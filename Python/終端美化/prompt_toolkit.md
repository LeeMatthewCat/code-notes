# 概念與原理

## 什麼是 prompt_toolkit？

這是一個強大的 Python 終端互動式命令列應用函式庫。

主要用來取代原生的 `input()`，提供如自動補全、語法高亮、多行輸入等進階功能。

> **[生動比喻]**：  
> - **原生的 `input()`**：像「陽春的投幣式電話」，按鍵撥號後只能直直講完掛斷，中途打錯字難以修改，無法自動預測也無法上色。  
> - **`prompt_toolkit`**：像「現代智慧型手機的鍵盤與作業系統」，打字時會即時自動預測單字、即時標記語法色彩、支援自由移動游標，甚至還能監聽按鍵事件自訂專屬熱鍵！  
> - **`Buffer` (文字編輯緩衝區)**：像「手機螢幕中央正在輸入的即時文字草稿紙」，每一筆輸入、刪除、游標位移與歷史紀錄都在這張草稿紙上即時計算。  
> - **`on_text_changed`**：像「草稿紙上的感應鈴鐺」，只要輸入框有任何字元變動，鈴鐺立刻響起並通知後台執行即時校驗、動態計數或預覽更新！

---

## 核心優勢與比較

為什麼我們需要 `prompt_toolkit`？

- **取代原生 `input()`**：提供更流暢的終端機體驗。
- **支援豐富互動**：輕鬆實作命令歷史記錄、語法高亮與自訂快捷鍵。
- **全螢幕應用**：甚至可用來打造如 `vim` 或 `nano` 般的全螢幕終端程式。

| 功能比較 | 原生 `input()` | `prompt_toolkit` |
| :--- | :--- | :--- |
| **自動補全** | [不支援] | [支援] 多種 Completer |
| **語法高亮** | [不支援] | [支援] Pygments 整合 |
| **多行輸入** | [不支援] | [支援] |
| **終端相容性** | 各系統表現不一 | 跨平台高度相容 |

---

# 核心 API 類別、函式與屬性字典對照表

