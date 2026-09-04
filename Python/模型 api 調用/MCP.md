# 概念與原理

## 什麼是 MCP (Model Context Protocol)？

**MCP (Model Context Protocol，模型上下文協議)** 是由 Anthropic 開源並迅速成為全球 AI 產業標準的**開放式跨系統通訊協議**。

它旨在為**大語言模型 (LLM / AI Agent)** 與**外部數據來源、本地工具、API 服務**之間建立一條安全、標準化且即插即用的雙向通訊管道。

無論是 Claude Desktop、Cursor、Google Antigravity、VS Code 還是自建的 AI Agent，只要支援 MCP，就能瞬間獲得存取本機檔案、執行資料庫查詢、操作 GitHub、調用瀏覽器等無限擴充能力！

> **🍿 生動白話比喻**：  
> - **過去的 Tool Calling**：就像每買一家廠商的手機（Claude、OpenAI、Gemini），就必須為它買一條專用充電線；想連 GitHub 寫一套、想連 SQL 資料庫又得重寫一套。  
> - **現代的 MCP 協議**：就像 **AI 界的「Type-C (USB-C) 統一標準連接埠」**！任何硬體（資料庫、Git、本機檔案）只要做成一個 MCP Server，所有 AI 客戶端（Claude、Cursor、Agent）只要插上 Type-C 線，就能全自動辨識並直接調用！

---

## MCP 的 3 大核心支柱 (Primitives)

MCP 將 AI 與外界互動的所有能力精準抽象為三大支柱：

```text
                        ┌── 1. Tools (工具)     ➔ LLM 主動呼叫執行 (有副作用/動作)
┌──────────────┐        ├── 2. Resources (資源) ➔ 類似檔案/數據來源 (只讀上下文)
│  MCP Client  │ ◄════► └── 3. Prompts (提示詞) ➔ 預設工作流引導範本 (結構化模板)
│ (Claude/IDE) │               ▲
└──────────────┘               │ (透過 Stdio 或 SSE 通訊)
                               ▼
                    ┌─────────────────────┐
                    │     MCP Server      │
                    │ (MCPServer/FastMCP) │
                    └─────────────────────┘
```

| 核心支柱 | 類比 | 核心功能與特徵 | 典型實戰範例 |
| :--- | :--- | :--- | :--- |
| **1. Tools (工具)** | **「動作與技能」** | 由 LLM 主動決定何時呼叫，可傳入參數、可執行操作並產生副作用 (Actions)。 | 查詢 SQL 資料庫、發送 Slack 訊息、執行 Bash 指令 |
| **2. Resources (資源)** | **「檔案與資料庫」** | 類似 REST API 或檔案系統，提供只讀數據 (Read-only Data)，供 Client 掛載給 LLM。 | 讀取專案日誌 `file:///logs/app.log`、讀取資料庫 Schema |
| **3. Prompts (提示詞)** | **「劇本與範本」** | 預先定義好帶參數的提示詞模板，引導使用者與 LLM 進行高品質特定任務對話。 | `code_review(language, code)`、`debug_traceback()` |

---

## 傳輸通訊機制：Stdio vs SSE

MCP 支援兩種主流的通訊架構：

1. **Stdio (標準輸入輸出 - Local 本地模式 ⭐ 最常用)**：
   - Client 直接啟動 Python 子進程，雙方透過本機的 `stdin` (標準輸入) 與 `stdout` (標準輸出) 傳遞 JSON-RPC 訊息。
   - **優點**：極致安全、零延遲、無需監聽任何網路連接埠（Claude Desktop / Cursor 預設模式）。
2. **SSE (Server-Sent Events / HTTP - Remote 遠端模式)**：
   - MCP Server 作為獨立的 Web 伺服器運行，透過 HTTP POST 與 SSE 進行跨網路/跨容器通訊。
   - **優點**：支援雲端集中部署、微服務架構與多客戶端共享。

---

## 📚 Python MCP 官方 SDK 匯入語法與版本演進

在官方 Python SDK 中，伺服器核心類別歷經了官方規範統一：

