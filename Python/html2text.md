# 概念與原理

## 什麼是 html2text？

Python 中最經典的 HTML 轉 Markdown 解析工具。

它能將複雜、囉唆的 HTML 網頁原始碼，自動剝離 `<script>`、`<style>` 與無用標籤，並將其完美轉譯為乾淨、語意化的 Markdown 文字格式！

在 AI 時代，它是 LLM 大語言模型爬蟲與 RAG（檢索增強生成）前處理的最佳神器，能大幅節省 Token 消耗並提升文本讀取品質！

> **🍿 生動白話比喻**：  
> 如果傳統的 HTML 網頁原始碼是**包含骨架、水泥、雜線與油漆的毛胚屋**；  
> 那麼 `html2text` 就是**自動精裝裝潢師**！把所有無用的泥作雜物拆掉，只留下結構最清晰、閱讀最舒服的 Markdown 木質家具（標題、內文、清單與表格）！

---

## 安裝與基礎載入

`html2text` 為第三方套件，需先透過 `pip` 安裝至虛擬環境中：

```Shell
pip install html2text
```

---

## 核心 API 快捷函式

如果你只需要進行單次、基礎的 HTML 轉 Markdown，可以直接使用快捷函式。

##### html2text.html2text() 單次轉換快捷函式

- **使用時機**：當你只需要快速將一段 HTML 字串轉為預設設定的 Markdown 文字，不需要自訂複雜過濾屬性時。
- **語法**：`html2text.html2text(html: str, bodywidth=None)`
- **參數說明**：
  - `html`：要轉換的 HTML 原始碼字串。
  - `bodywidth`：（可選）指定每行最大字元數，超過會自動換行。
- **回傳值**：
  - `str`：轉換後的 Markdown 格式字串。

```python
import html2text

html_content = "<h1>歡迎來到 Python 世界</h1><p>這是一段 <b>粗體文字</b> 與 <a href='https://python.org'>連結</a>。</p>"

# 單次快速轉換
markdown_text = html2text.html2text(html_content)
print(markdown_text)
# 輸出:
# # 歡迎來到 Python 世界
#
# 這是一段 **粗體文字** 與 [連結](https://python.org)。
```

---

## 靈活控制項 HTML2Text 類別

當你需要精細控制轉譯細節（例如過濾圖片、去除超連結、關閉自動換行）時，必須建立 `HTML2Text` 實例物件。

##### HTML2Text() 建立轉譯器實例

- **使用時機**：當你需要針對特定爬蟲情境進行客製化設定（如 LLM 文本清洗）時使用。
- **語法**：`h = html2text.HTML2Text()`
- **參數說明**：無參數（屬性由實例物件後續設定）。
- **回傳值**：
  - `HTML2Text`：回傳一個轉換器實例物件。

---

##### HTML2Text 控制屬性字典對照表
用**屬性**的方式做設定
```Python
h = html2text.HTML2Text()

h.ignore_images = True
...
```
在建立 `h = html2text.HTML2Text()` 實例後，常用於控制轉譯行為的設定屬性（屬性名稱後面**絕對不加括號**）：

| 屬性名稱 | 資料型別 | 預設值 | 屬性作用與說明 |
| :--- | :--- | :--- | :--- |
| **`ignore_links`** | `bool` | `False` | **是否忽略超連結**。若設為 `True`，會移除 `[標題](url)` 語法，僅保留純標題文字。 *(LLM 清洗常用)* |
| **`ignore_images`** | `bool` | `False` | **是否忽略圖片**。若設為 `True`，會自動過濾 `![alt](url)` 圖片標籤。 |
| **`ignore_emphasis`** | `bool` | `False` | **是否忽略斜體/粗體**。若設為 `True`，會移除 `**` 與 `*` 強調標記。 |
| **`ignore_tables`** | `bool` | `False` | **是否忽略表格**。若設為 `True`，會直接抹除 HTML 中的表格元素。 |
| **`body_width`** | `int` | `78` | **自動換行字元寬度**。設為 `0` 可**關閉自動換行**（保持單行不被硬拆，LLM 最推薦）。 |
| **`single_line_break`**| `bool` | `False` | **是否將單一 `<br>` 轉為單一換行**。設為 `True` 可防止段落間產生過多空行。 |
| **`bypass_tables`** | `bool` | `False` | **是否保留 HTML 原生表格**。若設為 `True`，不會將表格轉為 Markdown `|` 語法。 |

