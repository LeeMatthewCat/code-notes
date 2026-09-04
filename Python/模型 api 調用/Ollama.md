# 概念與原理

## 什麼是 Ollama？

目前全球 Open-Source 社群中最流行的**本地大語言模型 (Local LLM) 運作與管理引擎**。

在傳統使用 AI 大模型的開發流程中，我們通常需要向 OpenAI (ChatGPT)、Google (Gemini) 或 Anthropic (Claude) 申請 API Key，每次對話都會將資料傳送到雲端伺服器。除了需要付費訂閱或按 Token 計費外，也存在資安與商業機密外洩的隱憂。

`Ollama` 是一個專為本地端設計的模型管理與執行框架（基於 Rust / Go 與 `llama.cpp` 底層構建）。它能幫你自動完成開源模型（如 DeepSeek-R1, Qwen 2.5, Llama 3.3, Mistral）的**下載、記憶體配置、GPU 硬體加速（Apple Metal / NVIDIA CUDA）**，並在本地背景自動開啟一個監聽在 `http://localhost:11434` 的 REST API 伺服器，讓你可以用極簡的 Python 代碼免費、零延遲、100% 離線調用本地模型！

> **🍿 生動白話比喻**：  
> 如果下載開源模型的原始權重檔（GGUF / Safetensors）像是買回一堆**裸裝的顯卡晶片與電路零件**，要自己焊電路板與編寫驅動程式才能用；  
> 那麼 `Ollama` 就是**「一鍵式家用插電 AI 主機」**！你只要開機並說出模型名字（`ollama run qwen2.5`），它自動幫你組裝零件、開闢記憶體包廂並插上電源，你在本機就能直接享用免費的 AI 能力！

---

## 為什麼選擇 Ollama？（四大核心優勢）

1. **100% 免費與無限制調用**：不需綁定信用卡，完全沒有 API 額度限制或按 Token 扣款的費用負擔。
2. **極致隱私保護 (Zero Data Leak)**：所有的 Prompt 提示詞與上下文資料完全留在個人電腦，絕對不會傳上雲端或被拿去訓練模型。
3. **離線運作 (Offline Ready)**：無網路環境、搭飛機或高資安要求的密閉環境中皆能順暢執行。
4. **開放標準 OpenAI 介面相容**：Ollama 內建了符合 OpenAI API 規範的 Endpoints，能無縫接入 LangChain、AutoGPT、LlamaIndex 等 Agent 框架。

---

## 安裝 Ollama 與常用 CLI 指令

### 1. 安裝 Ollama

