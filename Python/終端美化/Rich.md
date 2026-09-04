這是一個在 Python 社群中最知名的終端機 UI 與文字美化套件。

主要用來取代原生的 `print()`，將枯燥的黑底白字轉換成媲美網頁等級視覺效果的控制台介面，支援全彩、Emoji、表格、進度條與語法高亮。

> **🍿 比喻單元**：
> 如果傳統內建的 `print()` 是**古老的文字黑白傳真機**；
> 那麼 `rich` 就是終端機界的 **CSS 樣式庫與 Modern UI 框架**！

---

## 安裝與基礎載入

`rich` 為第三方套件，需先透過 `pip` 安裝至虛擬環境中：

```Shell
pip install rich
```

### 基礎快速體驗

如果你不想建立複雜的控制台物件，可直接取代原生 `print()` 快速體驗。

##### print() 快速格式化輸出

- **使用時機**：不想建立 `Console` 實例，只想快速在單一腳本中印出漂亮、具備高亮與排版效果的字典或字串時。
- **語法**：`print(*objects, sep=' ', end='\\n', ...)`
- **參數說明**：
  - `*objects`：要輸出的文字或資料結構（如 `dict`, `list`）。
- **回傳值**：
  - `None`。

```python
...
from rich import print

# 1. 支援類似 HTML 的 markup 標籤樣式
print("[bold red]警告：[/bold red] [italic yellow]連線逾時！[/italic yellow]")

# 2. 自動為字典加上彩色高亮與美麗縮排
data = {"user": "Matthew", "skills": ["Python", "Git"]}
print(data) 
...
```

> **💡 小提醒**：`rich.print()` 會自動識別資料型態並進行縮排與著色，大幅提升除錯時的資訊閱讀效率！

---

## 控制台大腦 (Console)

`Console` 類別是整個 Rich 套件的指揮中心。除了基本輸出，還支援日誌、狀態與輸入。

### Markup 樣式對照

| 樣式類別 | 語法標記範例 | 說明 |
| :--- | :--- | :--- |
| **文字顏色** | `[red]`, `[#ff5733]` | 支援標準顏色名稱與 HEX 色碼 |
| **字體樣式** | `[bold]`, `[italic]` | 粗體、斜體排版 |
| **背景顏色** | `[on red]` | 設定背景底色 |
| **超連結** | `[link=https://...]文字[/link]` | 可點擊的超連結 |

##### Console() 建立控制台實例

- **使用時機**：開發正式 CLI 工具時，所有進階輸出都應統一交由實例化的 `Console` 管理。
- **語法**：`Console(...)`
- **參數說明**：多數保持預設即可。
- **回傳值**：
  - `Console`：控制台實例。

```python
...
from rich.console import Console

console = Console()
...
```

---

##### print() 控制台輸出

- **使用時機**：透過 `Console` 實例印出文字、標籤或各種 Rich UI 組件（如表格、面板）時。
- **語法**：`console.print(*objects, style=None, justify=None, ...)`
- **參數說明**：

| 參數名稱 | 期待型別 | 說明 |
| :--- | :--- | :--- |
| `*objects` | `Any` | 指定要輸出的內容，支援傳入多個。 |
| `style` | `str` | 設定整行輸出的預設樣式（如 `"bold on blue"`）。 |
| `justify` | `str` | 指定文字對齊（`"left"`, `"center"`, `"right"`）。 |
| `overflow` | `str` | 溢出處理（`"fold"`, `"crop"`, `"ellipsis"`）。 |

- **回傳值**：
  - `None`。

```python
...
from rich.console import Console

console = Console()

# 專注展示使用 justify 與 style 全域樣式參數
console.print("這是置右顯示的藍底文字", justify="right", style="on blue")
...
```

---

##### log() 輸出日誌

- **使用時機**：需要進行基礎除錯，希望系統自動加上執行時間戳記與程式碼行號時。
- **語法**：`console.log(*objects, log_locals=False, ...)`
- **參數說明**：
  - `log_locals`：布林值，設定為 `True` 會將呼叫當下的所有區域變數以表格印出。
