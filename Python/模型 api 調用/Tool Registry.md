# 概念與原理

## 什麼是 Tool Registry 工具註冊表？

**Tool Registry (工具註冊表)** 是現代 **AI Agent (智能體)** 與 **大語言模型 (LLM)** 進行 **Tool Use / Function Calling** 時的核心架構模式。

當我們讓 LLM（如 Gemini、OpenAI 或 Local Ollama）呼叫外部 Python 函式時，LLM 本身並無法直接執行程式碼，它只會輸出包含「工具名稱」與「參數」的 JSON 文字（例如：`{"name": "get_weather", "arguments": {"location": "台北"}}`）。

`Tool Registry` 的作用就是作為**全域中央調度中心**：
1. 自動將 Python 函式登記到清單字典中。
2. 自動產生 LLM 理解的工具描述格式 (Tool Schema)。
3. 當 LLM 傳回工具名稱時，精準尋找到對應的 Python 函式並安全執行！

> **🍿 生動白話比喻**：  
> LLM 像**「坐在辦公室裡的指揮官」**，它不會親自下場搬東西，只會在紙條上記錄：「請使用 `get_weather` 工具，地點是台北」。  
> `Tool Registry` 就像**「自動工具箱服務台」**。服務台收到紙條後，根據名字從工具箱取出實體工具 (`func`)，幫指揮官執行完畢後把結果傳回！

---

# 核心架構與 3 大運作步驟

使用工具註冊表模式開發 AI Agent 的標準流程包含 3 大步驟：

```text
[ Python 函式 ] ──(@register_tool 裝飾器)──> [ TOOL_REGISTRY 字典 ]
                                                     │
┌────────────────────────────────────────────────────┘
│
├── 1. 導出工具描述 JSON (Schema) ➔ 傳給 LLM 大模型
└── 2. 接收 LLM 回傳的工具調用請求 ➔ 從字典檢索 func 並執行
```

---

## 步驟 1：定義註冊器裝飾器與全域字典

使用 Python 裝飾器 `@register_tool`，只需在函式頭頂戴上帽子，就能自動完成註冊：

```python
# 1. 全域工具註冊字典 (Key: 函式名稱, Value: 函式物件本人)
TOOL_REGISTRY = {}

# 2. 裝飾器函式
def register_tool(func):
    """ 只要戴上 @register_tool 帽子的函式，人都會被自動放入 TOOL_REGISTRY 字典中 """
    TOOL_REGISTRY[func.__name__] = func
    return func  # 原封不動回傳原函式
```

---

## 步驟 2：使用裝飾器自動標註 Python 工具函式

在任何需要開放給 AI Agent 使用的 Python 函式上記述：

```python
@register_tool
def get_current_weather(location: str):
    """ 查詢指定地區的即時天氣狀況 """
    return f"{location} 今天晴空萬里，氣溫 25°C"

@register_tool
def calculate_tax(amount: float):
    """ 計算輸入金額的標準稅額 """
    return amount * 0.05
```

---

## 步驟 3：接收 LLM 請求並動態執行 (Dynamic Execution)

當 LLM 傳回工具呼叫 JSON 時，從 `TOOL_REGISTRY` 檢索出 `func` 並傳入參數執行：

```python
def execute_tool_call(tool_name: str, arguments: dict):
    """ 接收 LLM 的工具呼叫請求並執行 """
    # 1. 檢查工具是否在註冊表中
    if tool_name not in TOOL_REGISTRY:
        return f"❌ 錯誤：找不到名為 '{tool_name}' 的已註冊工具！"
    
    # 2. 從字典取出函式物件本人
    func = TOOL_REGISTRY[tool_name]
    
    # 3. 使用 **arguments 動態傳參並執行
    try:
        result = func(**arguments)
        return result
    except Exception as e:
        return f"❌ 工具執行失敗: {str(e)}"
```

---

