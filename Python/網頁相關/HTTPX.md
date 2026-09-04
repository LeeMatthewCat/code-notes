這是 Python 現代化、次世代的 HTTP 網路請求套件。

它是老牌套件 `requests` 的精神繼承者。不僅保留了幾乎一模一樣的簡單易用 API，更原生支援了 **非同步 (async/await)** 以及 **HTTP/2** 協定，是目前開發爬蟲與串接外部 Web API 的首選！

> **🍿 比喻單元**：
> 如果傳統的 `requests` 是一台**堅固耐用的手排老爺車**；
> 那麼 `httpx` 就是一台**配備自動駕駛 (非同步) 的特斯拉**，開法（語法）幾乎一樣，但速度與效能卻是劃時代的提升！

---

## 安裝與基礎載入

因為是第三方套件，需先透過 `pip` 安裝至虛擬環境中：

```Shell
pip install httpx
```

---

## 核心請求 API

如果你只是想要寫一個簡單的單次爬蟲，你可以直接使用模組層級的快捷函式。

##### httpx.get() 發送 GET 請求

- **使用時機**：當你想要向伺服器「索取」資料（例如：讀取網頁 HTML、抓取天氣 API 的 JSON 數據）時。
- **語法**：`httpx.get(url, params=None, headers=None, timeout=5.0, follow_redirects=False, ...)`
- **參數說明**：
  - `url`：目標網址字串。
  - `params`：放在網址後面的 Query String 參數字典（如 `?q=python`）。
  - `headers`：自訂的 HTTP 請求標頭字典（常用於偽裝成瀏覽器或夾帶 API Token）。
  - `timeout`：最大等待秒數，預設 5 秒。
  - `follow_redirects`：布林值，是否自動跟隨 301 / 302 重定向轉址（**預設 `False`**）。
- **回傳值**：
  - 回傳一個 `httpx.Response` 物件，它就像是伺服器寄回來的「包裹」。這個包裹裡裝了這幾樣重要的東西：
    - `.status_code` (int)：HTTP 狀態碼數字（如 `200` 代表成功，`404` 代表找不到，`500` 代表伺服器端錯誤）。
    - `.reason_phrase` (str)：HTTP 狀態碼對應的英文原因說明文字（如 `"OK"`、`"Not Found"`、`"Internal Server Error"`）。
    - `.text` (str)：伺服器回應的純文字內容（通常是 HTML 原始碼或純文字資料）。
    - `.json()` (method)：如果確定伺服器回傳的是 JSON 格式，呼叫這個方法會自動將其反序列化為 Python 字典 (Dictionary)。
    - `.headers` (dict)：伺服器回應的標頭資訊。

> **⚠️ 重大陷阱：`follow_redirects` 預設為 `False`**：  
> 與傳統 `requests` 套件（預設 `allow_redirects=True`）不同，HTTPX 的 `follow_redirects` **預設為 `False`**！若遇到 301/302 重定向（網址被永久暫時更改到另一網址），HTTPX 預設只會傳回 301/302 回應（並附帶 `Location` 標頭）而不會自動跳轉頁面。若需要自動跟隨重定向，**必須明確傳入 `follow_redirects=True`**！

```python
...
import httpx

# 專注展示發送夾帶參數的 GET 請求
headers = {"User-Agent": "My-Crawler/1.0"}
params = {"q": "Python"}

response = httpx.get("https://httpbin.org/get", params=params, headers=headers)
...
```

---

##### headers 請求標頭參數詳細說明

- **使用時機**：當你需要向伺服器證明身分（如帶入 Token）、偽裝成真實瀏覽器避免被反爬蟲封鎖，或指定傳輸格式時使用。
- **資料型別**：`dict[str, str]`（鍵值皆為字串的字典）。
- **實戰 4 大核心應用場景**：
  1. **User-Agent 偽裝**：防止被目標網站識別為 Python 爬蟲機器人。
  2. **Authorization 身分驗證**：夾帶 Bearer Token 或 API Key 通行證。
  3. **Referer 防盜鏈繞過**：模擬從官網點擊進來的請求來源網址。
  4. **Accept / Content-Type 格式指定**：指定期待接收 JSON 或表單格式。

