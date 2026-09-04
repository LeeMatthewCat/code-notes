# 概念與原理

## 什麼是 json 模組？

Python 內建標準庫中專為處理 JSON 資料交換格式設計的**核心序列化與資料轉換模組 (JSON Serialization Module)**。

JSON (JavaScript Object Notation) 是一種輕量級、易於人類閱讀與機器解析的跨語言開放標準文字格式。
在現代軟體開發、前後端 API 通訊、設定檔讀寫（如 `config.json`）及資料庫資料交換中，JSON 是事實上的世界通用標準。

Python 內建的 `json` 模組，專門負責在 **Python 資料結構（如 `dict`、`list`、`int`、`str`）** 與 **JSON 格式文字字串/檔案** 之間進行無損雙向轉換。

> **🍿 生動白話比喻**：  
> - **序列化（Dump / 打包）**：像是把立體的**「樂高城堡模型（Python 記憶體物件）」**拆解並壓平成**「說明書平面印刷文字（JSON 字串）」**，方便裝進信封郵寄給其他人；  
> - **反序列化（Load / 還原）**：像是收到信封後，根據說明書上的文字，重新在記憶體中拼裝出立體的**「樂高城堡（Python dict/list）」**！

---

## 序列化與反序列化核心概念

- **序列化 (Serialization / Encoding)**：
  - 將 Python 記憶體中的物件（如字典、列表）轉換為 JSON 格式的文字字串或檔案串流。
  - 對應函數：`json.dumps()`（轉成字串）、`json.dump()`（寫入檔案）。
- **反序列化 (Deserialization / Decoding)**：
  - 將 JSON 格式的文字字串或檔案串流解析還原為 Python 原生資料結構。
  - 對應函數：`json.loads()`（從字串解析）、`json.load()`（從檔案解析）。

> **💡 快速記憶法**：  
> - 帶有 **`s`**（如 `dumps`, `loads`）代表操作對象是 **`s`tring（字串）**；  
> - 沒有 **`s`**（如 `dump`, `load`）代表操作對象是 **File-like Object（檔案物件/串流）**。

---

## Python 與 JSON 資料型別對照表

Python 與 JSON 之間並非所有型別都 100% 一對一完全相同。兩者在轉換時有一套嚴格的預設映射規則：

### Python 轉 JSON (編碼 Encoding) 對照

| Python 資料型別 | JSON 資料型別 | 轉換說明與範例 |
| :--- | :--- | :--- |
| **`dict`** | `object` | 轉換為 JSON 物件 `{ "key": "value" }`（鍵會強制轉為字串） |
| **`list`, `tuple`** | `array` | 轉換為 JSON 陣列 `[1, 2, 3]`（元組會變成中括號陣列） |
| **`str`** | `string` | 轉換為雙引號字串 `"hello"` |
| **`int`, `float`** | `number` | 轉換為數字 `42`, `3.14` |
| **`True` / `False`** | `true` / `false` | 首字母大寫轉為 JSON 全小寫布林值 |
| **`None`** | `null` | Python 空值轉為 JSON 的 `null` |

### JSON 轉 Python (解碼 Decoding) 對照

| JSON 資料型別 | Python 資料型別 | 轉換說明 |
| :--- | :--- | :--- |
| **`object`** | `dict` | 還原為 Python 字典 |
| **`array`** | `list` | 還原為 Python 串列（**原本是 tuple 轉出去再轉回來也會變成 list！**） |
| **`string`** | `str` | 還原為 Python 字串 |
| **`number (int)`** | `int` | 還原為整數 |
| **`number (real)`** | `float` | 還原為浮點數 |
| **`true` / `false`** | `True` / `False` | 還原為 Python 首字母大寫布林值 |
| **`null`** | `None` | 還原為 Python `None` |

> **⚠️ 元組 (Tuple) 轉換陷阱**：  
> Python 的 `tuple` 序列化為 JSON 後會變成 `array`；當再次反序列化回 Python 時，**會變成 `list` 而不是 `tuple`**！如果程式碼邏輯嚴格依賴不可變的 `tuple`，需手動透過 `tuple()` 重新轉換。

---

# 四大核心 API 函數字典對照表

