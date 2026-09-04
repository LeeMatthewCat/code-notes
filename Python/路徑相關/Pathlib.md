# 概念與原理

## 什麼是 pathlib？

Python 3.4+ 官方標準庫中專為檔案與目錄操作設計的**現代物件導向路徑處理庫 (Object-Oriented Filesystem Paths)**（無需額外安裝，直接 `from pathlib import Path`）。

在傳統 Python 開發中，檔案路徑通常被當作普通的「純文字字串 (`str`)」來處理。當需要拼接路徑、檢查檔案是否存在或取得副檔名時，必須呼叫 `os` 與 `os.path` 模組中的大量函式（如 `os.path.join()`、`os.path.exists()`、`os.path.splitext()`）。

這種傳統文字字串處理方式存在兩大致命痛點：
1. **跨平台相容性災難**：Windows 使用反斜線 `\`，而 macOS / Linux 使用正斜線 `/`，手動字串相加極易引發跨平台崩潰。
2. **語法冗長、可讀性差**：多層路徑拼接與解析時，函式嵌套極度繁瑣且容易出錯。

`pathlib` 將檔案系統路徑抽象化為真正的 **`Path` 物件**。它重載了斜線運算子 `/`，讓路徑拼接就像寫數學公式一樣直觀，並將所有檔案操作（讀寫、檢查、遍歷、建立）直接封裝為物件的方法！

> **🍿 生動白話比喻**：  
> - **傳統 `os.path`**：像**「剪貼純文字標籤紙 🏷️」**。必須手動拿剪刀與膠水去拼湊檔名與斜線，稍不留神就貼錯斜線方向。  
> - **`pathlib.Path`**：像**「磁吸式積木組件 🧱」**。用斜線 `/` 一吸就完美對齊，而且積木本身自帶智能按鈕（點一下就自動讀檔 `.read_text()`、取得副檔名 `.suffix`、檢查是否存在 `.exists()`）！

---

## 深入剖析：什麼是 Path 物件？

`Path` 物件並不是普通的純文字字串 (`str`)，而是 Python 專為**「封裝檔案系統路徑邏輯與硬體 I/O 行為」**所設計的智慧類別實例。

它徹底改變了過去將路徑當成「死板字元」處理的思維，讓路徑自身具備了「結構感知」與「行動能力」。

### 1. Path 物件 vs 純字串 (str) 的本質差異

| 比較維度 | 普通字串路徑 (`str`) | 現代 `Path` 物件 (`pathlib.Path`) |
| :--- | :--- | :--- |
| **資料本質** | 僅是一長串純文字字元（如 `"/data/file.txt"`） | **封裝了檔案系統架構與操作行為的智慧物件實例** |
| **路徑拼接** | 容易拼錯斜線：`"dir" + "/" + "file"`（跨平台常出錯） | 像磁鐵般直接吸附：`Path("dir") / "file"`（自動適配系統） |
| **屬性提取** | 需手動切字串：`path.split("/")[-1].split(".")[0]` | 直接點出結構：`p.stem`、`p.suffix`、`p.parent` |
| **硬碟操作** | 毫無行動力，必須額外調用 `open()`、`os.remove()` | **自帶行動能力**：直接 `p.read_text()`、`p.mkdir()`、`p.exists()` |
| **型別標註** | `<class 'str'>` | `<class 'pathlib.PosixPath'>` 或 `<class 'pathlib.WindowsPath'>` |

---

### 2. Path 物件專屬「屬性」與「常用檢查方法」全能速查表

`Path` 物件最震撼的特色，就是將路徑的字串結構封裝為**靜態屬性**（免括號），並將硬碟狀態查詢封裝為**動態方法**（加括號）：

#### 🧱 一圖秒懂路徑屬性拆解圖

以路徑範例 `p = Path("/Users/matthew/projects/my_app/src/sales_report.tar.gz")` 進行結構拆解：

```text
/Users/matthew/projects/my_app/src/sales_report.tar.gz
├─────────────── p.parent ───────────────┤├──────── p.name ────────┤
│                                         ├─ p.stem ───┤├ p.suffix ┤
│                                                      ├── p.suffixes ──┤
├ p.anchor ┤
```

#### 📊 Path 物件屬性與核心檢查方法綜合對照表

| 成員名稱 | 類型 | 回傳型別 | 訪問硬碟 (I/O) | 功能作用與意義 | 範例回傳值 | 典型實戰用途 |
| :--- | :--- | :--- | :---: | :--- | :--- | :--- |
| **`p.name`** | **靜態屬性** | `str` | ❌ 否 | **完整檔名**（含副檔名） | `'sales_report.tar.gz'` | 日誌記錄、UI 顯示檔名 |
| **`p.stem`** | **靜態屬性** | `str` | ❌ 否 | **主體檔名**（去除最後一個副檔名） | `'sales_report.tar'` | 轉檔時保留原主檔名 |
| **`p.suffix`** | **靜態屬性** | `str` | ❌ 否 | **最後一個副檔名**（含點號 `.`） | `'.gz'` | 依副檔名分流處理檔案 |
| **`p.suffixes`** | **靜態屬性** | `list[str]` | ❌ 否 | **所有副檔名清單** | `['.tar', '.gz']` | 識別多重副檔名壓縮檔 |
| **`p.parent`** | **靜態屬性** | `Path` | ❌ 否 | **直接上一層父目錄**物件 | `Path('/Users/.../src')` | 寫檔前建目錄（`p.parent.mkdir()`） |
| **`p.parents`** | **靜態屬性** | 序列 | ❌ 否 | **所有祖先目錄序列** | `[0]`: 父目錄, `[1]`: 祖父目錄... | 向上回溯尋找專案根目錄 |
| **`p.parts`** | **靜態屬性** | `tuple[str]` | ❌ 否 | **路徑切分元組**（所有片段） | `('/', 'Users', 'matthew', ...)` | 檢查路徑是否包含特定層級 |
| **`p.anchor`** | **靜態屬性** | `str` | ❌ 否 | **根節點 / 磁碟機代號** | `'/'`（Windows: `'C:\'`） | 判斷系統根目錄起點 |
| **`p.drive`** | **靜態屬性** | `str` | ❌ 否 | **Windows 專屬磁碟機代號** | `''`（Windows: `'C:'`） | 判定 Windows 磁碟分割區 |
| **`p.root`** | **靜態屬性** | `str` | ❌ 否 | **根目錄符號** | `'/'`（Windows: `'\\'`） | 提取底層根目錄符號 |
| **`p.is_file()`** | **動態方法** | `bool` | ✅ 是 | **檢查是否為實體檔案** | `True` / `False` | 遍歷時過濾掉資料夾 |
| **`p.is_dir()`** | **動態方法** | `bool` | ✅ 是 | **檢查是否為實體資料夾目錄** | `True` / `False` | 遍歷時過濾掉一般檔案 |
| **`p.is_symlink()`** | **動態方法** | `bool` | ✅ 是 | **檢查是否為捷徑 / 符號連結** | `True` / `False` | 避免遞迴檢索死循環 |
| **`p.exists()`** | **動態方法** | `bool` | ✅ 是 | **檢查路徑在硬碟中是否真實存在** | `True` / `False` | 讀寫前防禦性檢查防崩潰 |

