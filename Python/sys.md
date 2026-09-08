# 概念與原理

## 什麼是 sys 模組？

`sys` 是 Python 內建的核心標準庫（無需安裝，直接 `import sys`）。

專門用於**與「Python 直譯器自身運行時環境 (Runtime Environment)」進行溝通、查詢與控制**。

與 `os`（操作作業系統外部檔案、目錄、環境變數）和 `platform`（診斷硬體與作業系統版本）不同，`sys` 模組專注於**直譯器內部的大腦中樞**，負責管理：
- 命令列引數傳遞 (`sys.argv`, `sys.orig_argv`)
- 模組搜尋與載入快取 (`sys.path`, `sys.modules`)
- 標準輸入輸出串流 (`sys.stdin`, `sys.stdout`, `sys.stderr`)
- 記憶體物件大小與字串駐留 (`sys.getsizeof()`, `sys.intern()`)
- 直譯器執行檔路徑與版本控制 (`sys.executable`, `sys.version_info`)
- 執行期例外追蹤與程式終止 (`sys.exc_info()`, `sys.exit()`)

> **三大模組分工對照**：  
> - **`os` 模組**：面向「外部作業系統」，負責在硬碟開資料夾、刪檔案、管理系統環境變數。  
> - **`platform` 模組**：面向「底層硬體與 OS 架構」，負責查詢 CPU 架構、作業系統版本。  
> - **`sys` 模組**：面向「Python 直譯器自身」，控制直譯器內部記憶體、搜尋路徑、命令列輸入與終止退出。

---

# 核心 API 函數與屬性字典對照表