- **回傳值**：
  - `None`。

```python
...
from rich.console import Console

console = Console()

# 專注展示附帶區域變數的日誌
console.log("資料庫初始化完成", log_locals=True)
...
```

---

##### status() 加載狀態

- **使用時機**：執行需要等待的耗時任務（如下載、連線）時，想顯示一個不斷旋轉的動畫避免使用者以為死機。
- **語法**：`console.status(status, spinner="dots", ...)`
- **參數說明**：
  - `status`：顯示在動畫旁邊的提示文字。
  - `spinner`：動畫樣式（預設為 `"dots"`）。
- **回傳值**：
  - `Status`：上下文管理器，需搭配 `with` 使用。

```python
...
import time
from rich.console import Console

console = Console()

# 專注展示 Spinner 動畫
with console.status("[bold green]正在下載資料...[/bold green]"):
    time.sleep(2)
...
```

---

##### input() 彩色輸入

- **使用時機**：取代原生 `input()`，希望能用彩色標籤裝飾輸入提示字元時。
- **語法**：`console.input(prompt="", password=False, ...)`
- **參數說明**：
  - `prompt`：包含 Markup 標籤的提示字串。
  - `password`：布林值，`True` 時隱藏輸入。
- **回傳值**：
  - `str`：使用者輸入的字串。

```python
...
from rich.console import Console

console = Console()

# 專注展示彩色輸入提示
name = console.input("[bold yellow]請輸入名稱：[/bold yellow] ")
...
```

---

##### .size / .width 偵測終端尺寸與長度

- **使用時機**：當你需要根據使用者當前的終端機視窗寬度或高度，動態調整文字排版、文字截斷長度、自適應生成分隔線、圖表或邊框時。
- **語法**：`.width` / `.height` / `.size`
- **屬性說明**：
  - `.width`：取得當前終端機的**寬度（欄數 / 字元長度 Columns）**，型別為 `int`。
  - `.height`：取得當前終端機的**高度（行數 / 列數 Rows）**，型別為 `int`。
  - `.size`：回傳一個 `ConsoleDimensions` 命名元組，包含 `width` 與 `height` 屬性。
- **回傳值**：
  - `int` 或 `ConsoleDimensions(width, height)`。

```python
...
from rich.console import Console

console = Console()

# 1. 取得終端機的寬度（字元長度）與高度
width = console.width
height = console.height
print(f"當前終端寬度：{width} 字元，高度：{height} 行")

# 2. 透過 console.size 一次取得長寬
size = console.size
print(f"終端尺寸: 寬 {size.width} x 高 {size.height}")

# 3. 實際應用：根據終端寬度動態調整排版或邊框
if console.width < 60:
    console.print("[yellow]視窗過窄，建議拉寬終端機以獲得最佳體驗！[/yellow]")
else:
    console.print("=" * console.width, style="cyan")
...
```

> **💡 小提醒**：
> - `rich` 會在使用者拉伸或縮小終端機視窗時，**自動動態更新** `console.width` 與 `console.height` 的數值。
> - 若需要固定特定寬度（例如生成固定格式的報表或輸出至純文字檔），可在建立實例時傳入 `Console(width=80)` 來手動鎖定寬度。

---

## 豐富組件 (Rich Components)

以下列出建立美觀儀表板的各式高階 UI 組件。

##### Text() 文字物件

- **使用時機**：當你需要進階地控制一段字串中不同片段的顏色與樣式，或者準備將文字組裝進表格、面板等其他組件時。
- **語法**：`Text(text="", style=None, justify=None, ...)`
- **參數說明**：
  - `text`：基礎字串內容。
  - `style`：套用於整段文字的樣式（如 `"bold red"`）。
  - `justify`：對齊方式（如 `"center"`）。