> **🍿 生動白話比喻**：  
> `headers` 就像是去大樓會客時佩戴的**「識別證與偽裝面具」**！  
> 如果你不戴（預設 headers），保安（反爬蟲系統）一看到你身上印著 `python-httpx/0.x` 就會直接把你擋在門外；帶上 `headers` 就等於戴上 `User-Agent: Chrome` 的面具，並出示 `Authorization: Bearer <Token>` 的通行證！

```python
import httpx

# 建立實戰等級的 headers 字典
custom_headers = {
    # 1. 偽裝成桌面版 Chrome 瀏覽器
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
    
    # 2. 夾帶 API 驗證 Token (Bearer 認證)
    "Authorization": "Bearer secret_api_token_123456",
    
    # 3. 指定期待接收 JSON 資料
    "Accept": "application/json",
    
    # 4. 繞過圖片防盜鏈 (模擬從官網點擊進來)
    "Referer": "https://example.com/"
}

response = httpx.get("https://httpbin.org/headers", headers=custom_headers)
print(response.json())
```

> **💡 Client 全域 Headers 技巧**：  
> 如果使用 `httpx.Client(headers=global_headers)`，全域設定的 `headers` 會自動應用到該 Client 發出的**每一次請求**中，不需要每次 `client.get()` 都重複傳入！

---

##### httpx.post() 發送 POST 請求

- **使用時機**：當你需要向伺服器「提交」資料（例如：送出登入表單、上傳檔案、建立新文章）時。
- **語法**：`httpx.post(url, data=None, json=None, headers=None, ...)`
- **參數說明**：
  - `data`：用於發送傳統 HTML 表單資料 (`application/x-www-form-urlencoded`)。
  - `json`：將字典自動序列化為 JSON 格式發送（現代 API 開發最常用）。
- **回傳值**：
  - `Response`：包含伺服器回應結果的物件。

```python
...
import httpx

# 專注展示發送 JSON 格式的 POST 請求
payload = {"username": "matthew", "password": "123"}
response = httpx.post("https://httpbin.org/post", json=payload)
...
```

---

## 回應結果解析 (Response)

當請求發出後，伺服器會回傳一個 `Response` 物件。你可以透過下列屬性與方法解析回傳內容：

##### .text 回應解碼字串屬性

- **使用時機**：當你需要讀取網頁的 HTML 原始碼、純文字、CSS 或 JavaScript 檔案內容時使用。
- **資料型別**：`str`（已解碼的 Unicode 字串）。
- **屬性特徵與重要警告**：
  - ⚠️ **絕對不能加括號 `()`**：`.text` 是屬性 (Property) 而非方法 (Method)。呼叫 `response.text()` 會觸發 `TypeError: 'str' object is not callable` 錯誤！
  - **自動編碼與解碼機制**：HTTPX 會讀取原始二進位位元組 (`.content`)，並依據回應標頭中的 `Content-Type`（如 `charset=utf-8`）或自動推測編碼，將其轉為字串。
- **亂碼解決方案（手動指定 `.encoding`）**：
  若存取舊版網站（如使用 `Big5` 或 `GBK` 編碼）導致 `.text` 顯示為中文亂碼時，可以在讀取 `.text` 前手動修正編碼：
  ```python
  response.encoding = "big5"  # 手動指定為 Big5 編碼
  print(response.text)        # 再次存取即可正常顯示中文！
  ```

- **三種 Response 內容讀取屬性對比**：

| 屬性/方法 | 回傳型別 | 適用情境與檔案型別 | 語法範例 |
| :--- | :--- | :--- | :--- |
| **`.text`** | `str` (字串) | 網頁 HTML 原始碼、純文字、CSS / JS | `print(response.text)` *(無括號)* |
| **`.content`** | `bytes` (位元組) | 下載圖片、PDF、音訊、影音與 ZIP 壓縮包 | `file.write(response.content)` *(無括號)* |
| **`.json()`** | `dict` / `list` | 呼叫 REST API 取得 JSON 資料 | `data = response.json()` *(有括號)* |