# 完整 AI Agent 工具註冊實戰範例

以下展示一個包含 **工具自動註冊、LLM 回傳模擬與動態分發執行** 的完整可執行範例：

```python
import json

# 1. 全域註冊字典
TOOL_REGISTRY = {}

# 2. 註冊裝飾器
def register_tool(func):
    TOOL_REGISTRY[func.__name__] = func
    return func

# 3. 定義業務工具函式
@register_tool
def query_user_balance(user_id: str):
    """ 查詢指定使用者的帳戶餘額 """
    db = {"user_101": 15800, "user_102": 320}
    balance = db.get(user_id, 0)
    return f"使用者 {user_id} 當前餘額為：${balance} TWD"

@register_tool
def send_email_notification(to: str, subject: str):
    """ 寄送 Email 通知給指定使用者 """
    return f"✅ 已成功發送主旨為 '{subject}' 的信件至 {to}"


# 4. 模擬 LLM 大模型傳回的 Tool Call 響應 (JSON 格式)
mock_llm_response = """
{
    "tool_name": "query_user_balance",
    "arguments": {
        "user_id": "user_101"
    }
}
"""

# 5. 主程式：解析 LLM 請求並向註冊表發起調用
if __name__ == "__main__":
    request = json.loads(mock_llm_response)
    name = request["tool_name"]
    args = request["arguments"]
    
    print(f"🤖 [AI 大腦請求] 欲呼叫工具: {name}, 參數: {args}")
    
    # 向註冊表請求執行
    if name in TOOL_REGISTRY:
        output = TOOL_REGISTRY[name](**args)
        print(f"⚙️ [工具執行成果]: {output}")
    else:
        print(f"❌ [錯誤] 未知的工具名稱: {name}")
```

---

# 3 大進階架構防護與技巧

## 1. 錯誤處理與缺工具回退機制 (Fallback)

在正式上線的 Agent 系統中，如果 LLM 傳回了一個不存在的工具名稱，必須給出優雅的回退處理，避免 Agent 崩潰：

```python
def safe_dispatch(tool_name: str, **kwargs):
    func = TOOL_REGISTRY.get(tool_name)
    if not func:
        # 回傳錯誤訊息給 LLM，讓 LLM 嘗試自我修正或改用其他工具
        return f"ToolNotFoundError: Tool '{tool_name}' is not registered. Available tools: {list(TOOL_REGISTRY.keys())}"
    return func(**kwargs)
```

---

## 2. 搭配 `importlib` 實現外掛動態自動載入 (Plugin Auto-loading)

不需要手動 `import` 所有工具檔案！可以建立一個 `tools/` 資料夾，並在系統啟動時用 `importlib` 自動掃描並載入所有模組，此時戴有 `@register_tool` 帽子的函式就會**自動完成註冊**：

```python
import os
import importlib

def auto_load_plugins(plugin_dir="tools"):
    """ 自動掃描並載入 tools/ 資料夾下的所有 Python 工具檔 """
    for file in os.listdir(plugin_dir):
        if file.endswith(".py") and not file.startswith("__"):
            module_name = f"{plugin_dir}.{file[:-3]}"
            importlib.import_module(module_name)
    print(f"✅ 已完成外掛自動載入！目前註冊表包含: {list(TOOL_REGISTRY.keys())}")
```

---

## 3. 字典 Key 與 Value 觀念複習

- **`TOOL_REGISTRY` 字典的 Key** ➔ `func.__name__`（純字串 `str`，如 `'query_user_balance'`）。
- **`TOOL_REGISTRY` 字典的 Value** ➔ `func`（**函式物件本人**，不帶括號的程式碼記憶體實體，拿出來加上 `(**kwargs)` 即可直接執行）。

> **恭喜！** 你已經徹底掌握了 AI Agent / LLM Function Calling 中 `Tool Registry` 工具註冊表模式的完整架構與設計實務！
