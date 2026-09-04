這是 Python 3 官方標準庫中專為**執行外部系統指令、呼叫終端機命令 (CLI) 以及管理子進程 (Subprocess)** 而設計的核心模組（推薦替代舊式 `os.system()`）。

> **🍿 比喻單元**：  
> Python 主程式像**「坐在辦公室裡的指揮官」**，`subprocess` 就像指揮官手中的**「無線電對講機」**。指揮官發出指令，對講機傳給作業系統執行，並把回傳碼與輸出文字安全帶回。

---

## 概念全景對照：threading vs queue vs subprocess

這三個模組在 Python 系統與並行程式設計中分工明確、各司其職：

| 比較維度 | `threading` (多執行緒) | `queue` (安全佇列) | `subprocess` (子進程/子行程) |
| :--- | :--- | :--- | :--- |
| **本質定位** | **程式內部的多個「工人 👷‍♂️」** | **工人之間的「安全輸送帶 🛤️」** | **呼叫系統外部的「外包公司 / CLI 🏢」** |
| **執行空間** | 同一個 Python 行程**內部**（共享記憶體空間） | 同一個 Python 行程**內部**（執行緒安全通訊） | 作業系統的**完全獨立外部行程**（獨立記憶體與環境） |
| **跨語言/工具** | 僅限執行 **Python 內部函式** | 僅限在 **Python 執行緒間傳遞物件** | 可執行 **任何外部系統指令與二進位程式**（如 `git`, `ffmpeg`, `ls`, Bash 等） |
| **資料通訊方式** | 共享全域變數（需搭配 `Lock` 防撞） | 透過 `put()` / `get()` 安全傳遞 Python 物件 | 透過標準串流二進位/文字（`stdin`, `stdout`, `stderr`） |
| **GIL 影響** | 受 GIL 限制（適合 I/O 密集型） | 本身為資料結構，內部自帶互斥鎖 | **完全不受 Python GIL 限制**（獨立 OS 行程） |
| **典型應用** | 同時發送多個 HTTP 請求、背景監聽 | 生產者-消費者任務緩衝、解耦任務流水線 | 呼叫 ffmpeg 影片轉檔、執行 git 指令、呼叫第三方 CLI 工具 |

> **💡 一秒記憶選型口訣**：  
> - **想開多個 Python 工人** ➔ 用 `threading`  
> - **工人之間要安全交接貨物** ➔ 用 `queue`  
> - **想叫外部終端指令或第三方軟體** ➔ 用 `subprocess`

---

## 核心執行函式

執行系統命令最常用的高階 API。使用前請確保已經 `import subprocess`。

##### subprocess.run() 同步執行指令

- **使用時機**：當你需要執行一條終端機命令，並希望 Python 主程式**阻塞等待**直到命令執行完畢，然後獲取其輸出結果或結束狀態碼時。適合 95% 以上的一般需求。
- **語法**：`subprocess.run(args, capture_output=False, text=False, encoding=None, errors=None, check=False, timeout=None, ...)`
- **參數說明**：

| 參數名稱 | 資料型別 | 說明 |
| :--- | :--- | :--- |
| `args` | `list[str]` | 要執行的命令與引數。強烈建議傳入串列以防止 Shell 注入。 |
| `capture_output` | `bool` | `True` 時會抓取終端機的 stdout 與 stderr 輸出內容，不在畫面上印出。 |
| `text` | `bool` | `True` 時將輸出二進位資料解碼為字串。通常搭配 `capture_output` 使用。 |
| `encoding` | `str` | 明確指定解碼的文字編碼格式（如 `"utf-8"` 或 Windows 的 `"cp950"`）。 |
| `errors` | `str` | 文字解碼失敗時的容錯策略（如 `"replace"` 替換為未知字元、`"ignore"` 忽略跳過），**防止因特殊字元拋出 `UnicodeDecodeError` 導致程式崩潰**。 |
| `check` | `bool` | `True` 時，若命令執行失敗 (Exit Code != 0) 會自動拋出 `CalledProcessError`。 |
| `timeout` | `float` | 設定最大執行秒數，超時拋出 `TimeoutExpired` 異常。 |

- **回傳值**：
  - `CompletedProcess`：包含執行結果（如 `.returncode`, `.stdout`, `.stderr`）的物件。