| 版本規格 | 套件安裝指令 | 官方標準匯入語法 (Import Statement) | 說明與現況 |
| :--- | :--- | :--- | :--- |
| **官方最新標準 ⭐<br>(SDK v2.0+)** | `pip install "mcp[cli]"`<br>`uv add "mcp[cli]"` | **`from mcp.server.mcpserver import MCPServer, Context`**<br>*(或 `from mcp.server import MCPServer`)* | **Anthropic 官方最新核心標準**！官方將高階裝飾器架構正式統一命名為 `MCPServer` |
| **官方過渡相容別名<br>(FastMCP Alias)** | `pip install "mcp[cli]"` | **`from mcp.server.fastmcp import FastMCP`** | 官方早期名稱（`FastMCP` 是 `MCPServer` 的相容別名，API 完全通用） |
| **社群獨立擴充包<br>(Standalone)** | `pip install fastmcp` | **`from fastmcp import FastMCP`** | Prefect 社群額外維護的第三方獨立封裝包 |

> **💡 核心結論**：  
> `MCPServer` 與 `FastMCP` 的內部裝飾器語法（`@mcp.tool()`、`@mcp.resource()`、`@mcp.prompt()`、`mcp.run()`）**100% 完全相同**！最新專案建議直接採用官方最新規範 **`from mcp.server.mcpserver import MCPServer`**。

---

# 核心 API 類別與方法字典對照表

以 Anthropic 官方最新標準 (**`mcp.server.mcpserver`**) 進行完整解析：

