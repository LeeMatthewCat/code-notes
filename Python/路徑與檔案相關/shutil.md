# 概念與原理

## 什麼是 shutil 模組？

`shutil`（Shell Utility）是 Python 內建的高階檔案與目錄操作標準函式庫（無需額外安裝，直接 `import shutil`）。

專門用於處理**跨檔案複製、整棵目錄樹遞迴複製與搬移、目錄強力遞迴清除、歸檔壓縮/解壓縮，以及系統級磁碟空間診斷**。

與專注於底層單一檔案系統呼叫的 `os` 模組，以及專注於物件導向路徑計算的 `pathlib` 模組相比，`shutil` 補齊了 Python 在**高階檔案系統批次維運**上的最後一塊拼圖。

> **三大檔案操作模組分工邊界對照**：  
> - **`pathlib` 模組**：面向「路徑計算與現代物件導向語法」，負責路徑拼接、副檔名解析、純路徑運算與基本文字讀寫。  
> - **`os` 模組**：面向「底層系統呼叫與行程」，負責環境變數讀寫、檔案權限修改、單一空目錄建立與刪除（無法直接複製檔案，也無法直接刪除含有內容的非空目錄）。  
> - **`shutil` 模組**：面向「高階檔案系統批次維運」，專門處理整棵目錄樹的遞迴複製 (`copytree`)、強力刪除 (`rmtree`)、檔案搬移 (`move`)、壓縮打包 (`make_archive`) 與指令路徑探測 (`which`)。

---

# 核心 API 函數字典對照表