- **macOS**：直接前往 [Ollama 官網](https://ollama.com) 下載 Mac 應用程式，或使用 Homebrew：
  ```shell
  brew install ollama
  ```
- 安裝完成後，Ollama 服務預設會在背景自動執行，並監聽在 `http://localhost:11434`。

---

### 2. 終端機常用 CLI 指令

```shell
# 1. 下載並立刻啟動對話 (若本地無此模型，會自動由雲端倉庫下載)
ollama run qwen2.5:7b

# 2. 列出本地已下載的所有模型
ollama list

# 3. 查看當前正在記憶體中運行的模型與佔用資源
ollama ps

# 4. 刪除指定的本地模型以釋放硬碟空間
ollama rm qwen2.5:7b
```

---

## Python 調用 Ollama 的 3 種主流方式

以下詳細介紹如何在 Python 專案中透過官方 SDK、OpenAI 相容介面與標準 HTTP REST 呼叫本地 Ollama。

---

## 方式一：官方 ollama Python SDK (推薦最簡潔)

官方提供的 Python 套件，語法極致簡潔且原生支援非同步 (Async) 與串流 (Stream)。

### 1. 安裝 SDK

使用 [[專案準備#Step 2：建立與管理虛擬環境 (使用現代極速 uv)|uv]] 或 pip 安裝：

```shell
uv add ollama
# 或 pip install ollama
```

---

##### ollama.list() 列出本地已安裝模型清單

- **使用時機**：在 Python 程式中動態獲取當前本地 Ollama 服務已下載的所有模型清單（常用於：啟動時供使用者互動選取模型、或在執行生成前防呆檢查本地是否已具備該模型）。
- **語法**：`response = ollama.list()`
- **參數說明**：
  - 無必填參數。
- **回傳值**：
  - `ListResponse` 物件，**包含 `.models` 列表**。
  - 每個模型元素包含以下核心屬性：

| 屬性名稱                        | 資料型別                    | 說明與範例                                                      |
| :-------------------------- | :---------------------- | :--------------------------------------------------------- |
| **`.model`**（或 **`.name`**） | `str`                   | 模型完整標籤名稱（如 `"qwen2.5:7b"`、`"deepseek-r1:7b"`）。             |
| **`.size`**                 | `int`                   | 模型佔用磁碟空間大小（單位：位元組 Bytes，除以 $1024^3$ 即為 GB）。                |
| **`.modified_at`**          | `datetime` / `str`      | 模型的最後修改或下載時間戳記。                                            |
| **`.digest`**               | `str`                   | 模型檔案的 SHA256 唯一識別雜湊值。                                      |
| **`.details`**              | `ModelDetails` / `dict` | 包含 `family`, `parameter_size`, `quantization_level` 等架構細節。 |
```JSON
{
  "models": [
    {
      "model": "llama3:latest",
      "modified_at": "2026-06-12T10:30:15.123456Z",
      "size": 4920739840,
      "digest": "a6990840bc4823...5c24e6",
      "details": {
        "parent_model": "",
        "format": "gguf",
        "family": "llama",
        "families": ["llama"],
        "parameter_size": "8.0B",
        "quantization_level": "Q4_0"
      }
    },
    {
      "model": "mistral:latest",
      "modified_at": "2026-05-20T08:15:22.654321Z",
      "size": 4372937728,
      "digest": "2ae6f6dd7a3d44...9d811b",
      "details": {
        "parent_model": "",
        "format": "gguf",
        "family": "llama",
        "families": ["llama"],
        "parameter_size": "7.2B",
        "quantization_level": "Q4_0"
      }
    },
    {
      "model": "nomic-embed-text:latest",
      "modified_at": "2026-04-10T14:02:00.000000Z",
      "size": 274302464,
      "digest": "0a109f422b4701...ff328a",
      "details": {
        "parent_model": "",
        "format": "gguf",
        "family": "nomic-bert",
        "families": ["nomic-bert"],
        "parameter_size": "137M",
        "quantization_level": "F16"
      }
    }
  ]
}
```
```python
import ollama

# 1. 取得本地所有模型清單
response = ollama.list()

print(f"📦 本地已安裝模型數量: {len(response.models)}")

# 2. 走訪並印出每個模型的名稱與佔用容量
for m in response.models:
    size_gb = m.size / (1024 ** 3)
    print(f"🔹 模型: {m.model:20} | 容量: {size_gb:.2f} GB | 更新時間: {m.modified_at}")

# 3. 實戰防呆檢查：確認目標模型是否已下載
installed_models = [m.model for m in response.models]
target = "qwen2.5:7b"

if target not in installed_models:
    print(f"⚠️ 本地尚未下載 {target}，請先至終端機執行 `ollama run {target}`！")
else:
    print(f"✅ 目標模型 {target} 已就緒，可直接調用。")
```

> **💡 屬性點選與字典相容性**：  
> `ollama` SDK 回傳的 `models` 同時支援**物件屬性點選** (`m.model` / `m.size`) 與**字典下標存取** (`m["model"]` / `m["size"]`) 兩種風格。

---

##### ollama.generate() 單次生成文字

- **使用時機**：當你不需要處理複雜多輪對話歷史，只需要輸入單一 Prompt 提示詞並取得 AI 簡短回答或文本生成時使用。
- **語法**：`ollama.generate(model: str, prompt: str, system="", images=None, options=None, stream=False, keep_alive="5m")`
- **參數說明**：
  - `model`：指定呼叫的本地模型名稱標籤（如 `"qwen2.5:7b"`）。
  - `prompt`：輸入給模型的單次提示詞字串。
  - `system`：（可選）設定系統指令（System Prompt）。
  - `options`：（可選）超參數設定字典（如 `temperature`, `top_p`）。
  - `stream`：布林值，是否開啟串流輸出，預設 `False`。
- **回傳值**：
  - `dict`：包含 `response` 欄位（生成的文字字串）與耗時資訊的字典。

```python
import ollama

# 呼叫 generate 直接傳入模型名稱與提示詞
response = ollama.generate(
    model="qwen2.5:7b",
    prompt="請用繁體中文簡短介紹什麼是離線大語言模型。"
)

# 取得生成內容
print(response["response"])
```

---

##### ollama.chat() 多輪對話與對話歷史

- **使用時機**：當你需要建立多輪對話（保留聊天歷史）、傳入 Tool 工具函數、指定結構化 JSON 輸出或處理多模態時使用。
- **語法**：`ollama.chat(model: str, messages: list[dict], tools=None, format=None, options=None, stream=False, think=True, keep_alive="5m")`
- **參數說明**：

| 參數名稱             | 資料型別           | 預設值     | 參數詳細作用與說明                                                      |
| :--------------- | :------------- | :------ | :------------------------------------------------------------- |
| **`model`**      | `str`          | *(必填)*  | **指定呼叫的本地模型名稱**（如 `qwen2.5:7b` 或 `deepseek-r1:7b`）。            |
| **`messages`**   | `list[dict]`   | *(必填)*  | **對話歷史紀錄清單**（格式如 `[{"role": "user", "content": "..."}]`）。      |
| **`tools`**      | `list`         | `None`  | **傳入本地 Python 函式或 JSON Schema 清單** (Tool Use)。                 |
| **`format`**     | `str` / `dict` | `None`  | **指定回傳格式**。可設為 `"json"` 或 Pydantic 的 `model_json_schema()`。    |
| **`options`**    | `dict`         | `None`  | **模型超參數字典**（設定 `temperature`, `top_p`, `num_ctx` 上下文長度等）。      |
| **`stream`**     | `bool`         | `False` | **是否開啟串流打字機效果**。若設為 `True` 則回傳 Generator 迭代物件。                 |
| **`think`**      | `bool`         | `True`  | **控制是否開啟推理模型 (如 DeepSeek-R1) 的思考過程**。若設為 `False` 可關閉思考，顯著降低延遲。 |
| **`keep_alive`** | `str` / `int`  | `"5m"`  | **模型在記憶體/顯存中的駐留時間**（如 `"5m"`, `"1h"`, `-1` 代表永久駐留）。            |

- **回傳值**：
  - `ChatResponse` / `dict`：回傳對話回應物件，包含 `message` 訊息與執行指標。

###### ollama.chat() 回傳物件 ChatResponse 欄位說明

| 欄位名稱 | 資料型別 | 說明與存取寫法範例 |
| :--- | :--- | :--- |
| **`message`** | `Message` / `dict` | **模型回傳的消息物件**（包含 `role`, `content`, `thinking`, `tool_calls` 等子屬性）。存取：`response.message` 或 `response["message"]` |
| **`model`** | `str` | **實際呼叫的模型名稱標籤**（如 `"qwen2.5:7b"`）。 |
| **`created_at`** | `str` | **回應產生的 ISO 8601 時間戳記**。 |
| **`done`** | `bool` | **回應生成是否完結**（非串流模式固定為 `True`）。 |
| **`done_reason`** | `str` | **停止生成的原因**（如 `"stop"` 代表正常生成完畢，`"length"` 代表達 Token 上限）。 |
| **`total_duration`** | `int` | **模型處理總耗時**（單位：奈秒 ns，除以 $10^9$ 即為秒數）。 |
| **`load_duration`** | `int` | **模型載入至顯存/記憶體的耗時**（單位：奈秒 ns）。 |
| **`prompt_eval_count`** | `int` | **輸入 Prompt 提示詞的 Token 數量**。 |
| **`prompt_eval_duration`** | `int` | **評估/處理輸入提示詞的耗時**（單位：奈秒 ns）。 |
| **`eval_count`** | `int` | **模型生成輸出的 Token 數量**（生成答覆的長度）。 |
| **`eval_duration`** | `int` | **模型生成解答的耗時**（單位：奈秒 ns）。 |

###### .message 子欄位與 .tool_calls 結構

| 訊息子屬性 | 資料型別 | 說明與存取寫法範例 |
| :--- | :--- | :--- |
| **`.role`** | `str` | **訊息角色**（通常為 `"assistant"`）。 |
| **`.content`** | `str` / `""` | **模型發言的主要文字內容**。若觸發 Function Call 則通常為 `""` 空字串。 |
| **`.thinking`** | `str` / `None` | **推理模型 (如 DeepSeek-R1) 的思考過程**（若開啟思考功能）。 |
| **`.images`** | `list` / `None` | **多模態輸入之圖片資料清單**。 |
| **`.tool_calls`** | `list[ToolCall]` / `None` | **模型決定調用的本地 Function 清單** (Tool Use)。每個 ToolCall 包含 `.function.name` (函式名) 與 `.function.arguments` (引數字典)。 |

> **💡 語法存取小提醒**：  
> 最新的 `ollama` Python SDK 回傳之物件同時支援**字典存取** (`response['message']['content']`) 與**物件屬性點選** (`response.message.content`) 兩種寫法；走訪 `tool_calls` 時也可使用 `call.function.name` 與 `call.function.arguments` 直接取得函式名稱與引數字典。

```python
import ollama

# 1. 建立包含對話歷史的清單
messages = [
    {"role": "system", "content": "你是一位精通 Python 的資深技術顧問，回答請簡明扼要。"},
    {"role": "user", "content": "你好，我是 Matthew，我想學 Ollama。"},
]

# 2. 傳送多輪對話
response = ollama.chat(model="qwen2.5:7b", messages=messages)

# 3. 印出模型回覆
print(f"AI: {response.message.content}")
```

---

##### 串流打字機效果 (stream=True)

- **使用時機**：當模型較大或輸出較長時，設定 `stream=True` 可將回傳改為 Generator 迭代物件，實現像 ChatGPT 網頁般的逐字打字機輸出。

```python
import ollama

# 1. 開開啟 stream=True
stream = ollama.chat(
    model="qwen2.5:7b",
    messages=[{"role": "user", "content": "請寫一首關於工程師深夜除錯的現代抒情詩。"}],
    stream=True
)

# 2. 迭代印出每一個 chunk 數據
for chunk in stream:
    print(chunk["message"]["content"], end="", flush=True)
print()  # 換行
```

---

##### 結構化 JSON 輸出 (format="json")

- **使用時機**：當你需要讓本地模型回傳合法的 JSON 字串，並結合 Pydantic 驗證資料型別時使用。

```python
import ollama
from pydantic import BaseModel

# 1. 定義 Pydantic 資料模型
class CityInfo(BaseModel):
    city_name: str
    country: str
    population_millions: float

# 2. 使用 format=CityInfo.model_json_schema() 強制輸出指定 Schema
response = ollama.chat(
    model="qwen2.5:7b",
    messages=[{"role": "user", "content": "請提供日本東京的基本城市資訊。"}],
    format=CityInfo.model_json_schema()
)

print(response.message.content)
```

---

## 推理型模型 (DeepSeek-R1) 思考過程處理

在處理推理型大模型（如 `deepseek-r1:7b` 或 `deepseek-r1:14b`）時，模型會在給出最終答案前產生思考鏈 (Chain-of-Thought)。

### 1. 預設自動分離思考內容或傳入 `think=False` 完全關閉

在最新版的 Ollama 服務與 Python SDK 中，系統**原生支援思考過程控制與自動拆分**：

- **`response.message.content`** ➔ **預設即為乾淨的最終回答**（Ollama 原生將思考脈絡分離至 `response.message.thinking` 欄位）。
- **`think=False` 參數** ➔ 若希望模型**完全不產生思考過程**（跳過思考鏈生成，極速降低延遲與 Token 消耗），直接在 `ollama.chat()` 傳入 `think=False` 即可！

```python
import ollama

# 方法 A：預設情況 (模型內部會思考，但 content 自動保持乾淨)
response = ollama.chat(
    model="deepseek-r1:7b",
    messages=[{"role": "user", "content": "9.11 與 9.9 哪一個數字比較大？"}]
)
print("💡 最終回答:")
print(response.message.content)

# 方法 B：傳入 think=False (直接讓模型關閉思考生成，大幅降低回應延遲)
fast_response = ollama.chat(
    model="deepseek-r1:7b",
    messages=[{"role": "user", "content": "9.11 與 9.9 哪一個數字比較大？"}],
    think=False  # 👈 顯式關閉思考過程！
)
print("⚡ 極速回答:")
print(fast_response.message.content)
```

---

### 2. 主動讀取並顯示思考過程 (Thinking Display)

若希望打造類似 DeepSeek 官網的「思考過程 (Thinking...)」展開效果，只需存取 **`thinking`** 欄位或在串流時動態處理：

```python
import ollama

# 方式 A：非串流直接讀取獨立的思考鏈 (thinking)
response = ollama.chat(
    model="deepseek-r1:7b",
    messages=[{"role": "user", "content": "9.11 與 9.9 哪一個數字比較大？"}]
)

# 讀取獨立的 thinking 屬性
if hasattr(response.message, "thinking") and response.message.thinking:
    print("🧠 [模型思考過程]:")
    print(response.message.thinking)

print("\n💡 [最終回答]:")
print(response.message.content)
```

```python
import ollama

# 方式 B：串流 (stream=True) 時實時分色渲染思考過程與最終回答
stream = ollama.chat(
    model="deepseek-r1:7b",
    messages=[{"role": "user", "content": "9.11 與 9.9 哪一個數字比較大？"}],
    stream=True
)

for chunk in stream:
    # 若當前 chunk 屬於思考過程
    if hasattr(chunk.message, "thinking") and chunk.message.thinking:
        print(f"\033[90m{chunk.message.thinking}\033[0m", end="", flush=True)
    # 若當前 chunk 屬於最終回答
    elif chunk.message.content:
        print(chunk.message.content, end="", flush=True)

print()
```

> **💡 DeepSeek-R1 思考過程擷取技巧**：  
> 1. **標籤攔截**：推理模型的內部思考脈絡被包在 `<think>` 與 `</think>` 之間，透過字串過濾可以優雅地將「內心獨白」與「回覆結果」在 UI 介面上獨立呈現。  
> 2. **非串流取出**：若未開啟 `stream=True`，也可直接透過正規表達式或字串切片解析 `response["message"]["content"]` 中的 `<think>` 區塊！

---

## 方式二：OpenAI 相容通道 (openai SDK)

Ollama 提供了相容 OpenAI API 的本地通道，這意味著你可以**完全不改動原有依賴 OpenAI SDK 的程式碼**，只需修改 `base_url` 即可切換成免費的本地模型！

```python
from openai import OpenAI

# 初始化 OpenAI 客戶端，指向本地 Ollama 的 /v1 通道
client = OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama" # 本地不需要真金鑰，隨意填寫非空字串即可
)

response = client.chat.completions.create(
    model="qwen2.5:7b", # 本地 Ollama 內有的模型
    messages=[
        {"role": "system", "content": "你是一位親切的助手。"},
        {"role": "user", "content": "請問 Python 中 list 與 tuple 的差異？"}
    ]
)

print(response.choices[0].message.content)
```

> **💡 架構設計極致好處**：  
> 當你開發 Agent 專案時，只要把 API Base URL 做成 [[專案準備#5. .env 與 .env.example|.env 秘密檔案]] 的變數，就能在「開發測試階段用本地免費 Ollama」與「生產上線階段用 OpenAI/Gemini 雲端服務」之間一秒無縫切換！

---

## 方式三：標準 HTTP REST API (httpx / requests)

如果你不希望在專案中引入額外的 SDK，可以直接使用 `httpx` 或 `requests` 發送 HTTP POST 請求：

```python
import httpx

# 呼叫 Ollama 本地 REST API
url = "http://localhost:11434/api/generate"
payload = {
    "model": "qwen2.5:7b",
    "prompt": "為什麼 Python 這麼受歡迎？",
    "stream": False # 關閉串流一次回傳
}

response = httpx.post(url, json=payload, timeout=60.0)
data = response.json()

print(data["response"])
```

---

# 高階工具調用 (Tool Use / Function Calling)

現代開源模型（如 `qwen2.5`、`llama3.1`）在 Ollama 中原生支援 Tool Use。

### 1. 工具該以何種形式傳入？

- **形式一（原生 Python 函式，推薦）**：直接傳入帶有 **Type Hints (型別標註)** 與 **Docstring (功能與參數說明)** 的 Python 函式列表（`ollama` SDK 會自動反射解析為 Schema）：
  ```python
  tools=[add_two_numbers]
  ```
- **形式二（標準 JSON Schema 字典）**：傳入符合 OpenAI / Ollama 規範的聲明字典列表：
  ```python
  tools=[{
      "type": "function",
      "function": {
          "name": "add_two_numbers",
          "description": "計算兩個數字相加的和",
          "parameters": {
              "type": "object",
              "properties": {
                  "a": {"type": "number", "description": "第一個數字"},
                  "b": {"type": "number", "description": "第二個數字"}
              },
              "required": ["a", "b"]
          }
      }
  }]
  ```

---

### 2. 發送請求後會回傳什麼？ (Return Value Structure)

當把 `tools=[...]` 傳給 `ollama.chat()` 時：
- **若模型決定呼叫工具**：`response.message.tool_calls` 會回傳一個 **`ToolCall` 物件列表**（否則為 `None`）。
- **每個 `ToolCall` 元素包含兩個核心欄位**：
  - `.function.name`：模型要求呼叫的**工具函式名稱字串**（如 `add_two_numbers`）。
  - `.function.arguments`：模型自動解析並帶入的**引數字典 (`dict`)**（如 `{a: 456, b: 789}`）。

```python
# 檢查模型是否觸發了工具調用
if response.message.tool_calls:
    for call in response.message.tool_calls:
        func_name = call.function.name         # 取得要呼叫的函式名稱 (如 'get_current_weather')
        func_args = call.function.arguments    # 取得模型的參數字典 (如 {'location': '台北', 'unit': 'celsius'})
        print(f"🛠️ 模型請求呼叫函式: {func_name}, 參數: {func_args}")
```

---

### 3. Tool Calling 對話歷史 (messages) 完整閉環範例

```python
import ollama

# 1. 定義本地工具函式 (形式一：必須帶有 Type Hints 與 Docstring)
def add_two_numbers(a: float, b: float) -> float:
    """計算兩個數字相加的和
    Args:
        a: 第一個數字
        b: 第二個數字
    """
    return a + b

# 2. 建立對話歷史並傳給 ollama.chat()
messages = [{"role": "user", "content": "請幫我算算 456 加 789 等於多少？"}]

response = ollama.chat(model="qwen2.5:7b", messages=messages, tools=[add_two_numbers])

# 3. 檢查回傳的 response.message.tool_calls 列表並完成閉環對話
if response.message.tool_calls:
    # 3-1. 將模型的 assistant 請求訊息加入對話歷史
    messages.append(response.message)
    
    for tool in response.message.tool_calls:
        print(f"🤖 本地模型要求調用工具: {tool.function.name}")
        print(f"📦 帶入的引數字典: {tool.function.arguments}")
        
        # 3-2. 執行本地 Python 工具函式
        if tool.function.name == "add_two_numbers":
            res = add_two_numbers(**tool.function.arguments)
            print(f"⚙️ 本地執行計算結果: {res}")
            
            # 3-3. 將工具執行結果封裝為 role="tool" 寫回對話歷史中
            messages.append({
                "role": "tool",
                "name": tool.function.name,
                "content": str(res)
            })

    # 3-4. 再次呼叫 ollama.chat() 讓模型讀取工具執行結果並產出最終回答
    final_response = ollama.chat(model="qwen2.5:7b", messages=messages)
    print(f"💡 AI 最終解答: {final_response.message.content}")
```

> **💡 規範差異提醒 (Ollama 原生 vs OpenAI Standard API)**：  
> 1. **無須 `id` 與 `tool_call_id`**：Ollama 原生 API 格式極致精簡，`tool_calls` 內只有 `function: {name, arguments}`，**完全不需要** `id` 或 `tool_call_id` 來做對接。  
> 2. **`role="tool"` 的標籤方式**：寫回工具結果時，Ollama SDK 透過 `"name": "add_two_numbers"` 讓模型精準識別結果來源。

> **💡 Tool Use 傳參與回傳防崩潰檢查**：  
> 若模型判斷該對話不需要呼叫工具（純文字回答），`response.message.tool_calls` 的值為 `None`。存取前務必先進行 `if response.message.tool_calls:` 檢查，避免觸發 `TypeError`！

> **💡 核心觀念：串流模式 (`stream=True`) 下 `message.tool_calls` 的出現時機與處理**：  
> 1. **出現時機**：在流式迭代 `for chunk in stream:` 的過程中，若模型判斷需要調用工具，`chunk.message.content` 通常會保持為 `""`（不輸出文字），而 **`chunk.message.tool_calls` 會在模型完成工具判定與引數建構的「特定 chunk」（通常是觸發 Tool Call 的該幀片段）中一次性出現**。若模型判定需要同時呼叫多個工具，這些工具會同時打包存在同一個 chunk 的 `tool_calls` 列表內回傳。  
> 2. **Ollama SDK 底層解析優勢**：在 OpenAI API 串流中 `tool_calls` 的參數是分段 (delta) 拼湊的；但在 **Ollama Python SDK** 中，當 `chunk.message.tool_calls` 出現時，`call.function.arguments` 已在底層自動被完整解析為字典 `dict`，可以直接提取解包使用！  
> 3. **串流捕獲判定寫法**：  
> ```python
> stream = ollama.chat(model="qwen2.5:7b", messages=messages, tools=[add_two_numbers], stream=True)
> 
> for chunk in stream:
>     # 時機 A：若有文字內容則實時印出打字機效果
>     if chunk.message.content:
>         print(chunk.message.content, end="", flush=True)
>     
>     # 時機 B：若抵達工具調用 Chunk，觸發 Tool 執行
>     if chunk.message.tool_calls:
>         for call in chunk.message.tool_calls:
>             print(f"\n🛠️ [串流觸發] 收到工具呼叫請求: {call.function.name}")
>             print(f"📦 參數內容: {call.function.arguments}")
> ```

---

# 實戰綜合範例：100% 離線隱私型彩色 Terminal AI 助手

以下結合 [[專案準備#Step 2：建立與管理虛擬環境 (使用現代極速 uv)|uv]] + `ollama` SDK + [[Rich#1. Rich Console (控制台大腦)|Rich 美化]]，打造一個完全離線、不花半毛錢、隱私滿分的個人控制台小助手：

```python
# local_ai_assistant.py
import sys
import ollama
from rich.console import Console
from rich.panel import Panel

console = Console()

MODEL_NAME = "qwen2.5:7b"

# 1. 印出歡迎面板
console.print(Panel(
    f"[bold green]🔒 本地離線 AI 助手已啟動 (模型: {MODEL_NAME})[/bold green]\n"
    f"[dim]所有對話 100% 保存在本機，輸入 'quit' 或 'exit' 離開[/dim]",
    border_style="green"
))

# 2. 初始化對話歷史
messages = [
    {"role": "system", "content": "你是一位親切且富有條理的 AI 助手，請用繁體中文回答，並使用簡明扼要的格式。"}
]

while True:
    try:
        user_input = console.input("\n[bold cyan]You > [/bold cyan]")
        if user_input.strip().lower() in ["quit", "exit"]:
            console.print("[dim]離線對話已結束。[/dim]")
            break
        
        # 紀錄使用者輸入
        messages.append({"role": "user", "content": user_input})
        
        console.print(f"[bold magenta]{MODEL_NAME} > [/bold magenta]", end="")
        
        # 串流發送對話
        stream = ollama.chat(model=MODEL_NAME, messages=messages, stream=True)
        
        full_response = ""
        for chunk in stream:
            content = chunk["message"]["content"]
            full_response += content
            console.print(content, end="", flush=True)
        console.print()
        
        # 將 AI 的回覆更新進對話歷史
        messages.append({"role": "assistant", "content": full_response})
        
    except KeyboardInterrupt:
        console.print("\n[dim]程式已被使用者手動中斷。[/dim]")
        break
    except Exception as e:
        console.print(f"\n[bold red]發生錯誤（請確定 Ollama 服務已啟動）：{e}[/bold red]")
        break
```