| 類別 / 屬性 / 函式名稱 | 型態 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- | :--- |
| **[[#sys.argv 命令列參數清單\|sys.argv]]** | 屬性 (`list[str]`) | **取得命令列傳入的引數清單** | 撰寫 CLI 腳本時讀取使用者傳入的參數 |
| **[[#sys.orig_argv 原始完整命令列參數\|sys.orig_argv]]** | 屬性 (`list[str]`) | **取得傳遞給 Python 直譯器的原始完整參數** | (Python 3.10+) 讀取包含 Python 旗標的完整呼叫指令 |
| **[[#sys.exit() 終止當前程式執行\|sys.exit()]]** | 函式 | **立即終止當前 Python 程式執行並回傳狀態碼** | 發生嚴重錯誤時中斷程式，或正常結束退出 |
| **[[#sys.path 模組搜尋路徑清單\|sys.path]]** | 屬性 (`list[str]`) | **模組搜尋路徑字串清單 (import 搜尋順序)** | 動態注入自訂套件目錄、解決 ModuleNotFoundError |
| **[[#sys.modules 已載入模組字典快取\|sys.modules]]** | 屬性 (`dict`) | **已載入記憶體的模組字典快取** | 檢查套件是否已被載入、模組單例快取管理 |
| **[[#sys.stdin 標準輸入串流\|sys.stdin]]** | 屬性 (`TextIO`) | **標準輸入串流 (Standard Input，檔案描述符 0)** | 從終端機或管道 (Pipe) 批次讀取大量輸入 |
| **[[#sys.stdout 與 sys.stderr 輸出與錯誤串流\|sys.stdout]]** | 屬性 (`TextIO`) | **標準輸出串流 (Standard Output，檔案描述符 1)** | 終端機文字輸出、原地進度條即時刷新 (flush) |
| **[[#sys.stdout 與 sys.stderr 輸出與錯誤串流\|sys.stderr]]** | 屬性 (`TextIO`) | **標準錯誤串流 (Standard Error，檔案描述符 2)** | 輸出錯誤診斷資訊，與標準輸出管道分離 |
| **[[#sys.getsizeof() 獲取物件記憶體大小\|sys.getsizeof()]]** | 函式 | **獲取 Python 物件佔用的記憶體大小（Bytes）** | 分析資料結構記憶體開銷、效能最佳化 |
| **[[#sys.intern() 字串駐留最佳化\|sys.intern()]]** | 函式 | **將字串加入全域駐留表，以指標比對取代字元比對** | 大量重複字典鍵查找最佳化、節省記憶體 |
| **[[#sys.getrecursionlimit() 與 sys.setrecursionlimit() 遞迴深度查詢與設定\|sys.getrecursionlimit()]]** | 函式 | **獲取 Python 最大遞迴深度限制** | 深度優先搜尋 (DFS) 或遞迴演算法前置檢查 |
| **[[#sys.getrecursionlimit() 與 sys.setrecursionlimit() 遞迴深度查詢與設定\|sys.setrecursionlimit()]]** | 函式 | **調整 Python 最大遞迴深度限制** | 避免深度遞迴演算法提早引發 RecursionError |
| **[[#sys.exc_info() 獲取當前執行期例外資訊\|sys.exc_info()]]** | 函式 | **獲取當前正在處理的例外 (type, value, traceback)** | 日誌模組底層、自訂例外攔截與錯誤分析 |
| **[[#sys.version_info 直譯器版本五元組\|sys.version_info]]** | 屬性 (`tuple`) | **Python 版本的具名五元組** | 程式啟動時進行安全版本比對 (`>= (3, 10)`) |
| **[[#sys.executable 直譯器執行檔路徑\|sys.executable]]** | 屬性 (`str`) | **當前執行的 Python 直譯器絕對路徑** | 排查虛擬環境 (venv / uv / conda) 是否正確啟用 |
| **[[#sys.platform 平台識別碼\|sys.platform]]** | 屬性 (`str`) | **平台識別碼** (`'darwin'`, `'win32'`, `'linux'`) | 輕量快速判斷當前作業系統家族 |
| **[[#sys.dont_write_bytecode 禁止寫入位元組碼快取\|sys.dont_write_bytecode]]** | 屬性 (`bool`) | **控制是否在磁碟產生 .pyc 快取檔案** | 容器化部署、唯讀檔案系統環境配置 |

---

# 核心功能與語法大解密

## 1. 命令列參數與程式終止控制

##### sys.argv 命令列參數清單

- **使用時機**：撰寫命令列工具 (CLI) 或自動化腳本時，獲取終端機傳入的參數。
- **資料型別**：`list[str]`（字串清單）。
  - **`sys.argv[0]`**：固定為**「當前執行的 Python 腳本名稱或檔案路徑」**。
  - **`sys.argv[1:]`**：使用者在終端機輸入的**後續所有引數**（一律解析為 `str` 字串型別）。

```python
# 假設在終端機執行：python main.py build --port 8080
import sys

print(f"腳本名稱: {sys.argv[0]}")        # 輸出: main.py
print(f"所有傳入參數: {sys.argv[1:]}")    # 輸出: ['build', '--port', '8080']

# 安全讀取參數並手動轉型
if len(sys.argv) > 3:
    port = int(sys.argv[3])
    print(f"設定埠號為: {port}")
```

> **注意事項**：  
> 讀取 `sys.argv[1]` 前**必須先以 `len(sys.argv)` 進行長度防呆檢查**，否則使用者未傳參數時會直接引發 `IndexError: list index out of range`。更複雜的 CLI 介面推薦使用標準庫 `argparse` 或第三方庫 `click` / `typer`。

---

##### sys.orig_argv 原始完整命令列參數

- **使用時機**：(Python 3.10+) 需要獲取傳遞給 Python 直譯器二進位執行檔的**最原始完整參數清單**（包含傳給 Python 本身的旗標，例如 `-m`, `-u`, `-X` 等）。
- **資料型別**：`list[str]`。

```python
# 假設在終端機執行：python3 -u -X dev script.py arg1
import sys

# sys.argv 只能拿到: ['script.py', 'arg1']
# sys.orig_argv 可拿到: ['python3', '-u', '-X', 'dev', 'script.py', 'arg1']
print(f"原始啟動參數: {sys.orig_argv}")
```

---

##### sys.exit() 終止當前程式執行

- **使用時機**：當遇到致命錯誤需要中斷程式，或在順利完成任務後指定 Exit Code 退出。
- **語法**：`sys.exit(arg=0)`
- **參數說明**：
  - `arg`：結束狀態碼（整數 `0` 代表**正常成功退出**；非 `0` 如 `1`、`2` 代表**異常失敗退出**；亦可直接傳入錯誤訊息字串，直譯器會將字串印到 `stderr` 並以狀態碼 `1` 退出）。
- **底層原理**：
  - `sys.exit()` 的本質是**在直譯器內部主動拋出 `SystemExit` 例外**。

```python
import sys

def run_task(file_path: str):
    if not file_path.endswith(".json"):
        sys.exit("錯誤：檔案格式必須為 .json！")  # 印出錯誤訊息並以 code 1 退出
        
    print("處理資料中...")
    sys.exit(0)  # 成功結束程式，回傳狀態碼 0

run_task("config.txt")
```

---

## 2. 模組搜尋與載入快取管理

##### sys.path 模組搜尋路徑清單

- **使用時機**：當 Python 出現 `ModuleNotFoundError`，或需要**動態將自訂專案目錄、外掛資料夾加入模組搜尋範圍**時使用。
- **資料型別**：`list[str]`。Python 在執行 `import xxx` 時，會依照這個清單的**先後順序**逐一搜尋檔案。

```python
import sys
from pathlib import Path

# 1. 查看當前搜尋路徑清單
print("Python 模組搜尋路徑:")
for p in sys.path:
    print(f"  - {p}")

# 2. 動態將專案根目錄插入至最優先搜尋位置 (索引 0)
project_root = str(Path(__file__).resolve().parent.parent)
if project_root not in sys.path:
    sys.path.insert(0, project_root)  # 最高優先級
```

> **sys.path.append vs sys.path.insert(0, ...) 優先級規則**：  
> - `sys.path.append(dir)`：把路徑加在**最後面**。若系統函式庫已有同名模組，會優先載入系統庫。  
> - `sys.path.insert(0, dir)`：把路徑插入在**最前面（索引 0）**。確保 Python **優先載入你的自訂模組**。

---

##### sys.modules 已載入模組字典快取

- **使用時機**：檢查某個套件是否已經被 Python 直譯器載入記憶體中，或動態自省與清理模組快取。
- **資料型別**：`dict[str, ModuleType]`（鍵為模組名稱字串，值為記憶體中的模組物件實例）。
- **底層特性**：Python 的 `import` 具有**單例快取機制 (Singleton)**。當第一次 import 時會將模組存入 `sys.modules`，後續 import 都是直接從該字典中讀取。

```python
import sys

# 1. 檢查 json 是否已被載入
print("載入前:", "json" in sys.modules)  # 輸出: False

# 2. 執行 import
import json
print("載入後:", "json" in sys.modules)  # 輸出: True

# 3. 取得已載入模組的實體路徑
print("json 模組檔案位置:", sys.modules["json"].__file__)
```

---

## 3. 標準輸入輸出串流 (Standard Streams)

Python 直譯器在啟動時預設連結三個標準 I/O 串流：`sys.stdin`、`sys.stdout`、`sys.stderr`。

---

##### sys.stdout 與 sys.stderr 輸出與錯誤串流

- **使用時機**：
  - `sys.stdout.write()`：底層輸出，不會自動追加換行符號。
  - `sys.stdout.flush()`：**強制將內部緩衝區文字立即刷新輸出到終端機**（終端打字機效果、原地進度條必備）。
  - `sys.stderr`：專門輸出警告與錯誤日誌，避免污染正常的標準輸出（在管線重導向時特別重要）。

```python
import sys
import time

# 1. 實現終端機原地動態倒數計時 (不換行 + 強制刷新)
for i in range(3, 0, -1):
    sys.stdout.write(f"\r倒數計時: {i} 秒...")
    sys.stdout.flush()  # 強制將緩衝區文字刷新至終端螢幕
    time.sleep(1)

sys.stdout.write("\n倒數結束！\n")

# 2. 將錯誤訊息發送到 stderr (與 stdout 分流)
sys.stderr.write("警告：資料庫連線回應較慢。\n")
```

---

##### sys.stdin 標準輸入串流

- **使用時機**：在 CLI 中接收從管線 (Pipe) 或檔案重定向輸入的大量資料（例如 `cat data.txt | python process.py`）。
- **常用方法**：`sys.stdin.read()`（讀取全部）、`sys.stdin.readline()`（讀取單行）或 `for line in sys.stdin:`（逐行串流遍歷）。

```python
import sys

# 批次讀取來自管線或終端機的所有輸入內容
print("請輸入資料 (結束請按 Ctrl+D / Ctrl+Z):")
input_data = sys.stdin.read()
print(f"共讀取到 {len(input_data)} 個字元。")
```

---

## 4. 記憶體開銷、字串駐留與遞迴邊界

##### sys.getsizeof() 獲取物件記憶體大小

- **使用時機**：分析 Python 物件、容器資料結構在記憶體中所佔用的位元組數 (Bytes)。
- **語法**：`sys.getsizeof(object, default=None)`
- **參數說明**：
  - `object`：要測量的 Python 物件。
  - `default`：若物件無法測量時回傳的預設值。
- **回傳值**：
  - `int`：物件佔用的記憶體大小（單位：Bytes）。

```python
import sys

empty_list = []
full_list = [i for i in range(1000)]
gen = (i for i in range(1000))

print(f"空串列大小:   {sys.getsizeof(empty_list)} Bytes")  # 約 56 Bytes
print(f"千筆串列大小: {sys.getsizeof(full_list)} Bytes")   # 約 8856 Bytes
print(f"產生器大小:   {sys.getsizeof(gen)} Bytes (極低記憶體開銷)")  # 約 104 Bytes
```

> **getsizeof 淺層計算陷阱**：  
> `sys.getsizeof()` **只計算外層容器本身的結構開銷**，不會遞迴累加容器內部元素所引用的子物件記憶體！若需精準測量深層結構，需使用第三方庫（如 `pympler`）。

---

##### sys.intern() 字串駐留最佳化

- **使用時機**：當程式中存在大量重複、不可變的字串（如處理巨量 JSON 的字典鍵、日誌標籤），使用字串駐留（String Interning）將其納入全域字串池，**節省記憶體並將字串相等比對（`==`）加速至指標級別（`O(1)`）**。
- **語法**：`interned_str = sys.intern(string)`
- **參數說明**：
  - `string`：要進行駐留處理的字串。
- **回傳值**：
  - `str`：指向全域唯一駐留實例的字串物件。

```python
import sys

# 1. 透過運行時拼接產生的相同字串，記憶體位址原本不同
s1 = "".join(["hello", "_", "world"])
s2 = "".join(["hello", "_", "world"])
print(s1 is s2)  # 輸出: False (位址不同)

# 2. 進行字串駐留 (Interning)
s1_intern = sys.intern(s1)
s2_intern = sys.intern(s2)
print(s1_intern is s2_intern)  # 輸出: True (指向同一塊記憶體位址，比對極快)
```

---

##### sys.getrecursionlimit() 與 sys.setrecursionlimit() 遞迴深度查詢與設定

- **使用時機**：Python 內建遞迴深度保護機制（預設上限為 `1000`）。當執行深層樹狀走訪、圖形演算法或深層遞迴時，查詢並安全調整上限。
- **語法**：
  - `sys.getrecursionlimit()`：查詢當前上限。
  - `sys.setrecursionlimit(limit)`：設定新上限。

```python
import sys

# 1. 查詢預設上限
print(f"預設遞迴深度限制: {sys.getrecursionlimit()}")  # 輸出: 1000

# 2. 安全調大上限以支援深度演算法
sys.setrecursionlimit(3000)
print(f"調整後限制: {sys.getrecursionlimit()}")        # 輸出: 3000
```

---

##### sys.exc_info() 獲取當前執行期例外資訊

- **使用時機**：在 `except` 區塊內部獲取當前正在處理之例外的完整除錯資訊（例外類別、實例、Traceback 呼叫棧），日誌框架與錯誤追蹤中樞必備。
- **語法**：`sys.exc_info()`
- **回傳值**：
  - `tuple`：三元組 `(type, value, traceback)`。若無例外則回傳 `(None, None, None)`。

```python
import sys

try:
    result = 10 / 0
except ZeroDivisionError:
    exc_type, exc_value, exc_tb = sys.exc_info()
    print(f"例外類型: {exc_type.__name__}")  # 輸出: ZeroDivisionError
    print(f"錯誤訊息: {exc_value}")         # 輸出: division by zero
    print(f"發生行號: {exc_tb.tb_lineno}")  # 輸出具體出錯行號
```

---

## 5. 直譯器環境、版本與設定

##### sys.version_info 直譯器版本五元組

- **使用時機**：在程式啟動的第一時間，**以最具結構化、最高效的方式進行 Python 運行版本防呆比對**。
- **資料型別**：具名元組 `sys.version_info(major, minor, micro, releaselevel, serial)`。

```python
import sys

# 1. 取得主要與次要版本號
print(f"Python 版本: {sys.version_info.major}.{sys.version_info.minor}.{sys.version_info.micro}")

# 2. 標準版本防呆攔截 (核心最佳實踐)
if sys.version_info < (3, 10):
    sys.exit("錯誤：本專案必須在 Python 3.10 以上環境運行！")
```

---

##### sys.executable 直譯器執行檔路徑

- **使用時機**：確認當前腳本正在被哪個 Python 執行檔執行（確認是否正確啟用了虛擬環境 `.venv` 或 `conda`），排查環境污染。
- **語法**：`sys.executable`

```python
import sys

print(f"當前使用的 Python 直譯器: {sys.executable}")
# 範例輸出: /Users/matthew/project/.venv/bin/python
```

---

##### sys.platform 平台識別碼

- **使用時機**：以最輕量的方式快速判斷當前作業系統家族，進行跨平台邏輯分支。
- **常見回傳值**：`'darwin'` (macOS)、`'win32'` (Windows)、`'linux'` (Linux)。

```python
import sys

if sys.platform == "darwin":
    print("目前運行於 macOS 系統")
elif sys.platform == "win32":
    print("目前運行於 Windows 系統")
elif sys.platform.startswith("linux"):
    print("目前運行於 Linux 系統")
```

---

##### sys.dont_write_bytecode 禁止寫入位元組碼快取

- **使用時機**：控制 Python 在載入模組時，是否將編譯後的位元組碼寫入磁碟的 `__pycache__/*.pyc` 檔案。
- **資料型別**：`bool`（可透過環境變數 `PYTHONDONTWRITEBYTECODE=1` 或程式內修改）。
- **適用場景**：唯讀檔案系統、Docker 容器極簡構建、外掛即時熱重載。

```python
import sys

# 禁止 Python 在硬碟生成 .pyc 快取檔案
sys.dont_write_bytecode = True
```

---

# 實戰除錯與核心天條

## 1. 裸寫 except Exception 誤吞 sys.exit() 導致程式無法退出

> **捕捉例外誤吞 SystemExit 陷阱**：  
> `sys.exit()` 的底層實作是拋出 `SystemExit`（它直接繼承自 `BaseException`，而不是常規的 `Exception`）。  
> 若程式碼中裸寫了 `except BaseException:` 或在 `try...finally` 中誤寫了攔截邏輯，會把退出訊號意外吞掉，導致程式**完全無法終止退出**！  
> **鐵律**：永遠只捕捉 `except Exception:`，絕不可隨意捕捉 `BaseException`。

---

## 2. 嚴禁使用字串直接比對 Python 版本

> **字串版本比對天條**：  
> 絕對不要使用 `sys.version > "3.9"`！因為字串比較是按 ASCII 字典順序進行比對，會導致 `"3.10" < "3.9"` 判定為 `True` 的荒謬重大 Bug！  
> **鐵律**：比對 Python 版本唯一標準寫法為：`sys.version_info >= (3, 10)`。

---

## 3. sys.path 動態注入目錄順序天條

> **模組搜尋覆蓋天條**：  
> 使用 `sys.path.append()` 會將自訂路徑放在搜尋清單的最末尾。如果 Python 環境中已存在同名模組（如標準庫或第三方套件），自訂模組將永遠不會被載入！  
> **鐵律**：若需確保自訂模組優先被載入，一律使用 `sys.path.insert(0, custom_dir)` 插入至索引 0。