```python
...
import subprocess

# 1. 基本執行 (等待結束)
subprocess.run(["sleep", "2"])

# 2. 專注展示捕獲純文字輸出，並啟用編碼容錯 (errors="replace")
result = subprocess.run(
    ["ls", "-l"],
    capture_output=True,
    text=True,
    encoding="utf-8",
    errors="replace"  # 遇到特殊或損壞字元時自動替換為 ?，徹底避免 UnicodeDecodeError 閃退
)
print(result.stdout)
...
```

> **💡 文字解碼容錯 `errors` 模式對照表**：  
> 當終端機輸出的資料包含未知的損壞字元或編碼衝突時，可透過 `errors` 決定 Python 的容錯策略：

| 模式 | 設定寫法 | 遇到非法字元時的反應 | 輸出文字範例 | 適用時機 |
| :--- | :--- | :--- | :--- | :--- |
| **嚴格模式 (預設)** | `errors="strict"` | ❌ **直接拋出 `UnicodeDecodeError` 並崩潰閃退** | 程式中斷報錯 | 開發除錯，要求字元 100% 精準時 |
| **替換模式 (推薦)** | `errors="replace"` | ⭕ **換成 `` 或 `?`，程式安全繼續跑** | `Hello  World` | 執行外部指令、讀取未知日誌時防崩潰 |
| **忽略模式** | `errors="ignore"` | ⭕ **直接當作沒看到，把壞字丟棄吃掉** | `Hello  World` | 只在乎畫面乾淨，不在意少量字元遺失時 |

---

## 進階非同步控制 (Popen)

當你需要建立非同步執行的背景進程，或是需要即時進行輸入/輸出串流互動時使用。

##### subprocess.Popen() 非同步進程控制
> Process open

- **使用時機**：想要執行一個需要長時間運行的背景程式，並希望 Python 主程式能「繼續往下跑」不被卡住時；或是需要即時串流讀取日誌輸出時。
- **語法**：`subprocess.Popen(args, stdin=None, stdout=None, stderr=None, ...)`
- **參數說明**：
  - `args`：要執行的命令與引數。
    > **💡 小提醒：什麼是「引數」？為什麼要用「串列」？**
    > **引數 (Arguments)** 就是接在指令後面的附加條件，像是飲料的「半糖、少冰」。例如在指令 `ls -l -a` 中，`-l` 與 `-a` 就是引數。
    > 傳入 `args` 時，強烈建議將命令與引數拆解成**字串串列 (List)**（例如：`["ls", "-l", "-a"]`）。這能讓作業系統精準區分「指令本體」與「參數」，100% 防止駭客利用惡意空白字元或特殊符號發動 **Shell Injection (指令注入)** 攻擊！
  - `stdout`/`stdin`/`stderr`：用來決定背景程式資料流的去向（詳見下方的「標準 I/O 導向設定」章節）。
- **回傳值**：
  - 回傳一個 `subprocess.Popen` 實例物件，這就像是你拿到這個背景程式的「**遙控器**」。透過這個遙控器，你可以隨時對它進行操作或監控，最常用的按鈕與屬性包含：
    - `.poll()`：檢查程式是不是已經跑完了。回傳 `None` 代表還在背景跑，回傳數字（通常是 `0`）代表已經順利結束。
    - `.wait(timeout=None)`：讓主程式停下來，卡死在這裡等它跑完。
    - `.terminate()` / `.kill()`：發出中斷訊號，強制關閉這個背景程式。
    - `.pid`：取得這個程式在作業系統中的身分證字號 (Process ID)。
    - `.communicate()`：與這個進程對話（送出 stdin 或讀取它的 stdout/stderr 最後輸出），呼叫此方法會等待程式結束。

```python
...
import subprocess

# 專注展示建立非同步背景進程
proc = subprocess.Popen(["ping", "8.8.8.8"], stdout=subprocess.PIPE)

# 程式不會卡在這裡，可以繼續往下執行其他邏輯
print("Ping 已經在背景開始執行了...")
...
```



---

##### 標準 I/O 導向設定 (stdin/stdout/stderr)
> Input/Output

在建立 `Popen` 或 `run` 時，你可以透過這三個參數來決定程式資料的流向。最常見的設定有以下幾種：

1. **`None` (預設值)**：不做任何事。程式的輸出會直接印在終端機螢幕上。
2. **`subprocess.PIPE`**：建立專屬通訊水管，攔截輸入與輸出，允許事後用 `.communicate()` 讀寫互動。
   > pipe -> 管道