- **回傳值**：
  - `Text`：可渲染的文字物件，能透過 `.append()` 方法無限串接帶有不同樣式的文字。

```python
...
from rich.console import Console
from rich.text import Text

console = Console()

# 1. 建立基礎文字物件，並指定全域樣式
title = Text("系統狀態：", style="bold underline")

# 2. 透過 append 串接不同顏色的文字片段
title.append("正常 ", style="green")
title.append(" (但負載偏高)", style="italic yellow")

console.print(title)
...
```

---

###### Text.from_markup() 標籤解析文字

- **使用時機**：當你習慣像寫 HTML 一樣，在字串中直接混插多種樣式標籤（如 `[red]...[/red]`），並希望它能轉化為安全的 `Text` 物件時。
- **語法**：`Text.from_markup(text: str, ...)`
- **參數說明**：
  - `text`：包含 Rich Markup 標籤（中括號）的原始字串。
- **回傳值**：
  - `Text`：已成功解析標籤的文字物件。

```python
...
from rich.console import Console
from rich.text import Text

console = Console()

# 專注展示直接解析標籤建立 Text
my_text = Text.from_markup("這是一段 [bold blue]藍色粗體[/bold blue] 與 [red]紅色[/red] 的混搭。")
console.print(my_text)
...
```

> **💡 小提醒**：一般的 `Text("[blue]嗨[/blue]")` 不會發揮顏色效果，因為它為了安全防呆，會直接把標籤當成普通文字印出來！若要解析中括號標籤，請務必使用 `from_markup()`。

---

##### Rule() 水平分隔線

- **使用時機**：想在終端畫面畫出一條能自動填滿視窗寬度的水平分隔線，用來區分不同輸出的區塊時。
- **語法**：`Rule(title="", style=None, align="center", ...)`
- **參數說明**：
  - `title`：會鑲嵌（穿插）在線條之中的文字，視覺上與線條同高（支援 Markup 樣式標籤）。
  - `style`：線條本身的顏色與樣式設定（如 `"bold red"`）。
  - `align`：標題文字的靠齊位置，預設為 `"center"`，也可設為 `"left"` 或 `"right"`。
- **回傳值**：
  - `Rule`：分隔線物件，需交由 `console.print()` 渲染。

```python
...
from rich.console import Console
from rich.rule import Rule

console = Console()

# 專注展示畫出帶有標題且靠左對齊的紅色分隔線
console.print(Rule("[bold yellow]報表結束[/bold yellow]", style="red", align="left"))
...
```

> **💡 小提醒**：`Rule` 的核心優勢在於它會**自動偵測並適應當前終端機的確切寬度**！這比手動 `print("-" * 50)` 聰明得多，不管使用者怎麼拉伸視窗，都永遠不用擔心折行而導致排版大亂。

---

##### Table() 繪製資料表

- **使用時機**：需要以整齊的欄列排版展示統計數據，並希望擁有圓角或網格邊框時。
- **語法**：`Table(title=None, box=box.ROUNDED, ...)`
- **參數說明**：
  - `title`：表格的頂部大標題。
  - `box`：邊框主題設定（如 `box.ROUNDED`, `box.MINIMAL`）。
- **回傳值**：
  - `Table`：表格物件，建立後需搭配 `.add_column()` 與 `.add_row()` 填入資料。

```python
...
from rich.console import Console
from rich.table import Table
from rich import box

console = Console()

# 1. 建立表格與邊框主題
table = Table(title="統計表", box=box.ROUNDED)

# 2. 新增欄位 (指定顏色與對齊)
table.add_column("姓名", style="magenta")
table.add_column("分數", justify="right")

# 3. 插入列資料
table.add_row("Matthew", "95")

console.print(table)
...
```

---

##### track() 迴圈包裝器

- **使用時機**：有一個明確長度的迴圈（如 `range` 或 `list`），想要快速替它加上一條精緻的進度條時。
- **語法**：`track(sequence, description="Working...", ...)`
- **參數說明**：
  - `sequence`：要迭代的可迭代物件（如串列）。
  - `description`：進度條左側的說明文字。
