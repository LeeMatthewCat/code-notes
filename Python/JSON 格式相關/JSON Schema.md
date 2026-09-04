# 概念與原理

## 什麼是 JSON Schema？

**JSON Schema** 是一種基於 JSON 格式的**宣告式資料結構與驗證標準規範 (Declarative Data Definition Standard)**。

如果說 JSON 是一份「已經填寫完畢的資料表單」，那麼 JSON Schema 就是這份表單的**「官方填寫規範與防呆規則說明書」**！

它用來精確約束與定義：
1. 資料必須包含哪些**必要欄位 (Required Fields)**。
2. 每個欄位的**資料型別 (Data Types)**（如字串、整數、布林值、陣列、巢狀物件）。
3. 數值的**邊界範圍**（如最小值 `minimum`、最大值 `maximum`）。
4. 字串的**長度與格式規則**（如 `pattern` 正規表達式、Email 格式）。
5. 陣列的**元素型別與數量限制**。

> **🍿 生動白話比喻**：  
> - **JSON 資料**：像旅客手裡填好的**「入境申報單」**。  
> - **JSON Schema**：像海關印發的**「申報單規格指南」**（標註：姓名必填、護照號碼必須為 9 碼英數字、年齡必須介於 0 到 120 歲）。  
> - **JSON Schema 驗證器 (Validator)**：像**「自動安檢掃描閘門」**，只要申報單有任何一欄不符規定，當場攔截並精準指出哪一行填錯！

---

## 為什麼 JSON Schema 成為現代 AI 與 API 的世界標準？

JSON Schema 已經統治了現代軟體開發與 AI Agent 架構：

```text
┌──────────────────────────────────────────────────────────────┐
│                     JSON Schema 核心應用領域                  │
├──────────────────────────────┬───────────────────────────────┤
│ 1. 現代 Web API 規範 (OpenAPI)│ 2. 大語言模型 Tool Calling     │
│    FastAPI / Swagger / gRPC  │    Gemini / OpenAI / Claude   │
│    定義前後端溝通資料規格       │    向 LLM 描述外部函式參數規格  │
├──────────────────────────────┼───────────────────────────────┤
│ 3. LLM 結構化輸出 (Structured)│ 4. 設定檔驗證 (Config Lint)   │
│    強制模型輸出 100% 合法 JSON │    VS Code / CI/CD Pipeline   │
└──────────────────────────────┴───────────────────────────────┘
```

1. **AI Agent Tool Use (Function Calling) 的通用語言**：  
   當我們讓 LLM 呼叫 Python 函式時，LLM **根本看不懂 Python 代碼**！所有 AI 廠商（OpenAI、Google Gemini、Anthropic Claude、MCP 協議）統一要求將 Python 函式參數轉譯為 **JSON Schema** 傳給模型！
2. **LLM 結構化輸出保證 (Structured Outputs)**：  
   傳入 JSON Schema，模型底層會透過約束採樣 (Constrained Sampling)，保證輸出的 JSON 100% 符合定義，絕不缺欄位或型別出錯。
3. **OpenAPI / Swagger 核心規範**：  
   FastAPI 等現代框架自動產生的 API 文檔，底層就是由 JSON Schema 構成。

---

# 核心關鍵字與語法字典對照表

JSON Schema 採用標準化的鍵值 (Key-Value) 語法進行約束：