3. **`subprocess.DEVNULL`**：無盡的黑洞。所有丟進去的輸出都會人間蒸發，適合用來隱藏不必要的報錯或日誌。
   > Device None
4. **`subprocess.STDOUT`**：(通常只用在 `stderr`) 把錯誤訊息合併接到標準輸出 (`stdout`) 管線上，統一處理。
5. **`實體檔案物件`**：直接傳入一個用 `open()` 打開的檔案，讓程式直接把日誌寫進實體檔案裡。

###### proc.communicate() 建立 PIPE 管道並讀寫

> **🍿 比喻單元**：
> 想像你在遙控一台掃地機器人 (背景程式)。如果沒設定管道，它跑完就默默消失了。
> 當你在 Popen 中設定了 `subprocess.PIPE`，就等於在你們之間接上了「**透明水管**」！
> - **stdin**：接上「送水管」，讓你用 `communicate(input=...)` 把命令像紙條一樣塞進去。
> - **stdout / stderr**：接上「接水管」，讓你用 `communicate()` 接住它源源不絕噴出的日誌。

- **使用時機**：當你需要一次性將資料寫入進程的 `stdin`，或者是讀取它所有的 `stdout` 與 `stderr` 內容，並等待其結束時。
- **語法**：`proc.communicate(input=None, timeout=None)`
- **參數說明**：
  - `input`：要發送給子進程的資料（搭配 `stdin=subprocess.PIPE` 使用）。
- **回傳值**：
  - `tuple`：回傳一個 `(stdout_data, stderr_data)` 的元組，讓你接住輸出結果。

```python
...
import subprocess

# 範例：啟動一個背景 python，並同時接上「輸入」與「輸出」的水管
proc = subprocess.Popen(
    ["python3", "-c", "name = input(); print(f'你好, {name}!')"],
    stdin=subprocess.PIPE,   # 允許我們灌入資料
    stdout=subprocess.PIPE,  # 允許我們接住輸出結果
    text=True                # 自動把位元組(bytes)轉成字串(str)
)

# 透過 communicate，把 "Matthew" 從 stdin 送水管灌進去，並把結果從 stdout 接水管接住
out_data, err_data = proc.communicate(input="Matthew")

print(out_data)
# 輸出結果: 你好, Matthew!
...
```

---

##### proc.wait() 等待進程結束

- **使用時機**：在建立 `Popen` 背景進程後，你在某個時刻需要暫停 Python 主程式，等待該進程執行完畢才繼續時。
- **語法**：`proc.wait(timeout=None)`
- **參數說明**：
  - `timeout`：最大等待秒數，超時會拋出異常。
- **回傳值**：
  - `int`：進程的 Exit Code (結束狀態碼)。

```python
...
# 專注展示等待背景進程結束
proc = subprocess.Popen(["sleep", "3"])
return_code = proc.wait()
...
```

---

##### 終止進程的兩大天條與觀念

當你需要透過 Python 關閉背景程式時，最忌諱的就是發出信件就跑掉，或者亂拔插頭。這兩兄弟一定要搭配 `.wait()` 來完美收尾！

> 💡 **核心觀念 1：為什麼發出終止信件後，一定要搭配 `wait()`？**
> 當你呼叫 `.terminate()` 或 `.kill()` 時，Python 只是對作業系統「**寄出一封名為終止的信件**」，這個動作是瞬間完成的，不代表程式已經真正完全關閉了！
> 如果你的主程式發完信後就立刻繼續往下跑甚至結束，那個背景進程的屍體就會殘留在系統中，變成佔用系統資源的**「殭屍進程 (Zombie Process)」**。
> ⚠️ **正確做法 (收屍)**：永遠要在發出終止指令的後面加上 `.wait()`。這能確保主程式停下來，親眼看著它確實斷氣，並將它的資源回收乾淨！

> 💡 **核心觀念 2：`terminate()` 與 `kill()` 的差異總結**

| 特性 | `.terminate()` (SIGTERM) | `.kill()` (SIGKILL) |
| :--- | :--- | :--- |
| **態度** | 溫和、給予緩衝時間 | 暴力、瞬間抹殺 |
| **程式能否攔截？** | 可以。程式能捕捉這個訊號，做最後的存檔或清理。 | 不行。由作業系統直接強制介入拔插頭。 |
| **風險** | 程式如果當機卡死，可能會無視這個訊號。 | 可能會導致寫入一半的資料毀損、或資料庫被鎖死。 |
| **使用建議** | **永遠優先使用！** (先禮) | 只有在 `terminate()` 無效時才使用。 (後兵) |

