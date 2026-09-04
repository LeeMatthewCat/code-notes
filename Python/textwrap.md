# 概念與原理

## 什麼是 textwrap 模組？

Python 內建的核心標準庫（無需額外安裝，直接 `import textwrap`）。

專門用於**純文字段落的格式化排版、自動折行 (Wrapping)、文字填充 (Filling)、縮排修剪 (Dedenting) 與文字截斷省略 (Shortening)**。

在 CLI 命令列工具、終端機日誌輸出、Docstring 提取美化以及 AI 提示詞 (Prompt) 格式化中，`textwrap` 能讓雜亂的多行字串瞬間變得整齊優雅。

> **🍿 生動白話比喻**：  
> - **沒有 textwrap 模組**：文字在終端機輸出時，像**「一長條完全不換行、直接爆出螢幕邊界且左側縮排混亂的散亂草稿」**。  
> - **有了 textwrap 模組**：就像給文字請了一位**「專業的文字排版師 / 裁縫師 📐」**：  
>   - **太寬了？** ➔ 自動按寬度剪裁換行（`fill` / `wrap`）。  
>   - **多行字串縮排太深？** ➔ 自動一秒剔除左側多餘的空白（`dedent`）。  
>   - **需要每一行都加前綴？** ➔ 一鍵在每行開頭加上 `> ` 或空格（`indent`）。  
>   - **句子太長想做摘要？** ➔ 自動在尾巴加上 `[...]` 智慧截斷（`shorten`）。

---

# 核心 API 函數字典對照表

