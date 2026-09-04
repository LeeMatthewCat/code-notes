# 概念與環境配置

## Gemini API 簡介

Google 開發的最新一代多模態大語言模型 (Multimodal AI Model) 開發者介面。

透過 Gemini API，可以用 Python 程式碼直接調用最新的 Gemini 模型（如 `gemini-2.5-flash`、`gemini-2.5-pro`）。

用於實現智慧問答、文本生成、程式碼編寫、圖片理解、語音辨識與資料結構化分析。

---

## 金鑰安防與環境隔離規範

要調用 Gemini API，必須先在 Google AI Studio 免費申請一把 API Key（格式如 `AIzaSy...`）。

> **⚠️ 資安安防警告**：  
> API Key 代表個人身份與帳單權限！**絕對不能寫死在 Python 程式碼中**，更**絕對不能 Commit 到 GitHub**！符合行業標準的做法是將金鑰存放在 `.env` 秘密檔案中。

```env
# .env 本地金鑰檔
GEMINI_API_KEY=AIzaSyYourActualSecretKeyHere
```

---

## SDK 安裝與 Client 初始化

Google 在 2025 年推出了全新一代官方 SDK：`google-genai`（取代舊版 `google-generativeai`）。

```shell
pip install google-genai python-dotenv pillow pydantic
```

```Python
import os
from dotenv import load_dotenv
from google import genai

# 1. 【初始化】：載入 .env 裡的 GEMINI_API_KEY 並自動初始化 Client
load_dotenv()
client = genai.Client()
```

---

# 核心 API 函數全解密

## client.models.generate_content() 與 client.models.generate_content_stream()

發送請求給指定 Gemini 模型以生成內容。

- **`generate_content()`**：同步阻塞呼叫，一次性返回完整內容 (`GenerateContentResponse`)。
- **`generate_content_stream()`**：串流呼叫，分段接收文字碎片 (`GenerateContentResponseStream` 迭代器)，實現打字機即時輸出體驗。

---

### 同步與串流差異比較表

| 比較維度 | `client.models.generate_content()` | `client.models.generate_content_stream()` |
| :--- | :--- | :--- |
| **呼叫模式** | 同步阻塞 (一次性返回完整內容) | 串流輸出 (分段接收文字碎片) |
| **輸入參數** | 兩者完全一致 (`model`, `contents`, `config`) | 兩者完全一致 (`model`, `contents`, `config`) |
| **返回值型別** | `GenerateContentResponse` 物件 | 可迭代的 `GenerateContentResponseStream` 迭代器 |
| **內容讀取方式** | 直接讀取 `response.text` | 使用 `for chunk in response_stream:` 迭代 |
| **適用場景** | 結構化資料生成 (JSON)、工具調用 (Tool Use)、後端處理。 | 聊天對話介面 (Chat UI)、長文生成、打字機即時反饋場景。 |

---

### 核心輸入參數對照表 (兩者共有)

| 參數名稱 | 資料型別 | 預設值 | 參數詳細作用與說明 |
| :--- | :--- | :--- | :--- |
| **`model`** | `str` | *(必填)* | **指定要呼叫的模型名稱**（如 `gemini-2.5-flash` 或 `gemini-2.5-pro`）。 |
| **`contents`** | `str` / `list` | *(必填)* | **發送給 AI 的內容**。可傳入純字串、PIL 圖片、或符合 `{"role":..., "parts":...}` 的列表。 |
| **`config`** | `types.GenerateContentConfig` | `None` | **模型設定物件**。用於指定系統提示詞、temperature、JSON Schema、tools 等。 |

---

### 返回值結構解密與比較

#### 1. GenerateContentResponse (同步模式返回值)

