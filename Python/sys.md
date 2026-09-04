# 概念與原理

## 什麼是 sys 模組？

Python 內建的核心標準庫（無需額外安裝，直接 `import sys`）。

專門用於**與「Python 直譯器自身運行時環境 (Runtime Environment)」進行溝通、查詢與控制**。

與 `os`（操作作業系統外部世界、檔案、資料夾）和 `platform`（診斷硬體與系統架構）不同，`sys` 模組專注於**直譯器內部的大腦中樞**，包含：命令列引數讀取 (`sys.argv`)、模組搜尋路徑管理 (`sys.path`)、標準輸入輸出串流 (`sys.stdin` / `sys.stdout`)、記憶體物件大小計算 (`sys.getsizeof`) 以及程式終止退出 (`sys.exit`)。

> **🍿 生動白話比喻**：  
> - **`os` 模組**：像 Python 通往外部作業系統的**「瑞士軍刀 🇨🇭」**（用來在外面開資料夾、刪檔案、跑指令）。  
> - **`platform` 模組**：像電腦與系統的**「身分證 🪪」**（查詢硬體型號與系統版本）。  
> - **`sys` 模組**：像 Python 直譯器的**「儀表板與中樞神經調節器 🧠」**（管理外面傳進來的參數是什麼、模組要去哪裡找、何時命令程式終止退出）。

---

# 核心 API 函數與屬性字典對照表

