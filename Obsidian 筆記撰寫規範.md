# Obsidian 筆記撰寫與格式化規範指南 (Python.md 風格)

本指南根據 `Python/Python.md` 的實際語法與視覺排版提煉而成，定義了在 Obsidian Vault 中建立、編輯與重構技術筆記時必須嚴格遵守的格式與結構標準。

---

## 1. 核心編輯鐵律：未收到指令部分保持原樣 (Preserve Unspecified Content)

1. **嚴禁擅自更動未受指定部分**：
   - 當使用者指定修改筆記中某一特定段落、標題或屬性時，**沒有收到修改指令的部分一律嚴禁隨意變動、重新編排或移位（除非明確指示重新編輯），必須 100% 保持原樣**！

---

## 2. 筆記文筆風格與語氣規範 (Writing Style & Tone)

1. **極簡短句與高頻分段 (Short Sentences & High-Frequency Line Breaks)**：
   - 避免使用長篇大論的複合句與冗長段落。
   - 每句話控制在 15–30 字內，善用斷行與空行，創造極高的閱讀呼吸感。
   - **語氣特徵**：客觀、精煉、直刀入木（如「所有的東西（包括資料）都是一個物件」、「也就是多個名稱可指向同一個值」）。

2. **漸進式敘事邏輯 (Progressive Narrative Sequence)**：
   - 內文段落鋪陳固定遵循：**核心定義 ➔ 箇條說明 ➔ 程式碼證明 ➔ 陷阱提醒**。

---

## 3. 標題與分頁結構規範 (Header & Sectioning Rules)

1. **標題層級與分佈方式 (Header Distribution)**：
   - 舉例 os 模組：
	   模仿以下分類方式，並做出**彈性調整**
	   ＃os.environ 環境變數
		   對環境變數作出解釋
	   ＃＃ os.environ.get() 取得環境變數
	   ＃＃ os.environ[] 更改環境變數 
	   ＃資料相關
	   ＃＃ os.makedirs() 建立資料夾
	   ＃＃資料路徑
	   ＃＃＃ os.path.dirname() 去掉最尾
	   ＃＃./ .. 目前資料夾／上層資料夾  

2. **標題命名與符號規範**：
   - 函數/方法標題名稱後面 **必須加上小括號 `()`**（例如 `## client.models.generate_content()`）。
   - 屬性/變數標題名稱後面 **絕對不能帶有括號**（例如 `function_calls`）。
   - 標題內包含中英文時，使用半角空格或斜線分隔（如 `## 可變 Mutable／不可變 Immutable`）。
   - 當此章節介紹的是這個函數時就應該要把章節名稱命名為：**函數() 函數功能**
   - **盡可能讓每個函數都有獨立的一個章節標題**，但可把相似的做合併，如果有像同為  os.environ 開頭的（ os.environ.get() 和  os.environ[]）在上方整合出一個上層標題- os.environ 環境變數

3. **避免檔名和一級標題重複**，通常一個模組的筆記檔名本身就是這個模組的名字，不避再 # 模組名
4. **標題層級分割線 `---`**：
   - 在主要章節與次要主題區塊（通常為 H2 或 H3 區塊）結束後，使用獨立一行的水平線 `---` 進行視覺分隔，前後保持空行。

---

## 4. 引用塊與小提醒規範 (Blockquotes & Callouts)

1. **注意事項與備註嚴禁做成獨立標題**：
   - **嚴禁**將「注意事項」、「小提醒」、「常犯陷阱」寫成獨立標題（如 `#### 💡 小提醒`）。
   - **一律使用 Markdown 引用塊 `>` (Blockquote)** 呈現。

2. **引用塊格式範例**：
   - **直白觀念解說**：
     > a 是標籤，而 3 才是真正的物件
   - **小提醒/注意事項**：
     > **💡 小提醒**：設定 `timeout` 時務必搭配 `try...except TimeoutExpired` 捕獲，避免程式中斷！

---

## 5. 內文排版與標點轉義規範 (Formatting & Escaping Rules)

1. **條列式說明 (Unordered Lists & Indentation)**：
   - 使用 `-` 作為無序列表標記。
   - 列表內部補充說明需縮排 4 個空格，並善用生動粗體標記關鍵字。

2. **括號與特殊符號轉義 (Escaping in Text)**：
   - 在內文箇條點或說明文字中，小括號如包含語法說明時可加上轉義符號（例如：`\(list\)`、`\(dict\)`）。
   - 底線或減號連結時視需要轉義（如 `\_` 或 `\-\>`）。
3. **為每個函數做完整的參數及回傳解釋：**
   - 要有完整的使用時機說明
   - 傳入的參數說明，若只有少數幾個，條列式說明，數量太多則用表格形式
   - 回傳也同上少數幾個則條列式，多則表格化
   - 最好要在下面附帶上簡易的使用範例

   **實際寫法範例**：
   ````markdown
   ### prompt() 單次互動提示符

   - **使用時機**：當你只需要在程式中偶爾向使用者詢問一次資料（例如問密碼、問名稱），不需要保留跨次對話的歷史紀錄時使用。
   - **語法**：`prompt(message: str, completer=None, is_password=False, key_bindings=None, ...)`
   - **參數說明**：
     - `message`：要顯示給使用者的提示文字字串。
     - `completer`：提供自動補全邏輯的物件（如 `WordCompleter`）。
     - `is_password`：布林值，設定為 `True` 時將隱藏輸入內容（變成星號）。
     - `key_bindings`：傳入 `KeyBindings` 物件（相當於按鍵訊號攔截器，用來自訂或覆寫如 Ctrl+C 的預設行為）。
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
   ````