- **`text`**：`str` / `None`。模型生成的純文字回應（自動過濾思考片段，僅保留最終解答）。
- **`parts`**：`list[types.Part]` / `None`。原始內容片段列表。
    - **`text`**：`str` / `None`。片段文字內容。
    - **`thought`**：`bool` / `None`。標示是否為模型推理思考 (Chain-of-Thought) 過程。
    - **`thought_signature`**：`bytes` / `None`。思考過程的數位簽名。
    - **`function_call`**：`types.FunctionCall` / `None`。模型發出的單一工具呼叫請求物件，包含 `.name` (函式名稱) 與 `.args` (參數字典)。
	    - **另外還要實作 `id` 和 `thought_signature`** 於 `FunctionCall` 裡
    - **`function_response`**：`types.FunctionResponse` / `None`。傳回給模型的工具執行結果物件，包含 `.name` (函式名稱) 與 `.response` (結果字典)。
- **`function_calls`**：`list[types.FunctionCall]` / `None`。模型請求呼叫的本地工具清單。
- **`candidates`**：`list`。模型生成的候選回應列表，包含生成元數據。
- **`usage_metadata`**：`types.UsageMetadata`。Token 使用量統計。

#### 2. GenerateContentResponseStream (串流模式返回值)

`GenerateContentResponseStream` 本質上是一個可迭代的產生器 (Generator)。

- **迭代特性**：支援 `__iter__`，使用 `for chunk in response_stream:` 進行遍歷。
- **單個 Chunk 的結構**：迭代出來的每個 `chunk` 本身就是一個 `GenerateContentResponse` 物件。
    - **`chunk.text`**：`str` / `None`。當前分段產出的最終文字碎片（自動排除思考內容）。
    - **`chunk.parts`**：`list[types.Part]` / `None`。當前 chunk 的內容片段清單。
        - **`part.text`**：`str` / `None`。當前 Chunk 的分段文字內容。
        - **`part.thought`**：`bool` / `None`。若為 `True`，表示 `part.text` 為即時輸出的思考過程碎片。
        - **`part.function_call`**：`types.FunctionCall` / `None`。若當前 Chunk 包含工具調用請求，包含 `.name` (函式名稱) 與 `.args` (參數字典)。
    - **`chunk.function_calls`**：串流模式下若觸發工具調用，指令通常在最後一個 chunk 中帶出。
		>是 SDK 提供的快捷屬性，會自動遍歷 `parts` 列表並提取出所有存在 `part.function_call` 的物件集合。若需要針對每一個 `Part` 進行細粒度檢查或構建底層
    - **`chunk.usage_metadata`**：在最後一個 `chunk` 中包含完整的 Token 使用量統計。

> **💡 `parts` 屬性何時會為 `None`？**：  
> 存取 `response.parts` 或 `chunk.parts` 前務必進行 `if response.parts:` 或 `if chunk.parts:` 防空檢查。  
> 1. **觸發安全/資安攔截 (Safety Block / Content Filter)**：輸入 Prompt 或模型輸出觸發過濾，未生成有效 Candidates。  
> 2. **Prompt 請求被預先拒絕 (Prompt Blocked)**：例如 `prompt_feedback` 攔截，伺服器直接拒絕回應。  
> 3. **無內容回應/異常空封包**：特定串流 chunk 或異常狀態下未包含內容片段。

> **💡 觀念釐清：`part.text` 貫穿「思考」與「回答」兩階段**：  
> `part.text` 是底層存放文字內容的通用容器。不論是在「思考階段 (`part.thought == True`)」還是在「正式回答階段 (`part.thought` 為 `None`/`False`)」，文字內容都是放在 `part.text` 屬性中。  
> - **思考階段**：`part.thought = True` ➔ `part.text` 為思考鏈文字。  
> - **回答階段**：`part.thought = None/False` ➔ `part.text` 為正式解答文字。  
> - **`response.text` / `chunk.text`**：僅為 SDK 提供的高階快捷屬性，自動篩選並拼接了回答階段的 `part.text`。