- **回傳值**：
  - 迭代產生器，可直接用在 `for` 迴圈中。

```python
...
import time
from rich.progress import track

# 專注展示為迴圈加上進度條
for i in track(range(100), description="[cyan]處理中..."):
    time.sleep(0.02)
...
```

---

##### Panel() 卡片面板

- **使用時機**：想要將某些資訊（如系統狀態、錯誤訊息）用一個漂亮的邊框包裝起來，像是一張獨立的卡片時。
- **語法**：`Panel(renderable, title=None, subtitle=None, expand=True, ...)`
- **參數說明**：
  - `renderable`：卡片內容（字串或任何 Rich 組件）。
  - `title`：頂端標題。
  - `expand`：布林值，`False` 時會根據內容收縮邊框，不撐滿視窗。
- **回傳值**：
  - `Panel`：面板物件，需交由 `console.print()` 輸出。

```python
...
from rich.console import Console
from rich.panel import Panel

console = Console()

# 專注展示建立不撐滿視窗的邊框卡片
card = Panel("記憶體使用率：42%", title="伺服器狀態", expand=False)
console.print(card)
...
```

---

##### Group() 群組渲染器

- **使用時機**：當你需要將多個不同的 UI 組件（例如兩段文字加上一條分隔線）**綑綁成單一個物件**時。最常用來解決 `Panel` 或 `Table` 欄位「只能塞入單一物件」的限制。
- **語法**：`Group(*renderables, fit=False, ...)`
- **參數說明**：
  - `*renderables`：要綑綁在一起的多個 Rich 組件（支援不限數量的傳入）。
- **回傳值**：
  - `Group`：包含所有傳入組件的群組實例。

```python
...
from rich.console import Console, Group
from rich.panel import Panel
from rich.text import Text
from rich.rule import Rule

console = Console()

# 1. 準備多個不同的組件
text1 = Text("這是第一行")
text2 = Text("這是第二行")
rule = Rule(style="red")

# 2. 將它們打包成一個 Group
my_group = Group(text1, rule, text2)

# 3. 成功把三個組件一起塞進只能吃一個參數的 Panel 裡！
console.print(Panel(my_group))
...
```

---

##### Padding() 縮排與內邊距

- **使用時機**：當你想要對文字、表格或其他組件進行左縮排、右縮排或是上下留白，以創造出更好的視覺排版時使用。
- **語法**：`Padding(renderable, pad, style=None, expand=True, ...)`
- **參數說明**：
  - `renderable`：要包裹進行縮排的目標組件（字串、Table、Panel 等）。
  - `pad`：縮排設定值，支援多種整數或元組 (Tuple) 寫法：
    - `(統一留白量,)`：上下左右皆留白指定的量。（如 `(2,)`）
    - `(上下, 左右)`：垂直與水平分開設定。（如 `(1, 4)` 代表上下空 1 行，左右空 4 格）
    - `(上, 右, 下, 左)`：CSS 順時針風格精確控制。（如 `(0, 0, 0, 4)` 代表純粹左側縮排 4 格）
  - `style`：應用於縮排空白區域的樣式（例如設定背景顏色）。
- **回傳值**：
  - `Padding`：帶有內邊距的新物件，交由控制台輸出。

```python
...
from rich.console import Console
from rich.padding import Padding

console = Console()

# 專注展示純左側縮排效果
text = "這是一段被縮排的文字！"
padded_text = Padding(text, (0, 0, 0, 4)) # (上, 右, 下, 左)

console.print(padded_text)
...
```

---

##### Syntax() 語法高亮

- **使用時機**：想要在終端機印出一段程式碼，並希望它有 IDE 等級的色彩高亮與行號顯示時。
- **語法**：`Syntax(code: str, lexer_name: str, theme="monokai", line_numbers=False, ...)`
- **參數說明**：
  - `code`：原始碼字串。
  - `lexer_name`：語言名稱（如 `"python"`）。
  - `theme`：色彩主題。
  - `line_numbers`：布林值，是否顯示左側行號。
