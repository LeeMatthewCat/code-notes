這是一個強大的 Python 終端互動式命令列應用函式庫。

主要用來取代原生的 `input()`，提供如自動補全、語法高亮、多行輸入等進階功能。

---

## 核心優勢與比較

為什麼我們需要 `prompt_toolkit`？

- **取代原生 `input()`**：提供更流暢的終端機體驗。
- **支援豐富互動**：輕鬆實作命令歷史記錄、語法高亮與自訂快捷鍵。
- **全螢幕應用**：甚至可用來打造如 `vim` 或 `nano` 般的全螢幕終端程式。

| 功能比較 | 原生 `input()` | `prompt_toolkit` 的 `prompt()` |
| :--- | :--- | :--- |
| **自動補全** | ❌ 不支援 | ✅ 支援多種 Completer |
| **語法高亮** | ❌ 不支援 | ✅ 支援 Pygments 整合 |
| **多行輸入** | ❌ 不支援 | ✅ 支援 |
| **終端相容性** | ⚠️ 各系統表現不一 | ✅ 跨平台高度相容 |

---

## 基礎互動與輸入

最基礎的用法就是直接匯入並使用 `prompt()`，能做到所有 `input()` 能做的事。

### prompt() 單次互動提示符

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

### PromptSession() 會話管理

- **使用時機**：當你需要建立一個像 Bash 或 Python Shell 那樣**持續運行的對話迴圈**，並希望自動保存歷史紀錄或共享共用設定檔（如自動補全器、自訂配色樣式）時使用。
- **語法**：`PromptSession(history=None, auto_suggest=None, completer=None, style=None, ...)`
- **參數說明**：由於它支援的參數極多（與 prompt 高度重疊），以下列出核心參數：

| 參數名稱 | 期待型別 | 說明 |
| :--- | :--- | :--- |
| `history` | `History` | 用於管理歷史紀錄的策略物件（如 `FileHistory`）。 |
| `auto_suggest` | `AutoSuggest` | 用於自動預測補齊的策略物件（如 `AutoSuggestFromHistory`）。 |
| `completer` | `Completer` | 一次性綁定的自動補全器，後續呼叫不用再傳。 |
| `style` | `BaseStyle` / `Style` | 全域樣式表物件（如 `Style.from_dict(...)`），統一設定提示字元、補全選單與狀態列的配色。 |

- **回傳值**：
  - `PromptSession`：會話實例，後續可透過 `.prompt()` 方法來啟動互動。

```python
...
from prompt_toolkit import PromptSession
from prompt_toolkit.styles import Style

# 1. 定義共用樣式表
my_style = Style.from_dict({
    'prompt': 'ansigreen bold',
    'bottom-toolbar': '#333333 bg:#ffffff',
})

# 2. 一次性綁定設定（包含 style）
session = PromptSession(style=my_style)

while True:
    # 3. 透過 session 呼叫 prompt，享受保留狀態與統一樣式的好處
    text = session.prompt('> ')
    ...
```

#### FileHistory() 持久化歷史紀錄與共享狀態

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

#### AutoSuggestFromHistory() 歷史自動建議

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

### 其他實用參數 (Other Parameters)

##### bottom_toolbar 底部狀態列

- **使用時機**：想在終端機輸入畫面的最底部，常駐顯示一段輔助資訊（如快捷鍵提示、目前模式、系統時間）時使用。
- **語法**：作為 `prompt(..., bottom_toolbar=...)` 的參數傳入。
- **參數說明**：
  - `bottom_toolbar`：可接受純字串、`HTML()` 格式化文字物件，或是一個會回傳文字的 `Callable` (函式，用於動態即時更新內容)。
- **回傳值**：無，純屬視覺渲染設定。

```python
...
from prompt_toolkit import prompt
from prompt_toolkit.formatted_text import HTML

# 1. 建立帶有顏色標籤的底部狀態文字
toolbar_text = HTML(" <b>[Ctrl-Q]</b> 離開 | <b>[Tab]</b> 補全")

# 2. 傳入 bottom_toolbar 參數
answer = prompt("請輸入：", bottom_toolbar=toolbar_text)
...
```

---

## 自動補全 (Auto-completion)

透過 `Completer`，能在使用者輸入時即時提供下拉選單建議。

### WordCompleter() 建立字詞補全清單

- **使用時機**：當你需要給使用者一個固定、已知的單字清單（例如指令、選項參數、路徑），讓他們能在輸入時透過 Tab 鍵快速補全時使用。最簡單且最常用的實作。
- **語法**：`WordCompleter(words: List[str], ignore_case: bool = False, WORD: bool = False, meta_dict: Optional[Dict[str, str]] = None, match_middle: bool = False, ...)`
- **參數說明**：
  - `words`：包含所有建議單字的串列 (List)。
  - `ignore_case`：布林值，設定為 `True` 時將忽略英文大小寫。
  - `WORD`：布林值，預設為 `False`。決定分詞邊界規則：
    - **`WORD=False` (預設小寫單字模式)**：使用常規單字字元 (`\w+`) 作為邊界，遇到 `-`、`.`、`/`、`:` 等標點符號時會被當作分隔符號截斷。
    - **`WORD=True` (大寫 WORD 模式 ⭐ CLI 必備)**：使用非空白字元 (`\S+`) 作為邊界（類似 Vim 的 WORD），只有遇到空格才截斷！
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

## 快捷鍵綁定 (Key Bindings)

就像是為終端機打造專屬的「快捷鍵設定面板」，允許開發者攔截鍵盤預設事件，並賦予全新的自訂行為。

> **💡 觀念釐清**：在終端機按下 `Ctrl+C` 預設是中斷程式，按下 `Enter` 預設是送出。透過快捷鍵綁定，你可以完全攔截這些訊號，例如把 `Tab` 鍵綁定成「插入四個空白」而非預設的跳轉。

### KeyBindings() 建立與攔截快捷鍵

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

## 實用對話框元件 (Dialogs)

提供類似 GUI 彈出視窗的終端機介面，非常適合用於確認操作或顯示訊息。

### yes_no_dialog() 確認對話框

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
result = yes_no_dialog(
    title='確認操作',
    text='您確定要刪除所有檔案嗎？'
).run()

if result:
    ...
```

---

## 格式化色彩與字體 (Formatted Text)

它擁有自己的微型 XML 解析器，提供比純 ANSI 碼更直覺的上色方式。

### print_formatted_text() 格式化輸出

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

### HTML() 標籤化上色系統

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

> **💡 小提醒**：`prompt_toolkit` 底層會完全接管終端機畫面，請**絕對避免**直接把帶有 ANSI 色碼的 `rich` 字串硬塞給 `prompt()`，這會導致亂碼與游標錯位！若想自訂色彩，請一律使用 `HTML()`。

---

### Style.from_dict() 自訂樣式表

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