| 類別 / 函式名稱 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- |
| **[[#textwrap.fill() 自動折行並填充字串\|textwrap.fill()]]** | **按指定寬度自動折行，回傳單一換行字串** | 終端機文字輸出排版、限制文字不超過螢幕邊界 |
| **[[#textwrap.wrap() 取得折行後的字串清單\|textwrap.wrap()]]** | **按指定寬度自動折行，回傳行字串清單** | 需要逐行處理、繪製帶邊框的終端表格 |
| **[[#textwrap.dedent() 去除共有縮排空白\|textwrap.dedent()]]** | **自動移除多行字串中所有行共有的前置空格** | 清理三引號 `"""..."""` 內部的縮排雜訊 |
| **[[#textwrap.indent() 為所有行添加前綴\|textwrap.indent()]]** | **在多行文字的每一行開頭加上指定前綴** | 製作 Markdown 引用塊 (`> `)、程式碼縮排 |
| **[[#textwrap.shorten() 智慧截斷並補上省略號\|textwrap.shorten()]]** | **按單詞邊界截斷文字並補上省略號** | 製作標題摘要、限制字數不破壞單詞完整性 |
| **[[#TextWrapper 封裝排版器\|textwrap.TextWrapper]]** | **建立可重複使用的排版器實例** | 專案中多次調用相同排版配置時提升效率 |

---

# 核心功能與語法大解密

## 1. 文字折行與填充 (Wrapping & Filling)

##### textwrap.fill() 自動折行並填充字串

- **使用時機**：將一長串連續文字按照指定的字元寬度進行自動折行，並直接產出以 `\n` 換行符號連接的單一格式化字串（最常用 ⭐）。
- **語法**：`textwrap.fill(text, width=70, initial_indent='', subsequent_indent='', ...)`
- **核心參數字典對照表**：

| 參數名稱 | 型別 | 預設值 | 說明與功能 |
| :--- | :--- | :--- | :--- |
| **`text`** | `str` | *(必填)* | 要排版的原始文字字串。 |
| **`width`** | `int` | `70` | 每一行容納的最大字元寬度上限。 |
| **`initial_indent`** | `str` | `""` | **第一行**開頭要額外插入的前綴字串（如 `"  "` 或 `"• "`）。 |
| **`subsequent_indent`** | `str` | `""` | **第二行及後續所有行**開頭要插入的前綴字串。 |
| **`break_long_words`** | `bool` | `True` | 是否強制拆斷長度超過 `width` 的超長單詞。 |
| **`break_on_hyphens`** | `bool` | `True` | 是否允許在連字號 (`-`) 處進行換行。 |

- **回傳值**：
  - `str`：完成自動折行排版後的單一字串。

```python
import textwrap

long_text = "Python 是一種直譯式、高階程式語言，具備優雅的語法與強大的標準庫，廣泛應用於人工智慧、數據分析與網頁後端開發。"

# 1. 按每行 20 個字元自動折行
formatted = textwrap.fill(long_text, width=20)
print("【基礎折行】:\n" + formatted)

# 2. 加上首行縮排與後續行對齊前綴 (懸掛縮排)
bullet_text = textwrap.fill(
    long_text,
    width=25,
    initial_indent="• [重要] ",
    subsequent_indent="         "  # 後續行縮排對齊
)
print("\n【清單項目排版】:\n" + bullet_text)
```

---

##### textwrap.wrap() 取得折行後的字串清單

- **使用時機**：與 `fill()` 邏輯相同，但**回傳的是每一行的字串列表 (`list[str]`)**，適合需要逐行繪製 UI 邊框或逐行遍歷時使用。
- **語法**：`textwrap.wrap(text, width=70, ...)`
- **回傳值**：
  - `list[str]`：折行後的各行字串清單。

```python
import textwrap

msg = "這是一段需要被切成一行一行的文字，用來放進終端機的對話框裡面。"

# 取得每行 12 字元的清單
lines = textwrap.wrap(msg, width=12)

for i, line in enumerate(lines, start=1):
    print(f"第 {i} 行: | {line} |")
```

---

## 2. 縮排清理與前綴添加 (Dedenting & Indenting)

##### textwrap.dedent() 去除共有縮排空白

- **使用時機**：在 Python 函式或類別內部撰寫**三引號多行字串 (`"""..."""`)** 時，原始字串會包含程式碼縮排產生的多餘前置空白。`dedent()` 能**自動尋找所有行最小的共有縮排並一鍵剔除**（AI Prompt 排版必備神器 ⭐）。
- **語法**：`clean_text = textwrap.dedent(multiline_text)`
- **回傳值**：
  - `str`：剔除共有縮排後的乾淨多行字串。

```python
import textwrap

def generate_prompt(user_query: str) -> str:
    # 原始字串包含了函式內部的 8 個空格縮排雜訊
    raw_prompt = """
        你是一位專業的 Python 工程師。
        請根據以下需求撰寫代碼：
        需求內容: {query}
    """
    # 一秒剔除共有縮排！
    return textwrap.dedent(raw_prompt).strip()

print("【清理後的 Prompt】:")
print(generate_prompt("請建立一個二元樹"))
```

> **💡 原理剖析**：  
> `dedent()` 會掃描字串中所有非空白行的最左側空格數，並將這個「最少空格數」從每一行的開頭統一剪掉，因此內部相對應的縮排層級**依然會完整保留**！

---

##### textwrap.indent() 為所有行添加前綴

- **使用時機**：需要為多行文字的**每一行開頭統一加上特定標記**（例如加上 Markdown 引用符號 `> `、註釋符號 `# ` 或 4 個空格）時使用。
- **語法**：`textwrap.indent(text, prefix, predicate=None)`
- **參數說明**：
  - `text`：多行文字字串。
  - `prefix`：要加在每行開頭的前綴字串（如 `"> "`、`"    "`）。
  - `predicate`：可選過濾函式。接收每一行字串，回傳 `True` 才加上前綴（**預設只對非空行加上前綴，忽略純空白行**）。
- **回傳值**：
  - `str`：加上前綴後的多行字串。

```python
import textwrap

code_snippet = """def hello():
    print("Hello World!")
    return True"""

# 1. 轉為 Markdown 引用塊 (每行前面加上 '> ')
quote_text = textwrap.indent(code_snippet, prefix="> ")
print("【Markdown 引用】:\n" + quote_text)

# 2. 為所有行統一縮排 4 個空格
tabbed_text = textwrap.indent(code_snippet, prefix="    ")
print("\n【統一縮排】:\n" + tabbed_text)
```

---

## 3. 文字截斷與省略 (Shortening & Truncating)

##### textwrap.shorten() 智慧截斷並補上省略號

- **使用時機**：需要將一段長文字壓縮在指定的寬度以內（如產生新聞摘要、卡片標題），並在結尾自動補上 `[...]`。
- **核心特性（單詞完整性保護）**：
  - 普通字串切片 `text[:30] + '...'` 會殘忍地把一個英文單詞或中文字**從中間砍成兩半**。
  - `textwrap.shorten()` 會**依照空格/單詞邊界進行智慧裁切**，確保單詞完整不破裂！
- **語法**：`textwrap.shorten(text, width=50, placeholder=' [...]', ...)`
- **參數說明**：
  - `text`：原始長文字。
  - `width`：截斷後的**總最大長度**（包含 `placeholder` 省略號本身的長度！）。
  - `placeholder`：省略標記字串（預設為 `' [...]'`）。
- **回傳值**：
  - `str`：截斷後的摘要字串。

```python
import textwrap

article = "Python is an easy to learn, powerful programming language with excellent libraries."

# 1. 限制總長度在 35 字元以內
summary = textwrap.shorten(article, width=35)
print(summary)
# 輸出: Python is an easy to learn, [...] (單詞完整保留！)

# 2. 自訂省略號標記
custom_summary = textwrap.shorten(article, width=35, placeholder="... (閱讀更多)")
print(custom_summary)
# 輸出: Python is an easy... (閱讀更多)
```

> **⚠️ shorten() 長度溢出報錯陷阱**：  
> 若傳入的 `width` 比省略號 `placeholder` 本身的長度還要短（例如 `placeholder=" [...]"` 長度為 6，但傳入 `width=4`），會直接拋出 `ValueError: placeholder is too large for max width`！

---

## 4. TextWrapper 類別實例化

##### TextWrapper 封裝排版器

- **使用時機**：在迴圈或高頻函式中，需要**多次使用同一組排版規則**時，建立 `TextWrapper` 實例可避免重複解析參數，提升執行效能。
- **語法**：`wrapper = textwrap.TextWrapper(width=..., initial_indent=..., ...)`

```python
import textwrap

# 建立專用的日誌格式排版器
log_wrapper = textwrap.TextWrapper(
    width=30,
    initial_indent="[LOG] ",
    subsequent_indent="      "
)

messages = [
    "系統初始化完成，正在載入資料庫連線池...",
    "偵測到未授權的連線請求，已自動攔截並記錄 IP。"
]

for msg in messages:
    print(log_wrapper.fill(msg))
```

---

# 實戰：打造 CLI 終端機格式化對話框

結合 `textwrap.wrap()` 與 `textwrap.dedent()`，可以輕鬆畫出整齊美觀的終端機訊息框：

```python
import textwrap

def print_box_message(title: str, content: str, box_width: int = 40):
    """繪製終端機文字對話框"""
    print("┌" + "─" * (box_width - 2) + "┐")
    print(f"│ {title:<{box_width - 4}} │")
    print("├" + "─" * (box_width - 2) + "┤")
    
    # 清理縮排並按框內寬度自動折行
    clean_text = textwrap.dedent(content).strip()
    wrapped_lines = textwrap.wrap(clean_text, width=box_width - 4)
    
    for line in wrapped_lines:
        print(f"│ {line:<{box_width - 4}} │")
        
    print("└" + "─" * (box_width - 2) + "┘")

# 呼叫範例
print_box_message(
    title="系統通知 (System Notice)",
    content="""
        伺服器將於今晚 24:00 進行例行維護升級。
        預計維護時間為 30 分鐘，期間 API 將暫停服務。
    """,
    box_width=36
)
```

---

# 實戰除錯與核心天條

## 1. 中文字元寬度視覺不齊問題 (全形 CJK vs 半形 ASCII)

> **⚠️ 終端機中文字寬陷阱**：  
> `textwrap` 內部是以「字元數量 (Character Count)」計算長度，而不是終端機的「顯示寬度 (Display Columns)」！  
> 一個中文字元算 1 個長度，但在終端機顯示時會佔用 2 個英文字位的寬度。若在純英文排版時效果完美，但在中英混排時若需精確對齊，建議搭配第三方庫（如 `rich` 或 `wcwidth`）進行計算。

---

## 2. dedent() 第一行非空格的截斷失效陷阱

> **⚠️ dedent 第一行縮排陷阱**：  
> 若多行字串的第一行緊接著三引號沒有換行（如 `"""第一行沒有空格`），第一行的縮排數就是 `0`！  
> 這會導致 `dedent()` 判定「最小共有空格數為 0」，從而**完全不進行任何縮排清理**！  
> **鐵律**：使用 `dedent()` 時，三引號後請務必先換行：  
> ```python
> # ⭕ 正確寫法（第一行換行，縮排被正確識別）
> text = """
>     這是第一行
>     這是第二行
> """
> textwrap.dedent(text)
> ```