> **💡 觀念釐清：`part.function_call` 與多工具並行 (Parallel Tool Calls)**：  
> - **單一 `part.function_call`**：只會存放**單一工具**的呼叫物件 (`types.FunctionCall`)，本身不是列表。  
> - **多工具並行呼叫**：當 Gemini 決定同時調用多個工具時，會在 `parts` 列表中放置**多個 `Part` 物件**（每個 `Part` 各自持有一個 `part.function_call`）。  
> - **`response.function_calls`**：則是 SDK 提供的快捷屬性，自動將 `parts` 中所有的 `part.function_call` 收集成一個 `list[types.FunctionCall]` 列表。

---

### 1. 單次問答 (同步基礎文字生成)

```Python
from dotenv import load_dotenv
from google import genai

# 1. 【初始化】：載入環境變數並建立 Client 物件
load_dotenv()
client = genai.Client()

# 2. 【調用模型】：同步發送生成請求
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="請用一句話簡短解釋什麼是 Python 虛擬環境？",
)
print(response.text)  # 輸出: Python 虛擬環境是一個獨立的隔離資料夾，用於管理專案專屬的套件與版本。
```

---

### 2. 串流打字機與思考過程輸出 (串流生成)

- **基礎串流打字機**

```Python
from dotenv import load_dotenv
from google import genai

# 1. 【初始化】：載入環境變數並建立 Client 物件
load_dotenv()
client = genai.Client()

# 2. 【調用模型】：以串流方式發送請求，取得可迭代的 Response Stream
response_stream = client.models.generate_content_stream(
    model="gemini-2.5-flash",
    contents="請寫一首關於 Python 工程師加班夜讀的現代抒情詩。"
)

# 3. 【串流接收】：迴圈接收連續送達的文字 Chunk 碎片並即時印出
for chunk in response_stream:
    print(chunk.text, end="", flush=True)
print()
```

- **分色顯示思考過程與最終回答**

```Python
from dotenv import load_dotenv
from google import genai
from google.genai import types

# 1. 【初始化】：載入環境變數並建立 Client 物件
load_dotenv()
client = genai.Client()

# 2. 【配置思考】：設定開啟思考過程傳回
config = types.GenerateContentConfig(
    thinking_config=types.ThinkingConfig(include_thoughts=True)
)

response_stream = client.models.generate_content_stream(
    model="gemini-2.5-flash",
    contents="9.11 與 9.9 哪一個數字比較大？",
    config=config,
)

# 3. 【分色印出】：檢查每個 chunk.parts，區分思考片段與最終回答
for chunk in response_stream:
    if chunk.parts:
        for part in chunk.parts:
            if part.thought:
                # 灰色印出思考過程
                print(f"[90m{part.text}[0m", end="", flush=True)
            elif part.text:
                # 正常印出最終回答
                print(part.text, end="", flush=True)
print()
```

---

### 3. 多模態 (圖片與檔案解讀)

```Python
from PIL import Image
from dotenv import load_dotenv
from google import genai

# 1. 【初始化】：載入環境變數並建立 Client 物件
load_dotenv()
client = genai.Client()

# 2. 【讀取圖片】：開啟本地圖片檔案
img = Image.open("sample_chart.png")

# 3. 【調用模型】：將圖片與文字說明作為列表傳入 contents
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[img, "請詳細分析這張圖片里的圖表數據，並整理出三大重點摘要。"]
)
print(response.text)  # 輸出: 根據圖表分析：1. 銷售額成長 20%...
```

---

### 4. 工具調用 (Tool Use)

> **💡 小提醒**：  
> 1. **工具打包形式**：必須打包成帶有 Type Hints (型別標註) 與 Docstring (功能說明) 的標準 Python 函式。  
> 2. **多工具註冊**：可同時將多個本地函式（例如：查詢天氣 `get_current_weather` 與 查詢景點 `get_city_attractions`）放入列表傳給 `tools=[...]`。模型會根據 Prompt 自動決定呼叫哪一個或同時呼叫多個工具。