| 函數名稱 | 操作對象 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- | :--- |
| **[[#json.dumps() 物件序列化為 JSON 字串\|json.dumps()]]** | **字串 (String)** | 將 Python 物件序列化為 **JSON 格式字串** | 呼叫 HTTP API 傳送 JSON Payload、日誌記錄、Redis 快取儲存 |
| **[[#json.dump() 物件直接寫入 JSON 檔案\|json.dump()]]** | **檔案 (File)** | 將 Python 物件序列化並直接寫入 **JSON 檔案** | 儲存設定檔、匯出資料報表至磁碟 |
| **[[#json.loads() JSON 字串解析為 Python 物件\|json.loads()]]** | **字串 (String)** | 將 **JSON 格式字串** 解析為 Python 物件 | 接收 Webhook 請求、解析 HTTP 回應內容（`response.text`） |
| **[[#json.load() 從 JSON 檔案直接讀取物件\|json.load()]]** | **檔案 (File)** | 從 **JSON 檔案** 讀取內容並解析為 Python 物件 | 程式啟動時載入本機 `config.json` 設定檔 |

---

# 核心功能與語法大解密

## 1. 記憶體字串操作 (In-Memory String Operations)

### json.dumps() 物件序列化為 JSON 字串

- **使用時機**：當你需要將 Python 字典或串列轉換為 JSON 格式字串，用來發送網路請求（如 HTTP POST）、寫入快取或印出除錯時使用。
- **語法**：`json.dumps(obj, ensure_ascii=True, indent=None, separators=None, sort_keys=False, default=None, ...)`
- **參數說明**：
  - `obj`：要被轉換的 Python 物件（如 `dict`、`list`）。
  - `ensure_ascii`：布林值，預設為 `True`。**處理中文時強烈建議設為 `False`**，否則中文會被轉成 `\u4e2d\u6587` 跳脫字元。
  - `indent`：縮排空格數（如 `2` 或 `4`），設定後會自動排版換行，增加人類可讀性。
  - `separators`：元組 `(item_separator, key_separator)`，可用於極致壓縮去除多餘空格（如 `(",", ":")`）。
  - `sort_keys`：布林值，設為 `True` 時會將字典鍵依照字母順序排序輸出。
  - `default`：傳入自訂函式，用來處理預設無法序列化的型別（如 `datetime`、`UUID`）。
- **回傳值**：
  - `str`：轉換後的 JSON 格式字串。

```python
import json

# 1. 基礎序列化
user = {"name": "小明", "age": 25, "is_admin": False, "skills": ["Python", "SQL"]}
json_str = json.dumps(user)
print(json_str)
# 輸出: {"name": "\u5c0f\u660e", "age": 25, "is_admin": false, "skills": ["Python", "SQL"]}

# 2. 解決中文編碼問題 (ensure_ascii=False)
json_str_cn = json.dumps(user, ensure_ascii=False)
print(json_str_cn)
# 輸出: {"name": "小明", "age": 25, "is_admin": false, "skills": ["Python", "SQL"]}

# 3. 美化輸出與排序鍵值 (indent=4, sort_keys=True)
pretty_json = json.dumps(user, ensure_ascii=False, indent=4, sort_keys=True)
print(pretty_json)
# 輸出:
# {
#     "age": 25,
#     "is_admin": false,
#     "name": "小明",
#     "skills": [
#         "Python",
#         "SQL"
#     ]
# }

# 4. 極致壓縮（去除所有多餘空格，用於節省網路傳輸流量）
compact_json = json.dumps(user, ensure_ascii=False, separators=(",", ":"))
print(compact_json)
# 輸出: {"name":"小明","age":25,"is_admin":false,"skills":["Python","SQL"]}
```

> **💡 中文輸出鐵律 `ensure_ascii=False`**：  
> Python 預設 `ensure_ascii=True` 會將所有非 ASCII 字元（如繁簡中文、日文、Emoji）全部轉義為 `\uXXXX` 形式。在繁體中文環境下，**請務必養成隨手加上 `ensure_ascii=False` 的好習慣**！

---

### json.loads() JSON 字串解析為 Python 物件

- **使用時機**：當你從第三方 API 獲取到 JSON 格式的回應字串（如 `response.text`），需要將其解碼為 Python 字典或串列進行資料提取時使用。
- **語法**：`json.loads(s, object_hook=None, parse_float=None, parse_int=None, ...)`
- **參數說明**：
  - `s`：包含有效 JSON 資料的文字字串（`str`）或二進位位元組（`bytes`/`bytearray`）。
  - `object_hook`：可選函式，在每個 JSON 物件（字典）被解析時攔截並進行自訂轉換（如轉為自訂 Class 物件）。
  - `parse_float`：可選函式，指定解析浮點數的型別（例如傳入 `decimal.Decimal` 以避免浮點數精度丟失）。
- **回傳值**：
  - `dict`、`list`、`str`、`int`、`float`、`bool` 或 `None`（對應解析出的 Python 物件）。

```python
import json

raw_json = '{"name": "小明", "age": 25, "is_admin": false, "profile": null}'

# 1. 將 JSON 字串解析為 Python 字典
data = json.loads(raw_json)

print(type(data))       # 輸出: <class 'dict'>
print(data["name"])     # 輸出: 小明
print(data["is_admin"]) # 輸出: False (JSON false 自動轉為 Python 布林值 False)
print(data["profile"])  # 輸出: None (JSON null 自動轉為 Python None)
```

> **⚠️ JSON 格式三大語法雷區**：  
> 1. **引號規定**：JSON 標準嚴格要求鍵名與字串值**必須使用雙引號 `"`**，使用 Python 單引號 `'` 會直接引發 `json.decoder.JSONDecodeError`！  
> 2. **布林值大小寫**：JSON 是小寫的 `true` / `false` / `null`，寫成大寫 `True` 會報錯。  
> 3. **尾隨逗號**：JSON 陣列或物件的最後一項**絕對不能有尾隨逗號（Trailing Comma）**，例如 `[1, 2, 3,]` 是非法格式！

---

## 2. 檔案串流讀寫 (File Streaming Operations)

### json.dump() 物件直接寫入 JSON 檔案

- **使用時機**：需要將記憶體中的大型資料結構、應用程式設定或抓取的資料，直接寫入硬碟上的 `.json` 檔案時使用。
- **語法**：`json.dump(obj, fp, ensure_ascii=True, indent=None, ...)`
- **參數說明**：
  - `obj`：要寫入的 Python 資料物件。
  - `fp`：已開啟且具備寫入權限的檔案物件（File-like Object，需支援 `.write()` 方法）。
  - `ensure_ascii`：設為 `False` 確保中文正常寫入。
  - `indent`：設定縮排使生成的檔案排版工整。
- **回傳值**：
  - `None`。

```python
import json
from pathlib import Path

data = {
    "app_name": "Antigravity",
    "version": "2.0.0",
    "settings": {
        "theme": "dark",
        "auto_save": True
    }
}

file_path = Path("config.json")

# 結合 context manager (with open) 寫入檔案，務必指定 encoding="utf-8"
with open(file_path, "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=4)

print("✅ 設定檔寫入完成！")
```

> **💡 開檔必加 `encoding="utf-8"`**：  
> 在 Windows 系統下，Python 開檔預設可能使用 `cp950` 或 `gbk` 編碼。寫入或讀取 JSON 檔案時，**務必明確指定 `encoding="utf-8"`**，徹底杜絕跨平台編碼亂碼問題！

---

### json.load() 從 JSON 檔案直接讀取物件

- **使用時機**：應用程式啟動時載入本機設定檔、讀取既有的 JSON 資料庫或讀取快取檔案時使用。
- **語法**：`json.load(fp, object_hook=None, ...)`
- **參數說明**：
  - `fp`：已開啟且具備讀取權限的檔案物件（需支援 `.read()` 方法）。
- **回傳值**：
  - 解析還原後的 Python 資料物件（通常為 `dict` 或 `list`）。

```python
import json
from pathlib import Path

file_path = Path("config.json")

# 從檔案直接讀取並解析
with open(file_path, "r", encoding="utf-8") as f:
    config = json.load(f)

print(config["app_name"])               # 輸出: Antigravity
print(config["settings"]["auto_save"])  # 輸出: True
```

---

# 進階技術與高級自訂處理

## 1. 自訂非標準型別序列化 (Handling Non-Serializable Types)

Python 預設的 `json` 模組無法直接序列化 `datetime`、`Decimal`、`UUID`、`set` 或自訂 Class 實例，直接執行會拋出 `TypeError: Object of type ... is not JSON serializable`。

### 方法 A：使用 `default` 參數（推薦輕量方案）

傳入一個自訂轉換函式，當遇到不認識的型別時由該函式負責轉換：

```python
import json
from datetime import datetime, date
from decimal import Decimal
from uuid import UUID, uuid4

def custom_serializer(obj):
    # 1. 處理日期與時間
    if isinstance(obj, (datetime, date)):
        return obj.isoformat()
    # 2. 處理高精度十進位數
    if isinstance(obj, Decimal):
        return float(obj)
    # 3. 處理 UUID
    if isinstance(obj, UUID):
        return str(obj)
    # 4. 處理集合 (set)
    if isinstance(obj, set):
        return list(obj)
    # 其他不支援的型別拋出原本的 TypeError
    raise TypeError(f"無法序列化型別: {type(obj)}")

# 測試包含多種非標準型別的字典
complex_data = {
    "order_id": uuid4(),
    "amount": Decimal("199.99"),
    "created_at": datetime.now(),
    "tags": {"vip", "paid"}
}

# 序列化時傳入 default 參數
json_output = json.dumps(complex_data, default=custom_serializer, ensure_ascii=False, indent=2)
print(json_output)
```

---

### 方法 B：繼承 `json.JSONEncoder`（物件導向架構方案）

透過自訂類別覆寫 `.default()` 方法：

```python
import json
from datetime import datetime

class CustomJSONEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.strftime("%Y-%m-%d %H:%M:%S")
        if hasattr(obj, "__dict__"):
            return obj.__dict__  # 自動將自訂 Class 物件轉為字典
        return super().default(obj)

class User:
    def __init__(self, name: str, email: str):
        self.name = name
        self.email = email

user_obj = User("Matthew", "matthew@example.com")

# 透過 cls 參數指定自訂 Encoder
json_data = json.dumps(user_obj, cls=CustomJSONEncoder, indent=2)
print(json_data)
# 輸出:
# {
#   "name": "Matthew",
#   "email": "matthew@example.com"
# }
```

---

## 2. 自訂反序列化 hook (Object Hook)

使用 `json.loads(..., object_hook=...)` 在解析時自動將 JSON 字典轉換為 Python 自訂物件（如 `dataclass`）：

```python
import json
from dataclasses import dataclass

@dataclass
class Product:
    id: int
    name: str
    price: float

def product_decoder(dct: dict):
    # 若字典中具備 Product 必要的欄位，自動實例化為 Product 物件
    if "id" in dct and "name" in dct and "price" in dct:
        return Product(id=dct["id"], name=dct["name"], price=dct["price"])
    return dct

json_str = '{"id": 101, "name": "機械鍵盤", "price": 2990.0}'

# 反序列化時傳入 object_hook
product = json.loads(json_str, object_hook=product_decoder)

print(type(product))       # 輸出: <class '__main__.Product'>
print(product.name)        # 輸出: 機械鍵盤
print(f"價格: {product.price}") # 輸出: 價格: 2990.0
```

---

# 實戰除錯與常見異常處理

## 1. 捕獲 JSONDecodeError 語法解析錯誤

當傳入的字串不符合 JSON 語法規範時，會拋出 `json.decoder.JSONDecodeError`。我們可以捕獲該例外並精確取得出錯的行號與字元位置：

```python
import json

# 故意提供一個帶有錯誤語法的 JSON (使用單引號且帶有尾隨逗號)
malformed_json = "{'name': 'Matthew', 'age': 20,}"

try:
    data = json.loads(malformed_json)
except json.JSONDecodeError as e:
    print(f"❌ JSON 解析失敗！")
    print(f"錯誤原因 (Message): {e.msg}")
    print(f"出錯行號 (Line): {e.lineno}")
    print(f"出錯欄位 (Col): {e.colno}")
    print(f"出錯字元索引 (Pos): {e.pos}")
```

---

## 2. 現代優雅寫法：搭配 Pathlib 快速讀寫

結合 Python 現代路徑庫 `pathlib.Path`，可以大幅精簡程式碼：

```python
import json
from pathlib import Path

data_path = Path("user_cache.json")

# 1. 快速寫入 JSON (Pathlib + json.dumps)
user_data = {"id": 1, "active": True}
data_path.write_text(json.dumps(user_data, ensure_ascii=False, indent=2), encoding="utf-8")

# 2. 快速讀取 JSON (Pathlib + json.loads)
if data_path.exists():
    cached_data = json.loads(data_path.read_text(encoding="utf-8"))
    print("讀取成功:", cached_data)
```

---

## 3. 進階延伸：JSON 格式約束與驗證 (JSON Schema)

當你需要對 JSON 資料進行**嚴格的結構規格定義、欄位型別約束、長度與數值防禦**，或要為 **AI Agent (Gemini / OpenAI / Claude / MCP)** 提供 Tool Calling 規格時，請參考專題筆記：
- 📘 詳見專題筆記：[[JSON Schema]]
- 🛡️ 搭配資料驗證庫：[[Pydantic]]