> **💡 深度解析：為什麼靜態屬性免加括號，而 `is_file()` / `is_dir()` 必須加括號？**  
> - **靜態屬性（如 `.stem`、`.suffix`）**：純粹從路徑文字字串拆解，**不需要連線硬碟 (No Disk I/O)**，因此是靜態屬性。  
> - **動態方法（如 `.is_file()`、`.is_dir()`）**：Python 必須**即時發送作業系統系統呼叫 (System Call) 去硬碟中確認該檔案的真實狀態與型態**，這是有 I/O 開銷的動作，因此設計為方法，**必須加上小括號 `()`**！

#### 💻 Python 屬性與檢查方法綜合示範代碼

```python
from pathlib import Path

# 建立 Path 智慧物件
p = Path("/Users/matthew/projects/my_app/src/sales_report.tar.gz")

# 1. 靜態屬性調用 (免加括號)
print(f"完整檔名 (name):     {p.name}")       # 'sales_report.tar.gz'
print(f"主體檔名 (stem):     {p.stem}")       # 'sales_report.tar'
print(f"副檔名 (suffix):     {p.suffix}")     # '.gz'
print(f"多重副檔名 (suffixes): {p.suffixes}")  # ['.tar', '.gz']
print(f"上一層父目錄 (parent): {p.parent}")    # /Users/matthew/projects/my_app/src
print(f"路徑元件元組 (parts):  {p.parts}")       # ('/', 'Users', 'matthew', 'projects', 'my_app', 'src', 'sales_report.tar.gz')

# 2. 動態方法調用 (必須加括號查詢硬碟)
print(f"路徑是否存在 (exists()):  {p.exists()}")
print(f"是否為檔案 (is_file()):  {p.is_file()}")
print(f"是否為資料夾 (is_dir()):   {p.is_dir()}")
```

---

### 3. 底層「工廠類別」運作機制 (PosixPath vs WindowsPath)

`Path` 在 Python 底層採用了**智慧工廠設計模式 (Factory Pattern)**：

- 當你在 **macOS / Linux** 呼叫 `Path("data.csv")` ➔ 底層自動產生 **`PosixPath`** 物件（以 `/` 運作）。
- 當你在 **Windows** 呼叫 `Path("data.csv")` ➔ 底層自動產生 **`WindowsPath`** 物件（以 `\` 運作）。

> **💡 開發者的極致好處**：  
> 開發時永遠只需 `from pathlib import Path`。同一套程式碼在 Mac 開發、部署到 Linux 伺服器、或發行到 Windows 客戶端，都能全自動無縫適配！

---

### 4. Path 物件與字串 (str) 相互轉換

```python
from pathlib import Path

# 1. 純字串 ➔ Path 物件 (將普通字串升級為智慧 Path 物件)
path_str = "/Users/matthew/Documents/report.pdf"
p = Path(path_str)

print(f"型別: {type(p)}")       # 輸出: <class 'pathlib.PosixPath'>
print(f"副檔名: {p.suffix}")    # 輸出: .pdf

