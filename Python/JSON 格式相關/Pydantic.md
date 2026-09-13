# 概念與原理

## 什麼是 Pydantic 套件？

Python 生態系中最流行、效能最強大的**資料驗證 (Data Validation) 與結構化解析 (Data Parsing)** 核心套件（`pip install pydantic`）。

Pydantic 完全基於 Python 原生的 **Type Hints（型別標註）**。在 Pydantic V2 版本中，核心引擎採用 **Rust (pydantic-core)** 全面重寫，驗證速度提升了 5 到 50 倍。

它是現代 Web 框架（**FastAPI**）、AI Agent 結構化輸出（**Gemini / OpenAI API Structured Outputs**）以及設定管理（**pydantic-settings**）的標準基石！

> **🍿 生動白話比喻**：  
> - **一般 Python 字典 / 類別**：像一個**「毫無安檢的自由市場」**。丟進來的資料就算型別錯誤、欄位缺失或格式離譜，也都照單全收，導致程式在後續執行時莫名其妙崩潰。  
> - **Pydantic (`BaseModel`)**：像一個**「嚴格的海關安檢閘門 🛂」**。所有外來資料進門前必須經過檢查：  
>   - **合法的**：自動放行並打包成乾淨的 Python 物件。  
>   - **格式稍有偏差的**（例如傳入字串 `"18"` 給 `int`）：**自動幫你扶正轉型**成整數 `18`。  
>   - **夾帶違規資料的**（例如負數年齡、格式錯誤的 Email）：**當場扣押**並開出精確到哪一行哪一欄的違規罰單 (`ValidationError`)！

---

## 序列化與反序列化核心概念 (Serialization & Deserialization)

- **為什麼叫「序列 (Serial)」？**：  
  記憶體裡的 Python 物件是多維立體的複雜網狀結構（包含指針、方法、巢狀關聯）；而硬碟和網路線只能傳輸**「一維線性的字元/位元組流（排排站的序列）」**。  
  - **序列化 (Serialization)**：將記憶體中立體複雜的物件**「打包壓扁成一維線性格式（如 JSON 字串或 Bytes）」**，以便儲存或透過網路傳輸。  
  - **反序列化 (Deserialization)**：將線性的 JSON 字串/Bytes**「100% 無損還原回記憶體中活生生的物件」**。  

```text
【記憶體立體物件】 ─── 序列化 (model_dump / dump_json) ──► 【一維線性資料 (JSON/Bytes)】
       ▲                                                            │
       └────────── 反序列化 (model_validate / validate_json) ───────┘
```

#### 序列化前後樣貌對比

- **1. 原本長什麼樣（記憶體中的立體 Python 物件）**：
```python
Order(
    order_id=8801,
    product="機械鍵盤",
    price=2990.0,
    created_at=datetime(2026, 8, 21, 10, 0, 0)  # 包含 Python 專屬的 datetime 時間物件
)
```

- **2. 序列化後變成什麼樣（拍扁後的一維 JSON 純文字字串）**：
```json
"{\"order_id\": 8801, \"product\": \"機械鍵盤\", \"price\": 2990.0, \"created_at\": \"2026-08-21T10:00:00\"}"
```

> **關鍵轉變**：  
> - 立體複雜的 Python 物件 ➔ 變成**一段扁平的純文字字串**（隨時可存入檔案或透過網路傳輸）。  
> - 特殊的 `datetime` 時間物件 ➔ 自動轉換為國際標準時間字串 `"2026-08-21T10:00:00"`（任何語言如 JS、Java 都能直接解析）。

---

## Pydantic V2 現代語法升級對照表 (V1 vs V2)

| 功能操作 | Pydantic V1 (舊式已棄用) | Pydantic V2 (現代標準 ⭐) | 核心差異與說明 |
| :--- | :--- | :--- | :--- |
| **轉為 Python 字典** | `model.dict()` | **`model.model_dump()`** | 效能更高，支援更多過濾選項 |
| **序列化為 JSON 字串** | `model.json()` | **`model.model_dump_json()`** | 底層 Rust 直接序列化，速度極快 |
| **從字典建立並校驗物件** | `Model.parse_obj(data)` | **`Model.model_validate(data)`** | 命名更語意化 |
| **從 JSON 字串反序列化** | `Model.parse_raw(json_str)` | **`Model.model_validate_json(json_str)`** | 一步完成 JSON 解析與資料校驗 |
| **取得 JSON Schema** | `Model.schema()` | **`Model.model_json_schema()`** | 產出標準 OpenAPI / AI 相容 Schema |
| **自訂欄位驗證器** | `@validator` | **`@field_validator`** | 語法更精準，支援 `mode='before'/'after'` |
| **模型全域設定** | 內部 `class Config:` | **`model_config = ConfigDict(...)`** | 改用型別提示更友好的字典類別 |

