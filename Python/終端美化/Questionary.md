# 概念與原理

## 什麼是 Questionary？

Python 社群中最優雅的**命令列互動式問答與表單打造工具 (Interactive CLI Prompts)**（`pip install questionary` 或 `uv add questionary`）。

在編寫命令列工具 (CLI) 或自動化腳本時，傳統內建的 `input()` 只能接受單調的純文字鍵入，無法提供鍵盤方向鍵移動選擇、複選框打勾、密碼遮罩、路徑自動補全或即時輸入驗證。

`questionary` 基於底層強大的 `prompt_toolkit` 構建而成（類似 JavaScript 社群著名的 `Inquirer.js`）。

它讓開發者能以極簡的 Python 語法，在終端機中輕鬆渲染出**單選選單 (Select)**、**多選框 (Checkbox)**、**確認按鈕 (Confirm)**、**密碼遮罩 (Password)**、**自動補全 (Autocomplete)** 以及 **路徑選擇 (Path)**，輕鬆打造媲美 Vite、Vue CLI 等現代前沿工具的互動體驗！

> **🍿 生動白話比喻**：  
> - **傳統 `input()`**：像一張**「毫無防呆的紙本填空考卷」**。填錯了只能拿橡皮擦塗改重寫，打錯一個字整個腳本就崩潰。  
> - **`questionary`**：像終端機裡的**「Google 表單 / 互動問答 App 📝」**。支援鍵盤上下鍵流暢滑動、空白鍵打勾選取、字數即時防呆檢查，把黑白終端機變成現代 GUI 軟體！

---

## 安裝與環境載入

`questionary` 是第三方套件，可透過現代專案工具 `uv` 或標準 `pip` 進行安裝：

```bash
# 推薦使用現代極速 uv
uv add questionary

# 或傳統 pip 安裝
pip install questionary
```

---

# 核心 API 函數字典對照表