# 2. Path 物件 ➔ 純字串 (降級轉回 str，用於傳給舊版不支援 Path 的第三方套件)
s = str(p)
print(f"轉換後字串: {s} | 型別: {type(s)}")  # 輸出: ... | 型別: <class 'str'>
```

---

## 傳統 os.path vs 現代 pathlib 震撼對比

| 操作需求 | 傳統 `os.path` 方式 (舊) | 現代 `pathlib` 方式 (新) | 核心差異與優勢 |
| :--- | :--- | :--- | :--- |
| **路徑拼接** | `os.path.join(base, "logs", "app.log")` | `Path(base) / "logs" / "app.log"` | 使用斜線運算子 `/` 直觀流暢 |
| **讀取純文字檔案** | `with open(p, "r", encoding="utf-8") as f:`<br>`    data = f.read()` | `p.read_text(encoding="utf-8")` | 一鍵極速讀取，免手動 open/close |
| **寫入純文字檔案** | `with open(p, "w", encoding="utf-8") as f:`<br>`    f.write("Hello")` | `p.write_text("Hello", encoding="utf-8")` | 一鍵極速寫入，自動關閉資源 |
| **建立多層資料夾** | `os.makedirs(dir_path, exist_ok=True)` | `dir_path.mkdir(parents=True, exist_ok=True)` | 語法更符合物件導向邏輯 |
| **取得檔名主體與副檔名** | `name = os.path.basename(p)`<br>`stem, ext = os.path.splitext(name)` | `p.name` (完整檔名)<br>`p.stem` (主體)<br>`p.suffix` (副檔名) | 屬性直接點出，無須手動切分字串 |
| **遞迴搜尋所有子目錄** | `os.walk(dir_path)` (需寫複雜迴圈) | `dir_path.rglob("*.py")` | 一行指令全自動遞迴檢索 |

---

# 核心 API 方法與屬性字典對照表

| 類別 / 方法 / 屬性名稱 | 型態 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- | :--- |
| **[[#Path() 建立與轉換路徑物件\|Path()]]** | 類別建構子 | **建立與轉換路徑物件** | 將字串路徑或多個路徑片段轉為跨平台 Path 物件 |
| **[[#Path.cwd() 取得當前工作目錄\|Path.cwd()]]** | 類別方法 | **獲取目前工作目錄絕對路徑** | 確認程式執行起點（等同於 `os.getcwd()`） |
| **[[#Path.home() 取得使用者家目錄\|Path.home()]]** | 類別方法 | **獲取當前使用者家目錄絕對路徑** | 存取個人檔案夾（等同於 `os.path.expanduser("~")`） |
| **[[#Path.expanduser() 展開波浪號 ~ 為家目錄絕對路徑\|Path.expanduser()]]** | 物件方法 | **展開路徑中的波浪號 (`~`) 為絕對路徑** | 讀取含 `~` 的使用者設定檔（防範 FileNotFoundError） |
| **[[#/ 跨平台路徑拼接運算子\|/ (斜線運算子)]]** | 運算子 | **直觀進行跨平台安全路徑拼接** | 組合目錄與檔名（完全取代繁瑣的 `os.path.join`） |
| **[[#Path.joinpath() 多路徑片段方法拼接\|p.joinpath()]]** | 物件方法 | **以方法方式拼接多個路徑片段** | 支援串列/元組解包傳入（`*parts`，與 `/` 底層等價） |
| **[[#Path.name 完整檔案或目錄名稱\|p.name]]** | 屬性 (`str`) | **取得完整檔案或目錄名稱** | 提取路徑最尾端的完整檔名（包含副檔名） |
| **[[#Path.stem 主體檔案名稱\|p.stem]]** | 屬性 (`str`) | **取得去除副檔名後的主體檔名** | 產出同名輸出檔（如轉檔時保留主檔名） |
| **[[#Path.suffix 最後一個副檔名\|p.suffix]]** | 屬性 (`str`) | **取得最後一個副檔名** (含點號 `.`) | 判定檔案類型（如 `.json`、`.py`、`.csv`） |
| **[[#Path.suffixes 所有副檔名清單\|p.suffixes]]** | 屬性 (`list[str]`) | **取得所有副檔名清單** | 處理多重副檔名壓縮檔（如 `.tar.gz`） |
| **[[#Path.parent 直接上一層父目錄\|p.parent]]** | 屬性 (`Path`) | **取得直接上一層父目錄物件** | 往上找上一層目錄、建立同目錄下的其他檔案 |
| **[[#Path.parents 所有祖先目錄序列\|p.parents]]** | 屬性 (序列) | **取得所有祖先層級目錄序列** | 依索引 `[0]`, `[1]` 向上回溯專案根目錄 |
| **[[#Path.parts 路徑切分元組\|p.parts]]** | 屬性 (`tuple[str]`) | **取得所有路徑片段切分元組** | 檢查路徑是否包含特定層級名稱（如 `"temp" in p.parts`） |
| **[[#Path.anchor 磁碟機根標籤\|p.anchor]]** | 屬性 (`str`) | **取得磁碟根節點標籤** | 檢查 Unix 根目錄 (`/`) 或 Windows 磁碟區 (`C:\`) |
| **[[#Path.drive 與 Path.root 磁碟代號與根標籤\|p.drive]]** | 屬性 (`str`) | **取得 Windows 磁碟機代號** | 判斷 Windows 磁區（如 `'C:'`，Unix 下為空字串 `""`） |
| **[[#Path.drive 與 Path.root 磁碟代號與根標籤\|p.root]]** | 屬性 (`str`) | **取得根目錄斜線標誌** | 提取根路徑符號（Unix 為 `'/'`，Windows 為 `'\\'`） |
| **[[#Path.exists() 檢查路徑是否存在\|p.exists()]]** | 物件方法 | **檢查路徑指向的檔案/資料夾是否存在** | 讀寫前進行防禦性檢查 |
| **[[#Path.is_file() 與 Path.is_dir() 類型檢查\|p.is_file()]]** | 物件方法 | **檢查路徑是否為常規檔案** | 走訪目錄時過濾掉資料夾 |
| **[[#Path.is_file() 與 Path.is_dir() 類型檢查\|p.is_dir()]]** | 物件方法 | **檢查路徑是否為資料夾目錄** | 遍歷目錄時過濾掉一般檔案 |
| **[[#Path.is_absolute() 絕對路徑檢查\|p.is_absolute()]]** | 物件方法 | **檢查路徑是否為絕對路徑** | 驗證輸入路徑是否為完整根路徑 |
| **[[#Path.stat() 取得詳細中繼資料\|p.stat()]]** | 物件方法 | **取得檔案大小 (Bytes) 與時間戳等中繼資料** | 檢查檔案大小 (`st_size`) 與修改時間 (`st_mtime`) |
| **[[#Path.read_text() 與 Path.write_text() 純文字讀寫\|p.read_text()]]** | 物件方法 | **一鍵讀取純文字內容為字串** | 免手動 open/close，極速讀取 `.txt`、`.md`、`.json` |
| **[[#Path.read_text() 與 Path.write_text() 純文字讀寫\|p.write_text()]]** | 物件方法 | **一鍵將字串寫入檔案 (自動覆寫/建立)** | 一鍵保存文字日誌與設定檔 |
| **[[#Path.read_bytes() 與 Path.write_bytes() 二進位讀寫\|p.read_bytes()]]** | 物件方法 | **一鍵讀取二進位資料為 bytes** | 讀取圖片、音訊、二進位模型檔 |
| **[[#Path.read_bytes() 與 Path.write_bytes() 二進位讀寫\|p.write_bytes()]]** | 物件方法 | **一鍵將 bytes 資料寫入檔案** | 儲存圖片、下載二進位檔案 |
| **[[#Path.mkdir() 建立資料夾目錄\|p.mkdir()]]** | 物件方法 | **建立資料夾目錄 (支援 parents/exist_ok)** | 初始化專案目錄結構 |
| **[[#Path.parent.mkdir() 防崩潰父目錄預建模式\|p.parent.mkdir()]]** | 語法模式 | **先確保父層資料夾存在再寫入檔案** | 防止深層目錄不存在引發 FileNotFoundError 崩潰 |
| **[[#Path.rename() 重新命名與移動\|p.rename()]]** | 物件方法 | **重新命名或移動檔案至新路徑** | 檔案整理、重命名歸檔 |
| **[[#Path.unlink() 與 Path.rmdir() 刪除檔案與空目錄\|p.unlink()]]** | 物件方法 | **刪除指定檔案 (支援 missing_ok=True)** | 清理暫存檔、快取清除 |
| **[[#Path.unlink() 與 Path.rmdir() 刪除檔案與空目錄\|p.rmdir()]]** | 物件方法 | **刪除指定的空資料夾目錄** | 移除空目錄 |
| **[[#Path.iterdir() 單層目錄遍歷\|p.iterdir()]]** | 物件方法 | **單層遍歷目錄下的所有子項目 (Path 迭代器)** | 掃描當前資料夾內的檔案與目錄 |
| **[[#Path.glob() 單層萬用字元檢索\|p.glob()]]** | 物件方法 | **單層萬用字元匹配搜尋 (回傳 Path 產生器)** | 搜尋當前層特定副檔名檔案（如 `*.py`） |
| **[[#Path.rglob() 全專案遞迴搜尋\|p.rglob()]]** | 物件方法 | **全專案深層遞迴萬用字元檢索 (Path 產生器)** | 遞迴搜尋所有子目錄深處的特定檔案 |

---

# 核心功能與語法大解密

## 1. 路徑建立與跨平台拼接 (Creation & Joining)

##### Path() 建立與轉換路徑物件

- **使用時機**：將字串路徑或多個路徑片段包裝成 `Path` 物件，自動相容當前作業系統的分隔符。
- **語法**：`p = Path(*pathsegments)`
- **回傳值**：
  - `Path` 物件（在 macOS/Linux 為 `PosixPath`，在 Windows 為 `WindowsPath`）。

```python
from pathlib import Path