- **回傳值**：
  - `Syntax`：高亮物件，交由控制台輸出。

```python
...
from rich.console import Console
from rich.syntax import Syntax

console = Console()

code = "def hello():\\n    print('hi')"

# 專注展示將字串轉為 Python 語法高亮
syntax = Syntax(code, "python", theme="monokai", line_numbers=True)
console.print(syntax)
...
```

---

##### Markdown() 渲染 Markdown

- **使用時機**：想要在純文字的終端機內，直接把 Markdown 語法（標題、清單、粗體）解析並渲染出層次感時。
- **語法**：`Markdown(markup: str, ...)`
- **參數說明**：
  - `markup`：Markdown 格式的原始字串。
- **回傳值**：
  - `Markdown`：渲染物件。

```python
...
from rich.console import Console
from rich.markdown import Markdown

console = Console()

# 專注展示渲染 MD 文件
md_text = "# 標題\\n這是 **粗體** 文字。"
console.print(Markdown(md_text))
...
```

---

##### Tree() 樹狀結構

- **使用時機**：需要以階層式視覺化來展示資料夾結構、JSON 架構或任何具有父子關係的組織圖時。
- **語法**：`Tree(label, ...)`
- **參數說明**：
  - `label`：根節點的顯示文字或組件。
- **回傳值**：
  - `Tree`：樹狀根節點物件。需透過其 `.add()` 擴展子節點。

```python
...
from rich.console import Console
from rich.tree import Tree

console = Console()

# 1. 建立根節點
tree = Tree("📁 src")

# 2. 加入子節點
tree.add("📄 main.py")
tree.add("📄 utils.py")

console.print(tree)
...
```

---

## 終端單元格寬度 (Cells)

在終端機的世界中，字串的**「字元個數」**並不等於在螢幕上實際佔用的**「格子寬度（Cell Width）」**。

- **半形英數（ASCII 如 `a`, `1`）**：佔用 **1 個 cell（半形）**。
- **全形中文字、全形標點、日韓文（CJK）與 Emoji（如 `中`, `！`, `🚀`）**：佔用 **2 個 cell（全形 / 雙倍寬度）**。
- **零寬字元（Zero-width）**：佔用 **0 個 cell**。

如果使用 Python 內建的 `len("你好")` 會得到 `2`，但在終端畫面上它實際上佔了 **4 格寬度**。若直接拿 `len()` 來做字串對齊或補空格，排版必定會錯位歪掉！`rich.cells` 模組正是為了解決此痛點而生。

---

##### cell_len() 計算終端顯示寬度

- **使用時機**：當你需要精確得知一段字串在終端機螢幕上實際佔用多少個「單元格（Cells）寬度」時使用（特別是字串中混雜了中文、全形符號或 Emoji 時）。
- **語法**：`cell_len(text: str) -> int`
- **參數說明**：
  - `text`：要測量終端顯示寬度的字串。
- **回傳值**：
  - `int`：該字串在終端畫面上實際佔用的格子寬度（單元格數量）。

```python
...
from rich.cells import cell_len

# 1. 英文字串：字元數等於 cell 寬度
text_en = "Python"
print(len(text_en))       # 輸出: 6 (6 個字元)
print(cell_len(text_en))  # 輸出: 6 (佔用 6 格)

# 2. 中文字串：每個中文字佔 2 格 cell
text_zh = "你好世界"
print(len(text_zh))       # 輸出: 4 (4 個字元)
print(cell_len(text_zh))  # 輸出: 8 (佔用 8 格!)

# 3. 混雜 Emoji 的字串
text_emoji = "進度 🚀"
print(len(text_emoji))       # 輸出: 5 (字元數，含空白)
print(cell_len(text_emoji))  # 輸出: 7 (進度佔4 + 空格佔1 + 火箭佔2 = 7格)
...
```