> **⚠️ 重大陷阱與深解：`body_width` 自動換行切割**：  
> - **字面含義**：`body_width` 代表「文字內文每一行的最大字元寬度限制」。
> - **預設行為 (78 字元)**：`html2text` 預設 `body_width = 78`。只要轉換出來的文字在單一行內超過 78 個字元，它就會**強制插入換行符號 `\n`，將後續文字拆斷至下一行**（原意是防止舊終端機或 Email 文字跑出螢幕外）。
> - **直觀對比示範**：
>   - ❌ **預設 `78` 效果**（句子被硬生生切為兩行）：
>     ```text
>     Python 是一種廣泛使用的程式語言，其設計哲學
>     強調程式碼的可讀性與簡潔的語法。
>     ```
>   - ✅ **設定 `0` 效果**（無限制，長句子完整維持單行）：
>     ```text
>     Python 是一種廣泛使用的程式語言，其設計哲學強調程式碼的可讀性與簡潔的語法。
>     ```
> - **LLM 爬蟲黃金設定**：將此屬性設定為 `0`（`h.body_width = 0`），代表**「關閉換行限制（寬度不限）」**。這樣能防止長句子被切斷，確保送給 AI (ChatGPT/Claude/RAG) 時獲得最完整流暢的文本語意！

> **💡 屬性深解：`single_line_break` 單一換行緊湊模式**：  
> - **字面含義**：`single_line_break` 代表「是否使用單一換行符號（`\n`）替代預設的雙換行（`\n\n`）」。
> - **預設行為 (`False`)**：`html2text` 預設遵守標準 Markdown 段落規範。遇到 HTML 中的 `<br>` 或換行標籤時，會在文字行之間**輸出兩個換行符 `\n\n`（產生一行空白行）**。
> - **直觀對比示範**（針對 HTML 內容 `第一行<br>第二行<br>第三行`）：
>   - ❌ **預設 `False` 效果**（間距鬆散，產生多餘空白行）：
>     ```text
>     第一行
> 
>     第二行
> 
>     第三行
>     ```
>   - ✅ **設定 `True` 效果**（排版緊湊，僅單一換行）：
>     ```text
>     第一行
>     第二行
>     第三行
>     ```
> - **使用建議**：若抓取的 HTML 網頁含有大量 `<br>`（如詩詞、地址、日誌或條列選單），且不希望轉出來的 Markdown 充滿稀疏空行時，手動設定 `h.single_line_break = True` 能讓文本緊湊又好看！

---

##### h.handle() 執行 HTML 轉換

- **使用時機**：完成 `HTML2Text` 實例屬性設定後，傳入 HTML 字串執行最終轉譯時。
- **語法**：`h.handle(html: str)`
- **參數說明**：
  - `html`：要進行轉譯的原始 HTML 字串。
- **回傳值**：
  - `str`：經客製化過濾後的 Markdown 格式字串。

```python
import html2text

html_data = """
<div class="article">
    <h2>LLM 爬蟲清洗示範</h2>
    <p>點擊 <a href="https://example.com">這裡</a> 查看詳情。</p>
    <img src="banner.jpg" alt="封面圖">
</div>
"""

# 1. 建立轉譯實例
h = html2text.HTML2Text()

# 2. 客製化屬性設定 (LLM 前處理黃金組合)
h.ignore_links = True    # 去除超連結
h.ignore_images = True   # 去除圖片
h.body_width = 0         # 關閉 78 字元硬換行

# 3. 執行轉換
clean_markdown = h.handle(html_data)

print(clean_markdown)
# 輸出:
# ## LLM 爬蟲清洗示範
#
# 點擊 這裡 查看詳情。
```