```Python
from dotenv import load_dotenv
from google import genai
from google.genai import types

# 1. 【初始化】：載入環境變數並建立 Client 物件
load_dotenv()
client = genai.Client()

# 2. 【定義工具 1】：查詢天氣
def get_current_weather(location: str) -> str:
    """獲取指定城市的當前即時天氣與氣溫資訊"""
    return f"{location} 目前多雲時晴，氣溫 26°C。"

# 3. 【定義工具 2】：查詢景點推薦
def get_city_attractions(location: str) -> str:
    """獲取指定城市的熱門旅遊景點列表"""
    return f"{location} 熱門景點包含：台北101、故宮博物院、陽明山國家公園。"

# 4. 【調用模型】：於 config 中註冊包含多個工具的清單
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="請問台北現在天氣怎麼樣？順便推薦我幾個台北的熱門景點！",
    config=types.GenerateContentConfig(tools=[get_current_weather, get_city_attractions])
)
print(response.text)  # 輸出: 台北目前天氣多雲時晴，氣溫 26°C。熱門景點推薦有：台北101、故宮博物院與陽明山國家公園。
```

#### .function_calls 手動工具調用指令解構 (多工具處理)

當需要手動控制工具執行或進行安全審核時，可透過 `response.function_calls` 遍歷 AI 發出的所有工具呼叫請求。

它本身是一個串列（未發出工具調用時為 `None`），每個元素包含：
- **`name`**：模型要求呼叫的工具函式名稱字串（如 `get_current_weather` 或 `get_city_attractions`）。
- **`args`**：模型為該工具自動解析產出的引數物件（⚠️ 非 Python 原生字典，建議先以 `dict(call.args)` 轉換）。

> **💡 小提醒**：  
> 當 Gemini 決定同時呼叫多個工具時，`response.function_calls` 會包含多個 `FunctionCall` 物件，建議以 `for call in response.function_calls:` 迴圈進行處理。

```Python
# 1. 【條件檢查與迴圈遍歷】：確保 AI 發出了工具調用請求並逐一處理
if response.function_calls:
    # 建立本地工具函數映射字典 (Mapping)
    tool_map = {
        "get_current_weather": get_current_weather,
        "get_city_attractions": get_city_attractions,
    }

    for call in response.function_calls:
        print(f"🤖 [AI 請求呼叫工具]: {call.name}")
        print(f"📦 [AI 帶入的參數字典]: {call.args}")
        
        # 2. 【動態匹配與執行】：根據工具名稱調用對應函式
        if call.name in tool_map:
            func = tool_map[call.name]
            result = func(**call.args)
            print(f"⚙️ [本地執行成果]: {result}")
```

#### types.Part.from_function_response() 回傳多工具執行結果 (FunctionResponse)

當**手動**攔截 AI 的多個 `FunctionCall` 請求並在本地執行完畢後，必須把所有執行成果封裝為 `FunctionResponse` 的 `Part` 物件傳回模型。

##### 核心參數說明
- **`name`** (`str`)：被呼叫的工具函式名稱（須與 AI 產出的 `call.name` 保持一致）。
- **`response`** (`dict`)：本地函式執行的結果字典（必須為 `dict` 格式，例如 `{"output": "..."}`）。

```Python
from google.genai import types

# 1. 【本地執行多工具】：遍歷調用並將結果分別封裝為 Part 物件列表
response_parts = []
tool_map = {
    "get_current_weather": get_current_weather,
    "get_city_attractions": get_city_attractions,
}

if response.function_calls:
    for call in response.function_calls:
        func = tool_map[call.name]
        tool_result = {"output": func(**call.args)}
        
        # 2. 【封裝 FunctionResponse】：將每個工具成果轉為 Part
        response_parts.append(
            types.Part.from_function_response(
                name=call.name,
                response=tool_result
            )
        )

# 3. 【續接對話】：將 原始 Prompt ➔ AI 的 FunctionCall 回應 ➔ 多工具執行成果 依序傳回模型
final_response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        types.Content(role="user", parts=[types.Part.from_text(text="請問台北現在天氣怎麼樣？順便推薦我幾個台北的熱門景點！")]),
        response.candidates[0].content,                         # 包含 AI 多個 FunctionCall 的 Content
        types.Content(role="user", parts=response_parts)        # 包含所有 FunctionResponse Parts 的 Content
    ],
    config=types.GenerateContentConfig(tools=[get_current_weather, get_city_attractions])
)
print(final_response.text)  # 輸出: 台北目前天氣多雲時晴，氣溫 26°C。熱門景點推薦有：台北101、故宮博物院與陽明山國家公園。
```

