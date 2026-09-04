# 概念與原理

## 什麼是 platform 模組？

Python 內建的核心標準庫（無需額外安裝，直接 `import platform`）。

專門用於**獲取當前運行的底層作業系統、硬體架構、主機名稱與 Python 直譯器版本**等豐富環境元數據。

與 `os.name` 僅回傳粗略的 `'posix'` 或 `'nt'` 相比，`platform` 模組能提供**極度精準、人類可讀且跨平台統一的系統硬體診斷資訊**（例如：明確區分 macOS、Windows 11、Linux Ubuntu，以及辨識 ARM64 / x86_64 處理器架構）。

> **🍿 生動白話比喻**：  
> - **`os` 模組**：像**「作業系統的工具箱 🧰」**（用來開關資料夾、刪除檔案、執行指令）。  
> - **`platform` 模組**：像**「電腦與 Python 的身分證 / 體檢診斷書 🪪」**（專門用來查：這台電腦是什麼作業系統？CPU 是 Intel 還是 Apple Silicon M 系列？Python 是哪個版本？執行在 64 位元還是 32 位元環境？）。

---

# 核心 API 函數字典對照表

| 類別 / 函式名稱 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- |
| **[[#platform.system() 取得作業系統名稱\|platform.system()]]** | **取得作業系統名稱** (`'Darwin'`, `'Windows'`, `'Linux'`) | 跨平台程式中進行頂層 OS 分支判斷 |
| **[[#platform.release() 與 platform.version() 發行版與內部版本\|platform.release()]]** | **取得作業系統發行版本號** | 檢查 Windows 10/11 或 Linux 核心版本 |
| **[[#platform.release() 與 platform.version() 發行版與內部版本\|platform.version()]]** | **取得作業系統詳細內部版本字串** | 深入診斷系統 Build 號或編譯日期 |
| **[[#platform.platform() 取得完整平台字串\|platform.platform()]]** | **產出單行、人類可讀的完整系統資訊字串** | 軟體崩潰回報 (Crash Report)、除錯日誌輸出 |
| **[[#platform.machine() 取得 CPU 架構\|platform.machine()]]** | **取得硬體 CPU 架構名稱** (`'arm64'`, `'x86_64'`) | 依據 CPU 架構動態載入對應的 C/C++ 二進位套件 |
| **[[#platform.machine() 取得 CPU 架構\|platform.processor()]]** | **取得處理器詳細型號名稱** | 記錄硬體環境、硬體診斷分析 |
| **[[#platform.architecture() 取得 Python 直譯器位元組架構\|platform.architecture()]]** | **取得 Python 直譯器的位元架構** (`64bit` / `32bit`) | 確認是否安裝了正確位元的 Python 環境 |
| **[[#platform.python_version() 與 python_version_tuple()\|platform.python_version()]]** | **取得當前 Python 版本字串** (`'3.12.3'`) | 檢查專案是否滿足最低 Python 版本依賴 |
| **[[#platform.python_version() 與 python_version_tuple()\|platform.python_version_tuple()]]** | **取得 Python 版本三元組** (`('3', '12', '3')`) | 便於使用元組進行版本大小比較運算 |
| **[[#platform.python_implementation() 與 python_compiler()\|platform.python_implementation()]]** | **取得 Python 實作名稱** (`'CPython'`, `'PyPy'`) | 依據直譯器特性進行效能最佳化或相容切換 |
| **[[#platform.node() 取得電腦主機名稱\|platform.node()]]** | **取得主機網路電腦名稱** | 分散式系統標記節點、識別叢集伺服器 |
| **[[#platform.uname() 一鍵取得系統全景元組\|platform.uname()]]** | **回傳包含所有核心系統資訊的具名元組** | 一次性取得系統、節點、核心與硬體架構集合 |

---

# 核心功能與語法大解密

## 1. 作業系統與平台識別 (OS & Platform Info)

##### platform.system() 取得作業系統名稱

- **使用時機**：在跨平台程式碼中，最常用的作業系統頂層分支判斷工具。
- **語法**：`platform.system()`
- **回傳值**：
  - `str`：標準作業系統名稱字串。常見回傳值：
    - `'Darwin'`：macOS 系統（蘋果底層 Unix 核心名稱）。
    - `'Windows'`：Windows 系統。
    - `'Linux'`：Linux 各大發行版（Ubuntu, Debian, CentOS 等）。
    - `'Java'`：Jython 運行環境。

```python
import platform

current_os = platform.system()
print(f"作業系統核心: {current_os}")

if current_os == "Darwin":
    print("🍎 正在運行於 macOS 系統！")
elif current_os == "Windows":
    print("🪟 正在運行於 Windows 系統！")
elif current_os == "Linux":
    print("🐧 正在運行於 Linux 系統！")
```

---

##### platform.platform() 取得完整平台字串

- **使用時機**：需要生成一段詳細、標準、人類可讀的單行系統字串，適合**寫入系統日誌 (Log) 或 Bug 錯誤回報**。
- **語法**：`platform.platform(aliased=False, terse=False)`
- **參數說明**：
  - `aliased`：布林值。設為 `True` 時，嘗試將部分別名調整為大眾常見名稱。
  - `terse`：布林值。設為 `True` 時，回傳較簡短的摘要資訊。
- **回傳值**：
  - `str`：包含 OS、版本、核心與架構的完整描述字串。

```python
import platform

# 取得完整系統字串
full_info = platform.platform()
print(f"完整系統資訊: {full_info}")
# macOS 輸出範例: macOS-15.0-arm64-arm-64bit
# Windows 輸出範例: Windows-11-10.0.22631-SP0
```

---

##### platform.release() 與 platform.version() 發行版與內部版本

- **使用時機**：需要區分具體是 Windows 10 還是 Windows 11，或需要記錄詳細的作業系統內部 Build 號時使用。
- **語法**：`platform.release()` / `platform.version()`

```python
import platform

print("發行版號 (Release):", platform.release())  # 例如: 24.0.0 (macOS) 或 11 (Windows)
print("內部版本 (Version):", platform.version())  # 例如: Darwin Kernel Version 24.0.0...
```

---

## 2. 硬體與處理器架構 (Hardware & Architecture)

##### platform.machine() 取得 CPU 架構

- **使用時機**：在部署 AI 深度學習框架（如 PyTorch、TensorFlow）或下載 C 語言編譯動態庫時，**判斷底層 CPU 是 ARM 架構還是 x86/x64 架構**。
- **語法**：`platform.machine()`
- **回傳值**：
  - `str`：CPU 架構代碼。常見回傳值：
    - `'arm64'` / `'aarch64'`：Apple Silicon (M1/M2/M3/M4) 或 ARM 處理器。
    - `'x86_64'` / `'AMD64'`：Intel 或 AMD 64 位元處理器。

```python
import platform

arch = platform.machine()
print(f"硬體 CPU 架構: {arch}")

if arch in ("arm64", "aarch64"):
    print("🚀 偵測到 ARM 架構（如 Apple M 系列晶片）")
elif arch in ("x86_64", "AMD64"):
    print("💻 偵測到 x86_64 架構（Intel / AMD）")
```

---

##### platform.architecture() 取得 Python 直譯器位元組架構

- **使用時機**：確認當前執行的 Python 直譯器本身是 **64 位元** 還是 **32 位元**（避免 32 位元 Python 無法存取 4GB 以上記憶體）。
- **語法**：`bits, linkage = platform.architecture()`
- **回傳值**：
  - `tuple[str, str]`：包含 `(位元數, 執行檔格式)` 的元組，如 `('64bit', '')` 或 `('64bit', 'ELF')`。

```python
import platform

bits, linkage = platform.architecture()
print(f"Python 直譯器位元: {bits}")  # 輸出: 64bit
```

---

## 3. Python 直譯器與環境自省 (Python Runtime Info)

##### platform.python_version() 與 python_version_tuple()

- **使用時機**：
  - `python_version()`：以**字串**形式取得版本號（用於顯示或日誌記錄）。
  - `python_version_tuple()`：以**元組**形式取得 `(主版本, 次版本, 微版本)`，**便於在程式啟動時進行版本檢查與防呆攔截**。
- **語法**：`platform.python_version()` / `platform.python_version_tuple()`

```python
import platform

# 1. 取得字串格式
ver_str = platform.python_version()
print(f"Python 版本: {ver_str}")  # 輸出例如: 3.12.3

# 2. 取得元組格式並進行版本檢查防呆
major, minor, patch = platform.python_version_tuple()

# 防呆檢查：要求 Python 必須 >= 3.10
if (int(major), int(minor)) < (3, 10):
    raise RuntimeError("❌ 本專案要求 Python 3.10 以上環境！")
```

---

##### platform.python_implementation() 與 python_compiler()

- **使用時機**：檢查當前運行的是標準的 CPython、還是具備 JIT 加速的 PyPy 等替代直譯器，以及編譯所使用的 C 編譯器。
- **語法**：`platform.python_implementation()` / `platform.python_compiler()`

```python
import platform

print("Python 實作:", platform.python_implementation())  # 輸出: CPython
print("C 編譯器:", platform.python_compiler())           # 輸出例如: Clang 15.0.0 (clang-1500.3.22.1)
```

---

## 4. 主機節點與綜合診斷 (Node & Uname)

##### platform.node() 取得電腦主機名稱

- **使用時機**：在多台伺服器叢集、分散式排程或微服務環境中，**識別當前程式是在哪一台主機上執行**。
- **語法**：`platform.node()`
- **回傳值**：
  - `str`：主機名稱（Computer Hostname）。

```python
import platform

host_name = platform.node()
print(f"當前電腦主機名稱: {host_name}")
```

---

##### platform.uname() 一鍵取得系統全景元組

- **使用時機**：需要**一次性取得所有核心系統屬性**時使用。
- **語法**：`info = platform.uname()`
- **回傳值**：
  - `uname_result` 具名元組（包含 `.system`, `.node`, `.release`, `.version`, `.machine`, `.processor` 屬性）。

```python
import platform

u = platform.uname()

print("系統全景診斷：")
print(f"  ├─ 作業系統: {u.system}")
print(f"  ├─ 主機名稱: {u.node}")
print(f"  ├─ 發行版本: {u.release}")
print(f"  ├─ 核心架構: {u.machine}")
print(f"  └─ 處理器  : {u.processor}")
```

---

# 實戰：跨平台專案自適應載入與環境報告生成

在開發現代跨平台應用程式時，結合 `platform` 可以自動完成環境適配與診斷報表：

```python
import platform

def generate_system_report() -> dict:
    """自動收集完整的環境診斷資訊字典"""
    return {
        "os": platform.system(),
        "os_release": platform.release(),
        "platform_str": platform.platform(terse=True),
        "cpu_arch": platform.machine(),
        "python_version": platform.python_version(),
        "python_impl": platform.python_implementation(),
        "bit_depth": platform.architecture()[0],
        "hostname": platform.node(),
    }

# 執行環境收集
report = generate_system_report()
for k, v in report.items():
    print(f"{k:15}: {v}")
```

---

# 實戰除錯與核心天條

## 1. macOS 系統名稱返回 'Darwin' 陷阱

> **⚠️ macOS 系統判斷陷阱**：  
> `platform.system()` 對 macOS 的回傳值是其底層 Unix 核心名稱 **`'Darwin'`**，**而不是 `'macOS'` 或 `'OSX'`**！  
> **鐵律**：判斷 Mac 系統時，請寫 `if platform.system() == "Darwin":`。

---

## 2. 避免使用版本字串直接做大小比對

> **⚠️ 字串版本比較陷阱**：  
> 切勿直接使用字串比較大小（例如 `"3.9"` > `"3.10"` 會得到 `True` 的荒謬錯誤，因為字串是按字典順序比對）！  
> **鐵律**：比較 Python 版本時，請使用 `platform.python_version_tuple()` 轉成數字元組 `(3, 10)`，或使用標準庫 `sys.version_info >= (3, 10)`。
