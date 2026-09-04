# 概念與原理

## 什麼是 Regex (正規表達式)？

**正規表達式 (Regular Expressions，簡稱 Regex)** 是一種用於描述、匹配、檢索、替換文字模式的強大迷你語言 (Pattern Matching Language)。

在 Python 開發中，無論是**驗證表單輸入**（如 Email、手機號碼、身分證字號）、**清洗爬蟲抓回來的網頁 HTML 雜質**、**解析系統 Log 日誌**，還是**龐大數據集中的特徵提取**，Regex 都是不可或缺的神級工具！

> **🍿 生動白話比喻**：  
> 傳統字串搜尋（如 `text.find("apple")`）像**「尋人啟事指定找張三這個人」**，只要名字少一個字就找不到了；  
> 而 **Regex 正規表達式** 像**「全自動智能通靈過濾篩網」**，你可以設定篩選條件：「找身高 175-180cm、穿紅色外套、且姓張的所有人」。不管對方叫張三還是張四，通通能被精確篩選捕捉出來！

---

## Regex 究竟長什麼樣？(直觀解密與經典範例拆解)

Regex 看起來像一串由**文字**與**特殊符號（如 `\d`, `+`, `^`, `$`, `[]`, `()`）**組合而成的「文字特徵密碼」。

以下是 3 個最常見的真實 Regex 外貌與構造拆解：

### 1. 台灣手機號碼 Regex ➔ `^09\d{8}$`
```text
  ^     09     \d{8}     $
  │     │        │       │
  │     │        │       └── 結尾：必須在第 10 個數字結束 (不可多字)
  │     │        └───────── 8 位數字：\d 代表數字，{8} 代表重複 8 次
  │     └────────────────── 固定開頭：前兩位必須是 "09"
  └──────────────────────── 開頭：從字串的第一個字元開始匹配
```
- **匹配成功**：`0912345678`
- **匹配失敗**：`0800000123`（開頭不對）、`09123`（數字長度不夠）

---

### 2. 電子郵件 Email Regex ➔ `[\w.-]+@[\w.-]+\.\w+`
```text
  [\w.-]+        @        [\w.-]+        \.        \w+
     │           │           │           │          │
     │           │           │           │          └── 域名副檔名 (如 com, org)
     │           │           │           └────────── 轉義點號 (代表真正的 '.')
     │           │           └────────────────────── 網域主體 (如 google, yahoo)
     │           └────────────────────────────────── 必須包含 @ 符號
     └────────────────────────────────────────────── 帳號名稱 (允許字母、數字、點與減號)
```
- **匹配成功**：`service@google.com`, `user.name_2026@my-domain.org`

---

### 3. 日期解構萃取 Regex ➔ `(\d{4})-(\d{2})-(\d{2})`
```text
  (\d{4})   -   (\d{2})   -   (\d{2})
    │           │               │
    │           │               └── 第 3 分組：2 位日期 (01~31)
    │           └────────────────── 第 2 分組：2 位月份 (01~12)
    └────────────────────────────── 第 1 分組：4 位年份 (如 2026)
```
- **匹配成功並分組萃取**：`2026-07-23` ➔ 成功拆解為 `年='2026'`, `月='07'`, `日='23'`！

---


## Regex vs Glob 萬用字元區別

初學者常將 `Glob` 通配符（如 `*.py`）與 `Regex` 搞混，兩者的適用場景完全不同：

| 比較維度 | Glob 萬用字元 (通配符) | Regex 正規表達式 |
| :--- | :--- | :--- |
| **主要定位** | 專門用於**檔案路徑與檔名匹配** (如 `pathlib` / Shell) | 專門用於**字串內部的複雜文字模式檢索、驗證與清洗** |
| **匹配萬用字元 `*` 的意思** | 代表「任意長度的字元」 (如 `*.txt`) | 代表「**前一個**字元重複 0 次或多次」 |
| **語法複雜度** | 極簡直觀，功能較單純 | 語法豐富強大，支援分組、斷言與複雜邏輯 |
| **常用模組** | `pathlib`, `glob`, `.gitignore` | Python 內建 `re` 模組 |

---

# 核心 API 函數字典對照表