---

####  SDK 自動代管 (Automatic Function Calling)

在使用 `google-genai` SDK 時，若要保留手動攔截與使用者授權 (Human-in-the-loop) 機制，必須明確關閉自動代管。
可以達成：
1. 攔截工具調用
2. 對 id 及簽名的自行管理

- **設定方式**：**在 `GenerateContentConfig` 中設定** `automatic_function_calling=types.AutomaticFunctionCallingConfig(disable=True)`。
```Python
... = ...generate_content_stream(
	...
	config=types.GenerateContentConfig(
		tools=...,
		automatic_function_calling=types.AutomaticFunctionCallingConfig(disable=True)
)
```

> **💡 小提醒**：否則 SDK 會在背景私自執行工具，導致自建的 Agent 邏輯失效，甚至可能引發空字串崩潰！

---

##### 工具調用 id 和思考簽名  thought_signature

**id** 是業界標準（如 OpenAI、Claude、Gemini），用於解決多工具並行時的結果對應問題。

- **運行機制**：當模型一次呼叫多個工具時，每個呼叫都會附帶一個唯一的 `id`。
- **回傳要求**：當我們將執行結果 (`FunctionResponse`) 傳回給模型時，必須附上對應的 `id`，模型才能正確對應哪個結果屬於哪個呼叫。
**thought_signature** 是 Google 具備推理能力的新模型（如 Gemini 2.0 Flash Thinking、3.5 Flash Lite）獨有的安全對齊機制。

- **結構位置**：當模型經過長期思考並決定呼叫工具時，API 會在 `Part` 屬性層級附上一段 `thought_signature`（與 `FunctionCall` 平級，包在 `Part` 內部）。
- **嚴格驗證**：當我們**回傳工具結果時**，如果沒有把這個簽名一併送回，Google API 會因為無法將結果對齊原先的內部推理脈絡，而直接回傳 `400 INVALID_ARGUMENT` 錯誤並拒絕回答。
---

> 為什麼要對 id 及簽名自管理？
> 因為在 agent 的流程為：「中斷 ➔ 本地處理/等待授權 ➔ 重新封裝 ➔ 再次發送」**的跨回合操作。我們必須自己保管好 API 發下來的「領取憑證 (`id`)」與「防偽圖章 (`thought_signature`)」，交卷的時候一起還回去，API 才會承認這份結果


我們將 id 及簽名要帶回給模型時要**自行做重新封裝**才能交還給模型：
```Python
fc = types.FunctionCall(name=t["name"], args=t["args"]) # 實作 FunctionCall

fc.id = t["id"]

part = types.Part(function_call=fc) # FunctionCall 組裝回 Part

part.thought_signature = t["thought_signature"] # 在 Part 貼上 thought_signature 標籤

parts.append(part) # 組裝好的 Part 放回串列
```

---

- 兩棲作戰的架構設計 (相容 Ollama)

	為了讓一套系統同時支援 Gemini 與 Ollama，在內部流通的 ToolCall 資料類別中，必須將 `id` 與 `thought_signature` 宣告為 `Optional` 並預設為 `None`。

	- **面對 Gemini 時**：手動放棄有缺陷的官方捷徑 `.from_function_call()`。改為手動實例化 `types.FunctionCall` 與 `types.Part`，將存下來的 ID 與簽名塞入後傳遞給 API。
	- **面對 Ollama 時**：Ollama SDK 在轉換歷史紀錄時只認得 `name` 和 `args`。包含 `None` 值的 `id` 與 `thought_signature` 欄位會被安全地過濾並忽略，完全不會引發報錯。

---

## types.GenerateContentConfig

用於配置 Gemini 模型的生成行為、溫度參數、角色設定、JSON 結構與工具調用。

