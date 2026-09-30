# 概念與原理

## 什麼是 mimetypes 模組？

`mimetypes` 是 Python 內建的檔案類型對照標準函式庫（無需額外安裝，直接 `import mimetypes`）。

核心職責在於處理**「檔案副檔名 (Filename Extension)」**與**「MIME 媒體類型 (MIME Type / Media Type)」**之間的雙向轉換。

在處理網路請求、檔案上傳、靜態資源託管或電子郵件附件時，伺服器必須透過正確的 `Content-Type` 標頭告知客戶端該如何解讀接收到的位元組串流。

`mimetypes` 模組內建了龐大的官方 IANA 標準資料庫，並在啟動時自動讀取作業系統自帶的對照表（如 Linux/macOS 的 `/etc/mime.types` 與 Windows 的登錄檔），提供極為高效的本地查表能力。

> **[生動比喻]**：  
> - **檔案副檔名（如 `.jpg`）**：如同一個人「隨意穿上的衣服外套」，任何人都能輕易更換或偽裝（將可執行檔改名為 `.jpg`）。  
> - **MIME 類型（如 `image/jpeg`）**：如同「國際公認的官方身份識別規格」，正式宣告該資料的本質結構與解析協定。  
> - **`mimetypes` 模組**：如同「海關的身分對照登記處」，負責在「外觀衣服標籤（副檔名）」與「通關身分代碼（MIME 類型）」之間建立標準化對照索引！

---

## MIME 類型結構剖析

MIME 類型由兩個主要字串構成，形式為 `type/subtype`：

```text
Content-Type: text/html
              └──┬──┘ └──┬──┘
              主要型態   子型態 (Subtype)
              (Type)
```

- **主要型態 \(Type\)**：定義資料的總類別，常見有 `text`（文字）、`image`（圖像）、`audio`（音訊）、`video`（視訊）、`application`（二進位應用資料）、`multipart`（多段封裝）。
- **子型態 \(Subtype\)**：定義特定格式細節，例如 `html`、`json`、`png`、`pdf`、`zip`。

---

## 現代語法升級與版本演進對照表

| 功能操作 | 舊式語法 (已棄用 / 易踩坑) | 現代標準語法 (推薦) | 核心差異與說明 |
| :--- | :--- | :--- | :--- |
| **路徑物件解析** | `mimetypes.guess_type(str(path))` | `mimetypes.guess_type(path)` | Python 3.8+ 原生支援 `os.PathLike`，可直接傳入 `pathlib.Path` 物件 |
| **多重副檔名反查** | 僅能呼叫 `guess_extension()` 取得單一副檔名 | `mimetypes.guess_all_extensions()` | Python 3.8+ 支援獲取該 MIME 類型對應的所有已知副檔名串列 |
| **URL 網址型態探測** | 直接傳入帶 Query 參數的完整網址 | 先以 `urlsplit(url).path` 清理後再查詢 | 避免 `?query=1` 或 `#hash` 被誤判為副檔名的一部分導致查詢失敗 |
| **資料庫狀態隔離** | 直接使用模組級 `mimetypes.add_type()` | 建立獨立 `mimetypes.MimeTypes()` 實例物件 | 避免修改全域狀態污染其他第三方套件或多線程併發衝突 |

---

# 核心 API 函式與屬性字典對照表