```python
import httpx

response = httpx.get("https://httpbin.org/html")

# 1. 正確存取 response.text 屬性 (取得 HTML 原始碼字串)
html_code = response.text
print(f"網頁 HTML 長度：{len(html_code)} 個字元")

# 2. 檢視與手動調整編碼 (解決中文亂碼狀況)
print(f"當前自動識別編碼：{response.encoding}")
if response.encoding != "utf-8":
    response.encoding = "utf-8"  # 強制指定 UTF-8 解碼

# 3. ❌ 常見寫法陷阱 (切勿加括號！)
# html = response.text()  # 錯誤！引發 TypeError: 'str' object is not callable
```

---

##### .content 原始二進位位元組屬性

- **使用時機**：當你需要下載或處理**非文字二進位檔案**（例如：圖片 JPG/PNG、PDF 文件、影音 MP3/MP4、ZIP 壓縮檔等）時使用。
- **資料型別**：`bytes`（原始二進位位元組數據）。
- **屬性特徵與重要警告**：
  - ⚠️ **絕對不能加括號 `()`**：`.content` 是屬性 (Property) 而非方法 (Method)。呼叫 `response.content()` 會觸發 `TypeError: 'bytes' object is not callable` 錯誤！
  - **最基層的真實數據**：`.content` 是伺服器透過網路傳送過來、未經任何文字編碼轉譯的原始 Raw Bytes。

```python
import httpx

# 發送請求下載 PNG 圖片
response = httpx.get("https://httpbin.org/image/png")

# 1. 存取二進位位元組數據 (bytes)
image_bytes = response.content
print(f"圖片檔案大小：{len(image_bytes)} bytes")

# 2. 必須以二進位寫入模式 ('wb') 將檔案存入硬碟
with open("logo.png", "wb") as f:
    f.write(response.content)
```

---

##### .encoding 文字編碼屬性

- **使用時機**：當你需要檢查 HTTPX 自動推測的字元編碼，或是發現網頁輸出中文亂碼（如舊版網站使用 Big5、GBK），需要**手動指定正確編碼**時使用。
- **資料型別**：`str`（例如 `"utf-8"`、`"big5"`、`"gbk"`、`"gb2312"`）。
- **屬性特徵與動態覆寫**：
  - ⚠️ **絕對不能加括號 `()`**：`.encoding` 是可讀寫的屬性 (Property)。呼叫 `response.encoding()` 會觸發 `TypeError: 'str' object is not callable` 錯誤！
  - **可讀可寫 (Get / Set)**：不僅可以印出來查看 `print(response.encoding)`，還可以直接給它賦值修改 `response.encoding = "big5"`。
- **HTTPX 自動識別編碼的 3 階機制**：
  1. **HTTP Header 優先**：優先讀取伺服器回應標頭 `Content-Type` 中的 `charset` 設定（如 `text/html; charset=utf-8`）。
  2. **內文自動推測**：若標頭無宣告，HTTPX 會剖析 HTML 內文與二進位字節來推測編碼。
  3. **預設降級 (Fallback)**：若完全無法識別，預設降級為 `"utf-8"`。

> **🍿 白話比喻：翻譯官的字典**：  
> `response.content` 是外國人講的**原始語音檔 (bytes)**；  
> `response.encoding` 就是告訴翻譯官**「現在要用哪一本字典 (UTF-8 / Big5) 來翻譯」**；  
> `response.text` 則是翻譯官最後翻出來的**中文譯本 (str)**！  
> 如果翻譯官用錯字典（例如拿 UTF-8 字典去翻 Big5 語音），譯本 (`.text`) 就會全變成天書亂碼！此時你只要手動換掉字典（`response.encoding = "big5"`），譯本就會瞬間恢復正常！