| 核心關鍵字 | 適用型別 | 核心功能與意義 | 語法範例 |
| :--- | :--- | :--- | :--- |
| **[[#1. type 資料型別宣告\|type]]** | 全型別 | **指定欄位的資料型別**（`string`, `number`, `integer`, `boolean`, `array`, `object`, `null`） | `"type": "string"` |
| **[[#2. properties 與 required 物件約束\|properties]]** | `object` | **定義物件內部所有子欄位的結構規格** | `"properties": { "name": {...} }` |
| **[[#2. properties 與 required 物件約束\|required]]** | `object` | **指定哪些欄位為必填項目 (不可省略)** | `"required": ["name", "age"]` |
| **[[#3. 數值範圍約束 (minimum / maximum)\|minimum / maximum]]** | `number` / `integer` | **設定數值的最小值與最大值 (包含邊界)** | `"minimum": 0, "maximum": 120` |
| **[[#4. 字串長度與正則 (minLength / pattern)\|minLength / maxLength]]** | `string` | **限制字串的最少與最多字元長度** | `"minLength": 8, "maxLength": 30` |
| **[[#4. 字串長度與正則 (minLength / pattern)\|pattern]]** | `string` | **使用正規表達式 (Regex) 進行字串格式校驗** | `"pattern": "^[a-zA-Z0-9_]+$"` |
| **[[#5. enum 枚舉候選清單\|enum]]** | 任意型別 | **限定欄位只能填入特定清單中的字面值** | `"enum": ["pending", "done"]` |
| **[[#6. 陣列元素與長度約束 (items / minItems)\|items]]** | `array` | **定義陣列內部所有元素的型別與規格** | `"items": { "type": "string" }` |
| **[[#6. 陣列元素與長度約束 (items / minItems)\|minItems / maxItems]]** | `array` | **限制陣列中元素的最小與最大數量** | `"minItems": 1, "maxItems": 10` |
| **[[#7. additionalProperties 多餘欄位控管\|additionalProperties]]** | `object` | **是否允許傳入未在 properties 定義的雜訊欄位** | `"additionalProperties": false` |
| **[[#8. description 欄位語意說明 (LLM 核心)\|description]]** | 全型別 | **為欄位撰寫人類與 AI 模型的白話解釋** | `"description": "使用者的電子郵件"` |

---

# 核心語法與結構大解密

### 範例：一份標準的 JSON Schema 結構

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "UserProfile",
  "description": "使用者個人基本資料表",
  "type": "object",
  "properties": {
    "username": {
      "type": "string",
      "description": "帳號名稱，僅限英數字與底線",
      "minLength": 3,
      "maxLength": 20,
      "pattern": "^[a-zA-Z0-9_]+$"
    },
    "age": {
      "type": "integer",
      "description": "使用者年齡",
      "minimum": 0,
      "maximum": 150
    },
    "role": {
      "type": "string",
      "description": "使用者系統權限角色",
      "enum": ["admin", "editor", "viewer"]
    },
    "tags": {
      "type": "array",
      "description": "使用者興趣標籤清單",
      "items": {
        "type": "string"
      },
      "minItems": 1
    }
  },
  "required": ["username", "role"],
  "additionalProperties": false
}
```

---

##### 1. type 資料型別宣告

- **使用時機**：定義目標資料的基本資料型別。
- **支援型別清單**：
  - `"string"`：字串文字。
  - `"integer"`：純整數（不可包含小數點，如 `42`）。
  - `"number"`：任何數值（包含浮點數與整數，如 `3.14`, `100`）。
  - `"boolean"`：布林值（`true` 或 `false`）。
  - `"object"`：鍵值字典物件（`{...}`）。
  - `"array"`：列表陣列（`[...]`）。
  - `"null"`：空值。

> **⚠️ `integer` vs `number` 陷阱**：  
> 若宣告 `"type": "integer"`，傳入 `18.5` 會驗證失敗！若欄位允許小數（如價格、經緯度），請務必使用 `"type": "number"`！

---

##### 2. properties 與 required 物件約束

- **`properties`**：字典物件，Key 為欄位名稱，Value 為該欄位的子 Schema 定義。
- **`required`**：字串陣列清單，列出**絕對不能為空或省略的必要欄位**。

```json
{
  "type": "object",
  "properties": {
    "id": { "type": "integer" },
    "email": { "type": "string" }
  },
  "required": ["id", "email"]
}
```

> **⚠️ 必填鐵律**：未寫入 `required` 陣列中的欄位，預設都是「可選 (Optional)」的！

---

##### 3. 數值範圍約束 (minimum / maximum)

- **`minimum`**：數值下限（`>=` 包含邊界）。
- **`maximum`**：數值上限（`<=` 包含邊界）。
- **`exclusiveMinimum` / `exclusiveMaximum`**：嚴格大於 `>` 或小於 `<`（不包含邊界）。

```json
{
  "type": "number",
  "description": "評分分數 (1.0 到 5.0 之間)",
  "minimum": 1.0,
  "maximum": 5.0
}
```

---

##### 4. 字串長度與正則 (minLength / pattern)

- **`minLength` / `maxLength`**：字元長度限制。
- **`pattern`**：傳入正規表達式（無需前後斜線 `/`），強制字串格式。

```json
{
  "type": "string",
  "description": "台灣手機號碼",
  "pattern": "^09\\d{8}$"
}
```

---

##### 5. enum 枚舉候選清單

- **使用時機**：限制欄位值只能在給定的白名單中選一個。

```json
{
  "type": "string",
  "description": "訂單處理狀態",
  "enum": ["pending", "paid", "shipped", "cancelled"]
}
```

---

##### 6. 陣列元素與長度約束 (items / minItems)

- **`items`**：宣告陣列內「每一個元素」必須符合的 Schema。
- **`minItems` / `maxItems`**：限制陣列中元素的個數。
- **`uniqueItems`**：若設為 `true`，要求陣列內所有元素不得重複。

```json
{
  "type": "array",
  "description": "商品 ID 列表",
  "items": {
    "type": "integer"
  },
  "minItems": 1,
  "uniqueItems": true
}
```

---

##### 7. additionalProperties 多餘欄位控管

- **`additionalProperties: false`**：**嚴格封閉模式**。若傳入 JSON 中帶有未在 `properties` 定義的未知欄位，直接判定違規拋錯！
- **`additionalProperties: true`**（預設）：允許夾帶其他多餘欄位。

---

##### 8. description 欄位語意說明 (LLM 核心)

- **AI Agent 生命線**：在呼叫大語言模型（如 Gemini、Claude、OpenAI）時，模型全靠 `description` 理解參數意圖。
- **鐵律**：永遠為欄位加上詳實清晰的 `description`，寫明單位、格式與業務意義！

---

# Python 生態實戰：如何操作與驗證 JSON Schema

在 Python 中，處理 JSON Schema 主要有兩大途徑：
1. **使用 `jsonschema` 官方標準庫進行手動驗證**。
2. **使用 `Pydantic V2` 自動產生 JSON Schema (推薦主流 ⭐)**。

---

### 途徑 1：使用 `jsonschema` 套件進行執行期驗證

安裝官方驗證器：

```shell
pip install jsonschema
```

#### 實戰代碼：驗證資料合法性與捕獲錯誤

```python
from jsonschema import validate, ValidationError

# 1. 定義驗證規格說明書 (JSON Schema)
user_schema = {
    "type": "object",
    "properties": {
        "name": {"type": "string", "minLength": 2},
        "age": {"type": "integer", "minimum": 0},
        "email": {"type": "string"}
    },
    "required": ["name", "age"],
    "additionalProperties": False
}

# 2. 測試合法資料
valid_data = {"name": "Matthew", "age": 25}
try:
    validate(instance=valid_data, schema=user_schema)
    print("✅ 合法資料驗證通過！")
except ValidationError as e:
    print(f"❌ 驗證失敗: {e.message}")

# 3. 測試違規資料 (年齡為負數，且夾帶多餘欄位)
invalid_data = {"name": "M", "age": -5, "hacker_field": True}
try:
    validate(instance=invalid_data, schema=user_schema)
except ValidationError as e:
    print(f"❌ 攔截到違規資料！")
    print(f"👉 違規路徑: {list(e.path)}")
    print(f"👉 違規原因: {e.message}")
```

---

### 途徑 2：使用 Pydantic V2 自動導出 JSON Schema (AI Agent 必備)

在現代開發中，我們極少純手寫龐大的 JSON Schema 字典，而是**定義 Pydantic 模型，再調用 `.model_json_schema()` 自動生成**：

```python
from pydantic import BaseModel, Field
from typing import Literal

# 1. 定義 Pydantic 模型
class WeatherToolInput(BaseModel):
    location: str = Field(
        description="要查詢天氣的城市名稱，例如: 台北市、東京都"
    )
    unit: Literal["celsius", "fahrenheit"] = Field(
        default="celsius",
        description="溫度計量單位"
    )
    days: int = Field(
        default=1,
        ge=1,
        le=7,
        description="預報天數 (1 到 7 天)"
    )

# 2. 一鍵導出為標準 JSON Schema
schema = WeatherToolInput.model_json_schema()

import json
print(json.dumps(schema, indent=2, ensure_ascii=False))
```

#### 輸出結果（直接可用於 Gemini / OpenAI / MCP Tool 參數定義）：

```json
{
  "title": "WeatherToolInput",
  "type": "object",
  "properties": {
    "location": {
      "description": "要查詢天氣的城市名稱，例如: 台北市、東京都",
      "title": "Location",
      "type": "string"
    },
    "unit": {
      "default": "celsius",
      "description": "溫度計量單位",
      "enum": ["celsius", "fahrenheit"],
      "title": "Unit",
      "type": "string"
    },
    "days": {
      "default": 1,
      "description": "預報天數 (1 到 7 天)",
      "maximum": 7,
      "minimum": 1,
      "title": "Days",
      "type": "integer"
    }
  },
  "required": ["location"]
}
```

#### 💡 若目標為頂層清單（如 `list[WeatherToolInput]`），需使用 `TypeAdapter` 包裝：

```python
from pydantic import TypeAdapter

# 原生 list 不是 Pydantic 類別，需用 TypeAdapter 包裝後呼叫 .json_schema()
list_adapter = TypeAdapter(list[WeatherToolInput])
list_schema = list_adapter.json_schema()
# 產出外層為 "type": "array" 的 JSON Schema！
```

---

# 實戰除錯與核心天條

## 1. required 欄位遺漏或寫錯位置

> **⚠️ required 放置天條**：  
> `required` 是屬於 `object` 層級的屬性，內容必須是**字串陣列**，而不是放在各別屬性內部！  
> **❌ 錯誤寫法**：`"properties": { "name": { "type": "string", "required": true } }`  
> **✅ 正確寫法**：`"properties": { "name": { "type": "string" } }, "required": ["name"]`

---

## 2. 浮點數誤設為 integer 導致校驗失敗

> **⚠️ 數值型別天條**：  
> `integer` 僅允許純整數（如 `10`），任何小數（如 `10.5`）都會被嚴格攔截！  
> **鐵律**：涉及價格、百分比、座標、度量衡等可能帶小數點的數值，型別請一律宣告為 `"type": "number"`！

---

## 3. Tool Calling 缺少 description 導致 LLM 亂傳參

> **⚠️ AI 提示詞說明天條**：  
> 大語言模型不會閱讀你的 Python 原始碼，它全憑 JSON Schema 裡的 `"description"` 來推論參數用途與格式。  
> **鐵律**：所有開放給 LLM 呼叫的欄位，都必須撰寫清楚白話的 `description`，並列舉示範範例！