| 類別 / 方法名稱 | 型態 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- | :--- |
| **[[#MCPServer 伺服器初始化\|MCPServer]]** | 類別 | **建立並初始化 MCP 伺服器實例** | 所有 MCP 服務的統一建構起點 |
| **[[#@mcp.tool() 定義與註冊工具\|@mcp.tool()]]** | 裝飾器 | **將 Python 函式註冊為 LLM 可調用的 Tool** | 提供算術、查庫、打 API 等功能給模型 |
| **[[#@mcp.resource() 定義與暴露資料資源\|@mcp.resource()]]** | 裝飾器 | **將資料或檔案註冊為可讀取之 Resource URI** | 暴露專案設定、系統狀態或日誌檔案 |
| **[[#@mcp.prompt() 定義可重用提示詞範本\|@mcp.prompt()]]** | 裝飾器 | **建立帶有參數的結構化對話 Prompt 範本** | 封裝代碼審查、重構或翻譯的工作流範本 |
| **[[#Context 運行時上下文與進度日誌\|Context]]** | 類別 | **在 Tool 執行期間發送日誌、報告進度與存取 Session** | 長時間耗時任務回報進度 (`ctx.info`) |
| **[[#mcp.run() 啟動伺服器服務\|mcp.run()]]** | 物件方法 | **啟動 MCP 伺服器並監聽請求 (預設 stdio)** | 程式進入點執行伺服器 |
| **[[#stdio_client() 與 ClientSession 自建客戶端\|stdio_client()]]** | 協程函式 | **建立與 Stdio MCP Server 的本地通訊管道** | 自行撰寫 Python Agent 客戶端連線 MCP |
| **[[#stdio_client() 與 ClientSession 自建客戶端\|ClientSession]]** | 類別 | **發送 Tool 呼叫、列出資源與讀取 Prompt 的客戶端會話** | 管理 Client 端的請求生命週期 |

---

# 核心功能與語法大解密

### 📦 官方套件安裝指南

安裝 Anthropic 官方提供的 `mcp` SDK：

```shell
# 現代 uv 方式 (極速推薦)
uv add "mcp[cli]"

# 或傳統 pip 方式
pip install "mcp[cli]"
```

---

##### MCPServer 伺服器初始化

- **使用時機**：建立 MCP Server 的核心實例，並為伺服器命名與配置相依套件。
- **語法**：`mcp = MCPServer(name="ServerName", dependencies=[...])`
- **參數說明**：
  - `name`：伺服器名稱（會顯示在 Claude Desktop / Cursor 等客戶端中）。
  - `dependencies`：伺服器依賴的第三方 Python 套件清單（可選）。
- **回傳值**：`MCPServer` 實例物件。

```python
from mcp.server.mcpserver import MCPServer

# 初始化名為 "SystemOps" 的 MCP 伺服器
mcp = MCPServer("SystemOps")
```

#### 📊 MCPServer 實例物件核心成員全覽表

當你執行 `mcp = MCPServer("MyServer")` 建立實例後，該 `mcp` 物件自帶以下核心屬性、裝飾器與操作方法：

| 成員名稱 | 類型 | 回傳型別 / 資料型態 | 核心功能與意義 | 典型實戰用途 |
| :--- | :--- | :--- | :--- | :--- |
| **`mcp.name`** | **實例屬性** | `str` | **伺服器註冊名稱** | 識別該 Server 的唯一名稱（顯示於 Claude / Cursor） |
| **`mcp.dependencies`** | **實例屬性** | `list[str]` | **相依第三方套件清單** | 記錄伺服器運行所需額外安裝的 Python 套件 |
| **[[#1. mcp.settings 全域組態設定物件\|mcp.settings]]** | **複合屬性** | `Settings` | **伺服器全域組態設定** | 調整日誌等級、超時時間等系統運行參數 |
| **[[#2. mcp._tool_manager 工具底層管理器\|mcp._tool_manager]]** | **底層複合屬性** | `ToolManager` | **工具核心底層管理器** | 掌管所有 Tool 的註冊、Schema 生成、參數校驗與調用執行 |
| **[[#3. mcp._resource_manager 資源底層管理器\|mcp._resource_manager]]** | **底層複合屬性** | `ResourceManager` | **資源核心底層管理器** | 掌管所有 Resource 的 URI 模板匹配、動態讀取與分發 |
| **[[#4. mcp._prompt_manager 提示詞底層管理器\|mcp._prompt_manager]]** | **底層複合屬性** | `PromptManager` | **提示詞核心底層管理器** | 掌管所有 Prompt 模板的註冊、參數填充與渲染 |
| **`@mcp.tool()`** | **裝飾器方法** | `Callable` | **註冊 Python 函式為 Tool** | 自動依 Type Hints 與 Docstring 產生 Tool Schema |
| **`@mcp.resource()`** | **裝飾器方法** | `Callable` | **註冊資料/檔案為 Resource** | 透過靜態或動態 URI 暴露唯讀上下文數據 |
| **`@mcp.prompt()`** | **裝飾器方法** | `Callable` | **註冊結構化 Prompt 範本** | 封裝可重用的任務工作流對話模板 |
| **`mcp.run()`** | **運作方法** | `None` | **啟動伺服器並進入事件監聽** | 支援 `transport="stdio"` 或 `transport="sse"` |
| **`mcp.add_tool()`**     | **操作方法** | `None` | **以函式呼叫方式動態註冊 Tool** | 適合無法使用裝飾器的動態批次註冊場景 |
| **`mcp.add_resource()`** | **操作方法** | `None` | **以手動方式註冊 Resource** | 動態批次註冊多個資源端點物件 |
| **`mcp.add_prompt()`**   | **操作方法** | `None` | **以手動方式註冊 Prompt** | 動態載入外部 Prompt 模板檔 |
| **`mcp.get_context()`**  | **操作方法** | `Context` | **獲取當前請求的上下文物件** | 在非同步函式外部手動取得當前 Context |

#### 💻 MCPServer 屬性與操作示範代碼

```python
from mcp.server.mcpserver import MCPServer

# 1. 建立伺服器實例並配置相依
mcp = MCPServer(name="SystemDevOps", dependencies=["httpx", "rich"])

# 2. 讀取實例屬性 (免加括號)
print(f"伺服器名稱: {mcp.name}")           # 輸出: SystemDevOps
print(f"相依套件:   {mcp.dependencies}")   # 輸出: ['httpx', 'rich']

# 3. 手動註冊 Tool (除裝飾器外，亦支援 add_tool 動態註冊)
def dynamic_ping() -> str:
    """ 檢查伺服器連線狀態 """
    return "pong"

# 手動將函式登記到 mcp 實例中
mcp.add_tool(dynamic_ping, name="ping_service", description="檢測服務存活狀態")
```

---

### 🧩 複合屬性深度二層展開 (Sub-attributes & Sub-methods)

當一個屬性本身是「管理物件」或「配置物件」時，它內部包含專屬的**子屬性 (Sub-attributes)** 與**子方法 (Sub-methods)**：

#### 1. mcp.settings 全域組態設定物件

`mcp.settings` 封裝了該伺服器的底層執行緒、超時與日誌行為：

| 子屬性名稱 | 型態 | 預設值 | 功能說明與用途 |
| :--- | :--- | :--- | :--- |
| **`mcp.settings.debug`** | `bool` | `False` | 是否開啟除錯模式（印出詳細協議日誌） |
| **`mcp.settings.log_level`** | `str` | `"INFO"` | 伺服器日誌過濾等級（如 `"DEBUG"`, `"INFO"`, `"ERROR"`） |
| **`mcp.settings.timeout`** | `float` | `30.0` | 工具執行與網路請求的最大超時秒數 |

---

#### 2. mcp._tool_manager 工具底層管理器

`mcp._tool_manager` 是負責工具註冊、Schema 轉換與分發調用的核心大腦：

| 子成員名稱                                | 類型      | 回傳型別              | 功能說明與用途                         |
| :----------------------------------- | :------ | :---------------- | :------------------------------ |
| **`mcp._tool_manager._tools`**       | **子屬性** | `dict[str, Tool]` | 保存所有已註冊 `Tool` 實例的內部字典          |
| **`mcp._tool_manager.add_tool()`**   | **子方法** | `Tool`            | 將 Python 函式解析並包裝為 `Tool` 物件加入字典 |
| **`mcp._tool_manager.get_tool()`**   | **子方法** | `Tool \| None`    | 根據工具名稱檢索已登記的工具實例                |
| **`mcp._tool_manager.list_tools()`** | **子方法** | `list[Tool]`      | 匯出符合 MCP 協議的所有工具 Schema 清單      |
| **`mcp._tool_manager.call_tool()`**  | **子方法** | `list[Content]`   | 解析 JSON 參數、注入 Context 並執行實體函式   |

```python
# 🔍 透過 _tool_manager 檢視底層 Schema
tool_obj = mcp._tool_manager._tools["ping_service"]
print(f"工具描述: {tool_obj.description}")
print(f"參數規格: {tool_obj.parameters}")
```

---

#### 3. mcp._resource_manager 資源底層管理器

`mcp._resource_manager` 負責 URI 路由匹配與靜態/動態資料的讀取分發：

| 子成員名稱 | 類型 | 回傳型別 | 功能說明與用途 |
| :--- | :--- | :--- | :--- |
| **`mcp._resource_manager._resources`** | **子屬性** | `dict[str, Resource]` | 靜態 URI 資源字典（Key 為完整 URI） |
| **`mcp._resource_manager._templates`** | **子屬性** | `list[ResourceTemplate]` | 動態參數化 URI 模板清單（如 `db://{table}`） |
| **`mcp._resource_manager.add_resource()`** | **子方法** | `Resource` | 註冊靜態資源或動態模板 |
| **`mcp._resource_manager.list_resources()`** | **子方法** | `list[Resource]` | 匯出當前所有可讀取之資源描述清單 |
| **`mcp._resource_manager.read_resource()`** | **子方法** | `list[Content]` | 依據傳入的 URI 路由至對應函式並讀取資料 |

---

#### 4. mcp._prompt_manager 提示詞底層管理器

`mcp._prompt_manager` 負責提示詞模板的參數注入與渲染：

| 子成員名稱 | 類型 | 回傳型別 | 功能說明與用途 |
| :--- | :--- | :--- | :--- |
| **`mcp._prompt_manager._prompts`** | **子屬性** | `dict[str, Prompt]` | 保存所有已登記 Prompt 模板的內部字典 |
| **`mcp._prompt_manager.add_prompt()`** | **子方法** | `Prompt` | 註冊對話提示詞模板 |
| **`mcp._prompt_manager.list_prompts()`** | **子方法** | `list[Prompt]` | 匯出所有可用的 Prompt 模板選單 |
| **`mcp._prompt_manager.get_prompt()`** | **子方法** | `GetPromptResult` | 帶入參數渲染出最終的完整對話 Message 序列 |

---

##### @mcp.tool() 定義與註冊工具

- **使用時機**：將 Python 函式暴露給 LLM 作為可調用的工具。**SDK 會全自動依據函式的「型別註記 (Type Hints)」與「Docstring (函式說明)」產生精準的 JSON Schema**！
- **語法**：
  ```python
  @mcp.tool()
  def tool_name(param: type) -> return_type:
      """ 工具描述文字（LLM 讀取此處來決定何時使用） """
      ...
  ```

```python
from mcp.server.mcpserver import MCPServer

mcp = MCPServer("CalculatorHelper")

@mcp.tool()
def add_numbers(a: float, b: float) -> float:
    """ 計算兩數相加的結果。
    
    Args:
        a: 第一個數字
        b: 第二個數字
    """
    return a + b

@mcp.tool()
def query_user_info(user_id: int) -> dict:
    """ 根據使用者 ID 查詢使用者詳細檔案。 """
    # 模擬資料庫查詢
    mock_db = {101: {"name": "Matthew", "role": "Engineer", "active": True}}
    return mock_db.get(user_id, {"error": "User not found"})
```

> **⚠️ 型別註記與 Docstring 鐵律**：  
> MCP 依賴 Python 的型別註記（如 `int`, `str`, `list[str]`）與 Docstring 來自動產生 LLM 理解的規格說明。**型別註記與 Docstring 不可省略，否則 LLM 會無法理解如何正確傳參！**

---

##### @mcp.resource() 定義與暴露資料資源

- **使用時機**：當你需要提供靜態或動態的上下文資料（例如檔案內容、系統環境、資料庫結構），供使用者或 LLM **以唯讀方式附加到對話中**時使用。
- **語法**：`@mcp.resource(uri_pattern)`
- **URI 規範**：支援靜態 URI（如 `config://app`）或動態參數化 URI（如 `users://{user_id}/profile`）。

```python
from mcp.server.mcpserver import MCPServer
import platform

mcp = MCPServer("SystemMonitor")

# 1. 靜態資源：提供系統即時環境資訊
@mcp.resource("system://info")
def get_system_info() -> str:
    """ 提供當前主機的作業系統與硬體平台資訊 """
    return f"OS: {platform.system()} | Release: {platform.release()} | Arch: {platform.machine()}"

# 2. 動態參數化資源：根據傳入的 project_name 讀取設定
@mcp.resource("projects://{project_name}/readme")
def get_project_readme(project_name: str) -> str:
    """ 動態讀取指定專案的 README 說明文件 """
    return f"# {project_name} 專案概覽\n這是一個由 MCP 自動提供的動態專案文檔。"
```

---

##### @mcp.prompt() 定義可重用提示詞範本

- **使用時機**：將複雜、專業的 Prompt 工程模板預先封裝在 Server 中。Client 介面中會出現選單，使用者點擊即可填入參數並一鍵產生完整對話上下文。
- **語法**：`@mcp.prompt()`

```python
from mcp.server.mcpserver import MCPServer

mcp = MCPServer("AssistantTools")

@mcp.prompt()
def code_review_prompt(language: str, code: str) -> str:
    """ 生成標準的代碼審查 (Code Review) 提示詞模板 """
    return f"""你是一位世界頂級的資深 {language} 架構師。
請對以下程式碼進行徹底的 Code Review：
1. 分析潛在的記憶體洩漏或效能瓶頸
2. 檢查邊界條件處理與異常防禦
3. 提供重構後的乾淨代碼

【待審查程式碼】：
```{language}
{code}
```
---

##### Context 運行時上下文與進度日誌

- **使用時機**：當 Tool 執行需要較長時間（如爬蟲、大檔處理），需要向 Client 發送**即時日誌訊息 (`ctx.info`)** 或**即時進度條 (`ctx.report_progress`)** 時使用。
- **使用方式**：在 Tool 函式參數中宣告 `ctx: Context`，MCP 會全自動注入上下文實例！

```python
from mcp.server.mcpserver import MCPServer, Context
import time

mcp = MCPServer("BatchProcessor")

@mcp.tool()
async def process_batch_files(files: list[str], ctx: Context) -> str:
    """ 批次處理檔案並向 Client 回報即時進度 """
    total = len(files)
    ctx.info(f"開始處理 {total} 個檔案...")
    
    for idx, filename in enumerate(files):
        # 報告進度 (當前進度, 總量)
        await ctx.report_progress(idx + 1, total)
        ctx.info(f"正在處理: {filename} ({idx + 1}/{total})")
        time.sleep(0.5)  # 模擬耗時運算
        
    return f"🎉 成功完成 {total} 個檔案的批次處理！"
```

---

##### mcp.run() 啟動伺服器服務

- **使用時機**：在主程式進入點啟動 MCP 監聽。
- **語法**：`mcp.run(transport="stdio")`
- **參數說明**：
  - `transport`：傳輸協定，可選 `"stdio"`（本機標準輸入輸出，預設）或 `"sse"`（網路 HTTP/SSE 模式）。

```python
if __name__ == "__main__":
    # 啟動 Stdio 伺服器（等待 Claude Desktop / Cursor 來連接）
    mcp.run()
```

> **🚨 Stdio 通訊致命天條：嚴禁在程式中使用 `print()`**：  
> 當使用 Stdio 模式時，進程的 `stdout` 是專門用來傳輸 JSON-RPC 協議封包的。若你在代碼中隨手寫了 `print("除錯訊息")`，會直接污染通訊數據流，導致 Client 解析 JSON 失敗而崩潰！  
> **鐵律**：輸出日誌請一律使用 `sys.stderr.write()` 或透過 `ctx.info()`！

---

##### stdio_client() 與 ClientSession 自建客戶端

- **使用時機**：當你要自己撰寫 Python 程式（如自訂 Agent 系統）去連接並調用任何現有的 MCP Server 時使用。

```python
import asyncio
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def main():
    # 1. 定義要啟動的 MCP Server 子進程參數
    server_params = StdioServerParameters(
        command="python",
        args=["my_mcp_server.py"],
        env=None
    )

    # 2. 建立連線並初始化 Session
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()
            
            # 3. 查詢該 Server 提供哪些 Tools
            tools = await session.list_tools()
            print(f"找到的可用工具: {[t.name for t in tools.tools]}")
            
            # 4. 調用指定工具
            result = await session.call_tool("add_numbers", arguments={"a": 10, "b": 25})
            print(f"執行結果: {result.content[0].text}")

if __name__ == "__main__":
    asyncio.run(main())
```

---

# 完整專案實戰：SQLite 智慧資料庫 MCP Server

以下展示一個工業級標準的完整 MCP Server 腳本，提供 SQLite 資料庫的 **資料表結構檢視 (Resource)** 與 **安全 SQL 查詢執行 (Tool)**：

```python
# sqlite_mcp_server.py
import sqlite3
from pathlib import Path
from mcp.server.mcpserver import MCPServer, Context

# 1. 建立 MCP 實例
mcp = MCPServer("SQLiteExplorer")
DB_PATH = Path("demo.db")

# 初始化測試資料庫
def init_db():
    conn = sqlite3.connect(DB_PATH)
    cursor = conn.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS products (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            price REAL NOT NULL,
            stock INTEGER NOT NULL
        )
    """)
    cursor.execute("DELETE FROM products")
    cursor.executemany("""
        INSERT INTO products (name, price, stock) VALUES (?, ?, ?)
    """, [
        ("MacBook Pro M3", 59900.0, 15),
        ("iPhone 16 Pro", 36900.0, 42),
        ("AirPods Pro 2", 7490.0, 80)
    ])
    conn.commit()
    conn.close()

init_db()

# 2. 定義 Resource：唯讀提供資料庫目前所有的資料表清單與欄位結構
@mcp.resource("db://schema")
def get_db_schema() -> str:
    """ 取得資料庫中所有資料表的結構定義 (Schema) """
    conn = sqlite3.connect(DB_PATH)
    cursor = conn.cursor()
    cursor.execute("SELECT sql FROM sqlite_master WHERE type='table'")
    schemas = [row[0] for row in cursor.fetchall() if row[0] is not None]
    conn.close()
    return "\n\n".join(schemas)

# 3. 定義 Tool：供 LLM 執行唯讀 SQL 查詢
@mcp.tool()
def execute_sql_query(query: str, ctx: Context) -> str:
    """ 執行 SELECT 查詢並回傳格式化資料結果。
    
    Args:
        query: 要執行的 SQL SELECT 語法 (例如: SELECT * FROM products WHERE price > 10000)
    """
    ctx.info(f"正在執行 SQL: {query}")
    
    # 安全防禦：防止破壞性寫入指令
    clean_query = query.strip().upper()
    if not clean_query.startswith("SELECT"):
        return "❌ 安全限制：此工具僅允許執行 SELECT 查詢！"
    
    try:
        conn = sqlite3.connect(DB_PATH)
        cursor = conn.cursor()
        cursor.execute(query)
        rows = cursor.fetchall()
        columns = [desc[0] for desc in cursor.description] if cursor.description else []
        conn.close()
        
        if not rows:
            return "查詢結果為空 (0 筆資料)。"
            
        # 格式化輸出
        header = " | ".join(columns)
        divider = "-" * len(header)
        body = "\n".join([" | ".join(str(val) for val in row) for row in rows])
        return f"{header}\n{divider}\n{body}"
    except Exception as e:
        return f"❌ SQL 執行錯誤: {str(e)}"

# 4. 啟動服務
if __name__ == "__main__":
    mcp.run()
```

---

# 連接 Claude Desktop / Cursor 設定教學

將編寫好的 MCP Server 掛載到 **Claude Desktop** 或 **Cursor** 中，只需配置一個簡單的 JSON 檔案：

### 1. 找到設定檔路徑
- **macOS (Claude Desktop)**：`~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows (Claude Desktop)**：`%APPDATA%\Claude\claude_desktop_config.json`

### 2. 編寫設定檔內容 (`claude_desktop_config.json`)

```json
{
  "mcpServers": {
    "sqlite_explorer": {
      "command": "uv",
      "args": [
        "run",
        "--with",
        "mcp[cli]",
        "python",
        "/Users/matthew/projects/sqlite_mcp_server.py"
      ]
    }
  }
}
```

重啟 Claude Desktop 後，聊天視窗右下角就會出現 **🔨 工具小鐵鎚圖示**，Claude 就能主動查詢你的 SQLite 資料庫了！

---

# 實戰除錯與核心天條

## 1. Stdio 模式下隨意 print 導致通訊協議中斷

> **⚠️ stdout 污染天條**：  
> Stdio 傳輸依賴標準輸出 (`sys.stdout`) 傳遞 JSON-RPC 封包。任何額外的 `print("hello")` 都會使 JSON 格式損壞，導致 Client 報出 `JSONDecodeError` 斷線崩潰！  
> **鐵律**：除錯訊息請一律使用 `ctx.info()`、`ctx.warning()` 或輸出至 `sys.stderr`！

---

## 2. 函式未寫型別註記導致 Schema 生成失敗

> **⚠️ 嚴格 Type Hints 天條**：  
> MCPServer 會在後台檢查函式簽名，若缺少參數型別註記（如 `def search(query):`），會無法產生 JSON Schema 或直接拋出初始化異常。  
> **鐵律**：每個 Tool 參數與回傳值**必須具備完整型別標註**（如 `def search(query: str) -> list[str]:`）！

---

## 3. 缺乏詳細 Docstring 導致 LLM 幻覺誤用工具

> **⚠️ 工具說明文檔天條**：  
> LLM 決定「要不要用這個工具」與「參數該填什麼」，完全取決於函式的 Docstring！如果沒有 Docstring，LLM 就如同盲人摸象。  
> **鐵律**：為每個 Tool 撰寫清晰的繁體中文 Docstring，詳述功能、參數含義與邊界條件！