### 核心參數字典對照表

| 參數名稱 | 資料型別 | 預設值 | 參數詳細作用與說明 |
| :--- | :--- | :--- | :--- |
| **`system_instruction`** | `str` | `None` | **角色人設與系統指令**。設定 AI 的身份、說話口吻或行為規則。 |
| **`temperature`** | `float` | `1.0` | **發散性/創意度**（範圍 `0.0`~`2.0`）。越低越嚴謹確定，越高越豐富有創意。 |
| **`max_output_tokens`** | `int` | `None` | **限制模型輸出的最大 Token 數量**。 |
| **`response_mime_type`** | `str` | `None` | **指定回傳格式**。若要強制回傳 JSON 可設為 `application/json`。 |
| **`response_schema`** | `BaseModel` / `dict` | `None` | **指定 JSON 回傳的數據結構**。常傳入 Pydantic 類別。 |
| **`tools`** | `list` | `None` | **傳入可供 AI 呼叫的 Python 本地函式清單** (Tool Use)。 |
| **`automatic_function_calling`** | `types.AutomaticFunctionCallingConfig` | `None` | **工具調用自動代管配置**。設定 `disable=True` 可關閉 SDK 背景自動執行工具，交由開發者手動攔截並自行管理 ID / 簽名。 |
| **`thinking_config`** | `types.ThinkingConfig` | `None` | **思考過程配置**。設定是否包含思考過程與思考 Token 預算。 |

---

### 1. 系統指令 (System Instruction) 與創意度

```Python
from google.genai import types

# 1. 【設定人設】：配置系統角色設定與 temperature
config = types.GenerateContentConfig(
    system_instruction="你是一位嚴格的 Python 高級資深架構師，語氣專業冷酷。",
    temperature=0.2,
)
```

---

### 2. 結構化 JSON 輸出 (配合 Pydantic)

```Python
from pydantic import BaseModel
from google.genai import types

# 1. 【定義結構】：使用 Pydantic 定義 JSON Schema
class Recipe(BaseModel):
    recipe_name: str
    ingredients: list[str]

# 2. 【設定配置】：強制回傳 JSON 並指定 response_schema
config = types.GenerateContentConfig(
    response_mime_type="application/json",
    response_schema=Recipe,
)
```

---

### 3. 推理思考設定 (ThinkingConfig)

```Python
from google.genai import types

# 1. 【配置思考預算】：設定思考 Token 預算上限與是否包含思考片段
config = types.GenerateContentConfig(
    thinking_config=types.ThinkingConfig(
        thinking_budget=2048,
        include_thoughts=True,
    )
)
```

---

### 4. 關閉工具自動代管 (手動攔截 Tool Calls)

```Python
from google.genai import types

# 1. 【關閉自動執行】：設定 disable=True 讓 SDK 不在背景私自呼叫工具，交由程式手動攔截
config = types.GenerateContentConfig(
    tools=[get_current_weather, get_city_attractions],
    automatic_function_calling=types.AutomaticFunctionCallingConfig(disable=True)
)
```

---

## client.chats.create()

建立一個具備記憶力的多輪對話 Session (`Chat`)，會自動在本地記錄與維護歷史訊息（透過 `chat.send_message()` 進行對話）。

### 核心參數字典對照表

| 參數名稱 | 資料型別 | 預設值 | 參數詳細作用與說明 |
| :--- | :--- | :--- | :--- |
| **`model`** | `str` | *(必填)* | 對話使用的模型名稱（如 `gemini-2.5-flash`）。 |
| **`config`** | `types.GenerateContentConfig` | `None` | 該對話 Session 的通用模型生成設定。 |
| **`history`** | `list[types.Content]` | `None` | 可傳入預先載入的歷史對話紀錄列表。 |

---

### 多輪對話語法示範

