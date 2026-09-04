# 概念與原理

## 什麼是 readability？

Python 中最知名的**網頁正文內容自動提取與去雜訊工具**（基於 `readability-lxml`）。

源自 Mozilla 瀏覽器內建的「閱讀模式 (Reader View)」演算法。它能自動分析網頁 DOM 樹與文字密度，將網頁中與核心內容無關的**廣告、導覽列 (Navbar)、側邊欄 (Sidebar)、頁尾 (Footer) 與彈出選單**自動剝離，精準萃取出文章真正的「標題 (Title)」與「正文 HTML (Summary)」！

在 AI / LLM 爬蟲開發中，它是解決網頁雜訊干擾的核心利器，常與 [[HTTPX]] 及 [[html2text]] 組成網頁文本清洗的黃金三劍客！

> **🍿 生動白話比喻**：  
> 如果抓下來的網頁 HTML 是**包裝極度過度、滿是廣告傳單與廢報紙包裹的大箱子**；  
> 那麼 `readability` 就是**「專業 X 光掃描過濾器」**！它能直接穿透外層無用的廢紙廣告，一秒幫你把箱子裡面最珍貴的寶物（新聞文章正文）精準夾出來！

---

## 安裝與基礎載入

`readability` 套件在 PyPI 上的名稱為 `readability-lxml`，需先透過 `pip` 安裝：

```Shell
pip install readability-lxml
```

> **💡 注意匯入名稱**：套件安裝名稱為 `readability-lxml`，但在 Python 程式碼中匯入時請使用 `from readability import Document`！

---

## 核心 API 與用法解析

`readability` 的核心為 `Document` 類別，透過傳入原始 HTML 字串建立文件物件進行剖析。

##### Document() 建立網頁文件剖析器

- **使用時機**：當你需要將 `httpx` 或 `requests` 抓取到的網頁原始 HTML 字串進行正文演算法剖析時使用。
- **語法**：`Document(input: str, min_text_length=25, retry_on_unk_tags=True, ...)`
- **參數說明**：
  - `input`：必填，要剖析的完整網頁 HTML 字串（如 `response.text`）。
  - `min_text_length`：（可選）正文文字最小長度閥值，預設為 25 個字元。
  - `retry_on_unk_tags`：（可選）遇到未知 HTML 標籤時是否重試剖析，預設 `True`。
- **回傳值**：
  - `Document`：回傳一個網頁文件剖析器實例物件（通常命名為 `doc`）。

---

##### doc.title() 提取網頁正文標題

- **使用時機**：當你需要從網頁中精準提取出文章的核心標題，並自動去除網站域名後綴時使用。
- **語法**：`doc.title()`
- **參數說明**：無參數。
- **回傳值**：
  - `str`：提取出的文章標題字串。

---

##### doc.summary() 提取網頁純淨正文 HTML

- **使用時機**：當你需要剝離導覽列與廣告，僅保留包含文章正文標籤（如 `<p>`, `<h1>`, `<img>`）的乾淨 HTML 時使用。
- **語法**：`doc.summary(html_partial=False)`
- **參數說明**：
  - `html_partial`：（可選）布林值，若設為 `True` 則僅回傳 `<div>` 正文區塊；預設 `False` 會包裹完整的 `<html><body>` 結構。
- **回傳值**：
  - `str`：經演算法過濾與去雜訊後的純淨正文 HTML 字串。

```python
import httpx
from readability import Document

# 1. 使用 HTTPX 抓取目標新聞網頁
url = "https://news.example.com/article/123"
response = httpx.get(url, follow_redirects=True)

# 2. 將 HTML 傳入 Document 建立剖析器
doc = Document(response.text)

# 3. 提取文章標題與純淨正文 HTML
article_title = doc.title()
clean_html = doc.summary()

print(f"📰 文章標題: {article_title}")
print(f"📄 正文 HTML 長度: {len(clean_html)} 字元 (已自動剝離廣告與側邊欄)")
```

---

## 實戰黃金組合：HTTPX + Readability + html2text (LLM 爬蟲三劍客)

將三個工具串聯，打造極致乾淨、適合直接餵給 AI (ChatGPT/Claude/RAG) 的 Markdown 文章提取流程：

```python
import httpx
import html2text
from readability import Document

def fetch_clean_article(url: str) -> dict:
    """下載網頁並提取乾淨的 Markdown 正文"""
    # Step 1: 發送請求抓取網頁
    response = httpx.get(url, headers={"User-Agent": "Mozilla/5.0"}, follow_redirects=True)
    response.raise_for_status()
    
    # Step 2: 使用 Readability 剝離廣告與側邊欄，萃取正文 HTML
    doc = Document(response.text)
    title = doc.title()
    clean_html = doc.summary()
    
    # Step 3: 使用 html2text 將正文 HTML 轉為高品質 Markdown
    h = html2text.HTML2Text()
    h.ignore_links = True    # 依需求可去除超連結
    h.ignore_images = True   # 去除圖片
    h.body_width = 0         # 關閉 78 字元自動換行限制
    
    markdown_content = h.handle(clean_html)
    
    return {
        "title": title,
        "markdown": markdown_content.strip()
    }

# 測試調用
# result = fetch_clean_article("https://news.ycombinator.com")
# print(result["title"])
# print(result["markdown"])
```

> **⚠️ 致命陷阱：短文字與動態渲染網頁**：  
> 1. **短文字網頁判定失敗**：如果目標網頁文字量極少（低於 25 字），Readability 演算法可能會判定找不到正文而傳回空內容。  
> 2. **JavaScript 動態渲染**：Readability 只處理靜態 HTML。若網頁內容是透過 React/Vue 在前端動態加載的，需先配合 Playwright 等工具渲染出完整 HTML 後再傳入 `Document`！