# 1. 傳入字串路徑
p1 = Path("/Users/matthew/Documents")

# 2. 傳入多個路徑片段 (自動依作業系統組合)
p2 = Path("src", "core", "engine.py")
print(p2)  # macOS/Linux: src/core/engine.py | Windows: src\core\engine.py
```

---

##### Path.cwd() 取得當前工作目錄

- **使用時機**：獲取程式執行當下的工作目錄絕對路徑（等同於傳統 `os.getcwd()`）。
- **語法**：`current_dir = Path.cwd()`
- **回傳值**：
  - `Path`：當前工作目錄的絕對路徑物件。

```python
from pathlib import Path

current_directory = Path.cwd()
print(f"當前工作目錄: {current_directory}")
```

---

##### Path.home() 取得使用者家目錄

- **使用時機**：獲取當前登入使用者的主目錄絕對路徑（等同於 `os.path.expanduser("~")`）。
- **語法**：`home_dir = Path.home()`
- **回傳值**：
  - `Path`：使用者家目錄物件（macOS: `/Users/username`, Linux: `/home/username`, Windows: `C:\Users\username`）。

```python
from pathlib import Path

home_directory = Path.home()
print(f"使用者家目錄: {home_directory}")
```

---

##### Path.expanduser() 展開波浪號 ~ 為家目錄絕對路徑

- **使用時機**：將路徑字串開頭代表家目錄的波浪符號 `~`（如 `"~/Documents/config.json"`）自動展開替換為當前使用者的實體家目錄絕對路徑。
- **語法**：`expanded_path = path_obj.expanduser()`
- **回傳值**：
  - `Path`：展開後的標準絕對路徑物件。

```python
from pathlib import Path

# 使用 .expanduser() 自動將 ~ 解析為絕對路徑
config_path = Path("~/Documents/config.json").expanduser()
print(config_path)  # 輸出例如: /Users/matthew/Documents/config.json
```

> **⚠️ 波浪號 `~` 不自動展開陷阱**：  
> Python 預設**不會自動把 `~` 展開為實體路徑**！若直接對 `Path("~/data.csv")` 進行讀寫，Python 會在當前目錄尋找名為 `~` 的真實資料夾，直接引發 `FileNotFoundError`！務必先呼叫 `.expanduser()`！

---

##### / 跨平台路徑拼接運算子

- **使用時機**：以最直觀的數學公式語法組合路徑。只要左側為 `Path` 物件，右側可為字串或另一個 `Path` 物件。
- **語法**：`full_path = path_obj / "sub_dir" / "file.txt"`

```python
from pathlib import Path

# 自動處理跨平台斜線 (macOS 使用 /，Windows 使用 \)
log_file = Path.home() / "logs" / "2026" / "app.log"
print(log_file)
```

---

##### Path.joinpath() 多路徑片段方法拼接

- **使用時機**：以方法呼叫形式拼接多個路徑片段。底層與 `/` 運算子完全等價，但**非常適合搭配可變長度參數清單進行解包傳入（`*parts`）**。
- **語法**：`new_path = path_obj.joinpath(*other)`
- **參數說明**：
  - `*other`：一個或多個字串路徑片段或 `Path` 物件。
- **回傳值**：
  - `Path`：拼接後的新 `Path` 物件。

```python
from pathlib import Path