> **💡 底層揭密：`.text`、`.encoding` 與 `.content.decode()` 的關係**：  
> - `.encoding` **只是一個變數標籤（設定樣式）**！單獨修改它（如 `response.encoding = "big5"`）**完全不會改變 `.content` (原始 bytes 資料)**！
> - `.text` 的底層實作本質，就是呼叫 Python 原生的 `.decode()`：  
>   `.text` $\equiv$ `.content.decode(response.encoding)`
> - **兩大完全等價的解碼寫法**：
>   - **寫法 A（修改編碼標籤，存取 `.text`）**：
>     ```python
>     response.encoding = "big5"
>     text = response.text  # 底層自動執行 response.content.decode("big5")
>     ```
>   - **寫法 B（手動 `.content.decode()`，適合處理壞字節）**：
>     ```python
>     # 直接對原始 bytes 手動指定編碼解碼，還可傳入 errors="ignore" 容錯防崩潰
>     text = response.content.decode("big5", errors="ignore")
>     ```

> **🛡️ 強大防崩潰武器：`errors="ignore"` 容錯解碼參數**：  
> - **問題背景**：爬取舊版網頁（如 Big5 / GBK）時，頁面內常混雜著無法被該字典識別的日文假名、表情符號或傳輸破損的壞位元組 (Bad Bytes)。預設情況下（`errors="strict"`），只要碰到一個無法識別的壞字節，Python 就會直接拋出 `UnicodeDecodeError` 導致整台爬蟲程式崩潰！
> - **`errors="ignore"` 作用**：命令 Python **「遇到看不懂或損壞的壞位元組直接無視跳過（Ignore），繼續解碼後續文字，絕對不要報錯崩潰！」**
> - **`errors="replace"` 替代方案**：遇到壞字節時不跳過，而是將壞字節替換為黑底問號 ``，避免漏字。
> - **實戰防崩潰寫法**：
>   ```python
>   # 強力防崩潰解碼組合拳：指定 Big5 並無視無效壞字節
>   clean_text = response.content.decode("big5", errors="ignore")
>   ```

> **⚠️ 重大觀念澄清：`response.encoding` 僅對文字有效，無法涵蓋二進位檔案！**：  
> - **文字檔才有 encoding**：`response.encoding` 專門服務於 `.text`。只有 HTML、TXT、CSS、JSON 等「文字檔」才存在字元編碼 (UTF-8 / Big5) 的概念。  
> - **二進位檔案完全無效**：圖片 (PNG/JPG)、PDF、影音 (MP4/MP3)、ZIP 壓縮檔等**二進位檔案完全沒有字元編碼的概念**！如果對圖片硬要讀取 `.text` 或設定 `response.encoding`，只會得到一堆不可讀的亂碼甚至破壞二進位結構。**處理非文字檔案時，請永遠使用 `response.content` 並以二進位模式 (`wb`) 寫入硬碟！**

```python
import httpx

# 假設請求一個舊版 Big5 編碼的繁體中文網站
response = httpx.get("https://example-big5-site.com.tw")

# 1. 檢視 HTTPX 自動識別出的編碼
print(f"原始識別編碼：{response.encoding}")

# 2. 若自動識別錯誤導致 response.text 出現亂碼，手動覆寫為 Big5
response.encoding = "big5"

# 3. 再次讀取 .text，即可以正確的 Big5 解碼顯示中文！
print(response.text)
```

---

##### .json() 解析 JSON 數據

- **使用時機**：當你確信伺服器回傳的是 JSON 格式的數據時，用來將其自動轉換為 Python 的字典 (`dict`) 或串列 (`list`)。
- **語法**：`response.json()`
- **參數說明**：無。
- **回傳值**：
  - `dict` 或 `list`：解析完畢的 Python 資料結構。

```python
...
response = httpx.get("https://jsonplaceholder.typicode.com/todos/1")

# 專注展示提取 JSON 數據
data = response.json()
print(data["title"])  # 直接以字典方式操作
...
```

---

##### .raise_for_status() 檢查錯誤碼

- **使用時機**：每次發起請求後，第一時間確保伺服器沒有報錯（例如 404 找不到、500 伺服器崩潰）。若有錯則立即中斷程式。
- **語法**：`response.raise_for_status()`
- **參數說明**：無。
- **回傳值**：無。若狀態碼為 4xx 或 5xx，會直接拋出 `HTTPStatusError` 異常。