| 類別 / 函式名稱 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- |
| **[[#questionary.select() 單選選單\|questionary.select()]]** | **單選選單提示器** | 讓使用者以方向鍵從多個選項中挑選一個 |
| **[[#questionary.text() 文字輸入與即時驗證\|questionary.text()]]** | **單行文字輸入與即時驗證** | 輸入使用者名稱、年齡、自訂文字設定 |
| **[[#questionary.confirm() 二元確認按鈕\|questionary.confirm()]]** | **Yes/No 二元確認按鈕** | 關鍵操作二度確認（如部署、刪除確認） |
| **[[#questionary.checkbox() 多選複選框\|questionary.checkbox()]]** | **多選複選框提示器** | 勾選要安裝的套件、批次挑選目標項目 |
| **[[#questionary.password() 密碼遮罩輸入\|questionary.password()]]** | **密碼與敏感資料遮罩輸入** | 讀取 API Key、資料庫密碼，防止旁人窺視 |
| **[[#questionary.autocomplete() 模糊搜尋與自動補全\|questionary.autocomplete()]]** | **模糊搜尋與自動補全提示器** | 從數百個城市、型號或長清單中快速搜尋 |
| **[[#questionary.path() 檔案與目錄路徑選擇\|questionary.path()]]** | **檔案與資料夾路徑選擇器** | 支援 Tab 鍵自動補全檔案路徑 |
| **[[#questionary.form() 組合多欄位表單工作流\|questionary.form()]]** | **將多個問題組合成完整表單** | 收集結構化資料，一次性回傳結果字典 |
| **[[#questionary.Choice 與 questionary.Separator\|questionary.Choice]]** | **自訂選項標籤、真值與預設勾選狀態** | 分離「顯示名稱」與「程式內部回傳值」 |
| **[[#questionary.Choice 與 questionary.Separator\|questionary.Separator]]** | **在選項清單中插入視覺分割線** | 選單分類美化、分隔不同群組選項 |
| **[[#questionary.Style 主題樣式美化\|questionary.Style]]** | **自訂問答組件的顏色與樣式** | 打造符合品牌色系或 Rich 風格的終端主題 |

---

# 核心功能與語法大解密

## 1. 核心問答提示器 (Core Prompts)

`questionary` 的所有提示器預設都是阻塞式的，呼叫 **`.ask()`** 即可暫停程式並等待使用者互動輸入。

---

##### questionary.select() 單選選單

- **使用時機**：需要使用者在多個固定選項中**僅挑選單一項目**時使用。支援上下方向鍵 (`↑` / `↓`) 或 `J` / `K` 鍵移動，按 `Enter` 確認。
- **語法**：`questionary.select(message, choices, default=None, style=None, ...).ask()`
- **常用參數說明**：
  - `message`：提示使用者的問題字串。
  - `choices`：選項清單，可傳入字串列表 `list[str]` 或 `Choice` / `Separator` 物件列表。
  - `default`：預設選中的選項值或名稱。
  - `use_shortcuts`：布林值。設為 `True` 時，允許直接按數字快捷鍵選取。
- **回傳值**：
  - `Any`：使用者選中的選項值（若按 `Ctrl+C` 中途中斷則回傳 `None`）。

```python
import questionary

# 1. 基礎單選選單
backend = questionary.select(
    "請選擇後端框架：",
    choices=["FastAPI", "Django", "Flask", "Litestar"],
    default="FastAPI"
).ask()

print(f"你選擇了: {backend}")
```

---

##### questionary.text() 文字輸入與即時驗證

- **使用時機**：接收使用者輸入的純文字，並可在鍵入當下進行**即時格式校驗 (Validation)**。
- **語法**：`questionary.text(message, default='', validate=None, ...).ask()`
- **常用參數說明**：
  - `message`：提示文字。
  - `default`：預設填入的文字字串（按 Enter 可直接採用）。
  - `validate`：驗證函式或類別。**若通過回傳 `True`；若失敗回傳錯誤訊息字串 (`str`)**。
- **回傳值**：
  - `str`：使用者輸入的文字字串。

```python
import questionary

# 定義驗證規則：通過回傳 True，失敗回傳錯誤字串
def validate_port(text: str) -> bool | str:
    if not text.isdigit():
        return "❌ 連接埠必須是純數字！"
    port = int(text)
    if not (1 <= port <= 65535):
        return "❌ 連接埠範圍必須介於 1 到 65535 之間！"
    return True

port = questionary.text(
    "請輸入伺服器連接埠 (Port):",
    default="8000",
    validate=validate_port
).ask()

print(f"設定埠號: {port}")
```

---

##### questionary.confirm() 二元確認按鈕

- **使用時機**：需要使用者進行 `(Y/n)` 或 `(y/N)` 的二元確認時使用。支援按 `y`、`n` 或 `Enter`。
- **語法**：`questionary.confirm(message, default=True, auto_enter=True, ...).ask()`
- **常用參數說明**：
  - `message`：確認提示文字。
  - `default`：布林值。設為 `True` 預設選中 `Yes`；設為 `False` 預設選中 `No`。
  - `auto_enter`：布林值。設為 `True` 時，使用者只要一按 `y` 或 `n` 鍵就會立即確認，無須額外按 Enter。
- **回傳值**：
  - `bool`：使用者選擇結果 (`True` 或 `False`)。

```python
import questionary

# 關鍵操作安全二度確認
should_clean = questionary.confirm(
    "確定要清空資料庫並重建所有資料表嗎？",
    default=False
).ask()

if should_clean:
    print("🚨 開始清空資料庫...")
else:
    print("🛡️ 已取消危險操作。")
```

---

##### questionary.checkbox() 多選複選框

- **使用時機**：讓使用者從清單中**自由勾選 0 個、1 個或多個項目**。支援方向鍵移動、`空白鍵 (Space)` 打勾/取消、`A` 鍵全選、`I` 鍵反選。
- **語法**：`questionary.checkbox(message, choices, validate=None, ...).ask()`
- **常用參數說明**：
  - `message`：提示文字。
  - `choices`：選項清單（可搭配 `Choice(..., checked=True)` 預設打勾）。
  - `validate`：驗證函式（例如限制至少勾選一項）。
- **回傳值**：
  - `list[Any]`：所有被勾選項目的值所組成的串列。

```python
import questionary

selected_tools = questionary.checkbox(
    "請勾選需要啟用的開發工具 (可多選)：",
    choices=[
        questionary.Choice("Ruff (程式碼檢查與排版)", checked=True),
        questionary.Choice("pytest (單元測試庫)", checked=True),
        questionary.Choice("mypy (靜態型別檢查)", checked=False),
        questionary.Choice("pre-commit (Git 提交前檢查鉤子)", checked=False)
    ],
    validate=lambda result: True if len(result) >= 1 else "❌ 請至少勾選一項工具！"
).ask()

print(f"已選取工具: {selected_tools}")
```

---

##### questionary.password() 密碼遮罩輸入

- **使用時機**：輸入金鑰、Token 或帳號密碼等敏感機密資料，畫面上會自動以 `****` 遮罩保護。
- **語法**：`questionary.password(message, ...).ask()`
- **回傳值**：
  - `str`：使用者輸入的真實密碼字串。

```python
import questionary

api_token = questionary.password(
    "請輸入你的 GitHub Personal Access Token (內容將被遮蔽):",
    validate=lambda val: True if len(val) >= 8 else "❌ Token 長度至少需 8 個字元！"
).ask()

print("🔐 Token 載入完畢！")
```

---

##### questionary.autocomplete() 模糊搜尋與自動補全

- **使用時機**：當候選清單項目過多（如數百個城市、商品型號、資料表名），使用者一邊鍵入文字時能**即時進行模糊匹配搜尋**。
- **語法**：`questionary.autocomplete(message, choices, ignore_case=True, match_middle=True, ...).ask()`
- **常用參數說明**：
  - `ignore_case`：布林值，是否忽略英文字母大小寫。
  - `match_middle`：布林值，是否支援從單詞中間關鍵字進行匹配。
- **回傳值**：
  - `str`：使用者最終確認挑選的項目字串。

```python
import questionary

cities = ["Taipei", "Tokyo", "New York", "London", "Paris", "Berlin", "Sydney", "Toronto"]

city = questionary.autocomplete(
    "請輸入或搜尋所在城市：",
    choices=cities,
    ignore_case=True,
    match_middle=True
).ask()

print(f"選擇城市: {city}")
```

---

##### questionary.path() 檔案與目錄路徑選擇

- **使用時機**：引導使用者指定磁碟上的檔案或資料夾路徑，支援按 `Tab` 鍵自動補全檔案系統路徑。
- **語法**：`questionary.path(message, default='', only_directories=False, ...).ask()`
- **常用參數說明**：
  - `only_directories`：布林值。設為 `True` 時，限制使用者只能選擇目錄資料夾。
  - `file_filter`：檔案過濾函式（例如只允許選擇 `.csv` 檔案）。
- **回傳值**：
  - `str`：使用者輸入或補全的檔案路徑字串。

```python
import questionary

config_file = questionary.path(
    "請選擇要匯入的設定檔路徑：",
    default="./config.yaml",
    only_directories=False
).ask()

print(f"目標檔案: {config_file}")
```

---

## 2. 表單與選項進階配置 (Form & Customization)

##### questionary.form() 組合多欄位表單工作流

- **使用時機**：將多個問題串聯成**一整套依序執行的表單 (Form)**。
- **語法**：`questionary.form(field1=prompt1, field2=prompt2, ...).ask()`
- **回傳值**：
  - `dict[str, Any]`：以關鍵字參數名稱為 Key，使用者回答為 Value 的字典物件。

```python
import questionary

# 一次性定義完整問卷
profile = questionary.form(
    name=questionary.text("1. 請輸入名稱:", default="Matthew"),
    role=questionary.select("2. 請選擇角色權限:", choices=["Admin", "Developer", "Guest"]),
    notifications=questionary.confirm("3. 是否啟用推播通知？", default=True)
).ask()

# 回傳完整字典: {'name': 'Matthew', 'role': 'Admin', 'notifications': True}
print("📋 建立完成的資料字典:", profile)
```

---

##### questionary.Choice 與 questionary.Separator

- **使用時機**：
  - `Choice`：需要將**「畫面上顯示的文字 (title)」**與**「程式內部實際取得的資料 (value)」**分開時使用。
  - `Separator`：在選單中插入水平分割線或群組標題。

```python
import questionary
from questionary import Choice, Separator

deploy_target = questionary.select(
    "請選擇部署目標環境：",
    choices=[
        Separator("=== 🧪 測試環境 ==="),
        Choice(title="本地端 (Localhost)", value="env_local"),
        Choice(title="測試伺服器 (Staging)", value="env_staging"),
        Separator("=== 🚀 正式環境 ==="),
        Choice(title="正式生產集群 (Production)", value="env_prod")
    ]
).ask()

# 使用者看見的是中文標題，但變數拿到的會是 'env_local'、'env_prod'！
print(f"部署環境代碼: {deploy_target}")
```

---

##### questionary.Style 主題樣式美化 (整合 prompt_toolkit.styles.Style)

- **使用時機**：自訂問答組件的色彩與字體樣式，打造符合品牌色系、Dracula 或 Cyberpunk 風格的終端互動主題。
- **底層原理**：`questionary` 內部完全採用 **`prompt_toolkit.styles.Style`** 作為樣式引擎。你可以直接從 `questionary` 匯入 `Style`，也可以使用 `prompt_toolkit.styles.Style.from_dict()`，兩者 100% 互通！
- **樣式語法規則**：
  - **前景色**：`fg:#hex`（如 `fg:#ff5555`）或 `fg:color_name`（如 `fg:ansigreen`, `fg:darkred`）。
  - **背景色**：`bg:#hex`（如 `bg:#282a36`）或 `bg:color_name`。
  - **字體樣式**：`bold`（粗體）、`italic`（斜體）、`underline`（底線）、`reverse`（反相）、`noinherit`（不繼承預設樣式）。

---

#### 🎨 Questionary 可設定色彩的 12 大 UI 部件 (Style Tokens) 全覽總表

在 Questionary 中，畫面上所有的文字、問號、游標、勾選框與錯誤提示，都分別對應專屬的 **Token 部件選擇器名稱**。下表列出全部可設定的部件名稱、對應 UI 位置以及其**內建預設樣式與顏色 (Default Style & Color)**：

| Token 部件名稱 | 對應 UI 元件 / 狀態部位 | 內建預設樣式與顏色 (Default Style) | 白話功能與視覺效果說明 |
| :--- | :--- | :--- | :--- |
| **`qmark`** | 問題前方的問號前綴標記 (`?`) | `fg:#5f819d bold`<br>*(灰藍色加粗)* | 標示問題的開頭起始符號 |
| **`question`** | 問題題目的主體文字 | `bold`<br>*(終端預設前景色加粗)* | 顯示問題的文字描述 |
| **`answer`** | 使用者確認輸入後的最終答案文字 | `fg:#28a745 bold`<br>*(成功綠色加粗)* | 送出後顯示的回答結果 |
| **`pointer`** | 單選/多選選單中指向目前項目的游標箭頭 (`❯`) | `fg:#6c71c4 bold`<br>*(藍紫色加粗)* | 指示鍵盤當前停駐的焦點項目 |
| **`highlighted`** | 當前被游標焦點選中的選項文字 | `noinherit`<br>*(不繼承，終端反白)* | 醒目顯示鍵盤游標當前聚焦的選項內容 |
| **`selected`** | 複選框 (Checkbox) 中已被選取/打勾的標記與文字 (`◉` / `✔`) | `fg:#28a745 bold`<br>*(成功綠色加粗)* | 標示已被打勾選中的項目 |
| **`separator`** | `Separator` 分隔線符號與分類標題文字 | `fg:#6c71c4`<br>*(藍紫色)* | 分隔不同群組選項的視覺分割線 |
| **`instruction`** | 括號內的即時操作導引提示文字（如 `(Use arrow keys)` / `(Press <space> to select)`） | `fg:#5f819d italic`<br>*(灰藍色斜體)* | 引導使用者如何使用鍵盤操作的說明 |
| **`text`** | 文字輸入框 (`text` / `password`) 中正在鍵入的純文字內容 | `""`<br>*(終端原生文字色)* | 使用者正在打字的文字流 |
| **`disabled`** | 被禁用的選項文字（如 `Choice(disabled="已額滿")`） | `fg:#858585 italic`<br>*(灰色斜體)* | 無法被選取的灰色唯讀選項 |
| **`validation-toolbar`** | 即時驗證失敗時出現在畫面底部的錯誤提示工具列 | `fg:#7f0000 bg:#ffaaaa italic`<br>*(粉紅底深紅字斜體)* | 提示使用者輸入不合規定的具體錯誤原因 |
| **`choice-switch`** | 開啟 `use_shortcuts=True` 時的數字快捷鍵標籤（如 `[1]`, `[2]`） | `fg:#6c71c4 bold`<br>*(藍紫色加粗)* | 數字快捷選取指示符 |

---

#### 🗺️ UI 部件視覺對應地圖 (ASCII 視覺示意)

```text
  [qmark]  [question]              [instruction]
     ?    請選擇要啟動的服務：  (Use arrow keys)
  
  [separator]
  ──────── 🌐 網路服務 ────────
  
  [pointer]  [highlighted]
     ❯       FastAPI 伺服器
             Django 後端
  
  [selected]
     ◉       PostgreSQL 資料庫 (已選取)
  
  [disabled]
     -       Redis 快取 (維護中)
  
  ──────────────────────────────
  [validation-toolbar]  ❌ 錯誤：此伺服器目前不可用！
```

---

#### 💻 自訂主題代碼示範

你可以使用 **元組列表語法 (`questionary.Style`)** 或 **字典語法 (`prompt_toolkit.styles.Style.from_dict`)** 進行自訂：

```python
import questionary
from questionary import Choice, Separator, Style

# 1. 方式 A：使用 questionary.Style (元組清單模式)
cyberpunk_theme = Style([
    ("qmark", "fg:#00ffff bold"),              # 亮青色問號
    ("question", "fg:#ffffff bold"),           # 純白粗體問題
    ("answer", "fg:#39ff14 bold"),             # 螢光綠答案
    ("pointer", "fg:#ff007f bold"),            # 霓虹粉箭頭
    ("highlighted", "fg:#ff007f bold underline"), # 霓虹粉加底線高亮
    ("selected", "fg:#39ff14 bold"),           # 螢光綠已勾選
    ("separator", "fg:#7928ca italic"),        # 紫色分割線
    ("instruction", "fg:#888888 italic"),      # 暗灰提示
    ("disabled", "fg:#444444 italic"),         # 深灰禁用選項
    ("validation-toolbar", "fg:#ffffff bg:#ff0055 bold") # 桃紅底白字報錯
])

# 2. 方式 B：使用 prompt_toolkit 的 Style.from_dict (字典模式)
from prompt_toolkit.styles import Style as PtStyle
dracula_theme = PtStyle.from_dict({
    "qmark": "#ff5555 bold",
    "question": "#f8f8f2 bold",
    "answer": "#50fa7b bold",
    "pointer": "#bd93f9 bold",
    "highlighted": "#ffb86c bold",
    "selected": "#50fa7b",
    "separator": "#6272a4",
    "instruction": "#6272a4 italic",
})

# 3. 在問答中套用自訂樣式
framework = questionary.select(
    "請選擇主要使用的框架：",
    choices=[
        Separator("=== 現代 API 框架 ==="),
        Choice("FastAPI (推薦)", "fastapi"),
        Choice("Litestar (高效能)", "litestar"),
        Choice("Flask 傳統框架 (維護中)", "flask", disabled="暫停支援")
    ],
    style=cyberpunk_theme
).ask()

print(f"你的選擇: {framework}")
```

---

# 實戰：打造現代 CLI 互動式專案腳手架

結合 `questionary` 表單問答與 [[Rich#1. Rich Console (控制台大腦)|Rich]] 彩色面板，打造極致專業的終端互動初始化工具：

```python
import sys
import questionary
from questionary import Choice, Style
from rich.console import Console
from rich.panel import Panel

console = Console()

# 1. 終端標頭面板
console.print(Panel(
    "[bold cyan]🚀 歡迎使用 Python 現代專案初始化工具[/bold cyan]\n"
    "[dim]快速建立符合標準規範的專案骨架[/dim]",
    border_style="cyan"
))

# 2. 自訂問答配色
theme = Style([
    ("qmark", "fg:#8be9fd bold"),
    ("question", "bold"),
    ("answer", "fg:#50fa7b bold"),
    ("pointer", "fg:#ff79c6 bold"),
    ("highlighted", "fg:#ffb86c bold")
])

# 3. 收集使用者配置
config = questionary.form(
    project_name=questionary.text("1. 請輸入專案名稱:", default="my_project"),
    framework=questionary.select(
        "2. 請選擇後端核心框架:",
        choices=[
            Choice("FastAPI (極速異步 Web)", value="fastapi"),
            Choice("Django (全功能框架)", value="django"),
            Choice("CLI 工具 (無 Web 依賴)", value="cli")
        ]
    ),
    use_ai=questionary.confirm("3. 是否整合 Google Gemini AI SDK？", default=True),
    tools=questionary.checkbox(
        "4. 請勾選開發輔助套件:",
        choices=[
            Choice("Rich (終端美化)", checked=True),
            Choice("Ruff (程式碼檢查)", checked=True),
            Choice("pytest (單元測試)", checked=True)
        ]
    ),
    style=theme
).ask()

# 4. 關鍵防呆：若使用者按下 Ctrl+C 梯退
if config is None:
    console.print("\n[bold red]❌ 使用者已取消操作。[/bold red]")
    sys.exit(0)

# 5. 成功展示
console.print(Panel(
    f"[bold green]🎉 專案 `{config['project_name']}` 配置完成！[/bold green]\n"
    f"核心框架: [cyan]{config['framework']}[/cyan] | 整合 AI: [yellow]{config['use_ai']}[/yellow]",
    title="建置成功",
    border_style="green"
))
```

---

# 實戰除錯與核心天條

## 1. 使用者按 Ctrl+C 產生 None 導致程式崩潰 (NoneType Error)

> **⚠️ Ctrl+C 中斷防呆天條**：  
> 當使用者在問答過程中按下 `Ctrl+C` 中途退出時，`.ask()` **預設不會拋出例外，而是回傳 `None`**！  
> 若直接對回傳值進行取值或下標索引（如 `answers['name']`），會立即拋出 `TypeError: 'NoneType' object is not subscriptable` 崩潰！  
> **鐵律**：呼叫 `.ask()` 後永遠加上防呆檢查：  
> ```python
> ans = questionary.text("請輸入名稱：").ask()
> if ans is None:
>     print("操作已取消")
>     sys.exit(0)
> ```

---

## 2. validate 驗證函式的回傳規則天條

> **⚠️ 驗證函式回傳值陷阱**：  
> 撰寫自訂 `validate` 函式時：  
> - **驗證通過**：必須明確回傳 **`True`**。  
> - **驗證失敗**：必須回傳**「錯誤提示文字字串 (`str`)」**，**切勿直接回傳 `False`**！若回傳 `False`，終端機只會顯示生硬的預設警告 `"Invalid input"`，使用者完全無法得知具體哪裡填錯。