base = Path.cwd()

# 1. 傳統傳入多個片段
p1 = base.joinpath("data", "raw", "2026", "sales.csv")
print(p1)

# 2. 核心殺手級優勢：支援動態清單解包 (*parts)
# （當路徑片段儲存在 List 中時，joinpath 比斜線運算子更優雅簡潔！）
sub_folders = ["logs", "system", "daily", "2026-08.log"]
log_path = base.joinpath(*sub_folders)
print(log_path)
```

> **💡 `.joinpath()` vs `/` 運算子如何選擇？**  
> - 已知固定路徑拼接 ➔ 首選 **`/` 斜線運算子**（如 `base / "logs" / "app.log"`，最直觀乾淨）。  
> - 動態路徑清單拼接 ➔ 首選 **`.joinpath(*list_of_dirs)`**（免寫迴圈或 `reduce`，一鍵解包組合）。

---

## 2. 路徑屬性解析 (Path Decomposition)

### 🧱 一圖秒懂路徑屬性拆解圖

以路徑範例 `p = Path("/Users/matthew/projects/my_app/src/sales_report.tar.gz")` 進行結構拆解：

```text
/Users/matthew/projects/my_app/src/sales_report.tar.gz
├─────────────── p.parent ───────────────┤├──────── p.name ────────┤
│                                         ├─ p.stem ───┤├ p.suffix ┤
│                                                      ├── p.suffixes ──┤
├ p.anchor ┤
```

---

##### Path.name 完整檔案或目錄名稱

- **使用時機**：從路徑中提取最尾端的完整檔案或資料夾名稱字串。
- **回傳值**：`str`（包含所有副檔名）。

```python
from pathlib import Path
p = Path("/Users/matthew/src/sales_report.tar.gz")
print(p.name)  # 輸出: 'sales_report.tar.gz'
```

---

##### Path.stem 主體檔案名稱

- **使用時機**：取得去除**最後一個副檔名**後的主體檔名（常在轉檔時保留原主名）。
- **回傳值**：`str`。

```python
from pathlib import Path
p = Path("/Users/matthew/src/sales_report.tar.gz")
print(p.stem)  # 輸出: 'sales_report.tar'
```

---

##### Path.suffix 最後一個副檔名

- **使用時機**：取得檔案最末端的副檔名字串（包含前置點號 `.`，若無則回傳空字串 `""`）。
- **回傳值**：`str`。

```python
from pathlib import Path
p = Path("/Users/matthew/src/main.py")
print(p.suffix)  # 輸出: '.py'
```

---

##### Path.suffixes 所有副檔名清單

- **使用時機**：當檔案含有多重副檔名（如 `.tar.gz` 或 `.py.bak`）時，回傳包含所有副檔名字串的清單。
- **回傳值**：`list[str]`。

```python
from pathlib import Path
p = Path("/Users/matthew/src/sales_report.tar.gz")
print(p.suffixes)  # 輸出: ['.tar', '.gz']
```

---

##### Path.parent 直接上一層父目錄

- **使用時機**：取得代表直接上級目錄的 `Path` 物件。
- **回傳值**：`Path`。

```python
from pathlib import Path

# 1. 常規多層路徑
p = Path("/Users/matthew/projects/my_app/src/main.py")
print(p.parent)  # 輸出: /Users/matthew/projects/my_app/src
```

> **⚠️ 找不到父目錄時會得到什麼？（退無可退時永遠得到自己本身）**：  
> **核心結論：當路徑已經退無可退、到達頂點時，再取 `.parent` 永遠會「得到自己本身」，絕不拋出錯誤！**  
> 1. **已達磁碟根目錄（如 `Path("/")` 或 `Path("C:/")`）**：  
>    根目錄已是系統最頂層，`Path("/").parent` ➔ **永遠得到根目錄自己本身 `Path('/')`**！  
> 2. **單層相對路徑與點號目錄（如 `Path("main.py")`、`Path(".")`）**：  
>    - `Path("main.py").parent` ➔ 回傳當前目錄點號 **`Path('.')`**。  
>    - `Path(".").parent` ➔ **永遠得到點號自己本身 `Path('.')`**！  
>    *💡 實體路徑解法*：若想取得硬碟真實父資料夾，請先轉為絕對路徑：`Path("main.py").resolve().parent`。

---

##### Path.parents 所有祖先目錄序列

- **使用時機**：取得一個由近到遠包含所有祖先層級目錄的可迭代序列，支援用索引 `[0]`, `[1]` 直接存取。
- **回傳值**：序列（`Sequence[Path]`）。

```python
from pathlib import Path

p = Path("/Users/matthew/projects/my_app/src/main.py")

print(p.parents[0])  # 父目錄: /Users/matthew/projects/my_app/src
print(p.parents[1])  # 祖父目錄: /Users/matthew/projects/my_app
print(p.parents[2])  # 曾祖父目錄: /Users/matthew/projects
```

> **⚠️ 找不到祖父目錄或層級不足時會得到什麼？（IndexError 陷阱）**：  
> `p.parents` 本質上是一個有限長度的不可變序列，**若層級不足硬取索引會直接拋出 `IndexError`**！  
> 1. **單層相對路徑（如 `p = Path("main.py")`）**：  
>    `list(p.parents)` 只包含 `[Path('.')]`（長度為 1）。  
>    - 取父目錄 `p.parents[0]` ➔ 得到 `Path('.')`。  
>    - 取祖父目錄 `p.parents[1]` ➔ **❌ 直接崩潰：`IndexError: tuple index out of range`**！  
> 2. **根目錄（如 `p = Path("/")`）**：  
>    `list(p.parents)` 是 **空清單 `[]`**（長度為 0）。  
>    - 取 `p.parents[0]` ➔ **❌ 直接崩潰：`IndexError`**！  
>  
> **🛡️ 安全存取祖先目錄的黃金法則**：  
> - **方法 A（推薦）：先解析為實體絕對路徑**：`Path("main.py").resolve().parents[1]`（會自動從真實硬碟路徑往上找祖父目錄）。  
> - **方法 B：長度防禦檢查**：`if len(p.parents) > 1: grand_parent = p.parents[1]`  
> - **方法 C：for 迴圈安全遍歷**：`for parent in p.parents:`（迴圈會自動在最頂層停下，永不越界）。

---

##### Path.parts 路徑切分元組

- **使用時機**：將路徑按目錄分隔符切分為各片段字串所組成的元組（常用於判定路徑中是否包含某層特定資料夾）。
- **回傳值**：`tuple[str, ...]`。

```python
from pathlib import Path