| 函式 / 物件名稱 | 回傳型別 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- | :--- |
| **[[#re.search() 搜尋第一個匹配項目\|re.search()]]** | `Match \| None` | **搜尋全字串，回傳第一個匹配成功項目** | 從日誌或長文字中提取第一個出現的目標（如 IP、Token） |
| **[[#re.match() 從字串開頭進行匹配\|re.match()]]** | `Match \| None` | **嚴格從字串開頭（索引 0）開始匹配** | 表單輸入格式驗證（如手機號碼、帳號格式） |
| **[[#re.findall() 找出所有匹配項目\|re.findall()]]** | `list[str \| tuple]` | **搜尋全字串，以 List 回傳所有匹配結果** | 爬蟲提取文章內所有 Email、超連結或特定標籤 |
| **[[#re.finditer() 找出所有匹配迭代器\|re.finditer()]]** | `Iterator[Match]` | **以產生器 (Iterator) 迭代回傳所有 Match 物件** | 大量資料批次處理，需同時取得匹配文字與其起訖位置 (`span`) |
| **[[#re.sub() 替換與文字清洗\|re.sub()]]** | `str` | **將匹配到的文字替換為新字串** | 資料清洗、去除 HTML 標籤、個資遮罩（如手機打碼） |
| **[[#re.subn() 替換並統計次數\|re.subn()]]** | `tuple[str, int]` | **替換文字並同時回傳替換成功次數** | 批次文字取代並統計異動數量 |
| **[[#re.split() 正則表達式字串切割\|re.split()]]** | `list[str]` | **依照多種正則模式切割字串** | 處理多種複雜分隔符（如依 `,`、`;`、空白同時切分） |
| **[[#re.compile() 預編譯效能優化\|re.compile()]]** | `Pattern` | **預先編譯正則規則為 Pattern 物件以提升效能** | 迴圈處理百萬筆數據、常駐 API 服務高頻重複匹配 |
| **[[#Match 物件核心方法 (取得分組與位置)\|Match 物件]]** | `re.Match` | **儲存匹配成功後的詳細資訊與位置** | 提取分組文字 (`.group()`)、起訖索引 (`.span()`) |

---

# 核心功能與語法大解密

Python 內建了 `re` 模組處理正規表達式，使用前只需匯入：

```python
import re
```

##### re.search() 搜尋第一個匹配項目

- **使用時機**：當你想在整個字串中尋找是否存在符合條件的子字串，只要找到第一個就立刻回傳時使用。
- **語法**：`re.search(pattern, string, flags=0)`
- **參數說明**：
  - `pattern`：要匹配的正規表達式字串（建議使用 `r"..."`）。
  - `string`：要被搜尋的目標文字字串。
  - `flags`：可選修飾標誌（如 `re.IGNORECASE`、`re.MULTILINE`）。
- **回傳值**：
  - `re.Match | None`：若匹配成功回傳 `Match` 物件，若無匹配回傳 `None`。

```python
import re

text = "User ID: 9527, Age: 25"
match = re.search(r"\d+", text)  # 尋找第一個連續數字

if match:
    print(match.group())  # 輸出: 9527
    print(match.span())   # 輸出: (9, 13)
```

---

##### re.match() 從字串開頭進行匹配

- **使用時機**：當你需要嚴格要求目標字串「從最開頭第 1 個字元起」就必須符合規則時使用（常用於表單格式驗證）。
- **語法**：`re.match(pattern, string, flags=0)`
- **參數說明**：
  - `pattern`：要匹配的正規表達式。
  - `string`：目標文字字串。
  - `flags`：可選修飾標誌。
- **回傳值**：
  - `re.Match | None`：若開頭即符合回傳 `Match` 物件，否則回傳 `None`。

```python
import re

text = "ID: 9527"
# match() 要求從開頭就符合 \d+，因此這裡會失敗回傳 None
print(re.match(r"\d+", text))          # 輸出: None
# search() 會搜尋全字串，因此成功找到
print(re.search(r"\d+", text).group()) # 輸出: 9527
```

---

##### re.findall() 找出所有匹配項目

- **使用時機**：當你想將文字中所有符合規則的內容一次性全部抽取出，放入串列中處理時使用。
- **語法**：`re.findall(pattern, string, flags=0)`
- **參數說明**：
  - `pattern`：要匹配的正規表達式。
  - `string`：目標文字字串。
  - `flags`：可選修飾標誌。
- **回傳值**：
  - `list[str] | list[tuple]`：所有匹配到的字串串列（若有無名分組則回傳元組串列）。

```python
import re

text = "蘋果 50 元，香蕉 30 元，西瓜 120 元"
prices = re.findall(r"\d+", text)
print(prices)  # 輸出: ['50', '30', '120']
```

---

##### re.finditer() 找出所有匹配迭代器

- **使用時機**：當處理大型文字檔案，想批次逐筆獲取 `Match` 物件且不想耗盡記憶體時使用。
- **語法**：`re.finditer(pattern, string, flags=0)`
- **參數說明**：
  - `pattern`：要匹配的正規表達式。
  - `string`：目標文字字串。
  - `flags`：可選修飾標誌。
- **回傳值**：
  - `Iterator[re.Match]`：回傳產生器迭代器，每次產出一個 `Match` 物件。

```python
import re

text = "蘋果 50 元，香蕉 30 元"
for match in re.finditer(r"\d+", text):
    print(f"找到數字: {match.group()}, 位置: {match.span()}")
```

---

##### re.sub() 替換與文字清洗

- **使用時機**：當你想將符合特定規則的文字替換為新內容，進行資料清洗或遮蔽敏感資訊時使用。
- **語法**：`re.sub(pattern, repl, string, count=0, flags=0)`
- **參數說明**：
  - `pattern`：要替換的正規表達式。
  - `repl`：替換後的新字串（或處理函式）。
  - `string`：目標文字字串。
  - `count`：最大替換次數（預設 `0` 代表全部替換）。
  - `flags`：可選修飾標誌。
- **回傳值**：
  - `str`：替換完成後的新字串。

```python
import re

phone = "0912-345-678 # 聯絡電話"
# 將非數字部分 (\D+) 或註解替換為空字串，完成電話號碼清洗
clean_phone = re.sub(r"\D", "", phone)
print(clean_phone)  # 輸出: 0912345678
```

---

##### re.subn() 替換並統計次數

- **使用時機**：當你執行文字取代的同時，還需要**精確知道總共成功替換了幾處**時使用。
- **語法**：`new_string, num_subs = re.subn(pattern, repl, string, count=0, flags=0)`
- **回傳值**：
  - `tuple[str, int]`：包含「替換後的新字串」與「替換總次數」的元組。

```python
import re

text = "cat, bat, rat, mat, cat"
result, count = re.subn(r"cat", "dog", text)
print(f"替換結果: {result}")  # 輸出: dog, bat, rat, mat, dog
print(f"總共替換了 {count} 處") # 輸出: 總共替換了 2 處
```

---

##### re.split() 正則表達式字串切割

- **使用時機**：當字串的分隔符號不單一（例如同時包含逗號、分號、空格或換行），傳統 `str.split()` 無法處理時使用。
- **語法**：`re.split(pattern, string, maxsplit=0, flags=0)`
- **回傳值**：
  - `list[str]`：切分後的子字串串列。

```python
import re

# 同時依據逗號、分號或空格切割字串
raw_data = "Python,Java;Rust   Go\tC++"
items = re.split(r"[,;\s]+", raw_data)
print(items)  # 輸出: ['Python', 'Java', 'Rust', 'Go', 'C++']
```

---

##### re.compile() 預編譯效能優化

- **使用時機**：當同一個 Regex 規則會在迴圈中重複調用上千次，需要大幅提升執行效能時使用。
- **語法**：`re.compile(pattern, flags=0)`
- **回傳值**：
  - `re.Pattern`：編譯完成的正規表達式 Pattern 物件。

```python
import re

# 1. 預先編譯 Regex 規則
email_pattern = re.compile(r"[\w.-]+@[\w.-]+\.\w+")

# 2. 重複高效使用
user_input = "contact@example.com"
if email_pattern.search(user_input):
    print("有效 Email！")
```

---

##### Match 物件核心方法 (取得分組與位置)

當 `re.search()` 或 `re.match()` 匹配成功時，會回傳 `re.Match` 物件：

| 方法 / 屬性 | 回傳型別 | 功能說明與用途 |
| :--- | :--- | :--- |
| **`match.group()`** 或 **`match.group(0)`** | `str` | 回傳**完整匹配到的整個字串** |
| **`match.group(1)`**, **`match.group(2)`** | `str` | 回傳第 1、第 2 個小括號分組所捕捉的子字串 |
| **`match.groups()`** | `tuple[str, ...]` | 一次性以元組回傳所有分組的捕捉結果 |
| **`match.groupdict()`** | `dict[str, str]` | 一次性以字典回傳所有具名分組 (`?P<name>`) 的結果 |
| **`match.start()`** | `int` | 回傳匹配內容在原字串中的**起點索引** |
| **`match.end()`** | `int` | 回傳匹配內容在原字串中的**終點索引** |
| **`match.span()`** | `tuple[int, int]` | 回傳包含 `(start, end)` 的元組 |

```python
import re

text = "Order-2026-0888"
m = re.search(r"Order-(\d{4})-(\d{4})", text)

if m:
    print(m.group(0))  # 'Order-2026-0888' (全體)
    print(m.group(1))  # '2026' (第 1 組)
    print(m.group(2))  # '0888' (第 2 組)
    print(m.groups())  # ('2026', '0888')
    print(m.span())    # (0, 15)
```

---

# 正規表達式核心語法全能手冊

在寫 Regex 模式時，請務必在字串前加上 **`r"..."` (Raw String 原始字串)**，以防止 Python 跳脫字元（如 `\n`、`\t`）干擾！

---

## 1. 預定義字元集 (Predefined Character Classes)

| 萬用記號 | 代表意義與說明 | 等價語法 | 成功匹配範例 |
| :--- | :--- | :--- | :--- |
| **`\d`** | 匹配任何**單一十進位數字** (Digit) | `[0-9]` | `'5'`, `'0'` |
| **`\D`** | 匹配任何**非數字**字元 | `[^0-9]` | `'A'`, `'!'`, `' '` |
| **`\w`** | 匹配單字字元 (**字母、數字、底線**，Word) | `[a-zA-Z0-9_]` | `'a'`, `'8'`, `'_'` |
| **`\W`** | 匹配非單字字元 (特殊符號、空白、標點) | `[^a-zA-Z0-9_]` | `'@'`, `'#'`, `' '`, `'-'` |
| **`\s`** | 匹配任何**空白字元** (空格、Tab `\t`、換行符 `\n`) | `[ \t\n\r\f\v]` | `' '`, `'\n'`, `'\t'` |
| **`\S`** | 匹配任何**非空白**字元 | `[^ \t\n\r\f\v]` | `'X'`, `'1'`, `'!'` |
| **`.`** | 匹配**除換行符 `\n` 之外**的任意單一字元 | — | `'a'`, `'9'`, `'#'`, `' '` |

---

## 2. 自訂字元集合與區間 (Custom Character Sets `[]`)

使用中括號 `[...]` 可以建立自訂的字元篩選池（只匹配括號內的**其中一個字元**）：

| 語法模式 | 說明與匹配範圍 | 實戰範例 | 成功匹配標的 |
| :--- | :--- | :--- | :--- |
| **`[abc]`** | 枚舉集合：匹配 `a` 或 `b` 或 `c` | `[aeiou]` | 匹配任何英文字母母音 |
| **`[a-z]`** | 連續範圍：匹配小寫字母 a 到 z | `[a-z]` | `'k'`, `'z'` |
| **`[A-Z]`** | 連續範圍：匹配大寫字母 A 到 Z | `[A-Z]` | `'G'`, `'W'` |
| **`[0-9]`** | 連續範圍：匹配數字 0 到 9 | `[0-9]` | `'7'` |
| **`[a-zA-Z0-9]`** | 多範圍組合：字母或數字 | `[a-zA-Z0-9]` | 帳號常用字元 |
| **`[^abc]`** | **排除取反**：在中括號開頭加上 `^` 代表**「非」** | `[^0-9]` | 任何非數字字元 |
| **`[\u4e00-\u9fa5]`** | **中文字元範圍 (Unicode)** ⭐ | `[\u4e00-\u9fa5]+` | 提取所有純中文字串 |

```python
import re

text = "Hello 世界 2026！Python"

# 1. 提取所有中文字
chinese = re.findall(r"[\u4e00-\u9fa5]+", text)
print(chinese)  # 輸出: ['世界']

# 2. 排除所有字母與數字，只留符號與空格
symbols = re.findall(r"[^a-zA-Z0-9\s]", text)
print(symbols)  # 輸出: ['世', '界', '！']
```

---

## 3. 數量表示符 (Quantifiers) 與貪婪/懶惰匹配

數量符號緊跟在字元或分組後面，用來指定該元素**重複出現的次數**：

| 數量記號 | 匹配次數說明 | 實戰範例 | 成功匹配標的 |
| :--- | :--- | :--- | :--- |
| **`*`** | 重複 **0 次或多次** (任意次，可完全沒有) | `go*d` | `'gd'`, `'god'`, `'good'` |
| **`+`** | 重複 **1 次或多次** (至少要有 1 次) | `go+d` | `'god'`, `'good'` (不包含 `'gd'`) |
| **`?`** | 重複 **0 次或 1 次** (可有可無) | `colou?r` | `'color'`, `'colour'` |
| **`{n}`** | 恰好重複 **n 次** | `\d{4}` | `'2026'` (恰好 4 位數) |
| **`{n,m}`** | 重複 **n 到 m 次** | `\d{2,4}` | `'12'`, `'123'`, `'1234'` |
| **`{n,}`** | 重複 **至少 n 次** (無上限) | `\d{3,}` | 3 位數以上的所有連續數字 |

### 🚨 貪婪 (Greedy) vs 懶惰 (Lazy / Non-greedy) 匹配

- **預設為貪婪匹配 (Greedy)**：`*` 與 `+` 會盡可能**吃掉最多**字元。
- **加 `?` 變為懶惰匹配 (Lazy)**：`*?` 與 `+?` 會盡可能**吃掉最少**字元（一旦找到最短吻合就停下）。

```python
import re

html = "<div>第一區塊</div><div>第二區塊</div>"

# 1. 貪婪匹配 (.*) - 會從第一個 <div> 一路吃到最後一個 </div>！
greedy_result = re.findall(r"<div>.*</div>", html)
print(greedy_result)
# 輸出: ['<div>第一區塊</div><div>第二區塊</div>']  <-- 錯把兩區塊當成一大包

# 2. 懶惰匹配 (.*?) - 遇到第一個 </div> 就停下，精確拆分標籤！
lazy_result = re.findall(r"<div>.*?</div>", html)
print(lazy_result)
# 輸出: ['<div>第一區塊</div>', '<div>第二區塊</div>']  <-- 完全精確！
```

---

## 4. 邊界與定位符 (Anchors & Word Boundaries)

定位符不匹配任何實體字元，而是用來**錨定特定位置 (Zero-width Position)**：

| 邊界記號 | 說明與功能 | 實戰範例 | 匹配行為解析 |
| :--- | :--- | :--- | :--- |
| **`^`** | 匹配**字串開頭**（多行模式下匹配每行開頭） | `^Hello` | 必須以 "Hello" 開頭才符合 |
| **`$`** | 匹配**字串結尾**（多行模式下匹配每行結尾） | `world$` | 必須以 "world" 結尾才符合 |
| **`\b`** | 匹配**單字邊界 (Word Boundary)** | `\bcat\b` | 只匹配獨立單字 `"cat"`，不匹配 `"category"` |
| **`\B`** | 匹配**非單字邊界** | `\Bcat\B` | 匹配位於單字內部的 cat（如 `"certificate"`） |
| **`\A`** | 永遠只匹配**全文字的最絕對開頭**（不受多行模式影響） | `\AStart` | 嚴格全檔開頭 |
| **`\Z`** | 永遠只匹配**全文字的最絕對結尾** | `End\Z` | 嚴格全檔結尾 |

```python
import re

text = "cat category concatenate copycat"
# 只搜尋獨立的單字 "cat"
print(re.findall(r"\bcat\b", text))  # 輸出: ['cat']
```

---

## 5. 邏輯分支與選擇 (Logical OR `|`)

使用豎線 `|` 代表「或 (OR)」，支援在多個匹配規則之間二選一或多選一：

```python
import re

text = "我喜歡 apple 和 banana，但不喜歡 grape。"
fruits = re.findall(r"apple|banana|grape", text)
print(fruits)  # 輸出: ['apple', 'banana', 'grape']

# 搭配括號限制作用範圍
text2 = "I have a cat and a dog."
pets = re.findall(r"I have a (cat|dog)", text2)
print(pets)  # 輸出: ['cat']
```

---

## 6. 分組捕捉 (Capturing Groups) 與高級分組技巧

使用小括號 `()` 可以將 Regex 的一部分括起來進行「分組」，方便事後萃取局部欄位：

### 6.1 傳統順序分組 (`(...)`)

```python
import re

text = "出生日期：2026-07-23"
pattern = r"(\d{4})-(\d{2})-(\d{2})"
match = re.search(pattern, text)

if match:
    print(match.group(0))  # 完整匹配結果: '2026-07-23'
    print(match.group(1))  # 第 1 個小括號 (年): '2026'
    print(match.group(2))  # 第 2 個小括號 (月): '07'
    print(match.group(3))  # 第 3 個小括號 (日): '23'
    print(match.groups())  # 一鍵匯出 Tuple: ('2026', '07', '23')
```

### 6.2 具名分組 `(?P<name>...)` (推薦！可讀性最高)

```python
import re

log_line = "IP: 192.168.1.100 - Status: 200"
pattern = r"IP:\s*(?P<ip>[\d.]+)\s*-\s*Status:\s*(?P<status>\d+)"
match = re.search(pattern, log_line)

if match:
    print(match.group("ip"))      # 輸出: '192.168.1.100'
    print(match.group("status"))  # 輸出: '200'
    print(match.groupdict())      # 一鍵匯出字典: {'ip': '192.168.1.100', 'status': '200'}
```

### 6.3 非捕獲分組 `(?:...)` (提升效能 ⭐)

如果你需要把多個字元括起來套用數量詞，但**不需要單獨提取它的值**，請使用 `(?:...)`。這樣 Python 不會耗費記憶體記錄它：

```python
import re

text = "http://google.com and https://yahoo.com"
# 只需要匹配協議，不需要個別儲存 http/https 分組
urls = re.findall(r"(?:https?://)[\w.]+", text)
print(urls)  # 輸出: ['http://google.com', 'https://yahoo.com']
```

---

## 7. 零寬斷言 (Lookaround Assertions - 高級黑魔法 ⭐)

斷言只用來**「檢查前後條件是否成立」**，**不消耗字元、也不會把條件文字包含在匹配結果中**：

| 斷言名稱 | 語法記號 | 意義說明 | 實戰範例 | 說明 |
| :--- | :--- | :--- | :--- | :--- |
| **正向肯定先行斷言** | **`(?=pattern)`** | **後面必須跟著**特定文字 | `\d+(?=元)` | 只抓取後面緊接著「元」的數字 |
| **正向否定先行斷言** | **`(?!pattern)`** | **後面不能跟著**特定文字 | `\d+(?!元)` | 抓取後面「不是」跟著「元」的數字 |
| **正向肯定後行斷言** | **`(?<=pattern)`** | **前面必須是**特定文字 | `(?<=\$)\d+` | 只抓取前面緊接著 `$` 的數字 |
| **正向否定後行斷言** | **`(?<!pattern)`** | **前面不能是**特定文字 | `(?<!\$)\d+` | 抓取前面「不是」`$` 的數字 |

```python
import re

text = "蘋果 50元，美金 $100，特價 30元，編號 999"

# 1. 抓取後面緊跟著「元」的數字 (正向肯定先行斷言)
ntd_prices = re.findall(r"\d+(?=元)", text)
print(ntd_prices)  # 輸出: ['50', '30'] (注意：回傳結果只有數字，不包含「元」！)

# 2. 抓取前面緊跟著「$」的數字 (正向肯定後行斷言)
usd_prices = re.findall(r"(?<=\$)\d+", text)
print(usd_prices)  # 輸出: ['100'] (注意：回傳結果只有數字，不包含「$」！)
```

---

## 8. 常用修飾標誌 (Regex Flags) 深度剖析

修飾標誌（Flags）用來**改變正則引擎的匹配行為與規則**。你可以傳給 `re.search()`、`re.findall()` 或 `re.compile()` 的 `flags` 參數。

### 📊 核心修飾標誌速查總表

| 標誌常數 | 簡寫 | 白話核心功能 | 解決的實戰痛點 |
| :--- | :---: | :--- | :--- |
| **[[#8.1 re.IGNORECASE (re.I)：忽略大小寫\|re.IGNORECASE]]** | `re.I` | **忽略英文大小寫** | 避免寫出極長且醜陋的 `[a-zA-Z]` 或手動轉 `.lower()` |
| **[[#8.2 re.MULTILINE (re.M)：多行模式\|re.MULTILINE]]** | `re.M` | **多行模式**（讓 `^` 與 `$` 錨定「每一行」） | 處理多行文字或日誌時，能逐行匹配行首與行尾 |
| **[[#8.3 re.DOTALL (re.S)：單行跨行穿透模式\|re.DOTALL]]** | `re.S` | **單行穿透模式**（讓萬用點 `.` 包含換行符 `\n`） | 爬蟲抓取跨多行的 HTML 區塊、文章段落 |
| **[[#8.4 re.VERBOSE (re.X)：詳細註釋與排版模式\|re.VERBOSE]]** | `re.X` | **詳細註釋模式**（忽略正則內空格與換行，支援 `#` 註解） | 將難以維護的「天書級」超長正則拆解成清晰代碼 |
| **[[#8.5 re.ASCII (re.A)：純 ASCII 模式\|re.ASCII]]** | `re.A` | **純 ASCII 模式**（讓 `\w`、`\d` 不匹配中文與 Unicode） | 嚴格驗證純英文帳號、避免中文字元被 `\w` 誤判 |

---

### 💡 多個 Flags 組合技（使用按位或運算子 `|`）

如果想要**同時套用多個標誌**，可以使用直線符號 `|`（Bitwise OR）串接：

```python
import re

# 同時啟用：忽略大小寫 (re.I) + 多行模式 (re.M) + 點號跨行 (re.S)
pattern = re.compile(r"^<div>.*?</div>$", re.I | re.M | re.S)
```

---

### 8.1 `re.IGNORECASE` (`re.I`)：忽略大小寫

- **痛點**：使用者輸入可能為 `Python`、`python` 或 `PYTHON`，傳統寫法要寫 `[pP][yY][tT][hH][oO][nN]`。
- **作用**：字母自動對應大寫與小寫。

```python
import re

text = "Learn Python, python3, and PYTHON_CORE."

# ❌ 未加標誌：只能精確匹配全小寫
print(re.findall(r"python\d?", text)) 
# 輸出: ['python3']

# ✅ 加上 re.IGNORECASE (或 re.I)：全數通抓！
print(re.findall(r"python\d?", text, flags=re.I)) 
# 輸出: ['Python', 'python3', 'PYTHON']
```

---

### 8.2 `re.MULTILINE` (`re.M`)：多行模式

- **痛點**：預設情況下，`^` 永遠只匹配**整篇文字的最開頭**，`$` 永遠只匹配**整篇文字的最尾端**。當字串包含多行文字（含 `\n`）時，無法抓出每一行開頭的資料。
- **作用**：讓 `^` 能夠匹配**「每一行的開頭」**，`$` 能夠匹配**「每一行的結尾」**。

```python
import re

log_data = """ERROR 2026-08-01 連線超時
INFO 2026-08-02 正常運行
ERROR 2026-08-03 資料庫崩潰"""

# ❌ 未加標誌：^ 只會看第 1 行的開頭，抓不到第 3 行的 ERROR！
print(re.findall(r"^ERROR.*", log_data))
# 輸出: ['ERROR 2026-08-01 連線超時']

# ✅ 加上 re.MULTILINE (或 re.M)：每一行都會獨立套用 ^ 錨點！
print(re.findall(r"^ERROR.*", log_data, flags=re.M))
# 輸出: ['ERROR 2026-08-01 連線超時', 'ERROR 2026-08-03 資料庫崩潰']
```

---

### 8.3 `re.DOTALL` (`re.S`)：單行跨行穿透模式

- **痛點**：萬用點 `.` 預設**會被換行符 `\n` 擋住（不匹配 `\n`）**！因此遇到跨越多行的 HTML 或 JSON 區塊時會直接匹配失敗。
- **作用**：讓萬用點 `.` 可以**真正匹配「包含 `\n` 換行符在內的任何字元」**。

```python
import re

html_snippet = """<article>
    <h1>Python 教學</h1>
    <p>正規表達式超好用！</p>
</article>"""

# ❌ 未加標誌：. 遇到換行符就停下，回傳空清單！
print(re.findall(r"<article>.*?</article>", html_snippet))
# 輸出: []

# ✅ 加上 re.DOTALL (或 re.S)：. 能夠順利穿透 \n 跨行抓取整個區塊！
result = re.findall(r"<article>.*?</article>", html_snippet, flags=re.S)
print(result[0])
# 輸出: 完整跨行的 <article>...</article> 內容！
```

---

### 8.4 `re.VERBOSE` (`re.X`)：詳細註釋與排版模式

- **痛點**：複雜的正規表達式擠在同一行時，就像外星文字（天書），幾個月後自己也看不懂。
- **作用**：**忽略正則表達式內部所有的普通空格與換行**，並允許你在正則中**直接用 `#` 寫中文註解**，代碼維護性提升 100 倍！

```python
import re

# 透過 re.VERBOSE (re.X) 將龐大複雜的手機號碼驗證邏輯拆解：
phone_pattern = re.compile(r"""
    ^                        # 必須從字串開頭開始
    (?:\+?886\s*|\(0\))?     # 國際區號：支援 +886 或 (0)，可有可無
    0?9\d{2}                 # 手機前綴：如 0912 或 912
    [-.\s]?                  # 分隔符：支援減號、點號或空白
    \d{3}                    # 中間 3 碼數字
    [-.\s]?                  # 分隔符
    \d{3}                    # 末尾 3 碼數字
    $                        # 必須以末尾結束
""", re.VERBOSE)

# 測試匹配
print(bool(phone_pattern.match("+886 912-345-678")))  # 輸出: True
print(bool(phone_pattern.match("0912.345.678")))      # 輸出: True
print(bool(phone_pattern.match("0912345678")))        # 輸出: True
```

---

### 8.5 `re.ASCII` (`re.A`)：純 ASCII 模式

- **痛點**：在 Python 3 中，`\w` 預設是 Unicode 模式，**會把中文字元、日文假名也視為單字字元 `\w`**！
- **作用**：強制讓 `\w`、`\W`、`\b`、`\d` 只匹配純 ASCII 字元（如 `[a-zA-Z0-9_]`），排除所有非英文字元。

```python
import re

text = "User_9527 測試帳號"

# ❌ 預設情況：\w+ 會連中文「測試帳號」一起抓進去！
print(re.findall(r"\w+", text))
# 輸出: ['User_9527', '測試帳號']

# ✅ 加上 re.ASCII (或 re.A)：嚴格只抓純英數字與底線！
print(re.findall(r"\w+", text, flags=re.A))
# 輸出: ['User_9527']
```

---

# 4 大高頻實戰專案腳本

### 實戰 1：Email 格式驗證與提取

```python
import re

def extract_emails(text: str) -> list[str]:
    # 匹配標準 Email 格式 (帳號@域名.頂級域)
    pattern = r"[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}"
    return re.findall(pattern, text)

content = "請寄信至 service@google.com 或 matthew.dev_2026@my-domain.org 聯絡我們。"
emails = extract_emails(content)
print(f"抽取出 Email: {emails}")
# 輸出: ['service@google.com', 'matthew.dev_2026@my-domain.org']
```

---

### 實戰 2：台灣手機號碼格式檢查與標準化

```python
import re

def normalize_tw_phone(raw_phone: str) -> str | None:
    # 先用 re.sub 清除空格、破折號與加號
    clean = re.sub(r"[\s\-\+]", "", raw_phone)
    
    # 若包含國際碼 886912345678 轉為 0912345678
    clean = re.sub(r"^8869", "09", clean)
    
    # 驗證是否為 09 開頭且剛好 10 位數字
    if re.match(r"^09\d{8}$", clean):
        return clean
    return None

print(normalize_tw_phone("+886 912-345-678")) # 輸出: '0912345678'
print(normalize_tw_phone("0987 654 321"))     # 輸出: '0987654321'
print(normalize_tw_phone("123456"))           # 輸出: None (無效)
```

---

### 實戰 3：HTML / Markdown 標籤去除與純文字清洗

```python
import re

html_content = "<h1>歡迎來到 <a href='#'>Antigravity AI</a></h1><p>這是<b>純文字</b>練習。</p>"

# 1. 使用懶惰匹配去除所有 HTML 標籤 <...>
clean_text = re.sub(r"<.*?>", "", html_content)
print(clean_text)  # 輸出: '歡迎來到 Antigravity AI這是純文字練習。'
```

---

### 實戰 4：日誌檔 Log 時間戳與 IP 位址自動解析萃取

```python
import re

log_data = """
2026-07-23 14:20:05 [INFO] 192.168.1.50 User logged in successfully.
2026-07-23 14:22:18 [ERROR] 10.0.0.12 Connection timeout.
"""

log_pattern = re.compile(
    r"(?P<timestamp>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})\s+"
    r"\[(?P<level>[A-Z]+)\]\s+"
    r"(?P<ip>[\d.]+)\s+"
    r"(?P<message>.*)"
)

for line in log_data.strip().splitlines():
    match = log_pattern.search(line)
    if match:
        data = match.groupdict()
        print(f"[{data['level']}] 時間: {data['timestamp']} | 來源 IP: {data['ip']} | 內容: {data['message']}")
```