| 函式名稱 | 回傳型別 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- | :--- |
| **[[#shutil.copy2() 完整複製檔案與元數據\|shutil.copy2()]]** | `str` / `Path` | **複製檔案內容並完整保留所有元數據 (時間戳記/權限)** | **檔案備份首選**！防止建置工具因修改時間變動而誤判重新編譯 |
| **[[#shutil.copy() 複製檔案與權限\|shutil.copy()]]** | `str` / `Path` | **複製檔案內容與權限標記 (chmod)，不保留時間戳記** | 產生新副本且希望更新時間戳記為當前時間時 |
| **[[#shutil.copyfile() 純檔案內容高速複製\|shutil.copyfile()]]** | `str` / `Path` | **僅複製檔案資料內容，不複製任何權限與元數據** | 複製到目標檔案並繼承目標系統預設權限時 |
| **[[#shutil.copyfileobj() 底層檔案串流複製\|shutil.copyfileobj()]]** | `None` | **直接透過檔案描述符/物件串流複製資料** | 大檔案分塊串流複製、跨網路 Socket 串流管線 |
| **[[#shutil.copytree() 遞迴複製整棵目錄樹\|shutil.copytree()]]** | `str` / `Path` | **遞迴複製整個資料夾及其內部所有檔案與子資料夾** | 專案範本複製、部署套件發布、全目錄批次備份 |
| **[[#shutil.rmtree() 強力遞迴刪除目錄樹\|shutil.rmtree()]]** | `None` | **強力遞迴刪除整個資料夾（包含所有內容，無懼非空）** | 清理臨時建置目錄 (dist/build)、測試快照重置 |
| **[[#shutil.move() 遞迴移動檔案或目錄\|shutil.move()]]** | `str` / `Path` | **將檔案或整個目錄搬移到新位置（支援重新命名）** | 檔案歸檔、批次分類整理、重構目錄階層 |
| **[[#shutil.disk_usage() 查詢磁碟空間使用狀態\|shutil.disk_usage()]]** | `namedtuple` | **查詢指定掛載點或路徑的硬碟空間 (total, used, free)** | 磁碟剩餘空間監控、大型寫入任務前置容量防呆 |
| **[[#shutil.which() 尋找系統可執行檔路徑\|shutil.which()]]** | `str \| None` | **在系統 PATH 中尋找指定可執行檔的絕對路徑** | 檢查外部 CLI 工具 (git, ffmpeg, docker) 是否安裝 |
| **[[#shutil.make_archive() 建立壓縮歸檔檔案\|shutil.make_archive()]]** | `str` | **將指定目錄打包壓縮成歸檔檔案 (zip, tar, gztar 等)** | 自動化備份打包、生成發行版 Release 壓縮包 |
| **[[#shutil.unpack_archive() 解壓縮歸檔檔案\|shutil.unpack_archive()]]** | `None` | **自動識別壓縮格式並將歸檔檔案解壓縮至目標目錄** | 自動化部署套件解壓、匯入資料封包解封 |
| **[[#shutil.get_archive_formats() 查詢支援壓縮格式\|shutil.get_archive_formats()]]** | `list[tuple]` | **取得當前執行環境所支援的所有壓縮打包格式清單** | 檢查環境是否支援 zip, tar, gztar, bztar 等格式 |
| **[[#shutil.get_unpack_formats() 查詢支援解壓縮格式\|shutil.get_unpack_formats()]]** | `list[tuple]` | **取得當前執行環境所支援的所有解壓縮格式清單** | 驗證外部壓縮檔格式是否可被原生解壓縮 |
| **[[#shutil.chown() 變更檔案擁有者與群組\|shutil.chown()]]** | `None` | **變更指定路徑的擁有者 (user) 與群組 (group)** | Linux 伺服器服務部署、容器權限降權配置 |

---

# 核心功能與語法大解密

## 1. 檔案複製四大函式深度對決

在 Python 中複製一個檔案有四種不同層次的函式，理解它們對**「內容」**、**「權限 (chmod)」** 與 **「時間戳記 (mtime/atime)」** 的保留程度是避免非預期行為的關鍵。

```text
copyfileobj() ──► 僅複製 open() 物件位元組串流
copyfile()    ──► 僅複製硬碟檔案內容 (不帶權限、不帶時間)
copy()        ──► 複製檔案內容 + 檔案權限 (chmod)
copy2()       ──► 複製檔案內容 + 完整元數據 (包含時間戳記 mtime/atime) 【最佳實踐】
```

---

##### shutil.copy2() 完整複製檔案與元數據

- **使用時機**：執行常態檔案備份時的**首選推薦函式**。完整保留原始檔案的建立時間、修改時間與讀寫執行權限，行為最接近 Unix 的 `cp -p`。
- **語法**：`shutil.copy2(src, dst, *, follow_symlinks=True)`
- **參數說明**：
  - `src`：來源檔案路徑（字串或 `Path` 物件）。
  - `dst`：目的地檔案路徑，或已存在的目的目錄（若傳入既有目錄，則會以來源檔名在該目錄下建立複本）。
  - `follow_symlinks`：布林值，預設為 `True`。若為 `False` 且 `src` 是符號連結，則直接複製符號連結本體而非指向的實體。
- **回傳值**：
  - `str` 或 `Path`：新複製檔案的最終絕對或相對路徑字串。

```python
import os
import shutil
from pathlib import Path

source = Path("project_config.json")
source.write_text('{"version": "1.0.0"}', encoding="utf-8")

# 1. 完整複製檔案並保留原始元數據 (包含修改時間)
backup_path = shutil.copy2(source, "project_config.backup.json")

print(f"備份成功，目標路徑: {backup_path}")
print(f"來源修改時間: {os.path.getmtime(source)}")
print(f"備份修改時間: {os.path.getmtime(backup_path)}")  # 兩者時間戳記完全一致！
```

---

##### shutil.copy() 複製檔案與權限

- **使用時機**：需要複製檔案內容與執行權限，但**希望新檔案的時間戳記刷新為當前複製時間**時使用。
- **語法**：`shutil.copy(src, dst, *, follow_symlinks=True)`
- **參數說明**：
  - `src`：來源檔案路徑。
  - `dst`：目的地檔案路徑或目的地資料夾路徑。
  - `follow_symlinks`：是否跟隨符號連結，預設為 `True`。
- **回傳值**：
  - `str` 或 `Path`：新檔案的最終路徑。

```python
import os
import shutil
import time

# 複製檔案，權限保留但修改時間會變更為現在時間
dest_file = shutil.copy("run_script.sh", "dist_script.sh")

# 檢查新檔案權限是否與原檔案一致
print(f"原檔案權限: {oct(os.stat('run_script.sh').st_mode)[-3:]}")
print(f"新檔案權限: {oct(os.stat(dest_file).st_mode)[-3:]}")
```

---

##### shutil.copyfile() 純檔案內容高速複製

- **使用時機**：僅需要將檔案內的位元組資料拷貝至另一個檔案，完全不在乎元數據與權限時使用。
- **語法**：`shutil.copyfile(src, dst, *, follow_symlinks=True)`
- **參數說明**：
  - `src`：來源檔案路徑。
  - `dst`：目的地**完整檔案路徑**（注意：`dst` **必須是具體檔案名稱，不能是目錄**，否則會引發 `IsADirectoryError`）。
- **回傳值**：
  - `str` 或 `Path`：新檔案的路徑。

```python
import shutil

# 高速拷貝純資料內容 (dst 必須指定檔案名稱)
shutil.copyfile("raw_dataset.csv", "backup_dataset.csv")
```

---

##### shutil.copyfileobj() 底層檔案串流複製

- **使用時機**：來源與目的地已經是以 `open()` 開啟的檔案物件、跨行程管道或網路 Socket 串流，需要以緩衝區方式分塊拷貝時使用。
- **語法**：`shutil.copyfileobj(fsrc, fdst, length=0)`
- **參數說明**：
  - `fsrc`：已開啟的來源檔案物件（具備 `.read()` 方法）。
  - `fdst`：已開啟的目的地檔案物件（具備 `.write()` 方法）。
  - `length`：緩衝區大小（位元組數）。預設 `0` 或負數代表由系統自動選擇最佳緩衝大小（通常為 64KB 或 1MB）。
- **回傳值**：
  - `None`。

```python
import shutil

# 適合拷貝超大型檔案，記憶體開銷恆定，不一次性讀入記憶體
with open("massive_video.mp4", "rb") as src, open("output_video.mp4", "wb") as dst:
    # 每次以 1MB 緩衝區進行串流傳輸
    shutil.copyfileobj(src, dst, length=1024 * 1024)
```

---

### 複製四大家族能力對比全景表

| 函式名稱 | 複製資料內容 | 複製檔案權限 (chmod) | 複製時間戳記 (mtime/atime) | `dst` 接受資料夾目錄 |
| :--- | :---: | :---: | :---: | :---: |
| **`shutil.copyfileobj()`** | 支援 | 不支援 | 不支援 | 否（僅吃檔案物件） |
| **`shutil.copyfile()`** | 支援 | 不支援 | 不支援 | **否**（若傳入目錄引發錯誤） |
| **`shutil.copy()`** | 支援 | **支援** | 不支援 | **支援**（自動使用原檔名） |
| **`shutil.copy2()` (首選)** | 支援 | **支援** | **支援** | **支援**（自動使用原檔名） |

---

## 2. 目錄樹遞迴操作 (複製、刪除與搬移)

##### shutil.copytree() 遞迴複製整棵目錄樹

- **使用時機**：需要將整個專案、範本資料夾或深層巢狀目錄完整複製一份時使用。
- **語法**：`shutil.copytree(src, dst, symlinks=False, ignore=None, copy_function=copy2, ignore_dangling_symlinks=False, dirs_exist_ok=False)`
- **參數說明**：
  - `src`：來源目錄路徑。
  - `dst`：目的地目錄路徑。
  - `dirs_exist_ok`：布林值，**極為重要的實用參數 (Python 3.8+)**。預設為 `False`（若目標目錄已存在會引發 `FileExistsError`）。設為 `True` 允許目標目錄已存在，會進行檔案合併與覆蓋。
  - `ignore`：傳入一個可呼叫函式，接收 `(目錄路徑, [內部檔案清單])` 並回傳應被忽略的名稱集合。通常搭配 `shutil.ignore_patterns()` 使用。
  - `copy_function`：底層用於複製單一檔案的函式，預設為 `shutil.copy2`。
- **回傳值**：
  - `str` 或 `Path`：複製後的目的地目錄路徑。

```python
import shutil
from pathlib import Path

# 1. 建立示範目錄結構
Path("src_app/core").mkdir(parents=True, exist_ok=True)
Path("src_app/core/main.py").write_text("print('App')", encoding="utf-8")
Path("src_app/temp.log").write_text("debug info", encoding="utf-8")

# 2. 遞迴複製目錄，並排除所有 .log 與快取檔案
dest = shutil.copytree(
    "src_app",
    "backup_app",
    ignore=shutil.ignore_patterns("*.log", "__pycache__"),
    dirs_exist_ok=True  # 允許目標目錄已存在，自動覆蓋合併
)

print(f"目錄樹複製完成: {dest}")
```

---

##### shutil.rmtree() 強力遞迴刪除目錄樹

- **使用時機**：需要徹底銷毀一個資料夾及其內部所有檔案與子資料夾時使用。在 `os.rmdir()` 無法刪除含有內容的非空目錄時，必須使用此函式。
- **語法**：`shutil.rmtree(path, ignore_errors=False, onexc=None)`
- **參數說明**：
  - `path`：欲刪除的目錄路徑。
  - `ignore_errors`：布林值。設為 `True` 時會忽略所有刪除過程中的權限與檔案不存在錯誤（靜默失敗）。
  - `onexc`：(Python 3.12+ 推薦，取代舊版 `onerror`) 例外處理回呼函式，用來處理特定檔案唯讀或無權限刪除的狀況。
- **回傳值**：
  - `None`。

```python
import os
import shutil
import stat
from pathlib import Path

# 處理 Windows / Linux 下唯讀檔案無法被 rmtree 刪除的防禦回呼
def handle_remove_readonly(func, path, exc_info):
    # 清除唯讀屬性，強制賦予寫入權限
    os.chmod(path, stat.S_IWRITE)
    func(path)

# 強力安全清除臨時目錄
target_dir = Path("build_artifacts")
if target_dir.exists():
    shutil.rmtree(target_dir, onexc=handle_remove_readonly)
    print("目錄樹已徹底移除！")
```

> **核彈級危險警告**：  
> `shutil.rmtree()` **不會將檔案移至系統資源回收筒 (Trash/Recycle Bin)**，而是直接進行磁碟級的永久抹除！  
> 呼叫前務必進行路徑檢驗，**嚴禁傳入空字串、根目錄 `/` 或未經驗證的使用者輸入路徑**！

---

##### shutil.move() 遞迴移動檔案或目錄

- **使用時機**：需要重新命名檔案、重新命名整個目錄，或將檔案/目錄搬遷至另一個資料夾時使用。
- **語法**：`shutil.move(src, dst, copy_function=copy2)`
- **參數說明**：
  - `src`：來源檔案或目錄路徑。
  - `dst`：目的地檔案或目錄路徑。
    - 若 `src` 與 `dst` 位在**同一個檔案系統磁區**，底層會直接呼叫 `os.rename()` 實現**秒級原子重新命名**。
    - 若 `src` 與 `dst` 位在**跨硬碟/不同磁區**，底層會自動先執行 `copytree()` 或 `copy2()` 複製，確認成功後再將原位置檔案安全刪除。
- **回傳值**：
  - `str` 或 `Path`：移動後的新路徑。

```python
import shutil
from pathlib import Path

Path("daily_task.log").write_text("task done", encoding="utf-8")
Path("archive_logs").mkdir(exist_ok=True)

# 將日誌檔案搬移至封存資料夾內
new_location = shutil.move("daily_task.log", "archive_logs/daily_task.log")
print(f"檔案已成功搬遷至: {new_location}")
```

---

## 3. 歸檔壓縮與解壓縮管理

`shutil` 內建支援常見的壓縮格式封裝與解壓，無需手動建立複雜的 `zipfile` 或 `tarfile` 流程。

---

##### shutil.make_archive() 建立壓縮歸檔檔案

- **使用時機**：將專案、日誌資料夾或備份資料打包成 `.zip` 或 `.tar.gz` 壓縮檔時使用。
- **語法**：`shutil.make_archive(base_name, format, root_dir=None, base_dir=None, ...)`
- **參數說明**：
  - `base_name`：壓縮檔產出的路徑與名稱（**無需副檔名**，系統會根據 `format` 自動追加 `.zip` 等）。
  - `format`：壓縮格式，如 `'zip'`, `'tar'`, `'gztar'`, `'bztar'`, `'xztar'`。
  - `root_dir`：作為壓縮檔案根目錄的路徑（壓縮時的基準點）。
  - `base_dir`：在 `root_dir` 內部開始進行壓縮的子目錄路徑（預設為當前目錄）。
- **回傳值**：
  - `str`：生成之壓縮檔案的絕對路徑名稱。

```python
import shutil

# 將 my_project 資料夾打包為 my_project_backup.zip
archive_file = shutil.make_archive(
    base_name="backup/my_project_backup",  # 輸出路徑 (不用寫 .zip)
    format="zip",                          # 壓縮格式
    root_dir="my_project"                  # 要被壓縮的來源資料夾
)

print(f"壓縮封裝完成: {archive_file}")
```

---

##### shutil.unpack_archive() 解壓縮歸檔檔案

- **使用時機**：自動識別格式並將壓縮檔案解封展開至指定目錄時使用。
- **語法**：`shutil.unpack_archive(filename, extract_dir=None, format=None)`
- **參數說明**：
  - `filename`：壓縮檔案的完整路徑。
  - `extract_dir`：解壓縮後的目標目錄（若目錄不存在會自動建立，預設為當前工作目錄）。
  - `format`：壓縮格式（預設為 `None`，系統會自動根據檔案副檔名進行智慧推斷）。
- **回傳值**：
  - `None`。

```python
import shutil

# 自動辨識 zip 或 tar 格式並解壓縮至 output_folder 目錄
shutil.unpack_archive("backup/my_project_backup.zip", extract_dir="restored_project")
print("解壓縮完成！")
```

---

##### shutil.get_archive_formats() 查詢支援壓縮格式

- **使用時機**：在不同作業系統環境下執行打包前，動態檢查支援哪些壓縮演算法。
- **語法**：`shutil.get_archive_formats()`
- **回傳值**：
  - `list[tuple[str, str]]`：格式名稱與說明的元組清單。

```python
import shutil

formats = shutil.get_archive_formats()
for name, desc in formats:
    print(f"支援格式: {name:<8} 說明: {desc}")
```

---

##### shutil.get_unpack_formats() 查詢支援解壓縮格式

- **使用時機**：動態查詢當前 Python 環境支援哪些副檔名的解壓縮操作。
- **語法**：`shutil.get_unpack_formats()`
- **回傳值**：
  - `list[tuple[str, list[str], str]]`：格式名稱、關聯副檔名與說明的元組清單。

```python
import shutil

for fmt in shutil.get_unpack_formats():
    print(f"解壓格式: {fmt[0]:<8} 對應副檔名: {fmt[1]}")
```

---

## 4. 系統環境與磁碟診斷

##### shutil.disk_usage() 查詢磁碟空間使用狀態

- **使用時機**：在執行巨量資料下載、解壓縮大型檔案或建立資料庫備份前，檢查硬碟剩餘空間是否足夠，進行容量防呆。
- **語法**：`shutil.disk_usage(path)`
- **參數說明**：
  - `path`：磁碟掛載點或任意檔案/目錄路徑（例如 `'/'`、`'C:\\'` 或當前目錄 `'.'`）。
- **回傳值**：
  - `namedtuple`（包含三個以 Bytes 為單位的整數欄位）：
    - `.total`：磁碟總容量。
    - `.used`：已使用容量。
    - `.free`：剩餘可用容量。

```python
import shutil

# 查詢當前硬碟空間
usage = shutil.disk_usage(".")

gb_factor = 1024 ** 3  # 轉為 GB 單位
total_gb = usage.total / gb_factor
used_gb = usage.used / gb_factor
free_gb = usage.free / gb_factor

print(f"磁碟總空間:   {total_gb:.2f} GB")
print(f"已使用空間:   {used_gb:.2f} GB ({used_gb / total_gb * 100:.1f}%)")
print(f"剩餘可用空間: {free_gb:.2f} GB")

# 容量防呆檢查
if free_gb < 5.0:
    print("警告：磁碟剩餘空間不足 5GB，請清理磁碟！")
```

---

##### shutil.which() 尋找系統可執行檔路徑

- **使用時機**：撰寫自動化腳本或呼叫 `subprocess` 外部指令前，探測使用者的系統中是否已安裝該命令（等同於在 Linux/macOS 終端機執行 `which <command>`，或在 Windows 執行 `where <command>`）。
- **語法**：`shutil.which(cmd, mode=os.F_OK | os.X_OK, path=None)`
- **參數說明**：
  - `cmd`：欲尋找的可執行檔名稱（例如 `'git'`、`'ffmpeg'`、`'python3'`、`'docker'`）。
  - `mode`：權限檢查旗標，預設檢查檔案存在且具備執行權限 (`os.F_OK | os.X_OK`)。
  - `path`：自訂搜尋的環境變數路徑字串（預設讀取系統的 `os.environ["PATH"]`）。
- **回傳值**：
  - `str`：找到的執行檔**絕對路徑字串**。
  - `None`：若在系統 PATH 內完全找不到該可執行檔，回傳 `None`。

```python
import shutil
import subprocess

# 1. 檢查系統中是否存在 git 命令
git_path = shutil.which("git")
if git_path:
    print(f"已找到 git 執行檔，路徑為: {git_path}")
    # 安全呼叫版本指令
    subprocess.run([git_path, "--version"])
else:
    print("錯誤：系統尚未安裝 git，請先安裝後再繼續！")

# 2. 檢查 ffmpeg 是否可用
if shutil.which("ffmpeg") is None:
    print("提醒：ffmpeg 不存在，將停用影片轉檔功能")
```

---

##### shutil.chown() 變更檔案擁有者與群組

- **使用時機**：在 Unix / Linux 伺服器上部署程式時，調整目錄或檔案的擁有者 (User) 與所屬群組 (Group)。（注意：Windows 系統不支援此函式）。
- **語法**：`shutil.chown(path, user=None, group=None)`
- **參數說明**：
  - `path`：檔案或目錄路徑。
  - `user`：新擁有者名稱（字串）或 UID（整數）。
  - `group`：新所屬群組名稱（字串）或 GID（整數）。
- **回傳值**：
  - `None`。

```python
import shutil
import sys

# 僅限 Unix / Linux 系統執行
if sys.platform != "win32":
    # 將 /var/www/html 目錄擁有者設定為 www-data
    shutil.chown("/var/www/html", user="www-data", group="www-data")
    print("權限擁有者更新完成！")
```

---

# 實戰避坑與開發天條

> **天條一：備份檔案永遠優先使用 `copy2()` 而非 `copy()`**  
> `copy()` 會將目標檔案的時間戳記重置為「當下建立時間」，這會導致許多自動化建置工具（如 Make、Webpack、增量備份腳本）因時間戳記改變而誤判檔案被修改，進而觸發不必要的全量重新建置。  
> 只有 `copy2()` 會忠實還原原始檔案的 `mtime` 與 `atime`。

> **天條二：`copytree()` 遇目標目錄已存在時的崩潰陷阱**  
> 在 Python 3.8 之前，只要 `dst` 目錄已存在，`copytree()` 就會立即拋出 `FileExistsError` 崩潰。  
> **最佳實踐**：務必顯式指定 `dirs_exist_ok=True`，確保在目標目錄已存在時能順暢執行增量檔案覆蓋與合併。

> **天條三：Windows 平台下 `rmtree()` 刪除唯讀檔案的崩潰防禦**  
> 在 Windows 檔案系統中，若資料夾內包含具備唯讀屬性 (Read-only) 的檔案（如 `.git` 內部的某些索引檔），直接執行 `shutil.rmtree()` 會拋出 `PermissionError: [WinError 5] 存取被拒`。  
> **標準防禦解法**：在呼叫 `rmtree()` 時傳入自訂的 `onexc`（或舊版 `onerror`）回呼函式，遇到權限阻擋時先透過 `os.chmod(path, stat.S_IWRITE)` 清除唯讀屬性，再重試刪除！

> **天條四：呼叫外部指令前一律先用 `which()` 進行健康檢查**  
> 避免直接用 `subprocess.run(["ffmpeg", ...])` 賭使用者電腦有沒有安裝工具，否則找不到指令時會直接拋出 `FileNotFoundError: [Errno 2] No such file or directory`。  
> 養成在發起子行程前，先以 `if shutil.which("ffmpeg"):` 進行探測的良好防禦習慣。