| 函式 / 屬性名稱 | 回傳型別 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- | :--- |
| **[[#mimetypes.guess_type() 猜測檔案 MIME 類型\|mimetypes.guess_type()]]** | `tuple[str \| None, str \| None]` | **根據檔名或 URL 猜測 MIME 類型與壓縮編碼** | 設定 HTTP Content-Type、決定郵件附件格式 |
| **[[#mimetypes.guess_extension() 查詢 MIME 類型對應副檔名\|mimetypes.guess_extension()]]** | `str \| None` | **根據 MIME 類型反查最適合的副檔名** | 接收 Web API 下載檔案時自動命名並補齊副檔名 |
| **[[#mimetypes.guess_all_extensions() 查詢 MIME 類型所有關聯副檔名\|mimetypes.guess_all_extensions()]]** | `list[str]` | **取得指定 MIME 類型對應的所有已知副檔名串列** | 檔案上傳白名單校驗、多副檔名相容性比對 |
| **[[#mimetypes.add_type() 動態註冊自訂 MIME 映射\|mimetypes.add_type()]]** | `None` | **動態新增或覆寫自訂副檔名與 MIME 類型對應關係** | 支援專案自訂資料格式、擴充新興格式支援 |
| **[[#mimetypes.init() 顯式初始化型別對照資料庫\|mimetypes.init()]]** | `None` | **載入系統預設檔案或指定檔案重新建立資料庫** | 啟動時批量載入外部自訂 mime.types 設定檔 |
| **[[#mimetypes.read_mime_types() 解析外部 MIME 定義檔\|mimetypes.read_mime_types()]]** | `dict[str, str] \| None` | **解析指定路徑之 mime.types 文字檔並返回字典** | 純文字讀取外部對照表且不污染全域資料庫 |
| **[[#mimetypes.MimeTypes 獨立資料庫類別實例\|mimetypes.MimeTypes]]** | `MimeTypes` | **建立獨立、隔離的 MIME 對照資料庫實例** | 多租戶架構、防禦全域污染、執行緒安全隔離 |
| **[[#mimetypes.types_map 官方標準 MIME 映射字典\|mimetypes.types_map]]** | `dict[str, str]` | **儲存官方標準副檔名到 MIME 類型的全域對照字典** | 唯讀除錯、快速檢視當前已載入之標準類型 |
| **[[#mimetypes.common_types 非標準通用 MIME 映射字典\|mimetypes.common_types]]** | `dict[str, str]` | **儲存常見非標準副檔名到 MIME 類型的對照字典** | 查詢非 IANA 官方註冊之通俗格式定義 |
| **[[#mimetypes.encodings_map 檔案壓縮編碼映射字典\|mimetypes.encodings_map]]** | `dict[str, str]` | **副檔名到壓縮演算法名稱的對照字典** | 識別 gzip、bzip2、xz 等傳輸編碼尾綴 |
| **[[#mimetypes.suffix_map 複合副檔名轉換映射字典\|mimetypes.suffix_map]]** | `dict[str, str]` | **簡寫副檔名映射到複合副檔名的對照字典** | 將 `.tgz` 自動正規化為 `.tar.gz` |

---

# 核心功能與語法大解密

## 1. 核心查詢與推測函式

##### mimetypes.guess_type() 猜測檔案 MIME 類型

- **使用時機**：當你需要根據檔案名稱、磁碟路徑或網路 URL，判斷檔案的 MIME 類型與傳輸壓縮編碼時使用。
- **語法**：`mimetypes.guess_type(url, strict=True)`
- **參數說明**：
  - `url`：檔案名稱、路徑（支援 `str` 或 `pathlib.Path` 物件）或 URL 字串。
  - `strict`：布林值，預設為 `True`。
      - 設為 `True` 時，僅搜尋 IANA 官方認證的標準 MIME 類型。
      - 設為 `False` 時，額外包含常見非標準通用類型（如早期影音格式）。
- **回傳值**：
  - `tuple[str | None, str | None]`：包含兩個元素的二元元組 `\(type, encoding\)`：
      - `type`：MIME 類型字串（如 `'application/json'`），若無法識別則為 `None`。
      - `encoding`：壓縮編碼演算法名稱（如 `'gzip'`, `'bzip2'`），若未壓縮則為 `None`。

```python
import mimetypes
from pathlib import Path

# 1. 解析普通檔案副檔名
mime_type, encoding = mimetypes.guess_type("report.pdf")
print(mime_type)  # 輸出: application/pdf
print(encoding)   # 輸出: None

# 2. 解析壓縮檔案 (識別雙重副檔名)
mime_type, encoding = mimetypes.guess_type("archive.tar.gz")
print(mime_type)  # 輸出: application/x-tar
print(encoding)   # 輸出: gzip

# 3. 原生支援 pathlib.Path 物件 (Python 3.8+)
file_path = Path("images/banner.png")
print(mimetypes.guess_type(file_path))  # 輸出: ('image/png', None)
```

---

##### mimetypes.guess_extension() 查詢 MIME 類型對應副檔名

- **使用時機**：當你只知道 MIME 類型（例如從 Web API 或 HTTP 標頭取得 `Content-Type: image/jpeg`），需要反查其標準副檔名以儲存檔案時使用。
- **語法**：`mimetypes.guess_extension(type, strict=True)`
- **參數說明**：
  - `type`：MIME 類型字串（例如 `'image/jpeg'`、`'application/json'`）。
  - `strict`：布林值，預設為 `True`。控制是否僅考慮官方標準類型。
- **回傳值**：
  - `str | None`：以句點開頭的小寫副檔名字串（例如 `'.jpg'`），若無法匹配則回傳 `None`。

```python
import mimetypes

# 1. 查詢常見圖像格式副檔名
ext_jpg = mimetypes.guess_extension("image/jpeg")
print(ext_jpg)  # 輸出: .jpg

# 2. 查詢文件格式副檔名
ext_pdf = mimetypes.guess_extension("application/pdf")
print(ext_pdf)  # 輸出: .pdf

# 3. 查詢未知類型
ext_unknown = mimetypes.guess_extension("application/x-custom-binary")
print(ext_unknown)  # 輸出: None
```

---

##### mimetypes.guess_all_extensions() 查詢 MIME 類型所有關聯副檔名

- **使用時機**：當你需要取得特定 MIME 類型對應的「所有可能副檔名」串列（例如比對副檔名白名單、驗證多種合法副檔名相容性）時使用。
- **語法**：`mimetypes.guess_all_extensions(type, strict=True)`
- **參數說明**：
  - `type`：MIME 類型字串。
  - `strict`：布林值，預設為 `True`。是否僅搜尋標準格式。
- **回傳值**：
  - `list[str]`：包含所有關聯副檔名的字串串列。若查無資料則回傳空串列 `[]`。

```python
import mimetypes

# 1. 獲取 JPEG 關聯的所有合法副檔名
all_exts = mimetypes.guess_all_extensions("image/jpeg")
print(all_exts)  # 輸出: ['.jpg', '.jpe', '.jpeg']

# 2. 進行白名單副檔名安全校驗
user_filename = "avatar.jpeg"
is_valid = any(user_filename.endswith(ext) for ext in mimetypes.guess_all_extensions("image/jpeg"))
print(is_valid)  # 輸出: True
```

---

## 2. 動態註冊與資料庫管理

##### mimetypes.add_type() 動態註冊自訂 MIME 映射

- **使用時機**：當系統預設對照表缺少某些專案自訂副檔名、新興檔案格式（如舊版環境尚未登錄之格式），需要手動擴充對照表時使用。
- **語法**：`mimetypes.add_type(type, ext, strict=True)`
- **參數說明**：
  - `type`：MIME 類型字串（例如 `'application/x-yaml'`）。
  - `ext`：檔案副檔名字串，必須帶有前導句點（例如 `'.yaml'`）。
  - `strict`：布林值，預設為 `True`。設定為 `True` 寫入標準對照表；設定為 `False` 寫入非標準表。
- **回傳值**：
  - `None`。

```python
import mimetypes

# 1. 註冊自訂檔案格式
mimetypes.add_type("application/x-myapp-data", ".mydata")

# 2. 立即驗證註冊結果
guessed_type, _ = mimetypes.guess_type("export_2026.mydata")
print(guessed_type)  # 輸出: application/x-myapp-data
```

---

##### mimetypes.init() 顯式初始化型別對照資料庫

- **使用時機**：模組在首次呼叫查詢函式時會自動延遲初始化。若你想在程式啟動時預先載入、重置資料庫，或明確載入特定外部 `mime.types` 檔案清單時調用。
- **語法**：`mimetypes.init(files=None)`
- **參數說明**：
  - `files`：可選參數。可傳入包含檔案路徑字串的序列（例如 `['/app/config/mime.types']`）。若為 `None`，則自動搜尋系統預設路徑與 Windows 註冊表。
- **回傳值**：
  - `None`。

```python
import mimetypes

# 1. 顯式初始化預設全域資料庫
mimetypes.init()
print(mimetypes.inited)  # 輸出: True

# 2. 擴充指定外部設定檔 (不覆蓋既有預設表，而是增量合併)
# mimetypes.init(files=["/etc/custom-apache.types"])
```

---

##### mimetypes.read_mime_types() 解析外部 MIME 定義檔

- **使用時機**：需要純粹解析特定 `mime.types` 格式文字檔，將其轉換為 Python 原生字典，且**完全不想污染或改變**全域資料庫時使用。
- **語法**：`mimetypes.read_mime_types(filename)`
- **參數說明**：
  - `filename`：目標 `mime.types` 格式檔案的路徑字串。
- **回傳值**：
  - `dict[str, str] | None`：鍵為副檔名（含句點）、值為 MIME 類型的字典。若檔案不存在或讀取失敗則回傳 `None`。

```python
import mimetypes

# 讀取自訂設定檔並提取字典
parsed_dict = mimetypes.read_mime_types("/etc/mime.types")
if parsed_dict:
    print(parsed_dict.get(".png"))  # 假設輸出: image/png
```

---

## 3. 物件導向與隔離資料庫

##### mimetypes.MimeTypes 獨立資料庫類別實例

- **使用時機**：在大型應用程式、多租戶架構或獨立套件中，需要維護完全隔離的 MIME 規則，避免多個模組因修改全域 `types_map` 而產生衝突時使用。
- **語法**：`mimetypes.MimeTypes(filenames=(), strict=True)`
- **參數說明**：
  - `filenames`：可選，初始化時需讀取的自訂檔案路徑序列。
  - `strict`：布林值，預設為 `True`。
- **回傳值**：
  - `mimetypes.MimeTypes` 物件實例，擁有與模組層級完全同名的方法（如 `.guess_type()`、`.add_type()`）。

```python
from mimetypes import MimeTypes

# 1. 建立獨立實例物件
custom_db = MimeTypes()

# 2. 在隔離實例中註冊格式 (完全不影響全域模組)
custom_db.add_type("application/vnd.company.doc", ".cdoc")

# 3. 實例查詢成功
print(custom_db.guess_type("manual.cdoc"))  # 輸出: ('application/vnd.company.doc', None)
```

---

## 4. 全域對照字典與內部結構屬性

##### mimetypes.types_map 官方標準 MIME 映射字典

- **使用時機**：底層全域字典，儲存所有官方標準副檔名（含句點小寫）到 MIME 類型的對照關係。適合進行只讀檢查或快速遍歷。
- **語法**：`mimetypes.types_map`
- **屬性型別**：`dict[str, str]`

```python
import mimetypes

mimetypes.init()
# 快速檢視副檔名映射數量與具體內容
print(type(mimetypes.types_map))  # 輸出: <class 'dict'>
print(mimetypes.types_map[".html"])  # 輸出: text/html
```

---

##### mimetypes.common_types 非標準通用 MIME 映射字典

- **使用時機**：儲存常見但非 IANA 官方正式標準的副檔名對照表。當 `guess_type(..., strict=False)` 時會一併檢索此字典。
- **語法**：`mimetypes.common_types`
- **屬性型別**：`dict[str, str]`

```python
import mimetypes

mimetypes.init()
print(type(mimetypes.common_types))  # 輸出: <class 'dict'>
```

---

##### mimetypes.encodings_map 檔案壓縮編碼映射字典

- **使用時機**：儲存常見壓縮檔案副檔名到編碼演算法的對照表。`guess_type()` 依據此表解析回傳值中的 `encoding` 欄位。
- **語法**：`mimetypes.encodings_map`
- **屬性型別**：`dict[str, str]`

```python
import mimetypes

mimetypes.init()
print(mimetypes.encodings_map.get(".gz"))   # 輸出: gzip
print(mimetypes.encodings_map.get(".bz2"))  # 輸出: bzip2
```

---

##### mimetypes.suffix_map 複合副檔名轉換映射字典

- **使用時機**：儲存簡化複合副檔名的重寫規則字典。例如將 `.tgz` 自動轉換為 `.tar.gz`，使模組能接續解析出其底層的真實資料類型與壓縮編碼。
- **語法**：`mimetypes.suffix_map`
- **屬性型別**：`dict[str, str]`

```python
import mimetypes

mimetypes.init()
print(mimetypes.suffix_map.get(".tgz"))  # 輸出: .tar.gz
```

---

# 實戰避坑與核心天條

## 1. 帶有查詢參數之 URL 造成 MIME 猜測徹底失效

> **[核心天條]**：傳入 URL 進行 MIME 猜測前，必須先以 `urllib.parse.urlsplit()` 去除 Query String 與 Hash 錨點！

- **錯誤症狀**：
  當傳入帶有查詢參數的下載連結時，`mimetypes.guess_type()` 回傳 `(None, None)`。
- **背後原理**：
  `mimetypes.guess_type` 底層使用 `posixpath.splitext` 切割副檔名。若網址尾端帶有 `?download=1`，副檔名會被識別為 `.pdf?download=1`，導致字串比對失敗。

```python
import mimetypes
from urllib.parse import urlsplit

raw_url = "https://example.com/assets/report.pdf?version=2&token=abc"

# [錯誤寫法]：直接傳入原始 URL，副檔名被截取錯誤
print(mimetypes.guess_type(raw_url))
# 輸出: (None, None)

# [正確寫法]：先取出純淨的路徑部分再交給 mimetypes 猜測
clean_path = urlsplit(raw_url).path
print(mimetypes.guess_type(clean_path))
# 輸出: ('application/pdf', None)
```

---

## 2. 嚴禁單純依賴副檔名進行檔案上傳安全性驗證

> **[核心天條]**：`mimetypes` 僅做字串規則比對，絕對不能作為防禦惡意檔案上傳的安全邊界！

- **錯誤症狀**：
  攻擊者將惡意後門木馬 `shell.php` 或 `virus.exe` 改名為 `avatar.png` 上傳，系統若僅呼叫 `mimetypes.guess_type()`，會誤判定為合法圖片放行。
- **背後原理**：
  `mimetypes` 完全不讀取檔案本體的二進位資料（Magic Numbers / File Signatures），純粹以檔名末段的文字進行比對。

```python
import mimetypes

malicious_file = "payload.exe.png"

# [錯誤寫法]：誤信副檔名即真實檔案類型
mime_type, _ = mimetypes.guess_type(malicious_file)
if mime_type == "image/png":
    # 危險！此處若直接存入伺服器公開目錄將造成遠端程式碼執行風險
    pass

# [正確寫法]：前端快篩用副檔名，後端安全檢驗必須校驗檔案開頭 Magic Bytes
def is_safe_image(file_bytes: bytes) -> bool:
    # PNG 標準開頭簽名: \x89PNG\r\n\x1a\n
    png_magic_signature = b"\x89PNG\r\n\x1a\n"
    return file_bytes.startswith(png_magic_signature)
```

---

## 3. Windows 平台登錄檔污染導致 JavaScript 解析異常

> **[核心天條]**：在提供靜態資源服務前，顯式加固常見關鍵格式的 MIME 映射！

- **錯誤症狀**：
  在 Windows 伺服器上部署 Web 服務時，瀏覽器拒絕執行 JavaScript 檔案，主控台報錯 `Refused to execute script from ... because its MIME type ('text/plain') is not executable`。
- **背後原理**：
  在 Windows 平台上，`mimetypes.init()` 會主動讀取登錄檔 `HKEY_CLASSES_ROOT`。某些本機軟體可能篡改了登錄檔，將 `.js` 錯誤註冊為 `text/plain`。

```python
import mimetypes

# [正確防禦寫法]：在 Web 框架啟動時顯式修正註冊，確保不受本機登錄檔污染影響
mimetypes.add_type("text/javascript", ".js", strict=True)
mimetypes.add_type("text/css", ".css", strict=True)

mime, _ = mimetypes.guess_type("bundle.js")
print(mime)  # 輸出: text/javascript
```

---

## 4. 全域狀態修改引發套件間相互干擾

> **[核心天條]**：模組內部或私有套件自訂型別時，優先使用 `MimeTypes` 實例而非修改全域字典！

- **錯誤症狀**：
  套件 A 執行了 `mimetypes.add_type()`，覆寫了預設副檔名規則，導致同一行程內的套件 B 運作邏輯異常崩潰。
- **背後原理**：
  `mimetypes.add_type()` 直接修改模組層級的共享字典 `mimetypes.types_map`，該變更在整個 Python 直譯器行程中全域可見。

```python
import mimetypes

# [錯誤寫法]：直接改寫全域設定，容易引發隱蔽的副作用與競態條件
# mimetypes.add_type("application/x-custom-bin", ".dat")

# [正確寫法]：建立獨立的 MimeTypes 實例物件，實現邊界隔離
from mimetypes import MimeTypes

app_mime_db = MimeTypes()
app_mime_db.add_type("application/x-custom-bin", ".dat")

print(app_mime_db.guess_type("data.dat"))  # 輸出: ('application/x-custom-bin', None)
```