| 類別 / 函式 / 屬性名稱 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- |
| **[[#prompt() 單次互動提示符\|prompt()]]** | **發起單次命令列互動輸入** | 簡易腳本、密碼輸入、單次問答互動 |
| **[[#PromptSession() 會話管理類別\|PromptSession()]]** | **建立具備歷史紀錄與共享設定的持續會話** | REPL 互動環境、CLI 交互式 Shell |
| **[[#FileHistory() 持久化歷史紀錄\|FileHistory()]]** | **將命令歷史持久化儲存至本機檔案** | 重啟程式後保留歷史輸入紀錄 |
| **[[#AutoSuggestFromHistory() 歷史自動建議\|AutoSuggestFromHistory()]]** | **根據歷史輸入自動於游標後方浮現補全建議** | 智慧命令列、減少重複長指令輸入負擔 |
| **[[#PromptSession() 的 bottom_toolbar 參數\|PromptSession() 的 bottom_toolbar 參數]]** | **在終端最底部呈現動態或靜態輔助狀態列** | 快捷鍵提示、動態字數統計、狀態資訊列 |
| **[[#WordCompleter() 建立字詞補全清單\|WordCompleter()]]** | **依據關鍵字清單提供下拉自動補全** | CLI 指令、參數選項、關鍵字智慧補全 |
| **[[#KeyBindings() 建立與攔截快捷鍵\|KeyBindings()]]** | **宣告自訂按鍵規則與信號攔截器** | 自訂熱鍵（如 Ctrl+Q 退出、Tab 縮排） |
| **[[#Buffer 核心文字編輯緩衝區類別\|Buffer]]** | **終端文字輸入與編輯的核心狀態大腦** | 底層文字操作、自訂 UI 控制項、複雜編輯器 |
| **[[#event.current_buffer 當前作用中緩衝區屬性\|event.current_buffer]]** | **快捷鍵事件中取得目前焦點所在的緩衝區實例** | 熱鍵事件中動態修改文字、移動游標、強制送出 |
| **[[#session.default_buffer 預設輸入緩衝區核心概念\|session.default_buffer]]** | **PromptSession 所持有的主要持久輸入緩衝區實例** | 狀態列動態計算字數、預先掛載變更監聽器、外部檢查文字 |
| **[[#default_buffer.text 緩衝區文字字串內容\|default_buffer.text]]** | **讀取或寫入當前緩衝區的文字純字串** | 檢查輸入內容、程式化設定或清空文字 |
| **[[#default_buffer.cursor_position 游標位置索引\|default_buffer.cursor_position]]** | **讀取或移動目前文字游標在字串中的索引位置** | 控制輸入游標跳轉、精確定位插入點 |
| **[[#default_buffer.document 文檔結構快照與語彙分析\|default_buffer.document]]** | **獲取文字文檔快照物件以進行行內語彙與游標周邊分析** | 即時提取游標前詞、分析游標行列位置、文檔內省 |
| **[[#default_buffer.on_text_changed 文字變更事件勾點\|default_buffer.on_text_changed]]** | **當緩衝區文字產生任何變更時觸發的事件監聽勾點** | 即時語法檢查、動態字數統計、輸入聯動預覽 |
| **[[#default_buffer.on_cursor_position_changed 游標移動事件勾點\|default_buffer.on_cursor_position_changed]]** | **當游標在文字中移動時觸發的事件監聽勾點** | 游標行列計數器、自動高亮括號匹配 |
| **[[#default_buffer.insert_text() 在游標處插入文字\|default_buffer.insert_text()]]** | **在當前游標所在位置精確插入一段文字字串** | 快捷鍵插入文字範本、自動補齊引號括號 |
| **[[#default_buffer.validate_and_handle() 驗證並提交內容\|default_buffer.validate_and_handle()]]** | **立即觸發校驗器並程式化模擬按下 Enter 送出** | 快捷鍵強制送出、多行編輯中的快捷提交 |
| **[[#default_buffer.reset() 清空並重置緩衝區\|default_buffer.reset()]]** | **清空文字內容、歸零游標並重置歷史還原棧** | Esc 鍵清空輸入、會話輪次手動重設 |
| **[[#Application 應用程式實例與全螢幕生命週期\|Application]]** | **prompt_toolkit 最高執行實例與事件迴圈指揮官** | 全螢幕 TUI 應用、多視窗儀表板、非文字輸入互動介面 |
| **[[#layout.find_all_controls() 遍歷佈局中所有 UI 控制項\|layout.find_all_controls()]]** | **搜尋並遍歷當前佈局樹狀結構中所有的 UI 控制項** | 走訪所有輸入緩衝區或格式化文字控制項、動態巡檢 |
| **[[#layout.find_all_windows() 遍歷佈局中所有視窗容器\|layout.find_all_windows()]]** | **搜尋並產生當前佈局中所有的 Window 視窗實體** | 檢查各視窗幾何尺寸維度、批次設定視窗屬性 |
| **[[#layout.focus() 與 layout.has_focus() 焦點切換與檢查\|layout.focus() / layout.has_focus()]]** | **切換鍵盤操作焦點至特定控制項或檢查焦點狀態** | 多視窗多輸入框切換、根據目前焦點動態啟用快捷鍵 |
| **[[#yes_no_dialog() 確認對話框\|yes_no_dialog()]]** | **彈出全螢幕 Yes/No 互動確認視窗** | 刪除前確認、重要操作二次確認 |
| **[[#print_formatted_text() 格式化輸出\|print_formatted_text()]]** | **支援彩色與樣式標籤的終端輸出函式** | 替代 print() 印出彩色訊息 |
| **[[#HTML() 標籤化上色系統\|HTML()]]** | **使用 HTML 風格標籤對文字進行樣式標記** | 簡潔且語意化地為文字與提示符上色 |
| **[[#Style.from_dict() 自訂樣式表\|Style.from_dict()]]** | **集中管理終端 UI 元件配色規則** | 統整命令列風格、主題切換 |

---

# 基礎互動與輸入

最基礎的用法就是直接匯入並使用 `prompt()`，能做到所有 `input()` 能做的事。

##### prompt() 單次互動提示符

- **使用時機**：當你只需要在程式中偶爾向使用者詢問一次資料（例如問密碼、問名稱），不需要保留跨次對話的歷史紀錄時使用。
- **語法**：`prompt(message: str, completer=None, is_password=False, key_bindings=None, bottom_toolbar=None, ...)`
- **參數說明**：
  - `message`：要顯示給使用者的提示文字字串。
  - `completer`：提供自動補全邏輯的物件（如 `WordCompleter`）。
  - `is_password`：布林值，設定為 `True` 時將隱藏輸入內容（變成星號）。
  - `key_bindings`：傳入 `KeyBindings` 物件（相當於按鍵訊號攔截器，用來自訂或覆寫如 Ctrl+C 的預設行為）。
  - `bottom_toolbar`：設定要在畫面底部常駐顯示的狀態列資訊。
- **回傳值**：
  - `str`：使用者輸入的最終結果字串。

```python
...
from prompt_toolkit import prompt

# 1. 專注展示基礎調用
answer = prompt("請輸入您的名字：")

# 2. 專注展示隱藏輸入的密碼提示框
password = prompt(..., is_password=True)
...
```

---

##### PromptSession() 會話管理類別

- **使用時機**：當你需要建立一個像 Bash、IPython 或資料庫 CLI 那樣**持續運行的對話迴圈**，並希望跨輪次自動保存歷史紀錄、共享全域配置（如配色主題、自動補全清單、自訂按鍵綁定）時使用。
- **語法**：`PromptSession(message=None, completer=None, validator=None, auto_suggest=None, style=None, key_bindings=None, bottom_toolbar=None, history=None, ...)`
- **參數說明**：`PromptSession` 與單次呼叫的 `prompt()` 共享絕大部分參數，以下將核心參數完整歸納：

| 參數名稱 | 期待型別 | 說明 |
| :--- | :--- | :--- |
| `message` | `str` / `HTML` | 預設終端提示符字串（如 `"> "` 或彩色 HTML 標籤）。 |
| `completer` | `Completer` | 全域自動補全器（如 `WordCompleter`），在每次呼叫時自動啟用。 |
| `validator` | `Validator` | 輸入內容即時驗證器，未通過驗證前禁止送出。 |
| `auto_suggest` | `AutoSuggest` | 歷史輸入自動預測建議（如 `AutoSuggestFromHistory`）。 |
| `style` | `Style` / `BaseStyle` | 全域外觀樣式表（透過 `Style.from_dict` 統一定義配色）。 |
| `key_bindings` | `KeyBindings` | 自訂按鍵規則與快捷鍵攔截器。 |
| `bottom_toolbar` | `str` / `HTML` / `Callable` | 底部常駐狀態列（支援字串或即時動態重新計算的回呼函式）。 |
| `rprompt` | `str` / `HTML` | 在提示行最右側對齊顯示的右側提示文字 (Right Prompt)。 |
| `multiline` | `bool` | 是否允許多行輸入（設為 `True` 時，按 Enter 換行，Esc+Enter 送出）。 |
| `is_password` | `bool` | 是否隱藏輸入內容（將輸入字元遮蔽為星號）。 |
| `placeholder` | `str` / `HTML` | 當輸入框為空時，以淡灰色呈現的佔位說明文字。 |
| `complete_while_typing` | `bool` | 使用者打字時是否主動彈出補全選單（預設為 `False`，需按 Tab 觸發）。 |
| `enable_history_search` | `bool` | 是否允許使用方向鍵或 Ctrl+R 快捷搜尋過往歷史指令。 |
| `mouse_support` | `bool` | 是否啟用滑鼠點擊與選單滾動。 |
| `history` | `History` | **[PromptSession 專屬]** 歷史紀錄儲存策略（如 `FileHistory`，必須在初始化時綁定）。 |

- **回傳值**：
  - `PromptSession`：會話實例，後續可多次重複呼叫 `.prompt()` 方法發起互動。

---

#### PromptSession(...) 建構參數 vs session.prompt(...) 調用參數深度對比

在實際開發中，許多參數（如 `completer`、`style`、`is_password`、`bottom_toolbar`）既可以寫在 `PromptSession(...)` 裡，也可以寫在 `session.prompt(...)` 裡。兩者存在關鍵的生命週期與行為差異：

| 比較維度 | `PromptSession(參數=...)` (建構式配置) | `session.prompt(參數=...)` (單次呼叫覆寫) |
| :--- | :--- | :--- |
| **設計定位** | **會話級全域預設值 (Session-wide Default)** | **單次提問動態覆寫 (Per-call Override)** |
| **作用範圍** | 作用於整個 Session 生命週期中的**所有後續呼叫** | 主要針對**當下這一次輸入請求**進行客製化 |
| **典型場景** | 配置整個 CLI 固定不變的主題、補全清單與快捷鍵 | 某一步需要特殊輸入（如忽然需要輸入密碼、切換臨時提示詞） |
| **專屬參數** | 支援 `history`、`input`、`output`（底層串流與歷史策略） | 支援 `default`（預填預設文字）、`accept_default`、`pre_run` |
| **狀態改寫機制** | 被動等待被單次調用覆寫 | **注意：傳入非 None 值會永久改寫 Session 內部對應屬性！** |

> **[核心天條]：session.prompt() 覆寫後具有「狀態黏滯性」！**  
> `prompt_toolkit` 底層設計中，當你在 `session.prompt(is_password=True)` 傳入某參數時，它不僅影響當前這一次，還會**直接改寫該 Session 實例上的內部屬性**！  
> - 若下一輪呼叫 `session.prompt()` 且未傳入該參數（預設為 `None`），它**不會還原**為建構時的預設值，而是繼續沿用上一輪被改寫的值！  
> - **最佳實踐**：若某輪使用了臨時特殊模式（如密碼輸入 `is_password=True`），在下一輪恢復一般輸入時，必須顯式傳入 `session.prompt(is_password=False)` 進行復原。  
> - 若要臨時清空補全器，傳入 `None` 無法清除（`None` 代表保持當前值），必須顯式傳入 `from prompt_toolkit.completion import DummyCompleter; session.prompt(completer=DummyCompleter())`！

```python
from prompt_toolkit import PromptSession
from prompt_toolkit.history import FileHistory
from prompt_toolkit.completion import WordCompleter
from prompt_toolkit.styles import Style

# 1. 在 PromptSession(...) 中配置全域共享的基礎設定（固定主題、持久歷史、共用補全）
session = PromptSession(
    history=FileHistory(".my_app_history"),          # 只能在建構式配置的參數
    completer=WordCompleter(["help", "status", "login", "exit"]),
    style=Style.from_dict({
        "prompt": "ansigreen bold",
        "bottom-toolbar": "#ffffff bg:#333333",
    }),
    bottom_toolbar="[狀態] 正常就緒"
)

# 2. 正常輪次：完全不傳參數，自動繼承 PromptSession 的全域配置
cmd = session.prompt("> ")

# 3. 特殊輪次：在 session.prompt(...) 中進行局部臨時覆寫 (例如輸入登入密碼)
if cmd == "login":
    # 使用 default 提供可編輯預設值（session.prompt 專屬參數）
    username = session.prompt("帳號 > ", default="admin")
    
    # 局部啟用密碼模式 (覆寫 is_password)
    password = session.prompt("密碼 > ", is_password=True)
    
    # 黏滯性防禦：下一輪若要輸入一般內容，顯式還原 is_password=False
    next_input = session.prompt("> ", is_password=False)
```

---

##### FileHistory() 持久化歷史紀錄

- **使用時機**：搭配 `PromptSession` 使用，希望即使關閉了程式，下一次打開依舊能用方向鍵找回昨天的輸入紀錄時。
- **語法**：`FileHistory(filename: str)`
- **參數說明**：
  - `filename`：要用來儲存歷史紀錄的本地檔案路徑。
- **回傳值**：
  - `FileHistory`：歷史管理策略物件。

```python
...
from prompt_toolkit import PromptSession
from prompt_toolkit.history import FileHistory

# 將歷史紀錄持久化寫入本地檔案中
session = PromptSession(history=FileHistory('.my_app_history'))
...
```

---

##### AutoSuggestFromHistory() 歷史自動建議

- **使用時機**：想提供類似 Fish Shell 那樣的高級體驗，在使用者打字時以灰色文字自動預測他們可能要打的句子。
- **語法**：`AutoSuggestFromHistory()`
- **參數說明**：無須傳入參數。
- **回傳值**：
  - `AutoSuggestFromHistory`：自動建議策略物件。

```python
...
from prompt_toolkit import PromptSession
from prompt_toolkit.auto_suggest import AutoSuggestFromHistory

# 專注展示開啟歷史紀錄自動建議
session = PromptSession(auto_suggest=AutoSuggestFromHistory())

# 假設你在這行曾經輸入過 "hello world"
... = session.prompt('> ')

# 當你下次迴圈只打出 "he" 時，游標後方會自動以灰色字體浮現 "llo world"！
...
```

---

##### PromptSession() 的 bottom_toolbar 參數

- **使用時機**：當你在 `PromptSession` 互動會話中，需要在終端最底部常駐顯示輔助資訊列（如熱鍵提示、即時字數統計、當前操作模式、Git 分支或時間），且希望隨著使用者打字即時動態更新時使用。
- **語法**：
  - 會話初始化綁定：`PromptSession(bottom_toolbar=...)`
  - 單次提問時動態傳入：`session.prompt(..., bottom_toolbar=...)`
  - 實例屬性動態賦值：`session.bottom_toolbar = ...`
- **參數說明**：
  - `bottom_toolbar`：支援以下四種型態傳入：
    - `str`：靜態純文字字串（如 `"[Ctrl+C] 離開"`）。
    - `HTML`：帶有色彩標籤的格式化文字物件（如 `HTML("<b>狀態：</b><ansigreen>正常</ansigreen>")`）。
    - `List[Tuple[str, str]]`：由樣式類別名稱與文字組成的元組串列（FormattedText 原生格式）。
    - `Callable[[], Any]`（核心進階用法）：**無參數的函式或 Lambda**，其回傳值為上述任一種文字格式。終端機在每次使用者按鍵、游標移動或畫面重新渲染時，都會**自動重新執行此函式**以實現即時動態更新！
- **回傳值**：
  - 無回傳值，純屬終端視覺渲染參數。

```python
import datetime
from prompt_toolkit import PromptSession
from prompt_toolkit.formatted_text import HTML
from prompt_toolkit.styles import Style

# 1. 定義狀態列樣式表 (底色與內部文字顏色)
style = Style.from_dict({
    'bottom-toolbar': '#ffffff bg:#333333',
    'bottom-toolbar.text': '#aaaaaa',
    'bottom-toolbar.key': 'bg:#555555 #00eeee bold',
})

session = PromptSession(style=style)

# 2. 定義動態狀態列產生函式 (每次畫面重繪皆會被即時呼叫)
def get_toolbar():
    # 直接讀取當前緩衝區的文字內容，實現隨打隨算的即時字數統計
    char_count = len(session.default_buffer.text)
    now_str = datetime.datetime.now().strftime("%H:%M:%S")
    return HTML(
        f" <b>[時間]</b> {now_str} | "
        f"<b>[字數]</b> {char_count} | "
        f"<b>[快捷鍵]</b> [Ctrl-Q] 退出"
    )

# 3. 綁定給 session (或在 session.prompt 中傳入)
session.bottom_toolbar = get_toolbar

while True:
    text = session.prompt("Prompt > ")
    if text.strip() == "exit":
        break
```

---

# 自動補全 (Auto-completion)

透過 `Completer`，能在使用者輸入時即時提供下拉選單建議。

##### WordCompleter() 建立字詞補全清單

- **使用時機**：當你需要給使用者一個固定、已知的單字清單（例如指令、選項參數、路徑），讓他們能在輸入時透過 Tab 鍵快速補全時使用。最簡單且最常用的實作。
- **語法**：`WordCompleter(words: List[str], ignore_case: bool = False, WORD: bool = False, meta_dict: Optional[Dict[str, str]] = None, match_middle: bool = False, ...)`
- **參數說明**：
  - `words`：包含所有建議單字的串列 (List)。
  - `ignore_case`：布林值，設定為 `True` 時將忽略英文大小寫。
  - `WORD`：布林值，預設為 `False`。決定分詞邊界規則：
    - **`WORD=False` (預設小寫單字模式)**：使用常規單字字元 (`\w+`) 作為邊界，遇到 `-`、`.`、`/`、`:` 等標點符號時會被當作分隔符號截斷。
    - **`WORD=True` (大寫 WORD 模式 [推薦] CLI 必備)**：使用非空白字元 (`\S+`) 作為邊界（類似 Vim 的 WORD），只有遇到空格才截斷！
    - **解決痛點**：若要補全包含連字號的 CLI 參數（如 `--help`、`--version`）或路徑（如 `src/main.py`），**必須設為 `WORD=True`**，否則輸入 `--` 時會被當成符號截斷而無法補全！
  - `meta_dict`：字典物件 (Dict)，為每個補全單字在下拉選單右側顯示即時說明提示文字（Meta Information）。
  - `match_middle`：布林值，設定為 `True` 時支援中綴匹配（輸入部分子字串即可觸發補全）。
- **回傳值**：
  - `WordCompleter`：可直接傳給 `prompt(completer=...)` 或 `PromptSession(completer=...)` 的補全器物件。

```python
from prompt_toolkit import prompt
from prompt_toolkit.completion import WordCompleter

# 1. 建立包含 CLI 指令與帶有「連字號 --」選項的補全清單
commands = ['--help', '--version', '--verbose', 'git-status', 'git-commit', 'exit']

# 2. 定義各選項的白話說明提示
meta_info = {
    '--help': '顯示幫助說明文檔',
    '--version': '查詢當前程式版本號',
    '--verbose': '啟用詳細日誌除錯模式',
    'git-status': '檢視工作區檔案狀態',
    'git-commit': '提交變更到倉庫',
    'exit': '退出終端對話'
}

# 3. 啟用 WORD=True 確保帶有符號的 '--help' 能被完整識別
cli_completer = WordCompleter(
    words=commands,
    ignore_case=True,
    WORD=True,          # 關鍵：允許補全包含 -- 等特殊符號的複合字詞！
    meta_dict=meta_info, # 顯示右側說明文字
    match_middle=True   # 支援中綴模糊匹配 (輸入 commit 也能找到 git-commit)
)

# 4. 傳入 prompt 啟動終端互動
text = prompt("CLI > ", completer=cli_completer)
print(f"執行指令: {text}")
```

---

# 快捷鍵綁定 (Key Bindings)

就像是為終端機打造專屬的「快捷鍵設定面板」，允許開發者攔截鍵盤預設事件，並賦予全新的自訂行為。

> **[觀念釐清]**：在終端機按下 `Ctrl+C` 預設是中斷程式，按下 `Enter` 預設是送出。透過快捷鍵綁定，你可以完全攔截這些訊號，例如把 `Tab` 鍵綁定成「插入四個空白」而非預設的跳轉。

##### KeyBindings() 建立與攔截快捷鍵

- **使用時機**：想要捕捉特定的按鍵組合（如 `Ctrl-Q`、`Tab`），並賦予它們特定行為（如退出程式、插入文字）時使用。
- **語法**：`KeyBindings()`
- **參數說明**：無須參數。
- **回傳值**：
  - `KeyBindings`：管理快捷鍵註冊的空清單。建立後，需透過其 `.add('按鍵組合')` 裝飾器來制定具體規則。

```python
...
from prompt_toolkit import prompt
from prompt_toolkit.key_binding import KeyBindings

# 1. 建立快捷鍵設定清單
bindings = KeyBindings()

# 2. 制定規則：攔截 'Ctrl-Q' (c-q)，並賦予離開程式的行為
@bindings.add('c-q')
def _(event):
    event.app.exit()
    ...

# 3. 將這份清單當作參數交給 prompt，攔截規則即可生效
text = prompt(..., key_bindings=bindings)
```

---

# 緩衝區與文字編輯核心 (Buffer & Text Manipulation)

在 `prompt_toolkit` 的底層架構中，所有文字輸入、游標移動與歷史紀錄，全部由核心的 **`Buffer`（緩衝區物件）** 全權掌管。

無論是單次 `prompt()`、長期的 `PromptSession`，或是全螢幕終端應用，文字永遠不是孤立的字串，而是被保存在 `Buffer` 之中。

---

## 1. 緩衝區架構基礎

##### Buffer 核心文字編輯緩衝區類別

- **使用時機**：當你需要建構自訂的命令列輸入框、打造多視窗文字編輯器，或需要底層精細控制文字輸入狀態與驗證規則時使用。
- **語法**：`Buffer(completer=None, auto_suggest=None, history=None, validator=None, on_text_changed=None, on_cursor_position_changed=None, multiline=True, read_only=False, ...)`
- **參數說明**：
  - `completer`：綁定此緩衝區專用的自動補全器。
  - `auto_suggest`：綁定此緩衝區專用的歷史建議器。
  - `history`：歷史紀錄管理物件。
  - `validator`：文字輸入即時校驗器（如 `Validator.from_callable`）。
  - `on_text_changed`：當文字內容發生變動時的回呼函式（接收 `Buffer` 作為唯一參數）。
  - `on_cursor_position_changed`：當游標移動時的回呼函式。
  - `multiline`：布林值，是否允許輸入多行文字。
  - `read_only`：布林值，是否為唯讀緩衝區。
- **回傳值**：
  - `Buffer`：文字編輯緩衝區實例。

```python
from prompt_toolkit.buffer import Buffer

# 1. 定義即時變更監聽函式
def handle_change(buf):
    print(f"緩衝區文字變動: {buf.text}")

# 2. 實例化獨立緩衝區
my_buffer = Buffer(on_text_changed=handle_change)
my_buffer.insert_text("Hello World")
```

---

##### event.current_buffer 當前作用中緩衝區屬性

- **使用時機**：在快捷鍵事件回呼函式（KeyBinding Handler）中，取得使用者當前正在打字、游標聚焦的 `Buffer` 物件實例（等同於 `event.app.current_buffer`）。
- **語法**：`event.current_buffer`
- **屬性型別**：`Buffer` 物件。

```python
from prompt_toolkit import prompt
from prompt_toolkit.key_binding import KeyBindings

bindings = KeyBindings()

# 按下 F2 鍵時，自動將輸入框清空
@bindings.add('f2')
def _(event):
    buffer = event.current_buffer
    buffer.reset()  # 操作當前作用中的緩衝區

text = prompt("> ", key_bindings=bindings)
```

---

## 2. default_buffer 預設輸入緩衝區及其專屬屬性

在 `PromptSession` 會話管理架構中，使用者在終端提示符後方輸入內容的「主文字輸入框」，本質上就是一個名為 `default_buffer` 的持久化 `Buffer` 物件實例（內部識別名稱為 `'DEFAULT_BUFFER'`）。

所有使用者打入的字元、移動的游標、即時校驗以及文檔快照，皆由這個 `default_buffer` 統一承擔。

| 概念維度 | `session.default_buffer` | `event.current_buffer` | `app.current_buffer` |
| :--- | :--- | :--- | :--- |
| **持有物件** | `PromptSession` 實例 | `KeyPressEvent` 按鍵事件參數 | `Application` 應用實例 |
| **取得時機** | 隨時可用（會話外部、回呼函式、主迴圈） | 僅在快捷鍵觸發時於事件常式中取得 | 在全螢幕應用或 UI 元件回呼中取得 |
| **角色定位** | 命令列輸入會話的「主要輸入框本體」 | 當下接收鍵盤輸入、擁有「焦點」的緩衝區 | 全螢幕多視窗佈局中，當前游標聚焦的視窗緩衝區 |

---

##### session.default_buffer 預設輸入緩衝區核心概念

- **使用時機**：在 `PromptSession` 會話中，需要從「外部（如狀態列回呼函式、背景監聽器、或是主迴圈）」存取使用者主要輸入框的 `Buffer` 物件時使用。
- **語法**：`session.default_buffer`
- **屬性型別**：`Buffer` 物件實例。
- **核心特點與用途**：
  - **持久存在**：不同於單次呼叫的 `prompt()` 會在每次執行後銷毀緩衝區，`PromptSession` 的 `default_buffer` 貫穿整個會話生命週期。
  - **狀態列即時聯動**：在 `bottom_toolbar` 函式中讀取 `session.default_buffer.text` 或其長度，能即時動態呈現打字進度與字數統計。
  - **預先掛載事件**：可以在會話啟動前，直接對其註冊變更監聽：`session.default_buffer.on_text_changed += my_handler`。
  - **文檔內省分析 (`document`)**：透過其 `document` 屬性，能提取如 `text_before_cursor`、`get_word_before_cursor()` 等豐富的行內文字上下文。

```python
from prompt_toolkit import PromptSession
from prompt_toolkit.formatted_text import HTML

session = PromptSession()

# 1. 預先掛載文字變更監聽器
def on_change(buf):
    # 當使用者輸入時即時監控
    pass

session.default_buffer.on_text_changed += on_change

# 2. 在動態狀態列中即時讀取 default_buffer 進行統計
def get_toolbar():
    # 存取 default_buffer 的純文字與 document 分析物件
    buf = session.default_buffer
    char_count = len(buf.text)
    word_before = buf.document.get_word_before_cursor()
    return HTML(f" <b>字數:</b> {char_count} | <b>前詞:</b> {word_before}")

session.bottom_toolbar = get_toolbar

text = session.prompt("> ")
```

---

##### default_buffer.text 緩衝區文字字串內容

- **使用時機**：讀取使用者當前輸入框中的完整純文字內容，或由程式碼強制賦值替換整個輸入框文字。
- **語法**：`session.default_buffer.text` (可讀寫屬性)
- **屬性型別**：`str` 字串。

```python
from prompt_toolkit import PromptSession

session = PromptSession()

# 讀取當前使用者輸入的完整文字長度
char_total = len(session.default_buffer.text)

# 亦可在熱鍵或事件中強制設定內容
session.default_buffer.text = "預設重設字串"
```

---

##### default_buffer.cursor_position 游標位置索引

- **使用時機**：獲取或設定游標在文字字串中的整數索引（從 `0` 到 `len(buffer.text)`），用於精確控制文字插入或跳轉。
- **語法**：`session.default_buffer.cursor_position` (可讀寫整數屬性)
- **屬性型別**：`int` 整數。

```python
from prompt_toolkit import PromptSession

session = PromptSession()

# 將游標瞬間移動至文字最開頭
session.default_buffer.cursor_position = 0

# 將游標瞬間移動至文字最末端
session.default_buffer.cursor_position = len(session.default_buffer.text)
```

---

##### default_buffer.document 文檔結構快照與語彙分析

- **使用時機**：當你需要分析當前輸入框內的文字語境、取得游標當前行/列位置、提取游標前方正在輸入的單詞（Word before cursor）以進行自訂動態補全、語意分析或狀態列顯示時使用。
- **語法**：`session.default_buffer.document` (或 `buffer.document`)
- **屬性型別**：`Document` 物件（不可變快照物件）。
- **常用屬性與方法說明**：
  - `document.text`：不可變快照文字內容。
  - `document.cursor_position`：游標於快照時的索引值。
  - `document.cursor_position_row`：游標所在行號（從 0 起算）。
  - `document.cursor_position_col`：游標所在欄號（從 0 起算）。
  - `document.text_before_cursor`：游標左側的所有文字字串。
  - `document.text_after_cursor`：游標右側的所有文字字串。
  - `document.get_word_before_cursor(WORD=False)`：提取游標正前方正在輸入的單詞。
  - `document.current_line_before_cursor`：當前行中游標左側的字串。
- **回傳值**：
  - `Document`：文檔快照實例。

```python
from prompt_toolkit import PromptSession

session = PromptSession()

# 透過 document 分析游標當前的文字語彙上下文
doc = session.default_buffer.document
current_word = doc.get_word_before_cursor()
row = doc.cursor_position_row
col = doc.cursor_position_col

print(f"目前單詞: {current_word}, 位置: 行 {row} 列 {col}")
```

---

##### default_buffer.on_text_changed 文字變更事件勾點

- **使用時機**：每當使用者輸入字元、刪除字元、剪下貼上，或程式碼動態修改文字時被自動觸發。極常用於即時字數統計、動態語法分析與即時聯動提示。
- **語法**：
  - 註冊監聽（掛載）：`session.default_buffer.on_text_changed += 回呼函式`
  - 註銷監聽（卸載）：`session.default_buffer.on_text_changed -= 回呼函式`
- **回呼簽名**：`callback(buffer: Buffer) -> None`（接收被修改的 `Buffer` 物件實例作為唯一引數）。
- **屬性型別**：`Event[Buffer]`（`prompt_toolkit.utils.Event` 事件發布訂閱物件）。

- **掛載語法解析（為什麼是 `+=` 而非 `=`？）**：
  初次接觸時常會疑惑：為什麼不能寫 `default_buffer.on_text_changed = on_change`？背後有著關鍵的架構考量：

  > **[生動比喻]**：  
  > - **直接賦值 `=`**：像「直接把學校講台上的廣播主機整台拆掉換成你自己的玩具」，不僅抹殺了系統原本的廣播功能，且整個學校只能有你一個人說話。  
  > - **運算子 `+=`**：像「在廣播主機的輸出面板上插上一條新的揚聲器訊號線」，原本的廣播系統完好無損，你的喇叭也會在廣播響起時同步播放！且任何人隨時都能插上（`+=`）或拔掉（`-=`）自己的訊號線。

  - **1. 本質是 Event 事件管理器而非普通回呼屬性**：  
    `on_text_changed` 並不是存放單一函式指標的變數，而是一個由 `prompt_toolkit.utils.Event` 實例化的事件發布器（Event Dispatcher）。它內部專門維護著一個處理函式清單（`self._handlers: list`）。
  - **2. Python 運算子多載機制 (`__iadd__` 與 `__isub__`)**：  
    `Event` 類別在底層多載了 Python 的原位加法與原位減法運算子：  
    - 執行 `event += handler` 時，等同於呼叫底層的 `event.add_handler(handler)`，將函式追加加入監聽清單中。  
    - 執行 `event -= handler` 時，等同於呼叫底層的 `event.remove_handler(handler)`，將該函式從監聽清單中安全移除。
  - **3. 多重廣播 (Multicast) 與防覆蓋特性**：  
    若使用一般的 `=` 賦值，會直接覆蓋摧毀 `Event` 物件本體，導致內建的事件派發邏輯徹底失效。使用 `+=` 可以同時掛載多個完全獨立的監聽函式（例如一個負責即時字數統計、另一個負責語法即時校驗），彼此並存不衝突。
  - **4. 發送者實例自動注入 (Sender Injection)**：  
    每當文字發生任何異動，`Event` 在呼叫 `fire()` 時會自動執行 `handler(self.sender)`，將當前發出事件的 `Buffer` 實例本體傳入回呼函式中，因此在 `on_change(buf)` 裡可以直接存取 `buf.text` 與 `buf.cursor_position`。
  - **5. 掛載生命週期原則（同一次運行中只需掛載一次）**：  
    在同一個 `PromptSession` 的生命週期中，**只需在主迴圈外部掛載一次即可**！  
    因為 `PromptSession` 的 `default_buffer` 是跨輪次持久存活的，且內部 `Event` 是以 Python 串列（`self._handlers.append`）保存回呼函式，**完全不會自動去重**。  
    若將掛載程式碼寫入 `while True:` 迴圈內部，每一輪迴圈都會再度 `append` 同一個函式，導致使用者打一個字卻連續觸發 2 次、3 次甚至幾十次回呼，引發嚴重的效能雪崩與重複處理。

> **[核心天條]：同一次運行中只需掛載一次！嚴禁寫在 while 迴圈內部造成重複疊加！**  
> - **[錯誤寫法]**：在 `while True:` 迴圈內部每次呼叫 `prompt()` 前執行 `session.default_buffer.on_text_changed += on_change`。第 10 輪時，每敲一個鍵就會連續執行 10 次回呼！  
> - **[正確寫法]**：在進入 `while True:` 迴圈之前（Session 初始化後）只掛載「一次」，整個會話全程自動生效。

```python
from prompt_toolkit import PromptSession

session = PromptSession()

# 1. 定義第一個監聽回呼：即時字數統計
def count_listener(buf):
    print(f"\n[字數統計] 目前文字長度: {len(buf.text)}")

# 2. 定義第二個監聽回呼：即時安全檢查
def security_listener(buf):
    if "sudo" in buf.text:
        print(f"\n[安全警告] 偵測到提權關鍵字！")

# 3. [正確寫法]：在進入互動迴圈前，於外部只掛載「一次」！
# default_buffer 會在整個 session 生命週期內持續沿用這份監聽清單
session.default_buffer.on_text_changed += count_listener
session.default_buffer.on_text_changed += security_listener

# 4. 若某階段不再需要特定監聽，使用 -= 即可隨時動態卸載
# session.default_buffer.on_text_changed -= security_listener

# 5. 持續互動的主迴圈
while True:
    # [錯誤寫法]：嚴禁在迴圈內寫 session.default_buffer.on_text_changed += ...
    # 否則每輪迴圈都會重複 append，導致打一個字連續觸發數十次回呼！
    text = session.prompt("輸入指令 > ")
    if text.strip() == "exit":
        break
```

---

##### default_buffer.on_cursor_position_changed 游標移動事件勾點

- **使用時機**：當使用者按方向鍵移動游標、點擊滑鼠或呼叫跳轉函式時觸發。適合用於動態取得游標當前所在單字、高亮對應標籤或更新行列計數器。
- **語法**：
  - 運算子附加：`session.default_buffer.on_cursor_position_changed += 回呼函式`
  - 運算子移除：`session.default_buffer.on_cursor_position_changed -= 回呼函式`
- **回呼簽名**：`callback(buffer: Buffer) -> None`。
- **屬性型別**：`Event[Callable[[Buffer], None]]`。

```python
from prompt_toolkit import PromptSession

session = PromptSession()

def on_cursor_move(buf):
    # 取得游標當前索引位置與文檔行列
    pos = buf.cursor_position
    row = buf.document.cursor_position_row
    col = buf.document.cursor_position_col

session.default_buffer.on_cursor_position_changed += on_cursor_move
```

---

##### default_buffer.insert_text() 在游標處插入文字

- **使用時機**：在快捷鍵或自動化流程中，於目前游標所在位置精確插入一段文字，並自動向後推移游標。
- **語法**：`session.default_buffer.insert_text(data, overwrite=False, move_cursor=True, fire_event=True)`
- **參數說明**：
  - `data`：要插入的字串。
  - `overwrite`：布林值，預設為 `False`。若為 `True`，則覆寫游標後方的文字。
  - `move_cursor`：布林值，預設為 `True`。插入後游標自動移至新文字末端。
  - `fire_event`：布林值，預設為 `True`。是否觸發 `on_text_changed` 事件。
- **回傳值**：
  - `None`。

```python
from prompt_toolkit import PromptSession

session = PromptSession()

# 程式化在目前游標處插入字串範本
session.default_buffer.insert_text("[自動補齊內容]")
```

---

##### default_buffer.validate_and_handle() 驗證並提交內容

- **使用時機**：在自訂按鍵或自動補全完成後，程式化模擬「按下 Enter 送出」的行為。它會先觸發驗證器，通過後立即結束此輪互動並回傳文字。
- **語法**：`session.default_buffer.validate_and_handle()`
- **參數說明**：無須傳入參數。
- **回傳值**：
  - `None`。

```python
from prompt_toolkit.key_binding import KeyBindings

bindings = KeyBindings()

@bindings.add('c-j')
def _(event):
    # 按下 Ctrl-J，立刻進行校驗並強制送出當前輸入
    buf = event.current_buffer
    buf.validate_and_handle()
```

---

##### default_buffer.reset() 清空並重置緩衝區

- **使用時機**：重置文字編輯器狀態，將文字內容清空、游標歸零並初始化復原歷史（Undo Stack）。
- **語法**：`session.default_buffer.reset(document=None, append_to_history=False)`
- **參數說明**：
  - `document`：可選，傳入全新的 `Document` 物件替代當前內容。若為 `None` 則清空為空字串。
  - `append_to_history`：布林值，是否將清空前的內容寫入歷史紀錄。
- **回傳值**：
  - `None`。

```python
from prompt_toolkit.key_binding import KeyBindings

bindings = KeyBindings()

@bindings.add('escape')
def _(event):
    # 按下 Esc 鍵時，清空當前輸入內容
    event.current_buffer.reset()
```

---

# 應用程式生命週期與最高指揮官 (Application)

在 `prompt_toolkit` 的架構體系中，**`Application`** 是整個系統的心臟與最高指揮官。

無論是高階封裝的 `prompt()` 函式、`PromptSession` 會話管理類別，或是像 `questionary` 這樣的第三方互動庫，其底層都是透過建立並驅動一個 `Application` 實例來運作。

#### 核心架構定位與角色職責

```text
  Application (最高指揮官 / 事件循環大腦)
       │
       ├── Layout (佈局樹狀結構：HSplit / VSplit / Window / UIControl)
       ├── KeyBindings (全域按鍵攔截規則)
       ├── Style (全域配色樣式表)
       ├── Clipboard (終端剪貼簿管理)
       └── Input / Output (跨平台終端輸入監聽與 ANSI 渲染輸出)
```

- **事件迴圈掌控者**：負責監聽使用者的鍵盤敲擊、滑鼠點擊、終端視窗尺寸調整（Resize），並排程執行背景任務。
- **畫面重繪指揮官**：當緩衝區文字變更或游標移動時，發起渲染管道（Render Pipeline）重新繪製終端畫面。
- **全螢幕與輸入模式**：管理終端是否進入 Alternate Screen（全螢幕緩衝區，結束後還原終端）以及是否啟用滑鼠事件。

---

##### Application 應用程式實例與全螢幕生命週期

- **使用時機**：
  - 需要開發多視窗、非文字輸入框為主的自訂互動工具（如選單、儀表板、全螢幕文字編輯器、終端儀表盤）。
  - 需要深度客製化事件迴圈、非同步生命週期或全螢幕終端介面時使用。
- **語法**：
  ```python
  from prompt_toolkit.application import Application

  app = Application(
      layout=layout,
      key_bindings=key_bindings,
      style=style,
      full_screen=False,
      mouse_support=False,
      color_depth=None,
  )
  ```
- **核心參數說明**：

| 參數名稱 | 資料型別 | 預設值 | 說明與作用 |
| :--- | :--- | :--- | :--- |
| **`layout`** | `Layout` | *(必填)* | 傳入佈局樹狀結構大腦，定義視窗與控制項的組織階層。 |
| **`key_bindings`** | `KeyBindings \| None` | `None` | 全域快捷鍵攔截規則清單。 |
| **`style`** | `BaseStyle \| None` | `None` | 全域配色樣式表實例（例如 `Style.from_dict(...)`）。 |
| **`full_screen`** | `bool` | `False` | 是否開啟全螢幕模式（Alternate Screen）。設為 `True` 時佔滿終端畫面，離開後還原原本的命令列歷史。 |
| **`mouse_support`** | `bool` | `False` | 是否啟用終端滑鼠事件監聽（如滾輪滾動、點擊聚焦）。 |
| **`color_depth`** | `ColorDepth \| None` | `None` | 強制指定終端色彩深度（如 True Color 24-bit 或 256 色）。 |

- **生命週期核心方法**：

| 方法名稱 | 語法 | 說明與用途 |
| :--- | :--- | :--- |
| **`run()`** | `app.run()` | 同步阻塞啟動應用程式並進入事件迴圈，直到呼叫 `exit()` 為止。 |
| **`run_async()`** | `await app.run_async()` | 原生非同步模式啟動事件迴圈，與 `asyncio` 協程無縫整合。 |
| **`exit()`** | `app.exit(result=None)` | 終止應用程式事件迴圈退出運行，並可選擇回傳結果物件。 |
| **`invalidate()`** | `app.invalidate()` | 程式化強制排程重繪目前終端畫面。 |

```python
from prompt_toolkit.application import Application
from prompt_toolkit.key_binding import KeyBindings
from prompt_toolkit.layout.containers import Window
from prompt_toolkit.layout.controls import FormattedTextControl
from prompt_toolkit.layout.layout import Layout

# 1. 建立快捷鍵清單：按 q 退出應用程式
kb = KeyBindings()

@kb.add("q")
def _(event):
    # 呼叫 event.app.exit() 終止 Application 事件迴圈
    event.app.exit(result="已安全離開全螢幕應用")

# 2. 定義 UI 控制項與視窗容器
control = FormattedTextControl(text="這是由 Application 驅動的全螢幕程式！按 q 退出。")
window = Window(content=control)
layout = Layout(window)

# 3. 實例化 Application 並以全螢幕模式啟動
app = Application(layout=layout, key_bindings=kb, full_screen=True)

# 4. 同步阻塞運行
exit_result = app.run()
print(f"程式結束回傳: {exit_result}")
```

---

# 佈局與視窗控制核心 (Layout & UI Hierarchy)

在 `prompt_toolkit` 的全螢幕或多視窗互動應用中，所有呈現於終端的視覺元件都組織在一棵階層樹狀結構中。

#### 核心架構階層關係：Layout、Window 與 UIControl

在 `prompt_toolkit` 中，三者的從屬關係為：

```text
  Application.layout (Layout 樹狀管理大腦)
         │
         ├── find_all_windows() 走訪所有可視視窗
         │         │
         │         └── Window (視窗幾何邊界、大小尺寸維度 Dimension)
         │                   │
         └── find_all_controls() 提取視窗內部掛載的控制項
                             │
                             └── UIControl (具體內容與邏輯)
                                   ├── BufferControl (文字輸入框、游標控制)
                                   └── FormattedTextControl (靜態文字、彩色選單)
```

- **`Window`**：負責「外觀幾何」與「尺寸限制」（寬度、高度、邊框、是否隱藏游標）。
- **`UIControl`**：負責「內部內容」與「使用者互動」（繪製文字 Token、接收滑鼠點擊、快取文字）。
- **`layout.find_all_controls()` 的底層實作**：內部即是走訪 `layout.find_all_windows()`，並依次取出各個 `window.content`。
- **第三方庫延伸（如 Questionary）**：例如在 [[Questionary#questionary.prompts.common.InquirerControl 選單控制項核心類別|Questionary 的 InquirerControl]] 中，就是繼承自 `FormattedTextControl`，因此透過 `layout.find_all_controls()` 即可直接遍歷並提取出該選單控制器。

---

##### layout.find_all_controls() 遍歷佈局中所有 UI 控制項

- **使用時機**：在應用程式初始化、快捷鍵回呼、動態更新或排查除錯時，需要搜尋、檢查或過濾目前畫面佈局樹狀結構中所有的 `UIControl` 控制項（例如尋找所有的 `BufferControl` 或 `FormattedTextControl`）。
- **語法**：`layout.find_all_controls()`
- **呼叫途徑**：
  - 透過 `Application` 實例：`app.layout.find_all_controls()`
  - 透過事件參數：`event.app.layout.find_all_controls()`
  - 透過 `PromptSession` 實例：`session.layout.find_all_controls()`
- **參數說明**：
  - 無須傳入參數。
- **回傳值**：
  - `Iterable[UIControl]`：可迭代的控制項生成器，每次產生一個 `UIControl` 物件（如 `BufferControl`、`FormattedTextControl` 等）。

```python
from prompt_toolkit import PromptSession
from prompt_toolkit.layout.controls import BufferControl, FormattedTextControl

session = PromptSession()

# 遍歷當前會話佈局中的所有控制項
for control in session.layout.find_all_controls():
    if isinstance(control, BufferControl):
        # 找到文字編輯緩衝區控制項 (例如預設輸入框 "DEFAULT_BUFFER")
        print(f"找到緩衝區控制項: {control.buffer.name}")
    elif isinstance(control, FormattedTextControl):
        # 找到格式化文字控制項 (如狀態列或靜態提示)
        print("找到格式化文字控制項")
```

---

##### layout.find_all_windows() 遍歷佈局中所有視窗容器

- **使用時機**：需要搜尋佈局中所有的 `Window` 視窗實體時使用（例如批次調整視窗尺寸、檢查可見度或設定 `always_hide_cursor`）。
- **語法**：`layout.find_all_windows()`
- **呼叫途徑**：
  - 透過 `Application` 實例：`app.layout.find_all_windows()`
  - 透過事件參數：`event.app.layout.find_all_windows()`
  - 透過 `PromptSession` 實例：`session.layout.find_all_windows()`
- **參數說明**：
  - 無須傳入參數。
- **回傳值**：
  - `Generator[Window, None, None]`：生成器，產生佈局中所有 `Window` 實例。

```python
from prompt_toolkit import PromptSession

session = PromptSession()

# 遍歷當前佈局樹狀結構中所有可視視窗容器
for window in session.layout.find_all_windows():
    # 檢查視窗是否掛載了特定控制項內容
    content = window.content
    print(f"視窗內容控制項型態: {type(content).__name__}")
```

---

##### layout.focus() 與 layout.has_focus() 焦點切換與檢查

- **使用時機**：在多視窗/多輸入框終端介面中，透過程式碼主動將鍵盤焦點切換到特定控制項、緩衝區或視窗，或檢查當前是否聚焦。
- **語法**：
  - 切換焦點：`layout.focus(value)`
  - 檢查焦點：`layout.has_focus(value)`
- **呼叫途徑**：
  - 透過 `Application` 實例：`app.layout.focus(...)`
  - 透過事件參數：`event.app.layout.focus(...)`
  - 透過 `PromptSession` 實例：`session.layout.focus(...)`
- **參數說明**：
  - `value`：可傳入 `UIControl` 實例、`Buffer` 實例、緩衝區名稱字串（如 `"DEFAULT_BUFFER"`）、`Window` 實例或任意容器物件。
- **回傳值**：
  - `focus()` 回傳 `None`；`has_focus()` 回傳 `bool`。

```python
from prompt_toolkit import PromptSession

session = PromptSession()

# 1. 透過緩衝區名稱將鍵盤操作焦點切換至預設輸入框
session.layout.focus("DEFAULT_BUFFER")

# 2. 檢查目前焦點是否停留在預設輸入框
is_focused = session.layout.has_focus("DEFAULT_BUFFER")
print(f"預設輸入框是否擁有焦點: {is_focused}")
```

---

# 實用對話框元件 (Dialogs)

提供類似 GUI 彈出視窗的終端機介面，非常適合用於確認操作或顯示訊息。

##### yes_no_dialog() 確認對話框

- **使用時機**：在執行不可逆操作（如刪除檔案、覆寫資料）前，需要跳出一個佔據終端畫面中央的醒目對話框，強迫使用者選擇 Yes 或 No 時。
- **語法**：`yes_no_dialog(title: str, text: str)`
- **參數說明**：
  - `title`：對話框頂部的標題。
  - `text`：對話框中間顯示的詢問文字。
- **回傳值**：
  - `Application`：會回傳一個應用程式物件，**必須加上 `.run()`** 才會真正執行並取得結果。
  - `.run()` 執行結果：布林值 (`True` 代表 Yes，`False` 代表 No)。

```python
...
from prompt_toolkit.shortcuts import yes_no_dialog

# 專注展示確認視窗的建立與執行，它會佔用整個終端畫面
dialog_app = yes_no_dialog(
    title='確認操作',
    text='您確定要刪除所有檔案嗎？'
)
result = dialog_app.run()

if result:
    ...
```

---

# 格式化色彩與字體 (Formatted Text)

它擁有自己的微型 XML 解析器，提供比純 ANSI 碼更直覺的上色方式。

##### print_formatted_text() 格式化輸出

- **使用時機**：想在終端機印出帶有 `HTML()` 或其他樣式的彩色文字，用來取代原生只能印出純白字的 `print()` 時。
- **語法**：`print_formatted_text(*values, sep=' ', end='\n', ...)`
- **參數說明**：
  - `*values`：要印出的多個物件（可以是字串，也可以是 `HTML` 物件）。
  - `sep` / `end`：與原生 `print()` 的換行與間距行為一致。
- **回傳值**：
  - `None`。

```python
...
from prompt_toolkit.formatted_text import HTML
from prompt_toolkit import print_formatted_text

# 1. 準備帶有 HTML 標籤的文字
styled_text = HTML("<b>粗體</b> 和 <ansired>紅色文字</ansired>")

# 2. 印出格式化文字
print_formatted_text(styled_text, ...)
```

---

##### HTML() 標籤化上色系統

- **使用時機**：想要直覺地為終端文字加上顏色、粗體或背景色，並將結果安全地傳遞給 `prompt()` 或 `print_formatted_text()` 顯示時。
- **語法**：`HTML(value: str)`
- **參數說明**：
  - `value`：包含類似 HTML 標籤（如 `<b>`, `<ansired>`）的格式化字串。
- **回傳值**：
  - `HTML`：一種 `FormattedText` 類別，能被 `prompt_toolkit` 安全解析為終端控制碼。

- **與網頁的差異**：它沒有任何排版功能（如 `<div>` 或 `<p>`），純粹只負責文字樣式。
- **巢狀與基礎字體**：支援 `<b>` (粗體)、`<u>` (底線)、`<i>` (斜體) 以及巢狀結構。
- **終端機專屬色彩**：內建大量終端專屬標籤，例如 `<ansired>`、`<bg:ansiblue>`。

```python
...
from prompt_toolkit import prompt
from prompt_toolkit.formatted_text import HTML

# 專注展示 HTML 標籤的多層次套用
styled_prompt = HTML("<b><ansiblue>請輸入</ansiblue> <ansired>名稱：</ansired></b>")

# 直接把 HTML 物件餵給 prompt，即可完美顯示色彩並保持游標穩定
answer = prompt(styled_prompt)
...
```

> **[小提醒]**：`prompt_toolkit` 底層會完全接管終端機畫面，請**絕對避免**直接把帶有 ANSI 色碼的 `rich` 字串硬塞給 `prompt()`，這會導致亂碼與游標錯位！若想自訂色彩，請一律使用 `HTML()`。

---

##### Style.from_dict() 自訂樣式表

- **使用時機**：當你需要為終端互動介面統一定義外觀主題（類似 CSS 樣式表），設定提示詞文字、補全下拉選單高亮、底部狀態列顏色，並注入給 `PromptSession` 或 `prompt()` 時使用。
- **語法**：`Style.from_dict(style_dict: dict)`
- **參數說明**：
  - `style_dict`：字典格式的樣式規則。鍵（Key）為目標類別名稱（如 `'prompt'`, `'bottom-toolbar'`, `'completion-menu.completion'`），值（Value）為樣式字串（支援前景色、`bg:` 背景色、`bold`, `italic`, `underline` 等）。
- **回傳值**：
  - `Style`：樣式表物件，可傳入 `PromptSession(style=...)` 或 `prompt(style=...)`。

- **空字串鍵 `''` 的特殊意義（全域預設 / 使用者輸入文字）**：
  - 在樣式字典中，**空字串鍵 `''` 代表「全域預設樣式 (Default / Root Style)」**（相當於 CSS 中的 `body` 或 `*` 選擇器）。
  - 它的主要作用是**定義所有「未指定特定 class 的普通文字」**，最常見的用途就是**改變使用者正在打字輸入時的文字顏色**！
  - 例如：`Style.from_dict({'': 'ansiblue'})` 會讓使用者在終端機打字輸入的所有內容一律呈現藍色。

```python
...
from prompt_toolkit import PromptSession
from prompt_toolkit.styles import Style
from prompt_toolkit.completion import WordCompleter

# 1. 建立自訂 UI 配色字典 (類似 CSS 規則)
custom_style = Style.from_dict({
    # '' (空字串)：全域預設樣式，用來控制「使用者打字輸入時」的預設文字顏色
    '': 'ansiblue',

    # 提示符標籤樣式
    'username': '#ansiblue bold',
    'at': '#888888',
    'host': '#ansigreen',
    'pound': 'bold',
    
    # 補全下拉選單樣式
    'completion-menu.completion': 'bg:#008888 #ffffff',
    'completion-menu.completion.current': 'bg:#00aaaa #000000 bold',
    
    # 底部狀態列樣式
    'bottom-toolbar': '#ffffff bg:#333333',
})

# 2. 注入給 PromptSession 全域生效
session = PromptSession(
    completer=WordCompleter(['status', 'commit', 'push']),
    style=custom_style
)

# 使用者在終端機輸入的文字將會自動呈現藍色 (ansiblue)
text = session.prompt('> ')
...
```

---

# 實戰避坑與核心天條

## 1. on_text_changed 內部修改文字引發遞迴爆棧

> **[核心天條]：切勿在 `on_text_changed` 監聽回呼內部直接改寫 `buffer.text`，否則會觸發無限遞迴！**

- **錯誤症狀**：
  程式報錯 `RecursionError: maximum recursion depth exceeded while calling a Python object` 並立即崩潰。
- **背後原理**：
  `buffer.text = ...` 或 `buffer.insert_text(...)` 會發出文字修改事件，該事件會再次觸發 `on_text_changed` 監聽函式，從而形成自己呼叫自己的死迴圈。

```python
from prompt_toolkit.buffer import Buffer

# [錯誤寫法]：直接在回呼內部修改 text，引爆無窮遞迴
# def bad_change_handler(buf):
#     buf.text = buf.text.upper()  # 再次觸發 on_text_changed -> 崩潰！

# [正確寫法]：使用防重入布林標記 (Reentrancy Guard)
_is_updating = False

def safe_change_handler(buf):
    global _is_updating
    if _is_updating:
        return
    
    _is_updating = True
    try:
        # 執行需要的文字自動轉換或修正
        pass
    finally:
        _is_updating = False
```

---

## 2. 混用 Rich ANSI 字串造成游標位置錯位

> **[核心天條]：嚴禁將帶有 ANSI 逸出碼的字串直接傳給 `prompt()`！**

- **錯誤症狀**：
  文字顯示正常但使用者按 Backspace 刪字時游標跳動混亂、刪除位置不對，或長字串自動換行時整行破版重疊。
- **背後原理**：
  `prompt_toolkit` 需要精確計算提示字串的字元可視寬度以維持終端機游標座標。原生 ANSI 色碼（如 `\x1b[31m`）會被誤算為可視寬度，導致游標計算產生偏差。必須一律使用 `HTML()` 包裹。

```python
from prompt_toolkit import prompt
from prompt_toolkit.formatted_text import HTML

# [錯誤寫法]：混用外部 ANSI 跳脫碼
# prompt("\033[91mName: \033[0m")

# [正確寫法]：使用 prompt_toolkit 原生 HTML 標記
prompt(HTML("<ansired>Name: </ansired>"))
```

---

## 3. CLI 參數補全漏設 WORD=True 造成連字號截斷

> **[核心天條]：補全帶有 `--` 連字號或路徑斜線的詞彙時，WordCompleter 必須顯式指定 `WORD=True`！**

- **錯誤症狀**：
  使用者在終端機打出 `--` 或 `git-` 時，下拉補全清單完全不彈出。
- **背後原理**：
  `WordCompleter` 預設以 `\w+`（字母數字底線）為單字邊界，連字號 `-` 被當成切詞標點截斷，導致前綴無法正確匹配候選清單。

```python
from prompt_toolkit.completion import WordCompleter

# [錯誤寫法]：預設 WORD=False，無法識別 --help
# completer = WordCompleter(['--help', '--version'])

# [正確寫法]：以非空白字元為分詞邊界
completer = WordCompleter(['--help', '--version'], WORD=True)
```

---

## 4. 快捷鍵回呼中執行阻塞任務造成終端凍結

> **[核心天條]：KeyBindings 處理常式必須極速返回，耗時 I/O 必須非同步委派！**

- **錯誤症狀**：
  按下自訂快捷鍵後，整個終端介面瞬間凍結無回應，使用者無法繼續輸入或退出。
- **背後原理**：
  `prompt_toolkit` 建立在單一執行緒事件迴圈上，任何在事件處理常式中的 `time.sleep()` 或同步網路請求都會直接阻塞畫面渲染引擎。

```python
import asyncio
from prompt_toolkit.key_binding import KeyBindings

bindings = KeyBindings()

# [正確寫法]：透過 app.create_background_task 發起非同步背景任務
@bindings.add('c-s')
def _(event):
    async def async_save():
        # 模擬非同步背景保存，完全不卡死使用者打字
        await asyncio.sleep(1)
    
    event.app.create_background_task(async_save())
```