```python
...
response = httpx.get("https://httpbin.org/status/404")

# 專注展示攔截 HTTP 錯誤
try:
    response.raise_for_status()
except httpx.HTTPStatusError as e:
    print(f"糟糕，連線失敗！狀態碼：{e.response.status_code}")
...
```

> **💡 小提醒**：`httpx` 預設就算遇到 `404 Not Found`，也不會讓程式崩潰，而是默默回傳一個 Response 給你。強烈建議你在取得 response 後，立刻呼叫 `raise_for_status()` 來排雷！

---

## 進階效能：連線池與非同步

如果你要撰寫的是**專業爬蟲**或**需要連續發送大量請求**的系統，絕對不要使用上面介紹的 `httpx.get()`，因為它每次都會重新建立與關閉連線（非常消耗資源）。你必須改用 Client 連線池！

##### httpx.Client() 同步連線池管理器

- **使用時機**：你需要對同一個網站連續發出 3 個以上的請求，希望能自動保持連線 (Keep-Alive) 以及共用 Cookie 時。
- **語法**：`httpx.Client(...)`
- **參數說明**：
  - 多數參數與 `get()` 相同，但可以傳入全域的 `headers` 或 `base_url` 供所有請求共用。
- **回傳值**：
  - `Client`：一個連線池客戶端，需搭配 `with` 上下文管理器使用。

```python
...
import httpx

# 專注展示使用連線池發送連續請求
with httpx.Client(base_url="https://httpbin.org") as client:
    # 這兩次請求將共用同一條底層 TCP 連線，速度極快！
    r1 = client.get("/get")
    r2 = client.post("/post", json={"hello": "world"})
...
```

---

##### httpx.AsyncClient() 非同步連線池管理器

- **使用時機**：你需要撰寫極致效能的非同步爬蟲，想要在短短幾秒內併發抓取 100 個以上的網頁時！
- **語法**：`httpx.AsyncClient(...)`
- **參數說明**：與同步 Client 相同。
- **回傳值**：
  - `AsyncClient`：非同步的連線池客戶端，需搭配 `async with` 與 `await` 使用。

```python
...
import httpx
import asyncio

async def fetch_data():
    # 專注展示建立非同步 Client
    async with httpx.AsyncClient() as client:
        # ⚠️ 注意這裡必須加上 await 才能真正發出請求
        response = await client.get("https://httpbin.org/get")
        print(response.status_code)

# 執行非同步函式
asyncio.run(fetch_data())
...
```

---

## 常見異常處理 (Exceptions)

當你爬蟲打錯網址，或者是對方的伺服器掛掉時，HTTPX 就會拋出例外錯誤。強烈建議使用 `try...except` 搭配這些專屬錯誤來攔截災情，防止程式崩潰。

> **🍿 比喻單元：HTTPX 例外樹狀繼承結構**：  
> - `HTTPError` (**H3 頂層父類別**：包含所有 HTTPX 內部產生的例外)  
>   ├── `RequestError` (**H4 父類別**：網路傳輸與請求層級錯誤)  
>   │     ├── `InvalidURL` (**H5 子類別**：網址格式錯誤)  
>   │     ├── `ConnectError` (**H5 子類別**：連線失敗)  
>   │     └── `TimeoutException` (**H5 子類別**：請求逾時)  
>   └── `HTTPStatusError` (**H4 父類別**：HTTP 狀態碼 4xx / 5xx 業務錯誤)

---

### httpx.HTTPError 頂層例外基類 (所有 HTTPX 例外總司令)

- **使用時機**：當你想**無腦包攬捕捉 HTTPX 拋出的任何例外**（不論是網址寫錯、網路通訊失敗，還是 4xx/5xx 狀態碼錯誤）時使用。
- **語法**：放在 `except httpx.HTTPError as exc:` 區塊中捕捉。

```python
import httpx

try:
    response = httpx.get("https://httpbin.org/status/500")
    response.raise_for_status()
    
except httpx.HTTPError as exc:
    print(f"🛡️ 成功攔截任何 HTTPX 相關錯誤：{type(exc).__name__} -> {exc}")
```

---

#### httpx.RequestError 網路與請求層例外基類