p = Path("/Users/matthew/projects/temp/cache.json")
print(p.parts)  # 輸出: ('/', 'Users', 'matthew', 'projects', 'temp', 'cache.json')

# 實戰應用：檢查檔案是否位於特定資料夾中
if "temp" in p.parts:
    print("⚠️ 此檔案位於暫存目錄中")
```

---

##### Path.anchor 磁碟機根標籤

- **使用時機**：取得路徑開頭的磁碟機根節點標籤（Unix 系統為 `'/'`，Windows 為 `'C:\'`）。
- **回傳值**：`str`。

```python
from pathlib import Path
p = Path("/Users/matthew/src/main.py")
print(p.anchor)  # Unix 輸出: '/'
```

---

##### Path.drive 與 Path.root 磁碟代號與根標籤

- **使用時機**：分別提取 Windows 磁碟機代號 (`drive`) 與根目錄斜線標籤 (`root`)。
- **回傳值**：`str`。

```python
from pathlib import Path

# Unix / macOS 路徑
p_unix = Path("/Users/matthew/main.py")
print(p_unix.drive)  # 輸出: '' (Unix 無磁碟機代號)
print(p_unix.root)   # 輸出: '/'

# Windows 路徑
p_win = Path("C:/Users/matthew/main.py")
print(p_win.drive)   # 輸出: 'C:'
print(p_win.root)    # 輸出: '\\'
```

---

## 3. 狀態與屬性檢查 (Status & Metadata)

##### Path.exists() 檢查路徑是否存在

- **使用時機**：讀寫前防呆檢查目標檔案或資料夾在磁碟中是否真實存在。
- **語法**：`path_obj.exists()`
- **回傳值**：`bool` (`True` / `False`)。

```python
from pathlib import Path

if Path("config.json").exists():
    print("✅ 設定檔存在！")
```

---

##### Path.is_file() 與 Path.is_dir() 類型檢查

- **使用時機**：遍歷目錄時過濾實體，確認目標是純檔案還是資料夾目錄。
- **語法**：`path_obj.is_file()` / `path_obj.is_dir()`
- **回傳值**：`bool`。

```python
from pathlib import Path

p = Path("main.py")
print("是否為檔案:", p.is_file())  # True
print("是否為目錄:", p.is_dir())   # False
```

---

##### Path.is_absolute() 絕對路徑檢查

- **使用時機**：檢查路徑是否為從根節點算起的絕對路徑。
- **語法**：`path_obj.is_absolute()`
- **回傳值**：`bool`。

```python
from pathlib import Path

print(Path("/etc/hosts").is_absolute())  # True
print(Path("src/main.py").is_absolute()) # False
```

---

##### Path.stat() 取得詳細中繼資料

- **使用時機**：取得檔案的詳細中繼資料結構體（包含檔案大小 `st_size`、最後修改時間 `st_mtime` 等，等同於 `os.stat()`）。
- **語法**：`info = path_obj.stat()`
- **回傳值**：`os.stat_result` 物件。

```python
from pathlib import Path

p = Path("main.py")
if p.exists():
    info = p.stat()
    print(f"檔案大小: {info.st_size} Bytes")
    print(f"最後修改時間戳: {info.st_mtime}")
```

---

## 4. 一鍵極速讀寫與檔案操作 (File I/O & Management)

##### Path.read_text() 與 Path.write_text() 純文字讀寫

- **使用時機**：
  - `read_text()`：一鍵讀取全份文字檔案內容為字串，自動關閉檔案（免手動 `open()` / `close()`）。
  - `write_text()`：一鍵將字串寫入檔案；若檔案不存在自動建立，存在則自動覆寫。
- **語法**：`p.read_text(encoding='utf-8')` / `p.write_text(data, encoding='utf-8')`

```python
from pathlib import Path

config = Path("settings.txt")

# 1. 一鍵寫入純文字 (務必指定 encoding="utf-8" 防亂碼)
config.write_text("APP_ENV=production\nDEBUG=False", encoding="utf-8")

# 2. 一鍵讀取純文字
content = config.read_text(encoding="utf-8")
print(content)
```

---

##### Path.read_bytes() 與 Path.write_bytes() 二進位讀寫

- **使用時機**：一鍵讀寫二進位檔案（如圖片、音訊、模型檔），直接與 `bytes` 物件互動。
- **語法**：`data = p.read_bytes()` / `p.write_bytes(b_data)`

```python
from pathlib import Path

# 讀取圖片並複製一份
image_data = Path("photo.png").read_bytes()
Path("photo_copy.png").write_bytes(image_data)
```

---

##### Path.open() 原生檔案物件開啟

- **使用時機**：處理數百 MB 以上的**超大型檔案**，需要逐行迭代讀取時使用。
- **語法**：`with p.open(mode='r', encoding='utf-8') as f:`

```python
from pathlib import Path

with Path("large_dataset.csv").open("r", encoding="utf-8") as f:
    for line in f:
        print(line.strip())
```

---

##### Path.mkdir() 建立資料夾目錄