---

## 6. 程式碼區塊規範 (Code Block Standards)

1. **明確指定語言標籤**：
   - 所有代碼區塊必須帶有語言標記（如 ````python` 或 ````Python`）。

2. **註釋與輸出標註規範**：
   - 代碼塊中帶有步驟標號與結構註釋（例如 `# 1. 定義 Protocol 介面協定`, `# 2. 具體類別 A`）。
   - 在 `print()` 後方或下一行直接以 `# 假設印出：...` 或 `# 輸出: ...` 標註預期執行結果。

   ```python
   name = "Matthew"
   print(id(name))  # 假設印出：4388921136 (這是舊石頭的位址)

   name = "MATTHEW"
   print(id(name))  # 假設印出：4388921584 (位置變了！證明這是一塊全新的石頭)
   ```
3.  減少不必要程式碼：
   - 專注於要解釋的函式或程式碼區塊，減少不必要的干擾
   - 例如這裡要解釋的是 `automatic_function_calling=types.AutomaticFunctionCallingConfig(disable=True)`的設定，就可以把函式起他要設定的值用 `...` 帶過（例如 `model=self.model_name,` 就被省略了）

```Python
... = ...generate_content_stream(
	...
	config=types.GenerateContentConfig(
		tools=...,
		automatic_function_calling=types.AutomaticFunctionCallingConfig(disable=True)
)
```
---

## 7. 表格規範 (Markdown Tables)

在介紹型別、命名慣例、API 參數或概念對照時，必須提供結構嚴謹的 Markdown 表格：

### 7.1 參數/型別對照表樣式

| 標註型別 / 類別 | 資料型別 | 說明與用法範例 |
| :--- | :--- | :--- |
| **`Optional[型別]`** | `Type` | **指定型別或為空** (`型別 \| None`)。範例：`x: Optional[int] = None` |
| **`List[型別]`** | `List` | **指定元素型別的串列**。範例：`names: List[str] = ["Alice", "Bob"]` |

### 7.2 概念比較表格樣式

| 比較維度 | 抽象基底類別 (ABC) | 結構化協定 (Protocol) |
| :--- | :--- | :--- |
| **型別系統** | **名義型別 (Nominal Typing)**<br>必須顯式繼承父類別 | **結構型別 (Structural Typing)**<br>只要介面/方法簽名符合即可 |
| **解耦能力** | **較低**（第三方類別必須明確繼承你的 ABC） | **極高**（第三方類別完全無需感知你的 Protocol 存在） |

### ⚠️ Markdown 表格排版鐵律
1. **最左側頂格**：表格必須從行首 `|` 開始，不能有前置空格縮排。
2. **絕無空行**：表格內部行與行之間**絕對不能有任何空行**，否則會破壞 Markdown 渲染。
3. **欄位對齊**：對齊列可使用 `:---`（左對齊）或 `---`（預設對齊）。
4. **儲存格內換行**：如需在表格儲存格內換行，使用 HTML 標籤 `<br>`。

---

## 8. 函式與 API 說明規範 (Function & API Explanation Standards)

本規範提煉自 `Python/Python 專案測試指南.md` 的函式與 API 說明格式。
適用於在筆記中解釋特定函式、方法、內建工具或類別屬性時。

1. **標題層級與命名 (Header Level & Naming)**：
   - 一律使用 `#####` (H5) 作為單個函式/方法的參考標題。
   - 方法/函數標題名稱後面 **必須加上小括號 `()`**。
     - 例如：`##### capsys.readouterr() 讀取並重設輸出快取`
   - 屬性/變數/參數標題名稱後面 **絕對不能帶有括號**。
     - 例如：`##### tmp_path 臨時目錄管理`

2. **段落排版結構 (Section Layout)**：
   - **使用時機**：一句話告訴讀者「什麼時候該用它」。
   - **語法**：標註具體調用方法簽名（需以反單引號包覆）。
   - **參數說明**：
     - 若參數只有少數幾個，請使用條列式說明。
     - 若參數數量太多或極為複雜，請改用 Markdown 表格形式歸納。
   - **回傳值**：說明傳回的資料型別與代表意義（少數幾個條列式，多則表格化）。
   - **程式碼證明**：在最下方附帶極度精簡的實用範例（適當使用 `...` 隱藏不必要邏輯）。
   - **主題分割線**：每個函式說明區塊結尾處必須加上獨立一行的 `---` 進行視覺分割。

3. **格式範本 (Template Example)**：

##### prompt() 單次互動提示符

- **使用時機**：當你只需要在程式中偶爾向使用者詢問一次資料（例如問密碼、問名稱），不需要保留跨次對話的歷史紀錄時使用。
- **語法**：`prompt(message: str, completer=None, key_bindings=None, ...)`
- **參數說明**：
  - `message`：要顯示給使用者的提示文字字串。
  - `completer`：提供自動補全邏輯的物件（如 `WordCompleter`）。
  - `is_password`：布林值，設定為 `True` 時將隱藏輸入內容（變成星號）。
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