| 類別 / 屬性 / 函式名稱 | 型態 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- | :--- |
| **[[#sys.argv 命令列參數清單\|sys.argv]]** | 屬性 (`list[str]`) | **取得命令列傳入的引數清單** | 撰寫 CLI 腳本時讀取使用者傳入的參數 |
| **[[#sys.exit() 終止當前程式執行\|sys.exit()]]** | 函式 | **立即終止當前 Python 程式執行** | 發生嚴重錯誤時中斷程式，或正常結束退出 |
| **[[#sys.path 模組搜尋路徑清單\|sys.path]]** | 屬性 (`list[str]`) | **模組搜尋路徑字串清單** | 動態注入自訂套件目錄、解決 `ModuleNotFoundError` |
| **[[#sys.modules 已載入模組字典快取\|sys.modules]]** | 屬性 (`dict`) | **已載入記憶體的模組字典快取** | 檢查某個套件是否已被載入、動態模組自省 |
| **[[#sys.stdin 標準輸入串流\|sys.stdin]]** | 屬性 (`TextIO`) | **標準輸入串流 (Standard Input)** | 從終端機或管線 (Pipe) 批次讀取大量輸入 |
| **[[#sys.stdout 與 sys.stderr 輸出與錯誤串流\|sys.stdout]]** | 屬性 (`TextIO`) | **標準輸出串流 (Standard Output)** | 終端機即時輸出文字、即時刷新 (`flush`) |
| **[[#sys.stdout 與 sys.stderr 輸出與錯誤串流\|sys.stderr]]** | 屬性 (`TextIO`) | **標準錯誤串流 (Standard Error)** | 輸出錯誤診斷資訊，與標準輸出日誌分離 |
| **[[#sys.getsizeof() 獲取物件記憶體大小\|sys.getsizeof()]]** | 函式 | **獲取物件佔用的記憶體大小（Bytes）** | 分析資料結構記憶體開銷、效能最佳化 |
| **[[#sys.getrecursionlimit() 與 sys.setrecursionlimit()\|sys.getrecursionlimit()]]** | 函式 | **獲取 Python 最大遞迴深度限制** | 深度優先搜尋 (DFS) 或遞迴演算法前置檢查 |
| **[[#sys.getrecursionlimit() 與 sys.setrecursionlimit()\|sys.setrecursionlimit()]]** | 函式 | **調整 Python 最大遞迴深度限制** | 避免深度遞迴演算法提早引發 RecursionError |
| **[[#sys.version_info 直譯器版本五元組\|sys.version_info]]** | 屬性 (`tuple`) | **Python 版本的具名五元組** | 程式啟動時進行安全版本比對 (`>= (3, 10)`) |
| **[[#sys.executable 直譯器執行檔路徑\|sys.executable]]** | 屬性 (`str`) | **當前執行的 Python 直譯器絕對路徑** | 排查虛擬環境 (venv / uv) 是否正確啟用 |
| **`sys.platform`** | 屬性 (`str`) | **平台識別碼** (`'darwin'`, `'win32'`, `'linux'`) | 輕量快速判斷當前作業系統家族 |

---

# 核心功能與語法大解密

## 1. 命令列參數與程式終止控制 (CLI Arguments & Exit)

##### sys.argv 命令列參數清單

- **使用時機**：撰寫輕量級腳本時，直接獲取在終端機執行時傳入的參數清單。
- **資料結構**：字串清單 `list[str]`。
  - **`sys.argv[0]`**：固定為**「當前執行的 Python 腳本檔案名稱或路徑」**。
  - **`sys.argv[1:]`**：使用者在終端機輸入的**後續所有引數**（一律為 `str` 型別）。

```python
# 假設在終端機執行：python main.py build --verbose 8080
import sys

print(f"腳本名稱: {sys.argv[0]}")  # 輸出: main.py
print(f"所有傳入參數: {sys.argv[1:]}")  # 輸出: ['build', '--verbose', '8080']

# 取得特定參數並轉型
if len(sys.argv) > 3:
    port = int(sys.argv[3])
    print(f"設定埠號: {port}")
```

> **⚠️ argv 長度安全檢查天條**：  
> 讀取 `sys.argv[1]` 前**必須先檢查 `len(sys.argv)`**，否則若使用者未傳參數會直接引發 `IndexError: list index out of range`！更複雜的 CLI 建議使用 `argparse` 或 `click` / `typer`。

---

##### sys.exit() 終止當前程式執行

- **使用時機**：在遇到致命錯誤時中斷程式，或在正常完成任務後指定狀態碼結束。
- **語法**：`sys.exit(arg=0)`
- **參數說明**：
  - `arg`：結束狀態碼（整數 `0` 代表**正常成功退出**；非 `0` 如 `1`、`2` 代表**異常失敗退出**；也可傳入錯誤訊息字串）。
- **底層原理**：
  - `sys.exit()` 的本質是**在內部主動拋出 `SystemExit` 例外**！

```python
import sys

def process_file(filename: str):
    if not filename.endswith(".csv"):
        print("❌ 錯誤：僅支援處理 .csv 格式檔案！", file=sys.stderr)
        sys.exit(1)  # 異常退出，回傳狀態碼 1
        
    print("✅ 檔案格式正確，開始處理...")
    sys.exit(0)      # 正常結束

process_file("data.txt")
```

---

## 2. 模組搜尋與載入管理 (Import Path & Modules Cache)

##### sys.path 模組搜尋路徑清單

- **使用時機**：當 Python 出現 `ModuleNotFoundError`，或需要**動態將自訂資料夾加入模組搜尋範圍**（例如將專案根目錄或外掛目錄加入）時使用。
- **資料結構**：字串清單 `list[str]`。Python 執行 `import` 時會按照這個清單的**先後順序**依序搜尋模組。

```python
import sys
from pathlib import Path

# 1. 取得當前模組搜尋路徑清單
print("當前 Python 搜尋路徑:")
for p in sys.path:
    print(f"  - {p}")

# 2. 動態將專案根目錄加入搜尋清單的最優先位置 (最高優先級 ⭐)
project_root = str(Path(__file__).resolve().parent.parent)
if project_root not in sys.path:
    sys.path.insert(0, project_root)
```

> **⚠️ sys.path.append vs sys.path.insert(0, ...) 優先級天條**：  
> - `sys.path.append(dir)`：把路徑加在**最後面**。若系統庫已有同名模組，會先載入系統庫。  
> - `sys.path.insert(0, dir)`：把路徑插入在**最前面（索引 0）**。確保 Python **優先載入你的自訂模組**！

---

##### sys.modules 已載入模組字典快取

- **使用時機**：檢查某個模組是否已經被 Python 直譯器載入至記憶體中，或動態自省已載入的套件。
- **資料結構**：字典 `dict[str, ModuleType]`（鍵為模組名稱，值為記憶體中的模組物件實例）。

```python
import sys

# 1. 檢查 json 模組是否已被載入
if "json" in sys.modules:
    print("json 模組已經載入記憶體中！")
else:
    print("json 模組尚未被載入。")

import json
print("載入後狀態:", "json" in sys.modules)  # 輸出: True
```

---

## 3. 標準輸入輸出串流 (Standard Streams)

Python 程式預設連結三個標準 I/O 串流：`sys.stdin` (標準輸入)、`sys.stdout` (標準輸出)、`sys.stderr` (標準錯誤)。

---

##### sys.stdout 與 sys.stderr 輸出與錯誤串流

- **使用時機**：
  - `sys.stdout.write()`：比 `print()` 更底層的輸出（預設不會自動加換行，也不會自動轉型）。
  - `sys.stdout.flush()`：**強制將緩衝區文字立即刷出到終端機螢幕**（打字機效果、進度條必備 ⭐）。
  - `sys.stderr`：輸出錯誤日誌，避免污染正常的資料輸出（例如管線重導向時）。

```python
import sys
import time

# 1. 實現終端機原地動態倒數計時 (不換行 + 強制刷新)
for i in range(3, 0, -1):
    sys.stdout.write(f"\r倒數計時: {i} 秒...")
    sys.stdout.flush()  # 強制立即將畫面刷新到終端機！
    time.sleep(1)

print("\n🚀 發射！")

# 2. 將錯誤日誌專門輸出至 stderr (與標準輸出分離)
sys.stderr.write("⚠️ 這是一條發送到 stderr 的警報訊息！\n")
```

---

##### sys.stdin 標準輸入串流

- **使用時機**：在 CLI 中接收從管線 (Pipe) 餵進來的大量文字資料（例如 `cat data.txt | python process.py`）。
- **語法**：`sys.stdin.read()` 或走訪 `for line in sys.stdin:`。

```python
import sys

# 批次讀取來自管線或終端機的所有文字 (Ctrl+D / Ctrl+Z 結束)
print("請輸入資料 (結束請按 Ctrl+D):")
input_data = sys.stdin.read()
print(f"共讀取到 {len(input_data)} 個字元。")
```

---

## 4. 記憶體開銷與運行時邊界 (Memory & Recursion Limit)

##### sys.getsizeof() 獲取物件記憶體大小

- **使用時機**：分析 Python 物件、資料結構佔用記憶體位元組數 (Bytes)。
- **語法**：`bytes_size = sys.getsizeof(object)`
- **回傳值**：
  - `int`：物件佔用的記憶體大小（單位：Bytes）。

```python
import sys

empty_list = []
num_list = [i for i in range(1000)]
gen_expr = (i for i in range(1000))

print(f"空串列大小: {sys.getsizeof(empty_list)} Bytes")
print(f"千筆串列大小: {sys.getsizeof(num_list)} Bytes")
print(f"產生器物件大小: {sys.getsizeof(gen_expr)} Bytes (極致省記憶體！)")
```

> **⚠️ getsizeof 淺層計算陷阱**：  
> `sys.getsizeof()` **只計算外層容器本身的記憶體**，不會遞迴累加容器內子物件的記憶體！例如 `sys.getsizeof(["hello"])` 算的是列表指針的大小，不包含字串 `"hello"` 本身的內容記憶體。

---

##### sys.getrecursionlimit() 與 sys.setrecursionlimit()

- **使用時機**：Python 預設有遞迴深度保護機制（通常上限為 `1000`）。當執行深層樹狀搜尋或深層遞迴演算法時，可查詢並安全調大上限。
- **語法**：
  - `sys.getrecursionlimit()`：查詢當前上限。
  - `sys.setrecursionlimit(limit)`：調整上限。

```python
import sys

# 1. 查詢預設遞迴上限
current_limit = sys.getrecursionlimit()
print(f"預設最大遞迴深度: {current_limit}")  # 輸出: 1000

# 2. 安全調大遞迴深度以支援深度演算法
sys.setrecursionlimit(3000)
print(f"調整後遞迴深度: {sys.getrecursionlimit()}")  # 輸出: 3000
```

---

## 5. 直譯器版本與環境路徑自省 (Version & Executable)

##### sys.version_info 直譯器版本五元組

- **使用時機**：在程式啟動時，**以最高效、最標準的方式進行 Python 版本比對防呆**。
- **資料結構**：具名元組 `(major, minor, micro, releaselevel, serial)`。

```python
import sys

# 1. 取得版本資訊
print(f"Python 主要版本: {sys.version_info.major}.{sys.version_info.minor}")

# 2. 標準版本防呆攔截 (核心寫法 ⭐)
if sys.version_info < (3, 10):
    sys.exit("❌ 本專案要求 Python 3.10 以上環境！")
```

---

##### sys.executable 直譯器執行檔路徑

- **使用時機**：確認當前執行腳本的 Python 直譯器究竟位於系統全域、還是位於特定的虛擬環境（venv / uv / conda）中，除錯環境必備。
- **語法**：`sys.executable`

```python
import sys

print(f"當前 Python 直譯器路徑: {sys.executable}")
# 範例輸出: /Users/matthew/project/.venv/bin/python
```

---

# 實戰除錯與核心天條

## 1. 裸寫 except Exception 誤吞 sys.exit() 導致無法退出

> **⚠️ 捕捉例外誤吞 SystemExit 陷阱**：  
> `sys.exit()` 的底層實作是拋出 `SystemExit`（它繼承自 `BaseException` 而不是 `Exception`）。  
> 若寫了 `except BaseException:` 或使用了 `try...finally` 裡的阻斷邏輯，會把退出訊號攔截下來，導致程式**完全無法退出**！  
> **鐵律**：永遠只捕捉 `except Exception:`，切勿隨意捕捉 `BaseException`！

---

## 2. 嚴禁使用字串比較 Python 版本

> **⚠️ 版本比對天條**：  
> 絕對不要用 `sys.version > "3.9"`！因為字串比較是字典序，會產生 `"3.10" < "3.9"` 為 `True` 的荒謬 Bug。  
> **鐵律**：版本比對唯一標準寫法：`sys.version_info >= (3, 10)`。