###### proc.terminate() 溫和終止進程

> **🍿 比喻單元**：
> 終止程式有分文跟武，`.terminate()` 是「**文**」的。
> 它就像是餐廳打烊時，經理走到客人桌邊客氣地說：「不好意思，我們準備打烊了，請您收拾一下東西。」程式收到這個通知 (`SIGTERM` 訊號) 後，會有時間存檔、關閉連線，然後自己優雅地退出。

- **使用時機**：當你想要停止一個背景程式，但希望它能好好地收尾，而不是突然斷線導致資料毀損時。
- **語法**：`proc.terminate()`
- **參數說明**：無。
- **回傳值**：`None`。

```python
...
import subprocess

proc = subprocess.Popen(["python3", "long_task.py"])

# 先發出溫和終止信件
proc.terminate()

# 務必加上這行，留在現場收屍！確保它真的結束了才繼續往下跑
proc.wait() 
...
```

---

###### proc.kill() 強制終止進程

> **🍿 比喻單元**：
> 這個就是「**武**」的了！
> 它就像是餐廳打烊時，經理直接把餐廳的總電源「啪！」一聲切斷，然後把客人連同桌子直接扔出大門 (`SIGKILL` 訊號)。程式完全沒有機會反抗，也來不及做任何存檔，瞬間被抹殺。

- **使用時機**：當背景進程完全卡死當機，連 `.terminate()` 都不理你，需要無條件強制拔插頭時。
- **語法**：`proc.kill()`
- **參數說明**：無。
- **回傳值**：`None`。

```python
...
import subprocess

proc = subprocess.Popen(["ping", "8.8.8.8"])

# 遇到無反應的進程，直接無條件拔插頭抹殺
proc.kill()

# 即使是被抹殺，依然要留在現場收屍！
proc.wait()
...
```

---

## 資安陷阱與防呆機制

使用系統指令時最容易犯下的嚴重錯誤。

##### Shell Injection 命令注入漏洞

當你將未經驗證的字串直接當作終端機指令執行時，攻擊者可以藉由分號 `;` 等符號夾帶惡意指令。

> **🍿 比喻單元**：
> 這就像你去銀行臨櫃寫提款單，金額欄本來填寫 `100` 元；結果惡意人士在後面塗改加上 `; 順便把開戶人的存款轉帳給黑幫`。如果櫃檯人員（系統 Shell）照單全收，兩條命令都會被一口氣執行完畢！

- **❌ 錯誤寫法 (暴露嚴重漏洞)**：
  使用 `shell=True` 且用字串拼接。
  ```python
  ...
  user_input = "notes.txt; rm -rf /" 
  
  # 系統會將分號後面的 rm -rf / 視為第二條指令執行，造成毀滅性後果！
  subprocess.run(f"cat {user_input}", shell=True)
  ...
  ```

- **✅ 安全寫法 (100% 防注入)**：
  不要開啟 `shell=True`，並且使用 **串列 (List)** 傳參。
  ```python
  ...
  user_input = "notes.txt; rm -rf /"
  
  # 系統會把 "notes.txt; rm -rf /" 鎖定為一個純字串檔名，絕對不會把它拆成指令執行！
  subprocess.run(["cat", user_input])
  ...
  ```

> **💡 小提醒**：永遠保持 `shell=False`（預設值），並且將所有指令與參數拆解成 `list` 傳入，就能從根本上杜絕 Shell Injection 的可能。

---

##### 逾時卡死防禦 (TimeoutExpired)

當你呼叫任何會等待的指令時（如 `run` 或 `wait`），若目標程式陷入無限迴圈，整個 Python 將會永久卡死。

> **💡 小提醒**：只要你設定了 `timeout` 參數，就**絕對必須**搭配 `try...except` 來捕獲 `subprocess.TimeoutExpired` 異常，否則超時的當下你的 Python 主程式就會直接崩潰閃退！

```python
...
import subprocess

try:
    # 專注展示設定超時防護
    result = subprocess.run(["sleep", "10"], timeout=3)
except subprocess.TimeoutExpired as e:
    print("⚠️ 命令執行超時，已強制中斷防卡死！")
...
```

---
