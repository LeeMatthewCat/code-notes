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
| `args` | `list[str]` / `str` | 要執行的命令與引數。強烈建議搭配 `shell=False` 傳入串列以防止 Shell 注入。 |
| `shell` | `bool` | 是否透過系統 Shell（如 `/bin/sh` 或 `cmd.exe`）來解析並執行命令。預設為 `False`。 |
| `stdin` / `stdout` / `stderr` | 串流設定 | 資料流向設定（如 `PIPE`, `DEVNULL`, 實體檔案）。詳見 [[#標準 I/O 導向設定 (stdin/stdout/stderr)\|標準 I/O 導向設定]]。 |
| `capture_output` | `bool` | `True` 時自動捕獲 stdout 與 stderr（等同於 `stdout=PIPE, stderr=PIPE`）。詳見 [[#標準 I/O 導向設定 (stdin/stdout/stderr)\|標準 I/O 導向設定]]。 |
| `input` | `str` / `bytes` | 直接傳送文字或二進位資料給子行程的 stdin。詳見 [[#標準 I/O 導向設定 (stdin/stdout/stderr)\|標準 I/O 導向設定]]。 |
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

##### shell 參數深度解密 (shell=False vs shell=True)

`shell` 參數決定了：**Python 是「直接呼叫目標二進位程式」，還是「先召喚一個系統 Shell 終端機，再由 Shell 去解讀命令」**。

```python
import subprocess

# 1. shell=False (預設，最佳實踐)：直接執行程式，傳入參數串列
subprocess.run(["ls", "-l", "my folder"])

# 2. shell=True：透過系統 Shell 解析整行字串 (支援管道與萬用字元)
subprocess.run("ls -l | grep txt", shell=True)
```

###### 1. 底層運作機制差異

- **`shell=False`（預設，推薦）**：
  
  Python 會透過作業系統底層的 `execve()` 系統呼叫，**直接定位二進位執行檔**（如 `/bin/ls`），並把後續的串列項目原封不動地傳給該程式的 `argv`。
  
  **不經過任何 Shell 解釋器**，因此輸入內容中的空白字元、分號、引號都不會被視為 Shell 特殊語法，徹底防範命令注入！

- **`shell=True`**：
  
  Python 會先啟動一個中介的 Shell 子行程：
  - **Linux / macOS**：等同於執行 `["/bin/sh", "-c", "你的命令字串"]`
  - **Windows**：等同於執行 `["cmd.exe", "/c", "你的命令字串"]`
  
  再由該 Shell 去進行命令分詞、解析環境變數、展開萬用字元與處理管道。

###### 2. shell=False 與 shell=True 全方位對照表

| 比較維度 | `shell=False` (預設，標準) | `shell=True` (特殊用途) |
| :--- | :--- | :--- |
| **參數推薦格式** | **串列 `list[str]`** (如 `["ls", "-la"]`) | **單一字串 `str`** (如 `"ls -la"`) |
| **底層執行方式** | 作業系統直接載入執行目標程式 | 先啟動 `/bin/sh` 或 `cmd.exe` 再轉發命令 |
| **管道 `\|` 與重定向 `>`** | 不支援（會被當作普通檔名字串） | **支援**（可使用 `\|`, `>`, `>>`, `&&`） |
| **萬用字元擴展 `*`** | 不支援（傳入 `*.txt` 不會展開） | **支援**（Shell 會自動展開所有符合的檔案） |
| **環境變數展開 `$VAR`** | 不支援（需手動用 `os.environ` 讀取） | **支援**（Shell 會自動替換 `$HOME` 或 `%USER%`） |
| **執行效能** | **極高**（少啟動一層 Shell 行程） | 略低（每次皆需額外開闢 Shell 子行程） |
| **安全性等級** | **極高**（免疫 Shell 注入攻擊） | **極低**（若拼接外部字串易遭 RCE 漏洞） |

###### 3. 什麼時候「非得使用」shell=True？與安全替代方案

1. **情境一：執行作業系統內建 Shell 指令**  
   在 Windows 下，諸如 `dir`, `echo`, `type`, `copy`, `cls` 並不是獨立的 `.exe` 執行檔，而是內建在 `cmd.exe` 中的內部命令。  
   - *解法*：在 Windows 執行此類指令時需設定 `shell=True`，或明確呼叫 `["cmd", "/c", "dir"]`。

2. **情境二：需要多個指令的管道 `|` 串接**  
   - *一般寫法*：`subprocess.run("cat access.log | grep 404", shell=True)`
   - *更安全且高效的純 Python 替代方案（使用 Popen 串接）*：
     ```python
     p1 = subprocess.Popen(["cat", "access.log"], stdout=subprocess.PIPE)
     p2 = subprocess.Popen(["grep", "404"], stdin=p1.stdout, stdout=subprocess.PIPE, text=True)
     p1.stdout.close() # 允許 p1 在 p2 結束時收到 SIGPIPE
     output, _ = p2.communicate()
     ```

3. **情境三：需要展開萬用字元（例如 `*.py`）**  
   - *更推薦的替代方案*：使用 Python 標準庫 `pathlib.Path().glob("*.py")` 取得檔案清單後，以 `shell=False` 傳入：
     ```python
     from pathlib import Path
     py_files = [str(f) for f in Path(".").glob("*.py")]
     subprocess.run(["black"] + py_files) # 安全且精準
     ```

---

## 進階非同步控制 (Popen)

當你需要建立非同步執行的背景進程，或是需要即時進行輸入/輸出串流互動時使用。

##### subprocess.Popen() 非同步進程控制
> Process open

- **使用時機**：想要執行一個需要長時間運行的背景程式，並希望 Python 主程式能「繼續往下跑」不被卡住時；或是需要即時串流讀取日誌輸出時。
- **語法**：`subprocess.Popen(args, stdin=None, stdout=None, stderr=None, bufsize=-1, close_fds=True, ...)`
- **參數說明**：
  - `args`：要執行的命令與引數。
    > **💡 小提醒：什麼是「引數」？為什麼要用「串列」？**
    > **引數 (Arguments)** 就是接在指令後面的附加條件，像是飲料的「半糖、少冰」。例如在指令 `ls -l -a` 中，`-l` 與 `-a` 就是引數。
    > 傳入 `args` 時，強烈建議將命令與引數拆解成**字串串列 (List)**（例如：`["ls", "-l", "-a"]`）。這能讓作業系統精準區分「指令本體」與「參數」，100% 防止駭客利用惡意空白字元或特殊符號發動 **Shell Injection (指令注入)** 攻擊！
  - `stdin` / `stdout` / `stderr`：用來決定背景程式資料流的去向。詳見 [[#標準 I/O 導向設定 (stdin/stdout/stderr)\|標準 I/O 導向設定]] 專章。
- **回傳值**：
  - 回傳一個 `subprocess.Popen` 實例物件，這就像是你拿到這個背景程式的「**遙控器**」。透過這個遙控器，你可以隨時對它進行操作或監控，最常用的按鈕與屬性包含：
    - `.poll()`：檢查程式是不是已經跑完了。回傳 `None` 代表還在背景跑，回傳數字（通常是 `0`）代表已經順利結束。
    - `.wait(timeout=None)`：讓主程式停下來，卡死在這裡等它跑完。
    - `.terminate()` / `.kill()`：發出中斷訊號，強制關閉這個背景程式。
    - `.pid`：取得這個程式在作業系統中的身分證字號 (Process ID)。
    - `.communicate()`：與這個進程對話（送出 stdin 或讀取它的 stdout/stderr 最後輸出），呼叫此方法會等待程式結束。

```python
import subprocess

# 專注展示建立非同步背景進程
proc = subprocess.Popen(["ping", "8.8.8.8"], stdout=subprocess.PIPE)

# 程式不會卡在這裡，可以繼續往下執行其他邏輯
print("Ping 已經在背景開始執行了...")
```

---

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

## 標準 I/O 導向設定 (stdin/stdout/stderr)

在呼叫 `subprocess.Popen()`、`subprocess.run()` 或 `check_output()` 時，可以透過這組參數精準控制子行程的資料流向。

### 1. 三大標準串流的核心定義
- **`stdin` (Standard Input，檔案描述符 0)**：程式的「**嘴巴**」，負責接收外部輸入的資料或文字。
- **`stdout` (Standard Output，檔案描述符 1)**：程式的「**正常喇叭**」，輸出正常的執行結果與文字日誌。
- **`stderr` (Standard Error，檔案描述符 2)**：程式的「**警報喇叭**」，專門輸出錯誤訊息、例外追蹤或除錯日誌。

---

### 2. stdin / stdout / stderr 可接受的 6 大合法值全景矩陣

| 設定值 | 意義與行為 | 適用串流 | 典型應用情境 |
| :--- | :--- | :--- | :--- |
| **`None` (預設值)** | **繼承父進程終端**，不攔截任何資料，輸出直接印在螢幕上。 | `stdin`, `stdout`, `stderr` | 一般指令執行，希望使用者直接在終端機看到畫面。 |
| **`subprocess.PIPE`** | **建立記憶體緩衝管道水管**，攔截資料並允許 Python 進行讀寫。 | `stdin`, `stdout`, `stderr` | 需要在 Python 變數中取得輸出內容，或跨進程串接水管。 |
| **`subprocess.DEVNULL`** | **丟入系統無底黑洞**（Linux `/dev/null`，Windows `NUL`），瞬間丟棄。 | `stdin`, `stdout`, `stderr` | **靜音執行**，隱藏不必要的背景提示或煩人的警告報錯。 |
| **`subprocess.STDOUT`** | **將 `stderr` 合併接到 `stdout` 管道**，按時間順序混合輸出。 | **僅限 `stderr`** | 統一抓取所有輸出日誌，避免錯誤訊息與正常訊息分離。 |
| **實體檔案物件** | 傳入以 `open()` 開啟的檔案（`'w'`, `'a'`, `'r'`），直接讀寫硬碟。 | `stdin`, `stdout`, `stderr` | 將輸出直接寫入 log 檔案，或從 txt 檔案餵入輸入。 |
| **整數檔案描述符 (int)** | 傳入作業系統底層的 File Descriptor（如 `0`, `1`, `2` 或 `os.open`）。 | `stdin`, `stdout`, `stderr` | 極底層的系統開發與通訊端點重新導向。 |

---

### 3. 與 I/O 相關的其他關鍵參數

- **`capture_output: bool`**（`subprocess.run()` 專屬）：
  - 設為 `True` 時，等同於自動宣告 `stdout=subprocess.PIPE, stderr=subprocess.PIPE`。
  - **注意**：設定 `capture_output=True` 時，不可同時再顯式指定 `stdout` 或 `stderr`，否則會拋出 `ValueError`。
- **`input: str | bytes`**（`subprocess.run()`, `check_output()`）：
  - 直接將字串或位元組資料送入子行程的 `stdin`（內部自動建立 PIPE 並透過 `.communicate()` 送出）。
  - **注意**：若傳入 `str`，必須搭配 `text=True` 或 `encoding="utf-8"`。
- **`bufsize: int`**（`Popen`, `run`）：
  - 設定 I/O 緩衝區模式：`0`（無緩衝）、`1`（行緩衝）、`>1`（指定緩衝區大小 Bytes）、`-1`（系統預設緩衝）。
- **`pipesize: int`**（Python 3.10+，`Popen`）：
  - 在 Linux 系統上調整底層作業系統管道的緩衝區大小（例如 `pipesize=1024*1024` 為 1MB，防範巨量資料造成管線堵塞死鎖）。
- **`close_fds: bool`**（`Popen`, `run`）：
  - 在子行程啟動前是否關閉除 0, 1, 2 以外繼承自父行程的檔案描述符（POSIX 預設為 `True`，Windows 預設在未重定向時為 `False`）。

---

### 4. 常用 subprocess 函數對 I/O 參數支援度速查表

| 函數名稱 | `stdin` | `stdout` | `stderr` | `input` 參數 | `capture_output` | 最佳使用情境 |
| :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| **`subprocess.run()`** | 支援 | 支援 | 支援 | 支援 | 支援 | **日常 95% 同步任務首選** |
| **`subprocess.Popen()`** | 支援 | 支援 | 支援 | 需用 `communicate()` | 不支援 (需顯式指定 PIPE) | **長時間背景任務、即時串流** |
| **`subprocess.check_output()`** | 支援 | **強制設為 PIPE** | 支援 | 支援 | 不支援 (內建自動抓 stdout) | 只在乎 stdout，失敗拋異常 |
| **`subprocess.check_call()`** | 支援 | 支援 | 支援 | 不支援 | 不支援 | 只在乎 Exit Code 是否為 0 |

---

### 5. 五大高頻 I/O 導向實戰程式碼範例

```python
import subprocess

# 範例 1：使用 DEVNULL 完全靜音執行 (丟棄所有輸出與錯誤)
subprocess.run(["ping", "-c", "1", "8.8.8.8"], stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)

# 範例 2：使用 subprocess.STDOUT 將錯誤訊息合併到標準輸出中
res = subprocess.run(
    ["python3", "-c", "import sys; print('正常訊息'); sys.stderr.write('報錯訊息\\n')"],
    stdout=subprocess.PIPE,
    stderr=subprocess.STDOUT, # 關鍵：把 stderr 接到 stdout 管道上
    text=True
)
print("合併後的輸出:\n", res.stdout)

# 範例 3：直接導向寫入實體檔案 (不佔用記憶體)
with open("task_output.log", "w", encoding="utf-8") as f:
    subprocess.run(["ls", "-la"], stdout=f, stderr=subprocess.STDOUT)

# 範例 4：使用 input 參數直接灌入文字給 stdin
res = subprocess.run(
    ["grep", "apple"],
    input="banana\napple\norange\npineapple\n", # 直接餵入字串
    capture_output=True,
    text=True
)
print("過濾結果:\n", res.stdout) # 輸出: apple, pineapple

# 範例 5：跨 Popen 進程接水管 (p1.stdout -> p2.stdin)
p1 = subprocess.Popen(["cat", "task_output.log"], stdout=subprocess.PIPE)
p2 = subprocess.Popen(["grep", "main"], stdin=p1.stdout, stdout=subprocess.PIPE, text=True)
p1.stdout.close() # 允許 p1 正常接收 SIGPIPE
out, _ = p2.communicate()
print("管道串接結果:", out)
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