> **💡 小提醒**：
> 在開發自訂終端表格、對齊標籤或進度條時，**請永遠使用 `cell_len()` 取代原生的 `len()`**，才能完美兼容中文與 Emoji 的視覺排版！
> （因為**中文佔據 2 格**，用 len() 會出錯）

---

##### set_cell_size() 固定單元格寬度

- **使用時機**：當你需要將字串強制調整為「指定格子寬度」時使用。若字串過短會自動用空格補齊；若字串過長則會智慧截斷（自動保護全形字不被切成半個亂碼）。
- **語法**：`set_cell_size(text: str, total: int) -> str`
- **參數說明**：
  - `text`：要調整寬度的原始字串。
  - `total`：期望在終端上呈現的精確單元格寬度（整數 `int`）。
- **回傳值**：
  - `str`：調整寬度後的字串。

```python
...
from rich.cells import set_cell_size, cell_len

# 1. 太短時：自動在右側補齊半形空格至剛好 10 格寬
short_text = set_cell_size("你好", 10)
print(f"'{short_text}'")        # 輸出: '你好      ' (4格中文 + 6格空格)
print(cell_len(short_text))     # 輸出: 10

# 2. 太長時：自動智慧截斷至剛好 6 格寬（不會切壞中文字）
long_text = set_cell_size("人工智慧模型", 6)
print(f"'{long_text}'")         # 輸出: '人工智' (剛好 6 格寬)
print(cell_len(long_text))      # 輸出: 6
...
```

---

##### chop_cells() 依單元格寬度切割字串

- **使用時機**：當你需要將一段長文字按照固定的「終端格子寬度」切成多行（例如實作自訂文字自動折行，或限制每行最大寬度）時使用。
- **語法**：`chop_cells(text: str, max_size: int, position: int = 0) -> list[str]`
- **參數說明**：
  - `text`：要進行切分的原始字串。
  - `max_size`：每段文字所允許的最大單元格寬度（`int`）。
  - `position`：起始位置偏移量（預設為 `0`）。
- **回傳值**：
  - `list[str]`：切分後的一組字串串列。

```python
...
from rich.cells import chop_cells, cell_len

text = "Rich終端美化套件非常強大！"

# 依每行最大 10 個單元格寬度進行切分
lines = chop_cells(text, max_size=10)

for line in lines:
    print(f"片段: '{line}' (寬度: {cell_len(line)})")
# 輸出範例:
# 片段: 'Rich終端' (寬度: 8)
# 片段: '美化套件' (寬度: 8)
# 片段: '非常強大' (寬度: 8)
# 片段: '！' (寬度: 2)
...
```

---

## 報錯與除錯 (Debug)

##### inspect() 物件探針

- **使用時機**：開發中途遇到不熟悉的物件，想快速印出它到底擁有什麼型別、方法與內部屬性時。
- **語法**：`inspect(obj, methods=False, ...)`
- **參數說明**：
  - `obj`：想要解剖的目標物件。
  - `methods`：布林值，`True` 代表連同方法一併列出。
- **回傳值**：
  - `None` (直接輸出至終端)。

```python
...
from rich import inspect

data = [1, 2, 3]

# 專注展示剖析 list 物件的內部方法
inspect(data, methods=True)
...
```

---

##### install() 全域報錯接管

- **使用時機**：希望當 Python 程式崩潰時，原本難懂的黑白 Traceback 錯誤堆疊，能被轉化成具有清楚高亮、標示出變數狀態的究極除錯畫面時。
- **語法**：`install(show_locals=False, ...)`
- **參數說明**：
  - `show_locals`：布林值，`True` 會將崩潰當下的所有區域變數數值印出，極度有助於除錯。
- **回傳值**：
  - `None`。

```python
...
from rich.traceback import install

# 專注展示全域接管未捕獲的例外報錯，並印出當下變數
install(show_locals=True)
...
```

---