```Python
from dotenv import load_dotenv
from google import genai

# 1. 【初始化】：載入環境變數並建立 Client 物件
load_dotenv()
client = genai.Client()

# 2. 【對話初始化】：建立對話 Session
chat = client.chats.create(model="gemini-2.5-flash")

# 3. 【第一輪對話】：發送訊息
response1 = chat.send_message("嗨！我是 Matthew，我住在台北。")
print(f"AI: {response1.text}
")  # 輸出: AI: 你好 Matthew！很高興認識你...

# 4. 【第二輪對話】：模型自動帶入歷史對話記憶
response2 = chat.send_message("請問我的名字是什麼？我住在幾度的城市？")
print(f"AI: {response2.text}")  # 輸出: AI: 你的名字是 Matthew，你住在台北。
```

---

## 底層多輪對話 Payload 格式 (role 與 parts)

### 1. 基礎純文字多輪對話歷史序列

```Python
contents = [
    # 1. 【用戶提問】與【工具結果】被強行合併成同一次發言 (role: "user")
    # 因為 agent.py 遺失了模型的 function_call 紀錄，
    # 而 gemini_provider.py 為了避免 API 報錯，將連續出現的 user 角色合併了 parts。
    {
        "role": "user",
        "parts": [
            {
                # 這是用戶原本的提問
                "text": "請問台北現在天氣怎麼樣？順便推薦我幾個台北熱門景點！"
            },
            {
                # 這是 agent.py 幫我們把多個工具結果「結合成一大串字串」後，
                # gemini_provider.py 把它當作純文字補充段落，貼在用戶提問的底下。
                "text": "execute tool result:......"
            }
        ]
    },
    
    # 注意：這裡【完全沒有】當初模型決定呼叫工具的紀錄，這段被抹除了。
    # (原本應該要有 role: "model" 以及 function_call 的片段)

    # 2. 模型看到上述整包用戶提供的「問題＋附帶資料」後，直接產出的最終解答 (role: "model")
    {
        "role": "model",
        "parts": [
            {
                "text": "台北目前天氣多雲時晴，氣溫 26°C。熱門景點推薦包含台北101、故宮博物院與陽明山！"
            }
        ]
    }
]
```

---

### 2. SDK 提供的形式


```Python
from google.genai import types

contents = [
    # 1. 用戶發起提問 (role: "user")
    types.Content(
        role="user",
        parts=[types.Part.from_text(text="請問台北現在天氣怎麼樣？順便推薦我幾個台北熱門景點！")]
    ),
    # 2. 模型決定同時呼叫兩個工具 (role: "model"，parts 包含兩個 Part.from_function_call)
    types.Content(
        role="model",
        parts=[
            types.Part.from_function_call(name="get_current_weather", args={"location": "台北"}),
            types.Part.from_function_call(name="get_city_attractions", args={"location": "台北"})
        ]
    ),
    # 3. 本地執行完兩個工具，將成果寫回給模型 (role: "user"，parts 包含兩個 Part.from_function_response)
    types.Content(
        role="user",
        parts=[
            types.Part.from_function_response(name="get_current_weather", response={"output": "台北目前多雲時晴，氣溫 26°C。"}),
            types.Part.from_function_response(name="get_city_attractions", response={"output": "台北熱門景點：台北101、故宮博物院、陽明山。"})
        ]
    ),
    # 4. 模型結合兩個工具執行結果產出的最終解答 (role: "model")
    types.Content(
        role="model",
        parts=[types.Part.from_text(text="台北目前天氣多雲時晴，氣溫 26°C。熱門景點推薦包含台北101、故宮博物院與陽明山！")]
    )
]
```

> **💡 規範與角色特點 (Gemini vs OpenAI / Ollama)**：  
> 1. **角色名稱為 `"model"`**：Gemini 使用 `"user"` 與 `"model"`（而非 OpenAI / Ollama 的 `"assistant"`）。  
> 2. **所有內容均包在 `parts` 列表中**：不論是純文字 `text`、`function_call` 還是 `function_response`，都是 `Part` 物件並放進 `parts` 陣列中。  
> 3. **`function_response` 歸類為 `user` 角色**：將本地工具執行成果傳回模型時，`role` 為 `"user"`。