- **使用時機**：建立新的資料夾目錄。
- **語法**：`path_obj.mkdir(mode=0o777, parents=False, exist_ok=False)`
- **核心參數說明**：
  - `parents`：布林值。設為 `True` 時，若上層多層資料夾不存在，會**自動一併遞迴建立**。
  - `exist_ok`：布林值。設為 `True` 時，**若資料夾已存在不拋出錯誤**。

```python
from pathlib import Path

# 安全遞迴建立多層目錄 (核心寫法 ⭐)
Path("build/output/logs").mkdir(parents=True, exist_ok=True)
```

---

##### Path.parent.mkdir() 防崩潰父目錄預建模式

- **使用時機**：在準備建立或寫入一個深層檔案前，**先自動確保該檔案所在的父資料夾已存在**，100% 避免 `FileNotFoundError` 崩潰（工程師必備黃金天條 ⭐）。

```python
from pathlib import Path

# 假設要寫入一個深層檔案 (此時 logs/2026/08/ 目錄尚不存在)
log_file = Path("logs/2026/08/system.log")

# 1. 先為檔案的父目錄自動建立好資料夾！
log_file.parent.mkdir(parents=True, exist_ok=True)

# 2. 現在可以 100% 安全寫入檔案了！
log_file.write_text("系統啟動正常", encoding="utf-8")
```

---

##### Path.rename() 重新命名與移動

- **使用時機**：更改檔名，或將檔案/目錄移動至全新路徑。
- **語法**：`new_path = path_obj.rename(target)`
- **回傳值**：指向新位置的 `Path` 物件。

```python
from pathlib import Path

# 1. 重新命名檔案
Path("old.txt").rename("new.txt")

# 2. 移動檔案至新資料夾
dest_dir = Path("archive")
dest_dir.mkdir(exist_ok=True)
Path("new.txt").rename(dest_dir / "new.txt")
```

---

##### Path.unlink() 與 Path.rmdir() 刪除檔案與空目錄

- **使用時機**：
  - `unlink()`：刪除指定的**單一檔案**（`missing_ok=True` 可在檔案不存在時不拋錯）。
  - `rmdir()`：刪除指定的**空資料夾目錄**（若目錄內仍有檔案會拋出 `OSError`）。

```python
from pathlib import Path

# 1. 安全刪除檔案 (Python 3.8+ missing_ok=True)
Path("temp.log").unlink(missing_ok=True)

# 2. 刪除空資料夾
if Path("empty_folder").exists():
    Path("empty_folder").rmdir()
```

> **💡 如何遞迴刪除包含檔案的非空資料夾？**  
> `pathlib` 不提供強制刪除非空目錄的方法。請使用標準庫 **`shutil.rmtree(Path("my_folder"))`** 進行安全遞迴刪除！

---

## 5. 目錄遍歷與萬用字元檢索 (Traversal & Glob Search)

##### Path.iterdir() 單層目錄遍歷

- **使用時機**：單層掃描當前目錄下的所有子檔案與子資料夾（非遞迴）。
- **語法**：`for item in path_obj.iterdir():`
- **回傳值**：回傳產生 `Path` 物件的迭代器（`generator`）。

```python
from pathlib import Path

# 遍歷當前目錄下的所有子項目
for item in Path.cwd().iterdir():
    # 產出的 item 都是標準 Path 物件，直接調用所有屬性！
    print(f"完整檔名 (name):     {item.name}")
    print(f"主體檔名 (stem):     {item.stem}")
    print(f"副檔名 (suffix):     {item.suffix}")
    print(f"上一層目錄 (parent): {item.parent}")
    print(f"是否為檔案:          {item.is_file()}")
    print("-" * 30)
```

> **⚠️ 迭代器本身 vs 產出元素的屬性關鍵觀念**：  
> - 迭代器**本身**只是產生器容器，**沒有** `.name`、`.suffix` 等路徑屬性（直接寫 `Path.cwd().iterdir().name` 會拋出 `AttributeError`）！  
> - 但從迭代器中**迭代取出的每一個元素 (`item`) 100% 都是真正的 `Path` 物件**，完整具備所有屬性與方法！

- **💡 實戰技巧：搭配列表推導式 (List Comprehension) 一行批量提取屬性**：

```python
from pathlib import Path
current = Path.cwd()

# 1. 一行提取所有子項目的「完整檔名」清單
all_names = [item.name for item in current.iterdir()]

# 2. 一行提取所有子項目的「主體檔名」清單
all_stems = [item.stem for item in current.iterdir()]

# 3. 一行過濾出目錄下所有的「.py 檔案物件」
py_files = [item for item in current.iterdir() if item.suffix == ".py"]
```

---

##### Path.glob() 單層萬用字元檢索

- **使用時機**：在當前目錄下使用萬用字元（如 `*.py`、`data_?.csv`）進行條件匹配搜尋。
- **語法**：`generator = path_obj.glob(pattern)`
- **回傳值**：回傳一個**迭代物件**，每次迭代產出的**直接是 `Path` 物件**！

```python
from pathlib import Path

# 搜尋當前目錄下的所有 Python 檔案
for py_file in Path.cwd().glob("*.py"):
    print(f"找到程式碼: {py_file.name} (大小: {py_file.stat().st_size} Bytes)")
```

> **💡 產生器 (Generator) 核心觀念**：  
> - **極度省記憶體**：產生器不會一次把硬碟中所有檔案載入記憶體，而是動態即時計算產生下一個 `Path` 物件。  
> - **⚠️ 產生器只能消耗一次**：一旦被 `for` 迴圈走過一遍就會變空。若需計算總數 `len()` 或重複遍歷，請先用 **`list(path.glob("*.py"))`** 轉為清單！

---

##### Path.rglob() 全專案遞迴搜尋

- **使用時機**：**遞迴萬用字元檢索 (Recursive Glob)**，自動深度搜尋當前目錄及其**所有子目錄與孫目錄**中的檔案。
- **語法**：`generator = path_obj.rglob(pattern)`（等同於 `path_obj.glob("**/" + pattern)`）。