- **使用時機**：當你希望**一次捕捉所有網路傳輸與請求層級的錯誤**（包含 `InvalidURL`、`ConnectError`、`TimeoutException` 等），但不想捕捉狀態碼錯誤 (4xx/5xx) 時使用。
- **語法**：放在 `except httpx.RequestError as exc:` 區塊中捕捉。

```python
import httpx

try:
    response = httpx.get("https://this-is-a-fake-domain.xyz")
    
except httpx.RequestError as exc:
    print(f"📡 網路請求層級錯誤 (包含網址錯誤、連不上、逾時等)：{exc}")
```

##### httpx.InvalidURL 無效網址格式

- **使用時機**：當傳入的網址字串格式不符合 URL 規範（例如：忘了寫 `http://` 或 `https://` 協定標頭、網址夾帶不合法字元、連線埠無效）時拋出。
- **語法**：放在 `except httpx.InvalidURL as exc:` 區塊中捕捉。
- **與 ConnectError 的關鍵差別**：
  - `InvalidURL`：**本地端解析網址即宣告失敗**，完全沒有發出任何網路請求。
  - `ConnectError`：網址語法正確且請求已發出，但**目標伺服器未回應、沒開機或 DNS 找不到**。

```python
import httpx

try:
    # ❌ 錯誤寫法：網址忘了寫 "https://" 協定開頭
    response = httpx.get("httpbin.org/get")
    
except httpx.InvalidURL as exc:
    print(f"⚠️ 網址格式無效！請檢查是否遺漏了 http:// 或 https://。詳細：{exc}")
```

---

##### httpx.ConnectError 網路連線失敗

- **使用時機**：當你要捕捉「**完全連不上目標伺服器**」（例如：你斷網了、對方伺服器關機、DNS 找不到該網域、連接埠沒開）的嚴重錯誤時。
- **語法**：放在 `except httpx.ConnectError as exc:` 區塊中捕捉。

```python
import httpx

try:
    # 故意打一個不存在的假網址，或者伺服器根本沒開的 port
    response = httpx.get("https://this-is-a-fake-domain.xyz")
    
except httpx.ConnectError as exc:
    print(f"🚨 嚴重錯誤：完全連不上伺服器！可能是你斷網了，或網域不存在。詳細：{exc}")
```

---

##### httpx.TimeoutException 請求逾時

- **使用時機**：當發送請求後，伺服器處理太慢，超過你所設定的 `timeout` 秒數（預設為 5.0 秒）仍未完成回應時拋出。
- **語法**：放在 `except httpx.TimeoutException as exc:` 區塊中捕捉。

```python
import httpx

try:
    # 故意設定極短的 0.001 秒 timeout 觸發逾時
    response = httpx.get("https://httpbin.org/delay/2", timeout=0.001)
    
except httpx.TimeoutException as exc:
    print("⏰ 請求逾時：伺服器回應太慢，超過設定時間！")
```

---

#### httpx.HTTPStatusError HTTP 狀態碼異常

- **使用時機**：當你主動呼叫 `response.raise_for_status()` 且伺服器回傳 4xx（如 404 找不到、403 禁止存取）或 5xx（500 伺服器崩潰）等非 2xx 狀態碼時拋出。
- **語法**：`response.raise_for_status()` 搭配 `except httpx.HTTPStatusError as exc:`。
- **重要屬性**：
  - `.response`：引發錯誤的 Response 物件（可存取 `.response.status_code` 狀態碼數字、`.response.reason_phrase` 文字說明）。
  - `.request`：發出該請求的 Request 物件。

```python
import httpx

try:
    response = httpx.get("https://httpbin.org/status/404")
    # 若狀態碼為 4xx 或 5xx，此行會主動引發 HTTPStatusError
    response.raise_for_status()
    
except httpx.HTTPStatusError as exc:
    # exc.response.reason_phrase 會印出對應的英文說明 "Not Found"
    print(f"🚨 HTTP 狀態碼錯誤：{exc.response.status_code} {exc.response.reason_phrase}")
    print(f"🔗 請求網址：{exc.request.url}")
```

---
