# 概念與原理

## 什麼是 os 模組？

Python 內建的核心標準庫（無需額外安裝，直接 `import os`）。

它提供了與**作業系統（Operating System）底層進行互動**的標準介面。包含：讀寫環境變數、跨平台檔案路徑處理、建立與刪除目錄、查詢系統處理程序等功能。

它抹平了 Windows、macOS 與 Linux 之間的底層系統差異，讓同一套 Python 程式碼可以在不同作業系統上無縫運行。

> **🍿 生動白話比喻**：  
> - **沒有 os 模組**：你的 Python 程式就像一個**「被關在房間裡的純演算法大腦」**，看不到外面的檔案夾、不知道自己在哪個路徑、也摸不到作業系統的環境變數。  
> - **有了 os 模組**：就像給 Python 裝上了**「通往作業系統的萬能瑞士軍刀 🇨🇭」**。隨時能出門建立資料夾、查看路徑、讀取系統密鑰或調用系統命令！

---

# 核心 API 函數字典對照表

| 類別 / 函式名稱 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- |
| **[[#os.environ 環境變數字典\|os.environ]]** | **讀取、修改作業系統環境變數字典** | 讀取 API 金鑰、資料庫連線字串、設定運行環境 |
| **[[#os.getenv() 便捷獲取環境變數\|os.getenv()]]** | **安全取得環境變數（支援預設值）** | 取出設定值且避免 `KeyError` 拋錯崩潰 |
| **[[#os.getcwd() 取得當前工作目錄\|os.getcwd()]]** | **獲取目前工作目錄絕對路徑** | 確認程式執行時的起點目錄位置 |
| **[[#os.chdir() 切換工作目錄\|os.chdir()]]** | **切換目前工作目錄** | 將執行焦點移至特定專案或資料夾路徑 |
| **[[#os.listdir() 列出目錄內容清單\|os.listdir()]]** | **列出目錄內所有檔案與子資料夾名稱** | 掃描資料夾清單、批次處理檔案 |
| **[[#os.scandir() 高效目錄掃描迭代器\|os.scandir()]]** | **高效目錄掃描迭代器（自帶檔案屬性）** | 遍歷目錄並同時獲取檔案類型/大小（效能比 `listdir` 快數倍） |
| **[[#os.mkdir() 與 os.makedirs() 建立資料夾\|os.makedirs()]]** | **遞迴建立多層目錄** | 初始化專案資料夾結構、確保存檔目錄存在 |
| **[[#os.remove() 與 os.rmdir() 刪除檔案與空目錄\|os.remove()]]** | **刪除單一檔案** | 清理臨時快取檔案、日誌清理 |
| **[[#os.remove() 與 os.rmdir() 刪除檔案與空目錄\|os.rmdir()]]** | **刪除空的目錄** | 移除不再需要的空資料夾 |
| **[[#os.rename() 與 os.replace() 重新命名與移動檔案\|os.rename()]]** | **重新命名或移動檔案/目錄** | 檔案整理、輸出重命名 |
| **[[#os.walk() 深度遍歷目錄樹\|os.walk()]]** | **深度遍歷目錄樹** | 遞迴掃描所有子資料夾內的特定副檔名檔案 |
| **[[#os.path.join() 跨平台安全拼接路徑\|os.path.join()]]** | **智慧安全拼接檔案路徑** | 跨平台組合檔案路徑（抹平 `/` 與 `\` 差異） |
| **[[#os.path.abspath() 取得標準絕對路徑\|os.path.abspath()]]** | **將相對路徑轉為標準絕對路徑** | 消除 `.` 與 `..`，取得完整精確路徑 |
| **[[#os.path.expanduser() 展開使用者家目錄波浪號 (~)\|os.path.expanduser()]]** | **展開使用者家目錄波浪號 (`~`) 為絕對路徑** | 讀取家目錄下的設定檔（如 `~/.config`、`~/Documents`） |
| **[[#os.path.dirname() 去除尾端取得父目錄路徑\|os.path.dirname()]]** | **取得路徑的父目錄部分** | 往上找上一層資料夾路徑 |
| **[[#os.path.basename() 取得檔案或目錄名稱\|os.path.basename()]]** | **取得路徑最尾端的檔案或目錄名稱** | 從完整路徑中提取純檔名 |
| **[[#os.path.split() 與 os.path.splitext() 拆分路徑與副檔名\|os.path.split()]]** | **拆分路徑為「目錄」與「檔名」** | 分離檔案所在位置與檔名本體 |
| **[[#os.path.split() 與 os.path.splitext() 拆分路徑與副檔名\|os.path.splitext()]]** | **拆分路徑為「主檔名」與「副檔名」** | 判斷副檔名（如 `.json`、`.csv`） |
| **[[#os.path.exists(), os.path.isfile(), os.path.isdir() 狀態判定\|os.path.exists()]]** | **判斷路徑/檔案是否存在** | 讀寫前進行防禦性檢查 |
| **[[#os.path.exists(), os.path.isfile(), os.path.isdir() 狀態判定\|os.path.isfile() / isdir()]]** | **判斷目標是否為純檔案 / 資料夾** | 走訪目錄時過濾掉非目標實體 |
| **[[#os.path.getsize() 獲取檔案大小\|os.path.getsize()]]** | **取得檔案大小（位元組 Bytes）** | 檢查檔案是否為空、日誌輪轉檢查 |
| **[[#os.name 作業系統核心名稱\|os.name]]** | **獲取作業系統平台核心名稱** | 依據平台差異 (`posix` vs `nt`) 執行分支邏輯 |

---

# 核心功能與語法大解密

## 1. os.environ 環境變數管理

環境變數是作業系統層級的全域鍵值對字典，常用於**隔離敏感資訊（如 API Key、密碼）或配置執行環境（如 `ENV=production`）**。

##### os.environ 環境變數字典

- **使用時機**：需要直接操作環境變數字典、走訪所有環境變數時使用。
- **資料結構**：一個類似 Python 原生字典的 `_Environ` 物件，鍵與值**一律強制為字串 (`str`)**。

```python
import os

# 1. 寫入或覆寫環境變數 (值必須是 str 字串)
os.environ["APP_MODE"] = "debug"
os.environ["MAX_WORKERS"] = "8"

# 2. 直接讀取 (若 Key 不存在會引發 KeyError 崩潰)
mode = os.environ["APP_MODE"]
print(f"目前模式: {mode}")  # 輸出: 目前模式: debug
```

---

##### os.environ.get() 安全取得環境變數

- **使用時機**：讀取環境變數時，**避免因變數未設定而拋出 `KeyError` 崩潰**，並可指定預設值。
- **語法**：`os.environ.get(key, default=None)`
- **參數說明**：
  - `key`：要查詢的環境變數名稱字串。
  - `default`：當找不到該環境變數時回傳的備用預設值（選填，預設為 `None`）。
- **回傳值**：
  - `str` 或 `default` 預設值。

```python
import os

# 1. 變數存在時：成功取得
api_key = os.environ.get("OPENAI_API_KEY", "預設_測試_KEY")

# 2. 變數不存在時：安全回傳預設值，完全不報錯！
db_port = os.environ.get("DB_PORT", "5432")
print(f"資料庫埠號: {db_port}")  # 輸出: 資料庫埠號: 5432
```

---

##### os.getenv() 便捷獲取環境變數

- **使用時機**：`os.environ.get()` 的快捷封裝函式，語法更簡短。
- **語法**：`os.getenv(key, default=None)`

```python
import os

# 單行安全取得環境變數
env_name = os.getenv("ENV", "development")
print(f"當前環境: {env_name}")  # 輸出: 當前環境: development
```

> **⚠️ 環境變數生命週期天條**：  
> 在 Python 程式中透過 `os.environ["KEY"] = "VAL"` 所做的修改，**只在當前 Python 處理程序及其衍生出的子處理程序中有效**！  
> 當 Python 程式結束後，**絕對不會**影響或修改作業系統本身的全域環境變數。

---

## 2. 檔案與目錄操作 (Directory & File Operations)

##### os.getcwd() 取得當前工作目錄

- **使用時機**：確認目前 Python 執行的起點目錄（Current Working Directory）。
- **語法**：`os.getcwd()`
- **回傳值**：
  - `str`：當前目錄的絕對路徑字串。

```python
import os

current_dir = os.getcwd()
print(f"目前工作目錄: {current_dir}")
```

---

##### os.chdir() 切換工作目錄

- **使用時機**：需要將當前程式的執行上下文路徑切換至其他資料夾時使用。
- **語法**：`os.chdir(path)`
- **參數說明**：
  - `path`：目標目錄的路徑字串。

```python
import os

# 切換到根目錄或上層目錄
os.chdir("..")
print(f"切換後的目錄: {os.getcwd()}")
```

---

##### os.listdir() 列出目錄內容清單

- **使用時機**：需要查看某個資料夾底下包含哪些檔案與子資料夾名稱時使用。
- **語法**：`os.listdir(path=".")`
- **參數說明**：
  - `path`：要掃描的目錄路徑（預設為 `.` 當前目錄）。
- **回傳值**：
  - `list[str]`：包含目錄下所有實體名稱的字串清單（**僅為檔名，不含完整路徑**）。

```python
import os

# 1. 取得當前目錄所有內容
items = os.listdir(".")
print("目錄內所有項目:", items)

# 2. 篩選出所有的 .py 檔案
py_files = [f for f in os.listdir(".") if f.endswith(".py")]
print("Python 程式清單:", py_files)
```

---

##### os.scandir() 高效目錄掃描迭代器

- **使用時機**：需要遍歷目錄內容，且同時需要檢查檔案類型（是檔案還是資料夾）、取得檔案大小或中繼資料時使用。其效能比 `os.listdir()` 搭配 `os.path.stat()` 快 2～20 倍！
- **語法**：`os.scandir(path=".")`
- **參數說明**：
  - `path`：要掃描的目錄路徑字串或路徑物件（預設為 `"."` 當前目錄）。
- **回傳值**：
  - `ScandirIterator`：回傳一個目錄掃描迭代器，迭代產出 `os.DirEntry` 物件。
- **核心 DirEntry 屬性與方法**：
  - `.name`：純檔案或目錄名稱字串（例如 `"app.py"`）。
  - `.path`：完整路徑字串（例如 `"./app.py"`）。
  - `.is_file(follow_symlinks=True)`：判斷是否為一般檔案（直接使用系統快取，無需額外系統呼叫）。
  - `.is_dir(follow_symlinks=True)`：判斷是否為目錄資料夾。
  - `.stat()`：取得檔案詳細中繼資料物件 `os.stat_result`（如大小 `st_size`、修改時間 `st_mtime`）。

```python
import os

# 1. 使用 with 語句確保迭代器資源及時釋放 (標準寫法 ⭐)
with os.scandir(".") as entries:
    for entry in entries:
        if entry.is_file():
            # 取得檔案大小 (Bytes)
            size = entry.stat().st_size
            print(f"📄 檔案: {entry.name} | 大小: {size} Bytes")
        elif entry.is_dir():
            print(f"📁 資料夾: {entry.name}")

# 2. 快速篩選出所有的 .py 檔案路徑清單
with os.scandir(".") as entries:
    py_files = [entry.path for entry in entries if entry.is_file() and entry.name.endswith(".py")]
    print("Python 檔案路徑清單:", py_files)
```

> **💡 效能暴增核心秘密**：  
> `os.listdir()` 僅回傳純檔名字串，若要進一步判斷 `os.path.isfile()` 或取得檔案大小，程式必須為每個檔案額外發起一次昂貴的作業系統呼叫 `stat()`；  
> 而 `os.scandir()` 在掃描目錄時，作業系統就已同步快取了檔案的屬性資訊。因此在包含大量檔案的目錄下，`os.scandir()` 的掃描速度能提升 **2 到 20 倍**！

> **💡 小提醒**：  
> 強烈建議搭配 `with os.scandir(...) as entries:` 語法使用，以確保在走訪完畢或發生例外時，能第一時間關閉並釋放作業系統的目錄資源與檔案描述符 (File Descriptor)。

---

##### os.mkdir() 與 os.makedirs() 建立資料夾

- **使用時機**：在磁碟上建立新的資料夾。
- **差異比較**：
  - `os.mkdir(path)`：**僅能建立單層資料夾**。若父目錄不存在，會直接拋出 `FileNotFoundError` 崩潰。
  - `os.makedirs(path, exist_ok=True)`：**可遞迴建立多層目錄**（相當於 `mkdir -p`）。**極度推薦使用！**
- **參數說明 (`os.makedirs`)**：
  - `path`：要建立的目錄路徑（如 `"logs/2026/08"`）。
  - `exist_ok`：布林值。設為 `True` 時，**若目錄已存在則直接忽略，不會報錯**！

```python
import os

# 1. 安全遞迴建立多層目錄 (核心寫法 ⭐)
os.makedirs("data/cache/temp_files", exist_ok=True)
print("目錄建立成功（或已存在）！")
```

> **⚠️ 建立目錄天條**：  
> 永遠記得加上 **`exist_ok=True`**！否則多個處理程序同時建立或第二次執行程式時，會觸發 `FileExistsError` 導致程式崩潰。

---

##### os.remove() 與 os.rmdir() 刪除檔案與空目錄

- **使用時機**：清理不需要的檔案或空資料夾。
- **語法**：
  - `os.remove(path)`：**專門刪除單一檔案**。若傳入資料夾路徑會拋錯。
  - `os.rmdir(path)`：**專門刪除單一「空」資料夾**。若資料夾內仍有檔案，會拋出 `OSError` 拒絕刪除。

```python
import os

# 1. 刪除檔案前先確認存在
if os.path.exists("temp.log"):
    os.remove("temp.log")
    print("檔案已刪除！")

# 2. 刪除空資料夾
if os.path.exists("empty_dir"):
    os.rmdir("empty_dir")
    print("空目錄已移除！")
```

> **💡 如何遞迴刪除包含檔案的非空資料夾？**  
> `os` 模組本身不提供非空目錄的強制刪除。必須搭配 Python 內建的 **`shutil.rmtree(path)`** 進行遞迴清理！

---

##### os.rename() 與 os.replace() 重新命名與移動檔案

- **使用時機**：更改檔案名稱，或將檔案移動至其他目錄路徑。
- **差異說明**：
  - `os.rename(src, dst)`：重新命名。在 Windows 下若目標檔案已存在會拋錯。
  - `os.replace(src, dst)`：**原子性覆蓋重新命名**（若目標檔案存在，會直接無聲覆蓋，跨平台一致）。

```python
import os

# 將 old_report.txt 重新命名並覆蓋為 report.txt
if os.path.exists("old_report.txt"):
    os.replace("old_report.txt", "report.txt")
```

---

##### os.walk() 深度遍歷目錄樹

- **使用時機**：需要**遞迴向下走訪整個資料夾結構**（包含所有子目錄與底下的全部檔案）時使用。
- **語法**：`for root, dirs, files in os.walk(top, topdown=True):`
- **產出三元組說明**：
  - `root` (`str`)：目前正在走訪的資料夾絕對路徑。
  - `dirs` (`list[str]`)：當前資料夾底下的所有**子目錄名稱清單**。
  - `files` (`list[str]`)：當前資料夾底下的所有**檔案名稱清單**。

```python
import os

# 深度遍歷掃描專案下的所有 .json 檔案
for root, dirs, files in os.walk("."):
    for file in files:
        if file.endswith(".json"):
            full_path = os.path.join(root, file)
            print(f"找到 JSON 檔案: {full_path}")
```

---

## 3. os.path 路徑處理與跨平台操作

在不同作業系統中，路徑分隔符號完全不同（Windows 使用反斜線 `\`，macOS / Linux 使用正斜線 `/`）。`os.path` 模組專門用來**自動處理跨平台路徑差異**。

> **💡 路徑基礎概念**：  
> - **`.`（單點）**：代表**「目前工作目錄」**。  
> - **`..`（雙點）**：代表**「上一層父目錄」**。  
> - **相對路徑**：相對於目前工作目錄的路徑（例如：`./data/app.log`、`../config.json`）。  
> - **絕對路徑**：從硬碟根目錄算起的完整路徑（例如：`/Users/matthew/app.py` 或 `C:\Users\app.py`）。

---

##### os.path.join() 跨平台安全拼接路徑

- **使用時機**：組合多段路徑字串。**絕對嚴禁使用字串 `+` 或 f-string 手動拼接斜線！**
- **語法**：`os.path.join(path1, path2, ...)`
- **核心優勢**：自動根據當前運行的作業系統，插入正確的分隔符號（Windows 填 `\`，Mac/Linux 填 `/`）。

```python
import os

base_dir = "Users"
user_folder = "matthew"
file_name = "config.json"

# 自動組合成跨平台安全路徑
safe_path = os.path.join(base_dir, user_folder, file_name)
print(safe_path)
# macOS / Linux 輸出: Users/matthew/config.json
# Windows 輸出: Users\matthew\config.json
```

---

##### os.path.abspath() 取得標準絕對路徑

- **使用時機**：將相對路徑（包含 `.` 或 `..`）轉換為完整、明確的系統絕對路徑。
- **語法**：`os.path.abspath(path)`

```python
import os

rel_path = "./data/../logs/app.log"
abs_path = os.path.abspath(rel_path)
print(f"標準絕對路徑: {abs_path}")
# 輸出類似: /Users/matthew/project/logs/app.log
```

---

##### os.path.expanduser() 展開使用者家目錄波浪號 (~)

- **使用時機**：將路徑開頭代表使用者家目錄的波浪符號 `~` 或 `~user`（例如 `"~/.zshrc"`、`"~/Documents"`）**自動替換展開為真實的作業系統家目錄絕對路徑**。
- **語法**：`expanded_path = os.path.expanduser(path)`
- **參數說明**：
  - `path`：包含 `~` 的路徑字串。
- **回傳值**：
  - `str`：展開後的標準絕對路徑字串。
    - **macOS**：`~/config.json` ➔ `/Users/matthew/config.json`
    - **Linux**：`~/config.json` ➔ `/home/matthew/config.json`
    - **Windows**：`~/config.json` ➔ `C:\Users\matthew\config.json`

```python
import os

# 1. 自動展開包含 ~ 的路徑
user_path = "~/Documents/data.csv"
real_path = os.path.expanduser(user_path)

print(f"原始路徑: {user_path}")
print(f"展開路徑: {real_path}")
# macOS 輸出例如: /Users/matthew/Documents/data.csv

# 2. 僅傳入 "~" 取得家目錄本身 (等同於 pathlib 的 Path.home())
home_dir = os.path.expanduser("~")
print(f"當前使用者家目錄: {home_dir}")
```

> **⚠️ 致命天條：Python 檔案操作不會自動展開 `~`**：  
> 在終端機 (bash/zsh) 中可以使用 `cat ~/.zshrc`，但若在 Python 寫 `open("~/.zshrc")`，Python 會在當前目錄下尋找名為 `~` 的「真實資料夾」，直接拋出 `FileNotFoundError` 崩潰！  
> **鐵律**：任何接收使用者輸入或包含 `~` 的路徑，在讀寫前**務必先通過 `os.path.expanduser()` 展開**！

---

##### os.path.dirname() 去除尾端取得父目錄路徑

- **使用時機**：從一個完整檔案路徑中，**剝除檔名，只取出上層資料夾路徑**。
- **語法**：`os.path.dirname(path)`

```python
import os

file_path = "/Users/matthew/project/src/main.py"

# 1. 取得 main.py 所在的資料夾
parent_dir = os.path.dirname(file_path)
print(parent_dir)  # 輸出: /Users/matthew/project/src

# 2. 連續呼叫兩次 ➔ 往上找上一層
grand_parent = os.path.dirname(parent_dir)
print(grand_parent)  # 輸出: /Users/matthew/project
```

---

##### os.path.basename() 取得檔案或目錄名稱

- **使用時機**：從完整路徑中，**只取出最後一個檔名或資料夾名稱**。
- **語法**：`os.path.basename(path)`

```python
import os

file_path = "/Users/matthew/project/data/orders.csv"
file_name = os.path.basename(file_path)
print(f"純檔案名稱: {file_name}")  # 輸出: orders.csv
```

---

##### os.path.split() 與 os.path.splitext() 拆分路徑與副檔名

- **使用時機**：
  - `os.path.split(path)`：將路徑拆解為 **`(父目錄, 檔名)`** 元組。
  - `os.path.splitext(path)`：將路徑拆解為 **`(檔名前綴, 副檔名)`** 元組。

```python
import os

full_path = "/data/reports/annual_2026.pdf"

# 1. 拆分目錄與檔名
folder, filename = os.path.split(full_path)
print(f"目錄: {folder}, 檔名: {filename}")
# 輸出: 目錄: /data/reports, 檔名: annual_2026.pdf

# 2. 拆分主檔名與副檔名 (抓副檔名神器 ⭐)
file_stem, ext = os.path.splitext(filename)
print(f"主檔名: {file_stem}, 副檔名: {ext}")
# 輸出: 主檔名: annual_2026, 副檔名: .pdf
```

---

##### os.path.exists(), os.path.isfile(), os.path.isdir() 狀態判定

- **使用時機**：讀取或寫入檔案前的安全檢查防呆。

| 檢查函式 | 判定條件 | 回傳值 |
| :--- | :--- | :--- |
| **`os.path.exists(path)`** | 目標檔案或資料夾是否存在 | `bool` |
| **`os.path.isfile(path)`** | 目標存在且**必須是一個普通檔案** | `bool` |
| **`os.path.isdir(path)`** | 目標存在且**必須是一個目錄資料夾** | `bool` |

```python
import os

target = "data/users.json"

if os.path.exists(target):
    if os.path.isfile(target):
        print("這是一個存在的檔案！")
    elif os.path.isdir(target):
        print("這是一個資料夾！")
else:
    print("目標完全不存在！")
```

---

##### os.path.getsize() 獲取檔案大小

- **使用時機**：檢查檔案容量大小（以位元組 Bytes 為單位）。
- **語法**：`os.path.getsize(path)`

```python
import os

# 取得檔案大小 (Bytes)
if os.path.isfile("data.csv"):
    size_bytes = os.path.getsize("data.csv")
    print(f"檔案大小: {size_bytes / 1024:.2f} KB")
```

---

## 4. 系統與處理程序管理 (System & Process Management)

##### os.system() 執行簡易 Shell 命令

- **使用時機**：在系統終端機執行極簡單的命令（如 `clear`、`cls`）。
- **語法**：`exit_code = os.system(command_string)`
- **回傳值**：
  - `int`：命令結束狀態碼（`0` 代表成功）。

```python
import os

# 執行終端機指令
exit_code = os.system("echo 'Hello OS!'")
print(f"結束狀態碼: {exit_code}")  # 輸出: 0
```

> **💡 進階命令調用提醒**：  
> `os.system()` 無法捕獲命令的輸出文字，且容易有 Shell 注入風險。需要獲取命令輸出或進行複雜命令調用時，**一律優先推薦使用 `subprocess.run()`**！

---

##### os.cpu_count() 與 os.getpid() 取得系統核心數與程序 ID

- **使用時機**：
  - `os.cpu_count()`：動態獲取 CPU 邏輯核心數，常作為 `ProcessPoolExecutor` 或 `ThreadPoolExecutor` 的最大線程數設定。
  - `os.getpid()`：獲取當前運行的 Python 處理程序 Process ID（PID），用於記錄除錯日誌。

```python
import os

# 1. 取得 CPU 核心數量
cores = os.cpu_count()
print(f"CPU 核心數: {cores}")

# 2. 取得當前 Process ID
pid = os.getpid()
print(f"當前 Python 行程 PID: {pid}")
```

---

##### os.name 作業系統核心名稱

- **使用時機**：判斷目前底層作業系統的家族型態。
- **屬性值**：
  - `'posix'`：代表 Linux、macOS、Unix 系統。
  - `'nt'`：代表 Windows 系統。

```python
import os

if os.name == "nt":
    print("目前運行在 Windows 系統！")
else:
    print("目前運行在 POSIX 系統 (macOS / Linux)！")
```

---

# 實戰除錯與核心天條

## 1. 忘記加上 exist_ok=True 導致目錄建立崩潰

> **⚠️ 目錄重複建立崩潰陷阱**：  
> `os.makedirs("path/to/dir")` 若未帶 `exist_ok=True`，當該資料夾已存在時會直接拋出 `FileExistsError`！  
> **鐵律**：永遠寫成 `os.makedirs(path, exist_ok=True)`。

---

## 2. 嚴禁手動用字串拼接斜線路徑

> **⚠️ 跨平台斜線衝突陷阱**：  
> 絕對不要寫 `f"{folder}/{file}"` 或 `folder + "\\" + file`！這會導致程式移到不同作業系統時路徑解析錯誤。  
> **鐵律**：路徑拼接一律使用 `os.path.join(folder, file)`。

---

## 3. os.remove() 與 os.rmdir() 的能力邊界

> **⚠️ 刪除功能誤用陷阱**：  
> - `os.remove()` **不能刪除資料夾**（會拋出 `IsADirectoryError` 或 `PermissionError`）。  
> - `os.rmdir()` **不能刪除非空的資料夾**（會拋出 `OSError: Directory not empty`）。  
> **鐵律**：刪除包含檔案的整個資料夾，請使用 `shutil.rmtree(path)`。