---

# 核心 API 類別與方法字典對照表

| 類別 / 函式名稱 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- |
| **[[#BaseModel 基礎資料模型定義\|BaseModel]]** | **所有資料模型的基礎父類別** | 定義 API 請求/回應結構、資料庫傳輸物件 (DTO) |
| **[[#Field() 欄位進階約束\|Field()]]** | **設定欄位約束、預設值、別名與說明** | 限制數值範圍 (`ge=0`)、字串長度、指定 `description` |
| **[[#model.model_dump() 與 model.model_dump_json() 序列化\|model.model_dump()]]** | **將模型物件轉換為原生 Python 字典** | 寫入資料庫、傳遞給其他只吃 dict 的函式庫 |
| **[[#model.model_dump() 與 model.model_dump_json() 序列化\|model.model_dump_json()]]** | **將模型物件直接序列化為 JSON 字串** | 網路傳輸、快取存檔、API 回傳 |
| **[[#BaseModel.model_validate() 與 model_validate_json() 反序列化\|BaseModel.model_validate()]]** | **傳入字典進行驗證並建立模型實例** | 接收 API 請求 Body、讀取資料庫回傳的 dict |
| **[[#BaseModel.model_validate() 與 model_validate_json() 反序列化\|BaseModel.model_validate_json()]]** | **傳入 JSON 字串進行驗證並建立實例** | 接收外部 Webhook、讀取本地 JSON 檔案 |
| **[[#1. JSON Schema 生成雙雄對照 (BaseModel vs TypeAdapter)\|BaseModel.model_json_schema()]]** | **生成標準的 JSON Schema 字典** | 傳給 LLM 進行結構化輸出 (Structured Outputs) |
| **[[#TypeAdapter() 原生型別與頂層容器適配器\|TypeAdapter()]]** | **免定義 BaseModel，直接校驗任意型別/清單** | 驗證頂層清單 `list[User]`、字典或原生型別 `int` |
| **[[#@field_validator() 單一欄位自訂校驗\|@field_validator()]]** | **自訂單一欄位的進階校驗規則** | 檢查密碼強度、校驗自訂格式、過濾非法字元 |
| **[[#@model_validator() 跨欄位全模型聯合校驗\|@model_validator()]]** | **跨欄位聯合校驗或整體驗證** | 比對「密碼」與「確認密碼」是否一致 |
| **[[#ConfigDict 模型全域設定\|ConfigDict]]** | **配置模型全域行為** | 禁止未知額外欄位 (`extra='forbid'`)、自動去除前後空白 |

---

# 核心功能與語法大解密

## 1. 基礎模型定義與智慧型別轉換

##### BaseModel 基礎資料模型定義

- **使用時機**：定義資料結構、欄位型別標註與預設值，讓傳入的資料在建立當下自動完成校驗與轉型。
- **語法**：繼承 `BaseModel`，並在類別內部使用 Type Hints 宣告欄位。
- **核心特性（智慧型別轉換 Coercion）**：
  - 若傳入的型別與標註不符，但**可以安全轉換**（如傳入字串 `"25"` 給 `int`、傳入 `"true"` 給 `bool`），Pydantic 會自動完成轉換！
  - 若無法轉換（如傳入 `"abc"` 給 `int`），會立即拋出 `ValidationError`。

```python
from pydantic import BaseModel, ValidationError

# 1. 定義資料模型
class UserProfile(BaseModel):
    user_id: int
    username: str
    is_vip: bool = False  # 帶有預設值 (選填欄位)
    bio: str | None = None

# 2. 正常建立：自動完成字串轉 int、字串轉 bool
user = UserProfile(
    user_id="1001",    # 自動轉為 int: 1001
    username="Matthew",
    is_vip="true"      # 自動轉為 bool: True
)
print(user.user_id)  # 輸出: 1001 (型別為 int)
print(user.is_vip)   # 輸出: True (型別為 bool)

# 3. 異常防護：傳入非法資料
try:
    bad_user = UserProfile(user_id="不是數字", username="Alice")
except ValidationError as e:
    print(f"❌ 驗證攔截報錯:\n{e}")
```

---

##### model.model_dump() 與 model.model_dump_json() 序列化

- **使用時機**：需要將校驗完成的 Pydantic 物件，轉回 Python 原生 `dict` 或 `JSON 字串` 時使用。
- **常用參數說明**：
  - `mode`：序列化模式（預設 `'python'` 產出原生型別，`'json'` 會將 datetime / UUID 等轉為字串）。
  - `include` / `exclude`：指定**只包含**或**排除**特定欄位名稱集合。
  - `exclude_unset`：布林值。設為 `True` 時，**只會匯出建立時有明確賦值的欄位**（忽略使用預設值的欄位，常用於 HTTP PATCH 局部更新）。
  - `exclude_none`：布林值。設為 `True` 時，過濾掉所有值為 `None` 的欄位。
  - `by_alias`：布林值。設為 `True` 時，使用 `Field(alias="...")` 定義的別名輸出。

```python
from pydantic import BaseModel

class Article(BaseModel):
    title: str
    views: int = 0
    draft: bool = True
    secret_note: str | None = None

art = Article(title="Python Pydantic 指南", views=150)

# 1. 轉為原生字典
print(art.model_dump())
# 輸出: {'title': 'Python Pydantic 指南', 'views': 150, 'draft': True, 'secret_note': None}

# 2. 排除未手動賦值的欄位 (draft 靠預設值，會被排除) 與 None 欄位
patch_data = art.model_dump(exclude_unset=True, exclude_none=True)
print(patch_data)
# 輸出: {'title': 'Python Pydantic 指南', 'views': 150}

# 3. 直接序列化為 JSON 字串
json_str = art.model_dump_json(indent=2)
print(json_str)
```

---

##### BaseModel.model_validate() 與 model_validate_json() 反序列化

- **使用時機**：從外部資料來源（如資料庫傳回的字典、HTTP 接收到的 JSON 原始字串）快速建立並校驗 Pydantic 物件。
- **語法**：
  - `Model.model_validate(dict_data)`
  - `Model.model_validate_json(json_string)`

```python
from pydantic import BaseModel

class Product(BaseModel):
    id: int
    name: str
    price: float

# 1. 從原生字典建立並驗證
raw_dict = {"id": 1, "name": "機械鍵盤", "price": "2990.5"}
p1 = Product.model_validate(raw_dict)
print(f"從字典建立: {p1.name}, 價格: {p1.price}")

# 2. 直接從 JSON 字串建立並驗證 (免先呼叫 json.loads)
raw_json = '{"id": 2, "name": "無線滑鼠", "price": 1200}'
p2 = Product.model_validate_json(raw_json)
print(f"從 JSON 建立: {p2.name}, 價格: {p2.price}")
```

---

##### TypeAdapter() 原生型別與頂層容器適配器

- **使用時機**：當你要驗證或解析的資料是**頂層清單（如 `list[Item]`）、字典（`dict[str, int]`）、原生型別（`int`）或聯合型別（`Union`）**，**不想為此額外定義一個無意義的 `BaseModel` 包裝類別**時使用。
- **語法**：`adapter = TypeAdapter(目標型別標註)`
- **核心方法差異對照表**：

| 方法名稱 | 處理方向 | 接收輸入型別 (Input) | 回傳輸出型別 (Output) | 核心功能與適用情境 |
| :--- | :--- | :--- | :--- | :--- |
| **`adapter.validate_python(data)`** | 📥 **輸入校驗**<br>(Python ➔ 物件) | Python 原生物件<br>(`list`, `dict`, `str`, `int` 等) | 校驗與轉型後的物件<br>(如 `list[Item]`, `list[int]`) | **驗證記憶體中的 Python 物件**。<br>自動將格式相容的資料扶正轉型（例如將 `["10", 20]` 轉為 `[10, 20]`）。 |
| **`adapter.validate_json(json_data)`** | 📥 **輸入校驗**<br>(JSON ➔ 物件) | JSON 字串 / 位元組<br>(`str` 或 `bytes`) | 校驗與轉型後的物件<br>(如 `list[Item]`, `list[int]`) | **免先呼叫 `json.loads()`**。<br>底層 Rust 直接一步到位將 JSON 字串解析並完成驗證，速度極快。 |
| **`adapter.dump_python(obj)`** | 📤 **輸出序列化**<br>(物件 ➔ Python) | 已驗證的 Python 物件 | 原生基礎型別<br>(如 `list[dict]`, `dict`, `int` 等) | **轉為純 Python 原生資料結構**。<br>便於直接傳遞給其他只接受原生 `dict/list` 的第三方庫或資料庫。 |
| **`adapter.dump_json(obj)`** | 📤 **輸出序列化**<br>(物件 ➔ JSON) | 已驗證的 Python 物件 | JSON 位元組串<br>(`bytes`) | **直接序列化為 JSON 二進位資料**。<br>極速產出，適合直接作為 HTTP API Response 回傳或寫入快取。 |
| **`adapter.json_schema()`** | 📋 **結構描述**<br>(無資料轉換) | *(無須傳入資料)* | 標準 JSON Schema 字典<br>(`dict`) | **產出型別結構藍圖**。<br>供大語言模型 (LLM) 進行結構化輸出限制，或產生 OpenAPI / Swagger 介面文件。 |

```python
from pydantic import BaseModel, TypeAdapter

class Item(BaseModel):
    id: int
    name: str

# 1. 定義「頂層清單」適配器：直接驗證 list[Item]
items_adapter = TypeAdapter(list[Item])

# 直接從 JSON 字串驗證頂層陣列 (免宣告外層 ItemList 模型！)
json_data = '[{"id": 1, "name": "蘋果"}, {"id": "2", "name": "香蕉"}]'
items = items_adapter.validate_json(json_data)

print(items)
# 輸出: [Item(id=1, name='蘋果'), Item(id=2, name='香蕉')]

# 2. 驗證原生型別清單 list[int]（自動字串轉整數）
int_list_adapter = TypeAdapter(list[int])
clean_numbers = int_list_adapter.validate_python(["10", 20, "30"])
print(clean_numbers)  # 輸出: [10, 20, 30] (全部轉成 int！)
```

> **💡 深入註解：`adapter.json_schema()` 具體在做什麼？**  
> 它**不處理任何真實資料**，而是把 Python 的型別標註**自動翻譯成標準的「JSON 規格說明書 (JSON Schema)」**：  
> - **範例**：`TypeAdapter(list[int]).json_schema()`  
> - **產出結果**：`{"type": "array", "items": {"type": "integer"}}` ，如果只有單層結構如 `TypeAdapter(int)`，就只會有 `{"type": "integer"}`
> - **核心用途**：丟給 **AI (LLM)** 進行結構化輸出約束（命令 AI 必須回傳整數陣列），或丟給 **FastAPI / Swagger** 自動渲染 API 介面文件。
> - 拿著這個規格說明書給 **`json_schema()`** 就可以將原先如寫 "dict" 轉 "object"
---

## 2. 欄位約束與元數據 (Field)

##### Field() 欄位進階約束

- **使用時機**：需要對欄位設定預設值、取值範圍限制（如大於零、字串長度）、正則表達式、別名對照或欄位說明時使用。
- **語法**：`欄位名稱: 型別 = Field(default=..., gt=..., lt=..., ...)`
- **核心參數字典對照表**：

| 參數名稱                            | 適用型別    | 說明與範例                                                                                                            |
| :------------------------------ | :------ | :--------------------------------------------------------------------------------------------------------------- |
| **`default`**                   | 任意      | 欄位的**靜態預設值**。適合不可變的常數（如 `default=0`、`default="user"`）。若無預設值使用 `...` (Ellipsis)。                     |
| **`default_factory`**           | 可呼叫函式  | 欄位的**動態工廠函式**。傳入函式名稱（如 `list`、`datetime.now`、`uuid4`），每次建立新實例時獨立呼叫生成，杜絕記憶體共享污染與時間鎖死。 |
| **`gt` / `ge`**                 | 數值型別    | 大於 (`>`) / 大於等於 (`>=`)。範例：`gt=0`、`ge=18`。（greater than / greater equal）                                          |
| **`lt` / `le`**                 | 數值型別    | 小於 (`<`) / 小於等於 (`<=`)。範例：`lt=100`、`le=120`。（less than / less equal）                                             |
| **`min_length` / `max_length`** | 字串 / 容器 | 限制字串或清單的最短 / 最長長度。範例：`min_length=6`。                                                                             |
| **`pattern`**                   | 字串      | 正則表達式匹配字串。範例：`pattern=r"^09\d{8}$"` (台灣手機門號)。                                                                    |
| **`alias`**                     | 字串      | 外部傳入與輸出時使用的欄位別名（如對接前端駝峰命名 `userId`）。（alias，別名）                                                                   |
| **`description`**               | 字串      | 欄位說明文件（會被寫入 `model_json_schema()` 供 AI 或 Swagger 閱讀）。                                                            |

###### default vs default_factory 核心差異 (靜態死值 vs 動態工廠)

在 Python 與 Pydantic 中，欄位預設值分為「靜態固定值」與「動態工廠產出」兩種機制：

1. **`default` (靜態死值)**：
   - 在程式啟動、Python 載入類別的瞬間**只計算一次**，並將結果固定在記憶體中。
   - 適用於**不可變的基礎型別常數**（如整數 `0`、字串 `"active"`、布林值 `False`）。

2. **`default_factory` (動態工廠製造機)**：
   - 傳入一個**函式本體（可呼叫物件 Callable，絕不能帶小括號 `()`）**。
   - **每當建立一個全新的模型物件時，Pydantic 就會現場重新呼叫該函式一次**，產出全新且獨立的物件。

> **生動比喻單元**：  
> - **`default`**：像餐廳一早煮好一整鍋湯（同一個記憶體位址）。後續來的每位客人都不拿新碗，直接把湯匙伸進同一鍋湯裡喝；只要一位客人在鍋裡撒胡椒粉，所有客人的湯全部被污染！  
> - **`default_factory`**：像廚房的點餐機（工廠函式）。每來一位新客人，點餐機就現場現煮一碗獨立的新湯端給該客人，彼此完全隔離、互不影響。

###### 兩大經典致命陷阱與程式碼證明

1. **陷阱一：可變容器 (List / Dict) 記憶體共享污染**
   - **錯誤寫法**：`tags: list[str] = Field(default=[])`  
     所有未傳入 `tags` 的物件實例，內部皆指向同一個全域空串列。A 物件修改了清單，B 物件的清單會跟著被竄改！
   - **正確寫法**：`tags: list[str] = Field(default_factory=list)`  
     每次建立新實例時自動執行 `list()`，分配完全獨立的記憶體空間。

2. **陷阱二：動態時間 (datetime) 與唯一識別碼 (UUID) 靜態鎖死**
   - **錯誤寫法**：`created_at: datetime = Field(default=datetime.now())`  
     `datetime.now()` 帶了括號，會在 Python 載入模組的當下一口氣算完並固定。三天後建立的新訂單，時間依然是三天前的伺服器啟動時間！
   - **正確寫法**：`created_at: datetime = Field(default_factory=datetime.now)`  
     傳入 `datetime.now` 函式名稱本身。每次建立新訂單時，現場看錶產出當下時間。

> **語法天條：`default_factory` 傳入值嚴禁帶有小括號**  
> - 錯誤：`default_factory=list()` 或 `default_factory=datetime.now()`（帶括號會立馬執行，退化為靜態死值）  
> - 正確：`default_factory=list` 或 `default_factory=datetime.now`（傳入函式本體）

```python
from datetime import datetime
import uuid
from pydantic import BaseModel, Field

class OrderItem(BaseModel):
    # 1. 靜態預設值：直接給固定常數
    status: str = Field(default="pending")
    
    # 2. 動態工廠預設值：每次建立新訂單，自動生成全新的獨立 UUID 與當下時間
    order_id: str = Field(default_factory=lambda: str(uuid.uuid4()))
    created_at: datetime = Field(default_factory=datetime.now)
    
    # 3. 容器型別動態工廠：每個實例各自擁有獨立的空串列，互不干擾
    tags: list[str] = Field(default_factory=list)

# 測試實例 1
order1 = OrderItem()
order1.tags.append("特急件")

# 測試實例 2
order2 = OrderItem()

print("訂單 1 ID:", order1.order_id)
print("訂單 1 標籤:", order1.tags)  # 輸出: ['特急件']

print("訂單 2 ID:", order2.order_id)  # 生成了全新的 UUID
print("訂單 2 標籤:", order2.tags)  # 輸出: [] (完全不受訂單 1 污染！)
```

---

###### 基礎約束與別名映射範例

```python
from pydantic import BaseModel, Field

class Account(BaseModel):
    # 限制字串長度 3~20 字元，且為必填
    username: str = Field(..., min_length=3, max_length=20, description="帳號名稱")
    
    # 限制年齡在 18 ~ 100 歲之間
    age: int = Field(ge=18, le=100, description="使用者年齡")
    
    # 限制正則：台灣手機號碼格式
    phone: str = Field(pattern=r"^09\d{8}$", description="10碼手機號碼")
    
    # 別名映射：前端傳入 snake_case 或 camelCase 均可透過別名支援
    user_token: str = Field(..., alias="accessToken")

# 建立物件 (傳入別名 accessToken)
acc = Account(
    username="matthew_dev",
    age=25,
    phone="0912345678",
    accessToken="tok_xyz123"
)
print(acc.user_token)  # 輸出: tok_xyz123
```

---

## 3. 自訂校驗器 (@field_validator & @model_validator)

##### @field_validator() 單一欄位自訂校驗

- **使用時機**：當內建的 `Field` 約束無法滿足需求（例如：檢查密碼是否包含特殊符號、檢查 Email 黑名單網域、文字強制小寫）時使用。
- **語法**：`@field_validator('欄位名', mode='after')`
- **參數說明**：
  - `mode='after'` (預設)：在 Pydantic 完成基礎型別轉換**之後**執行驗證（`v` 的型別已被轉正）。
  - `mode='before'`：在 Pydantic 型別轉換**之前**執行（`v` 是原始輸入的原始值）。
- **規範要求**：必須宣告為類別方法（通常第一個參數為 `cls`），驗證通過**必須將最終的值 `return v` 回傳**；若驗證失敗拋出 `ValueError`。

```python
from pydantic import BaseModel, field_validator

class RegisterForm(BaseModel):
    email: str
    nickname: str

    @field_validator("email")
    @classmethod
    def validate_company_email(cls, v: str) -> str:
        v = v.lower().strip()  # 自動清理前後空格與轉小寫
        if not v.endswith("@company.com"):
            raise ValueError("必須使用公司內部專屬信箱 (@company.com)！")
        return v

    @field_validator("nickname")
    @classmethod
    def forbid_admin(cls, v: str) -> str:
        if "admin" in v.lower():
            raise ValueError("暱稱不可包含敏感詞彙 'admin'！")
        return v
```

---

##### @model_validator() 跨欄位全模型聯合校驗

- **使用時機**：需要**同時比對多個欄位之間的關聯邏輯**（例如：「密碼」與「確認密碼」是否一致、「開始日期」是否小於「結束日期」）時使用。
- **語法**：`@model_validator(mode='after')`
- **規範要求**：在 `mode='after'` 下，方法的參數為 `self`，驗證通過**必須回傳 `self`**。

```python
from pydantic import BaseModel, model_validator

class ChangePasswordForm(BaseModel):
    new_password: str
    confirm_password: str

    @model_validator(mode="after")
    def check_passwords_match(self) -> "ChangePasswordForm":
        if self.new_password != self.confirm_password:
            raise ValueError("兩次輸入的密碼不相符，請重新確認！")
        return self
```

---

## 4. 模型設定與巢狀結構 (ConfigDict & Nested Models)

##### ConfigDict 模型全域設定

- **使用時機**：自訂資料模型的底層行為，如：禁止傳入未知欄位、自動修剪所有字串空格、支援按原欄位名取值等。
- **語法**：在類別最上方宣告 `model_config = ConfigDict(...)`。
- **常用配置項**：
  - `extra='forbid'`：**禁止傳入未定義的額外欄位**（若傳入多餘欄位直接報錯，提高安全性）。
  - `str_strip_whitespace=True`：**自動去除所有字串欄位的前後空格**。
  - `populate_by_name=True`：設定了 `alias` 後，同時允許用「欄位本名」或「別名」建立物件。

```python
from pydantic import BaseModel, ConfigDict, Field, ValidationError

class StrictConfigModel(BaseModel):
    model_config = ConfigDict(
        extra="forbid",                # 禁止未知欄位
        str_strip_whitespace=True,     # 自動 strip 空格
        populate_by_name=True          # 允許用本名或別名賦值
    )

    name: str
    user_id: int = Field(alias="userId")

# 1. 自動去除字串前後空格
obj = StrictConfigModel(name="   Matthew   ", userId=101)
print(f"修剪後名稱: '{obj.name}'")  # 輸出: 修剪後名稱: 'Matthew'

# 2. 傳入未知欄位當場被 extra='forbid' 攔截報錯
try:
    StrictConfigModel(name="Alice", userId=102, hacker_key="1234")
except ValidationError as e:
    print("❌ 攔截到未授權的額外欄位！")
```

---

##### 巢狀模型 (Nested Models)

- **使用時機**：處理複雜的層級結構資料（如：一筆訂單包含多筆商品明細、一個用戶包含一個地址物件）。
- **用法**：直接將另一個繼承 `BaseModel` 的類別作為欄位的型別標註，或使用 `list[子模型]`。

```python
from pydantic import BaseModel

class Address(BaseModel):
    city: str
    street: str
    zip_code: str

class Customer(BaseModel):
    name: str
    addresses: list[Address]  # 巢狀子模型列表

# 傳入包含多層字典的資料，Pydantic 自動遞迴解析並驗證每一層！
data = {
    "name": "Matthew",
    "addresses": [
        {"city": "台北市", "street": "信義路五段", "zip_code": "110"},
        {"city": "新北市", "street": "文化路一段", "zip_code": "220"}
    ]
}

customer = Customer.model_validate(data)
print(f"主顧客: {customer.name}")
print(f"第一組地址城市: {customer.addresses[0].city}")  # 輸出: 台北市
```

---

# 實戰：JSON Schema 生成與 AI 結構化輸出

現代大語言模型（如 **Gemini API** 或 **OpenAI API**）與 **FastAPI OpenAPI** 文件，其底層結構化協定完全基於 Pydantic 生成的 JSON Schema。

---

### 1. JSON Schema 生成雙雄對照 (BaseModel vs TypeAdapter)

在 Pydantic V2 中，生成 JSON Schema 根據**目標資料型態**分為兩大流派：

| 產生來源 | 呼叫語法 | 適用情境 | 核心原理與為什麼要這樣設計？ |
| :--- | :--- | :--- | :--- |
| **`BaseModel`** | **`Model.model_json_schema()`** | 自訂單一資料模型類別（如 `User`） | `User` 本身就是 Pydantic 類別，**自帶類別方法**可以直接呼叫。<br>方法帶有 `model_` 前綴，是為了**防止與使用者自訂的欄位名稱衝突**（例如自訂了 `schema` 欄位）。 |
| **`TypeAdapter`** | **`adapter = TypeAdapter(...)`<br>`adapter.json_schema()`** | 頂層清單（如 `list[User]`）、字典或原生型別 | 原生型別（如 `list[User]`、`int \| str`）**不是 Pydantic 類別**，根本沒有任何方法！<br>**必須先用 `TypeAdapter` 套上適配器外殼**，再透過 `adapter.json_schema()` 生成 Schema！ |

> **🔑 記憶口訣**：  
> - **自建模型類別** ➔ 找類別自己：`Model.model_json_schema()`  
> - **清單 / 原生泛型** ➔ 找適配器包裝：`adapter = TypeAdapter(list[Model])` ➔ `adapter.json_schema()`

#### ⚠️ 避坑對比代碼

```python
from pydantic import BaseModel, TypeAdapter

class Item(BaseModel):
    name: str
    price: float

# ==========================================
# 情況 1：直接由 BaseModel 生成（單一物件）
# ==========================================
item_schema = Item.model_json_schema()
# 產出: { "type": "object", "properties": { "name": ..., "price": ... } }

# ==========================================
# 情況 2：頂層是清單 list[Item]
# ==========================================
# ❌ 錯誤寫法：list[Item].model_json_schema() ➔ 拋出 AttributeError 崩潰（原生 list 沒有此方法！）
# ✅ 正確寫法：先用 TypeAdapter 包裝成適配器實例，再呼叫 .json_schema()
adapter = TypeAdapter(list[Item])
list_schema = adapter.json_schema()
# 產出: { "type": "array", "items": { "$ref": "#/$defs/Item" } }
```

---

### 2. Python 型別自動轉換 JSON Schema 規格對照表

呼叫 `json_schema()` 時，Pydantic **內建了完整的型別翻譯引擎**，自動將 Python 型別轉換為標準 JSON Schema 規格，**完全不需要自己手寫字典手動轉換**（例如自動將 `dict` 轉為 `"object"`、`list` 轉為 `"array"`）：

| Python 原生型別標註                        | Pydantic 自動生成的 JSON Schema 規格                              |
| :----------------------------------- | :--------------------------------------------------------- |
| **`dict`** 或自訂 **`BaseModel`**       | ➔ **`"type": "object"`**                                   |
| **`list`** / **`set`** / **`tuple`** | ➔ **`"type": "array"`**                                    |
| **`str`**                            | ➔ **`"type": "string"`**                                   |
| **`int`**                            | ➔ **`"type": "integer"`**                                  |
| **`float`**                          | ➔ **`"type": "number"`**                                   |
| **`bool`**                           | ➔ **`"type": "boolean"`**                                  |
| **`None`**                           | ➔ **`"type": "null"`**                                     |
| **`Union[int, str]`**                | ➔ **`"anyOf": [{"type": "integer"}, {"type": "string"}]`** |
| **`Literal["A", "B"]`**              | ➔ **`"enum": ["A", "B"]`**                                 |
要使用內部型別要在 **`"type"`** 拿出


---

### 3. Schema 生成常用參數

`model_json_schema()` 與 `adapter.json_schema()` 支援以下常用參數：

- **`by_alias=True`**：布林值。設定為 `True` 時，Schema 的欄位名稱會使用 `Field(alias="...")` 定義的別名。
- **`mode='validation'` (預設)**：產出用於**驗證輸入資料**的 Schema。
- **`mode='serialization'`**：產出用於**序列化輸出資料**的 Schema。

---

### 3. 在 Field 中自訂豐富的 Schema 屬性

除了 `description`，`Field` 還支援多種直接寫入 JSON Schema 的元數據屬性：

- **`description`**：欄位功能描述（AI 理解參數意義的關鍵）。
- **`examples`**：提供範例資料列表（例如 `examples=["https://example.com"]`）。
- **`json_schema_extra`**：自訂字典，直接注入額外的自訂 Schema 欄位（例如 `{"deprecated": True}`）。

```python
from pydantic import BaseModel, Field, TypeAdapter

# 1. BaseModel 生成 Schema (豐富元數據)
class BookReview(BaseModel):
    book_title: str = Field(
        description="書籍名稱",
        examples=["原子習慣", "被討厭的勇氣"]
    )
    score: int = Field(ge=1, le=5, description="評分 1 到 5 顆星")
    summary: str = Field(description="50字以內的一句話評語")
    tags: list[str] = Field(default=[], description="標籤清單")

print("--- [1] BaseModel Schema ---")
print(BookReview.model_json_schema())

# 2. TypeAdapter 生成頂層清單的 Schema
list_adapter = TypeAdapter(list[BookReview])

print("--- [2] TypeAdapter List Schema ---")
print(list_adapter.json_schema())
```

> 💡 **進階深入**：若想深入了解產出的 Schema 結構（`properties`, `required`, `enum`, `pattern`）或手動使用 `jsonschema` 驗證，請參閱專題筆記：[[JSON Schema]]。

---

### 4. 搭配 Gemini API 結構化輸出調用示範

```python
from google import genai
from google.genai import types

client = genai.Client()

# Gemini SDK 會自動在底層呼叫 BookReview.model_json_schema() 進行結構化約束
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="請為《原子習慣》寫一份評分與評語",
    config=types.GenerateContentConfig(
        response_mime_type="application/json",
        response_schema=BookReview  # 直接傳入 Pydantic 類別！
    )
)
print(response.text)
```

---

# 實戰除錯與核心天條

## 1. 忘記 Pydantic V2 方法名稱變更 (AttributeError)

> **⚠️ V1 到 V2 語法廢棄陷阱**：  
> 在 Pydantic V2 中呼叫舊版的 `model.dict()` 或 `Model.parse_obj()` 會觸發 Deprecation 警告，甚至在未來被徹底移除！  
> **牢記口訣**：V2 全面採用 `model_` 前綴：  
> - `.dict()` ➔ **`.model_dump()`**  
> - `.json()` ➔ **`.model_dump_json()`**  
> - `.parse_obj()` ➔ **`.model_validate()`**  
> - `.schema()` ➔ **`.model_json_schema()`**

---

## 2. 避免使用原生可變物件預設值陷阱 (Pydantic 自動深拷貝)

> **💡 Pydantic 自動深拷貝保護**：  
> 在原生 Python 函式中寫 `def f(items=[])` 會導致所有呼叫共享同一個列表；但在 Pydantic 欄位中寫 `items: list[str] = []` 是**完全安全的**！Pydantic 會在內部自動為每個實例建立獨立的深拷貝副本，無需強制使用 `Field(default_factory=list)`（但寫 `default_factory` 是更加嚴謹的好習慣）。