```python
from pathlib import Path

# 一行指令遞迴搜尋整個專案下所有的 .json 設定檔
all_json = list(Path.cwd().rglob("*.json"))
print(f"專案內總共包含 {len(all_json)} 個 JSON 設定檔！")
```

---

### Glob 萬用字元語法規則總覽與對照表

| 通配符記號 | 規則名稱 | 說明與匹配範圍 | 實戰範例 | 成功匹配目標 | 不匹配目標 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `*` | **單層任意字元** | 匹配 0 個或多個字元，**但不能跨越目錄斜線 `/`** | `*.py` | `main.py`, `app.py` | `src/main.py` (不可跨資料夾) |
| `**` | **跨層遞迴目錄** | 匹配任意層級的子資料夾與孫資料夾（僅在 `rglob` 或 `glob("**/*")` 使用） | `**/*.json` | `config.json`, `data/users.json` | — (全層級掃描) |
| `?` | **單一任意字元** | 恰好匹配 1 個任意字元（不包含 `/`） | `test_?.py` | `test_1.py`, `test_a.py` | `test_12.py`, `test_.py` |
| `[abc]` | **特定字元集合** | 匹配括號內列出的**任意一個**字元 | `file_[12].txt` | `file_1.txt`, `file_2.txt` | `file_3.txt` |
| `[a-z]` | **連續字元範圍** | 匹配指定範圍內的任意一個字母或數字 | `doc_[a-c].md` | `doc_a.md`, `doc_c.md` | `doc_z.md`, `doc_1.md` |
| `[!abc]` 或 `[^abc]` | **排除字元集合** | 匹配**不在**括號內的任意一個字元 | `data_[!0-9].txt` | `data_a.txt`, `data_x.txt` | `data_5.txt` |

---

## 6. 各式常見文檔 (.txt, .md, .json, .csv) 讀取全攻略

```python
import json
import csv
from pathlib import Path

# 1. 讀取純文字 / Markdown 檔
md_content = Path("README.md").read_text(encoding="utf-8")

# 2. 讀取 JSON 結構化檔
data = json.loads(Path("user.json").read_text(encoding="utf-8"))

# 3. 讀取 CSV 表格檔
with Path("sales.csv").open("r", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row)
```

---

# 實戰綜合範例：自動化 Downloads 檔案分類清理器

結合 `pathlib` 的遞迴檢索、多層資料夾建立與 [[Rich#1. Rich Console (控制台大腦)|Rich]] 美化面板：

```python
from pathlib import Path
from rich.console import Console
from rich.panel import Panel

console = Console()
target_dir = Path.home() / "Downloads"

EXTENSION_MAP = {
    "圖片": [".jpg", ".jpeg", ".png", ".gif", ".webp"],
    "文件": [".pdf", ".docx", ".xlsx", ".txt", ".csv"],
    "壓縮包": [".zip", ".tar", ".gz", ".7z"],
    "程式碼": [".py", ".js", ".html", ".json", ".sh"]
}

def organize_downloads():
    if not target_dir.exists():
        console.print(f"[bold red]❌ 找不到目標資料夾：{target_dir}[/bold red]")
        return

    console.print(Panel(f"[bold cyan]📁 開始整理資料夾：{target_dir}[/bold cyan]", border_style="cyan"))
    moved_count = 0
    
    for item in target_dir.iterdir():
        if item.is_dir():
            continue
            
        file_suffix = item.suffix.lower()
        for category, extensions in EXTENSION_MAP.items():
            if file_suffix in extensions:
                dest_dir = target_dir / category
                dest_dir.mkdir(exist_ok=True)
                dest_file = dest_dir / item.name
                item.rename(dest_file)
                console.print(f" [green]已移動:[/green] {item.name} ➔ [bold yellow]{category}/[/bold yellow]")
                moved_count += 1
                break

    console.print(f"\n[bold green]🎉 整理完成！總共歸類了 {moved_count} 個檔案。[/bold green]")

if __name__ == "__main__":
    organize_downloads()
```

---

# 實戰除錯與核心天條

## 1. 波浪號 (~) 不自動展開引發 FileNotFoundError

> **⚠️ ~ 家目錄未展開天條**：  
> Python 原生檔案 I/O 不會自動解析波浪號 `~`！若傳入 `"~/data.csv"`，Python 會在當前目錄尋找名為 `~` 的實體資料夾。  
> **鐵律**：任何包含 `~` 的路徑，在讀寫前**務必先呼叫 `.expanduser()`**！

---

## 2. 寫入深層檔案未建父目錄崩潰

> **⚠️ 父目錄缺失天條**：  
> 當寫入深層路徑檔案（如 `Path("logs/2026/08/app.log").write_text(...)`）時，若上層資料夾不存在會直接拋錯崩潰！  
> **鐵律**：寫入前永遠執行 **`p.parent.mkdir(parents=True, exist_ok=True)`**！

---

## 3. Windows 讀寫中文亂碼天條

> **⚠️ 強制 UTF-8 天條**：  
> 在 Windows 系統上，`read_text()` 與 `write_text()` 預設可能採用系統當地的 ANSI/CP950 編碼，導致中文字元亂碼！  
> **鐵律**：呼叫 `read_text()` 與 `write_text()` 時**永遠顯式帶入 `encoding="utf-8"`**！

---

## 4. parents 祖先層級不足時硬取索引引發 IndexError

> **⚠️ parents 序列越界天條**：  
> 對單層相對路徑（如 `Path("app.py")`）硬取 `p.parents[1]`（祖父層），或對根目錄 `Path("/")` 取 `p.parents[0]` 時，會直接拋出 `IndexError: tuple index out of range` 崩潰！  
> **鐵律**：取未知深度的祖先目錄時，**先用 `p.resolve()` 轉為絕對路徑**，或使用 `for parent in p.parents:` 安全遍歷！
