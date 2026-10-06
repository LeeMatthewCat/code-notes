# 概念與原理

## 什麼是 itertools 模組？

`itertools` 是 Python 內建的高效能迭代器工具庫（無需額外安裝，直接 `import itertools`）。

它以底層 C 語言高度最佳化實作，提供了一系列用於操作迭代器（Iterables）的標準構建區塊（Building Blocks）。

核心理念是**惰性求值 (Lazy Evaluation / Generator-based)**，即「按需計算、隨用隨取」，絕不在記憶體中一次性產生完整列表。即使處理數億筆龐大資料甚至無窮序列，記憶體開銷始終保持在 $O(1)$ 常數級別。

> **[生動比喻]**：  
> - **傳統 List 運算**：像「外送餐廳一口氣把 10,000 碗牛肉麵全部煮好堆滿整間廚房」，佔滿所有桌面與通道（記憶體爆滿崩潰），客人還沒吃完麵就全爛了。  
> - **`itertools` 迭代器**：像「頂級迴轉壽司軌道」，廚師只在客人伸手拿走一盤時，才在輸送帶上現場現捏下一盤。桌面永遠只保留一盤壽司的空間，即使軌道 24 小時無休運轉，廚房記憶體也永遠維持清爽！

---

## 為什麼需要 itertools？三大核心優勢

- **記憶體使用極小化 ($O(1)$)**：避免 `list` 串接或列表推導式產生的龐大中間暫存資料。
- **C 語言原生執行速度**：內部以原生 C 迴圈驅動，比手寫 Python `while` 或巢狀 `for` 迴圈快數倍。
- **函數式編程組合性 (Composability)**：各種迭代器函式可像樂高積木般互相串接組合，形成乾淨優雅的資料處理管道 (Data Pipeline)。

---

# 核心 API 函式字典對照表

| 類別 / 函式名稱 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- |
| **[[#count() 無窮計數迭代器\|count()]]** | **產生等差數列的無窮生成器** | 自動自增 ID 產生器、enumerate 步長自訂 |
| **[[#cycle() 循環重複迭代器\|cycle()]]** | **將有限序列無限循環重複產出** | 輪詢調度 (Round-robin)、交替顏色樣式輪播 |
| **[[#repeat() 固定重複迭代器\|repeat()]]** | **重複產出指定物件特定次數或無窮次** | 搭配 map 傳遞不變常數參數、預填固定初值 |
| **[[#islice() 迭代器高效切片\|islice()]]** | **以惰性串流方式對任意迭代器進行範圍切片** | 大型檔案分頁截取、無窮生成器前 N 筆截斷 |
| **[[#takewhile() 滿足條件前持續截取\|takewhile()]]** | **在斷言條件為真時持續產出，首遇假值立即永久終止** | 讀取有序串流前置有效區段、過濾標頭設定行 |
| **[[#dropwhile() 滿足條件時持續跳過\|dropwhile()]]** | **在斷言條件為真時持續捨棄，首遇假值後產出其餘所有元素** | 略過檔案開頭註釋行、跳過前導無效數值 |
| **[[#filterfalse() 反向過濾保留假值\|filterfalse()]]** | **保留斷言函式回傳為 False (或評估為假) 的所有元素** | 反向資料過濾、剔除有效值保留例外記錄 |
| **[[#compress() 布林遮罩過濾器\|compress()]]** | **依據布林選擇序列真假值篩選資料元素** | 特徵選取、向量遮罩匹配、平行篩選清單 |
| **[[#chain() 與 chain.from_iterable() 鏈結串接迭代器\|chain() / from_iterable()]]** | **將多個可迭代物件無縫銜接為單一連續迭代器** | 批次合併多個資料源、二維列表平坦化 (Flatten) |
| **[[#zip_longest() 最長拉鍊配對補齊\|zip_longest()]]** | **依最長迭代器長度配對元素，較短者以指定值補齊** | 合併長度不一清單、補齊缺失欄位表格 |
| **[[#starmap() 參數元組解包映射\|starmap()]]** | **從迭代器中取出元組元素解包傳入函式進行計算** | 批次座標距離計算、多參數函式向量化運算 |
| **[[#accumulate() 累加與累計運算器\|accumulate()]]** | **逐步計算前綴累加和、累積乘積或前綴極值** | 財務累積損益計算、跑步計時累計、前綴最大值 |
| **[[#pairwise() 相鄰元素成對滑動視窗\|pairwise()]]** | **(Python 3.10+) 產生長度為 2 的相鄰重疊滑動視窗元組** | 連續差值計算、軌跡相鄰節點距離、相鄰狀態轉移 |
| **[[#batched() 定長批次分組切塊\|batched()]]** | **(Python 3.12+) 將迭代器拆分為固定大小的批次元組** | 批次寫入資料庫、分批呼叫 LLM API、分塊處理 |
| **[[#groupby() 連續相鄰元素鍵值分組\|groupby()]]** | **將連續具備相同 Key 的相鄰元素聚合成組** | 依狀態/日期將已排序資料分群、連續重複項壓縮 |
| **[[#tee() 複製分裂獨立迭代器\|tee()]]** | **將單一迭代器分裂為多個互相獨立的迭代器副本** | 需多次遍歷單次性串流、同一資料流分叉處理 |
| **[[#product() 笛卡兒積多重迴圈展開\|product()]]** | **計算多個可迭代物件的笛卡兒積，取代巢狀迴圈** | 網格參數搜尋 (Grid Search)、全排列測試組合 |
| **[[#permutations() 排列生成器\|permutations()]]** | **產出考慮順序且元素不重複的所有排列組合 (nPr)** | 旅行推銷員路徑規劃、密碼字典暴破、排程窮舉 |
| **[[#combinations() 組合生成器\|combinations()]]** | **產出不考慮順序且元素不重複的組合 (nCr)** | 樂透選號、特徵兩兩配對分析、團隊搭檔選拔 |
| **[[#combinations_with_replacement() 可重複選取組合生成器\|combinations_with_replacement()]]** | **產出不考慮順序但允許元素重複挑選的組合** | 硬幣找零枚數組合、擲骰子點數組合窮舉 |

---

# 1. 無窮迭代器 (Infinite Iterators)

無窮迭代器不會自動停止，呼叫時必須透過 `islice()`、`for` 迴圈搭配 `break` 或條件中斷，切勿直接傳入 `list()`。

##### count() 無窮計數迭代器

- **使用時機**：需要一個無限遞增或遞減的數列時使用，例如作為自增流水號 ID、產生時間刻度，或搭配 `zip()` 取代手動維護的計數器變數。
- **語法**：`itertools.count(start=0, step=1)`
- **參數說明**：
  - `start`：計數起始數值（支援整數或浮點數，預設為 `0`）。
  - `step`：每次累加步長（支援整數、浮點數或負數，預設為 `1`）。
- **回傳值**：
  - 無窮數值迭代器。

```python
import itertools

# 1. 基本計數器：從 10 開始，每次加 2
counter = itertools.count(start=10, step=2)

print(next(counter))  # 輸出: 10
print(next(counter))  # 輸出: 12
print(next(counter))  # 輸出: 14

# 2. 實戰應用：為無索引資料流自動打上時間刻度或編號
tasks = ["資料下載", "特徵提取", "模型訓練"]
numbered_tasks = list(zip(itertools.count(1), tasks))
print(numbered_tasks)  # 輸出: [(1, '資料下載'), (2, '特徵提取'), (3, '模型訓練')]
```

---

##### cycle() 循環重複迭代器

- **使用時機**：需要將一個既有集合的元素無限週期性重複播放時使用，例如輪詢分配任務、終端交替背景顏色、回合制玩家輪替。
- **語法**：`itertools.cycle(iterable)`
- **參數說明**：
  - `iterable`：要循環輪播的可迭代物件（串列、字串、元組等）。
- **回傳值**：
  - 無窮循環迭代器。

> **[小提醒]**：`cycle()` 內部會快取輸入序列的所有元素。若傳入無窮生成器或極大檔案，會導致記憶體持續膨脹！請確保傳入的 `iterable` 為有限且大小可控的集合。

```python
import itertools

# 1. 建立紅綠燈狀態輪播器
traffic_lights = itertools.cycle(["紅燈", "綠燈", "黃燈"])

# 2. 模擬連續 5 次切換
states = [next(traffic_lights) for _ in range(5)]
print(states)  # 輸出: ['紅燈', '綠燈', '黃燈', '紅燈', '綠燈']
```

---

##### repeat() 固定重複迭代器

- **使用時機**：
  - 需要固定產出某個相同物件時使用。
  - 最常見是搭配 `map()` 或 `starmap()`，為多引數運算提供固定的常數引數。
- **語法**：`itertools.repeat(object[, times])`
- **參數說明**：
  - `object`：要重複輸出的目標物件實體。
  - `times`：可選，重複次數整數。若未指定則為無窮重複。
- **回傳值**：
  - 固定物件迭代器。

```python
import itertools

# 1. 有限次數重複
dots = list(itertools.repeat("點", 3))
print(dots)  # 輸出: ['點', '點', '點']

# 2. 實戰搭配 map：對串列每個數字進行固定的 3 次方計算
bases = [1, 2, 3, 4]
exponents = itertools.repeat(3)  # 提供固定常數 3
cubes = list(map(pow, bases, exponents))
print(cubes)  # 輸出: [1, 8, 27, 64]
```

---

# 2. 串流切片與條件過濾 (Slicing & Filtering)

##### islice() 迭代器高效切片

- **使用時機**：
  - 需要對**一般迭代器或生成器**（無法使用 `[start:stop]` 切片語法者）進行範圍截取時使用。
  - 大型日誌檔案分頁讀取、無窮序列截取前 N 筆，且**完全不將資料載入記憶體**。
- **語法**：
  - `itertools.islice(iterable, stop)`
  - `itertools.islice(iterable, start, stop[, step])`
- **參數說明**：
  - `iterable`：目標可迭代物件。
  - `start`：起始索引（從 0 開始；若為單一引數形式則為 stop）。
  - `stop`：結束索引（不包含此索引；若為 `None` 則截取至迭代器末尾）。
  - `step`：步長（可選，必須為正整數，**不支援負數步長**）。
- **回傳值**：
  - 切片迭代器。

```python
import itertools

# 1. 截取無窮 count() 的前 5 筆
infinite_seq = itertools.count(10, step=5)
first_five = list(itertools.islice(infinite_seq, 5))
print(first_five)  # 輸出: [10, 15, 20, 25, 30]

# 2. 範圍切片與步長：從索引 2 取到索引 8，步長為 2
data = [0, 10, 20, 30, 40, 50, 60, 70, 80, 90]
sliced = list(itertools.islice(data, 2, 8, 2))
print(sliced)  # 輸出: [20, 40, 60]
```

---

##### takewhile() 滿足條件前持續截取

- **使用時機**：在串流開頭持續截取資料，**一旦遇到第一個不符合條件的元素，立刻永久終止迭代**。適用於已排序資料的前段有效資料提取。
- **語法**：`itertools.takewhile(predicate, iterable)`
- **參數說明**：
  - `predicate`：條件斷言函式，接收元素並回傳布林值。
  - `iterable`：目標可迭代物件。
- **回傳值**：
  - 條件符合之元素迭代器。

```python
import itertools

numbers = [1, 3, 5, 8, 9, 2]

# 只要是小於 6 的奇數或數字就持續拿，直到遇到 8 (不滿足 < 6) 立即停止
# 注意：後面的 2 即使小於 6 也不會被取得！
result = list(itertools.takewhile(lambda x: x < 6, numbers))
print(result)  # 輸出: [1, 3, 5]
```

---

##### dropwhile() 滿足條件時持續跳過

- **使用時機**：在串流開頭持續丟棄資料，**一旦遇到第一個不符合條件的元素，從該元素起產出其後續的所有剩餘資料**（後續元素不再受條件檢驗）。
- **語法**：`itertools.dropwhile(predicate, iterable)`
- **參數說明**：
  - `predicate`：條件斷言函式。
  - `iterable`：目標可迭代物件。
- **回傳值**：
  - 跳過前置元素後的剩餘元素迭代器。

```python
import itertools

lines = [
    "# 設定檔註釋 A",
    "# 設定檔註釋 B",
    "host=localhost",
    "port=8080",
    "# 結尾註釋",
]

# 跳過檔案開頭所有以 # 開頭的註釋行，首遇設定項後全數保留
valid_lines = list(itertools.dropwhile(lambda s: s.startswith("#"), lines))
print(valid_lines)  # 輸出: ['host=localhost', 'port=8080', '# 結尾註釋']
```

---

##### filterfalse() 反向過濾保留假值

- **使用時機**：與 Python 內建 `filter()` 相反。保留斷言函式回傳 `False`（或評估為假）的項目，免除手動撰寫 `lambda x: not condition(x)`。
- **語法**：`itertools.filterfalse(predicate, iterable)`
- **參數說明**：
  - `predicate`：條件斷言函式。若傳入 `None`，則保留所有本身為假值 (Falsy) 的元素（如 `0`, `""`, `None`）。
  - `iterable`：目標可迭代物件。
- **回傳值**：
  - 過濾後元素迭代器。

```python
import itertools

data = [12, -5, 0, 8, -22, 17]

# 找出所有非負數（即不滿足 x < 0 的元素）
non_negatives = list(itertools.filterfalse(lambda x: x < 0, data))
print(non_negatives)  # 輸出: [12, 0, 8, 17]
```

---

##### compress() 布林遮罩過濾器

- **使用時機**：擁有一個資料串流與另一個等長的布林旗標串流（Selectors），需要依照布林遮罩（True 挑出、False 捨棄）進行向量化篩選時使用。
- **語法**：`itertools.compress(data, selectors)`
- **參數說明**：
  - `data`：原始資料可迭代物件。
  - `selectors`：由布林值或 Truthy/Falsy 評估值組成的選擇器序列。
- **回傳值**：
  - 遮罩篩選後之元素迭代器。

```python
import itertools

students = ["Alice", "Bob", "Charlie", "David"]
passed_exam = [True, False, True, False]

# 依據 passed_exam 的 True/False 挑選通過考試的學生
passed_students = list(itertools.compress(students, passed_exam))
print(passed_students)  # 輸出: ['Alice', 'Charlie']
```

---

# 3. 序列組合、分塊與運算 (Combining, Batching & Aggregating)

##### chain() 與 chain.from_iterable() 鏈結串接迭代器

- **使用時機**：
  - 需要將多個不同可迭代物件平滑串聯為單一連續迭代器時使用。
  - **效能核心**：取代 `list1 + list2`，完全無需在記憶體中建立合併後的全新列表！
  - `from_iterable()`：專門用於將二維可迭代結構（如巢狀清單）平坦化 (Flatten)。
- **語法**：
  - 多參數形式：`itertools.chain(*iterables)`
  - 單一容器形式：`itertools.chain.from_iterable(iterable)`
- **回傳值**：
  - 連續鏈結迭代器。

```python
import itertools

# 1. 基本 chain：依序接合多個獨立容器
list_a = [1, 2, 3]
tuple_b = ("a", "b")
set_c = {100}

combined = list(itertools.chain(list_a, tuple_b, set_c))
print(combined)  # 輸出: [1, 2, 3, 'a', 'b', 100]

# 2. chain.from_iterable：極速二維巢狀列表展平 (Flatten)
matrix = [[1, 2], [3, 4], [5, 6]]
flat = list(itertools.chain.from_iterable(matrix))
print(flat)  # 輸出: [1, 2, 3, 4, 5, 6]
```

---

##### zip_longest() 最長拉鍊配對補齊

- **使用時機**：
  - 類似內建 `zip()`，但當傳入的多個容器長度不同時，**不會在最短者結束時截斷**，而是以**最長者為基準**，短的自動以 `fillvalue` 補齊。
- **語法**：`itertools.zip_longest(*iterables, fillvalue=None)`
- **參數說明**：
  - `*iterables`：多個待配對的可迭代物件。
  - `fillvalue`：當較短序列耗盡時，用來補齊缺失位置的預設值（預設為 `None`）。
- **回傳值**：
  - 成對元組迭代器。

```python
import itertools

names = ["Alice", "Bob", "Charlie"]
scores = [95, 88]

# 1. 預設補 None
paired_none = list(itertools.zip_longest(names, scores))
print(paired_none)  # 輸出: [('Alice', 95), ('Bob', 88), ('Charlie', None)]

# 2. 指定補齊預設值
paired_zero = list(itertools.zip_longest(names, scores, fillvalue=0))
print(paired_zero)  # 輸出: [('Alice', 95), ('Bob', 88), ('Charlie', 0)]
```

---

##### starmap() 參數元組解包映射

- **使用時機**：當容器中的元素已經是以元組形式預先打包好的引數（例如 `[(2, 3), (4, 5)]`），需要直接對每個元組進行解包 (`func(*args)`) 運算時使用。
- **語法**：`itertools.starmap(function, iterable)`
- **參數說明**：
  - `function`：計算函式。
  - `iterable`：元素為參數元組的可迭代物件。
- **回傳值**：
  - 計算結果迭代器。

```python
import itertools

# 每一項元組包含 (底數, 指數)
pairs = [(2, 5), (3, 2), (10, 3)]

# starmap 自動等價於 pow(*pair)
results = list(itertools.starmap(pow, pairs))
print(results)  # 輸出: [32, 9, 1000]
```

---

##### accumulate() 累加與累計運算器

- **使用時機**：
  - 計算前綴和（累積加總）、前綴乘積或連續累積極值時使用。
  - 每個產出的元素代表「當前位置以前所有元素運算後的累積結果」。
- **語法**：`itertools.accumulate(iterable[, func, *, initial=None])`
- **參數說明**：
  - `iterable`：目標數值可迭代物件。
  - `func`：二元運算函式（接收兩個參數，預設為 `operator.add` 累加）。
  - `initial`：(Python 3.8+) 起始底數。若指定則產出結果的第一個元素為 `initial`。
- **回傳值**：
  - 累計值迭代器。

```python
import itertools
import operator

data = [1, 2, 3, 4, 5]

# 1. 預設前綴和 (累加)
cumsum = list(itertools.accumulate(data))
print(cumsum)  # 輸出: [1, 3, 6, 10, 15]

# 2. 自訂運算：累積連乘 (階乘流)
cumprod = list(itertools.accumulate(data, operator.mul))
print(cumprod)  # 輸出: [1, 2, 6, 24, 120]

# 3. 追蹤前綴最大值 (Running Maximum)
prices = [10, 8, 15, 12, 20, 18]
running_max = list(itertools.accumulate(prices, max))
print(running_max)  # 輸出: [10, 10, 15, 15, 20, 20]
```

---

##### pairwise() 相鄰元素成對滑動視窗

- **使用時機**：在 **Python 3.10+** 引入。將序列以長度為 2、步長為 1 的滑動視窗重疊輸出。極度適合計算相鄰差值、路徑相鄰座標距離或狀態轉換。
- **語法**：`itertools.pairwise(iterable)`
- **參數說明**：
  - `iterable`：目標可迭代物件（若長度小於 2 則產出為空）。
- **回傳值**：
  - 相鄰二元組迭代器 `((s0, s1), (s1, s2), (s2, s3), ...)`。

```python
import itertools

points = ["A", "B", "C", "D"]

# 產出相鄰路徑段落
segments = list(itertools.pairwise(points))
print(segments)  # 輸出: [('A', 'B'), ('B', 'C'), ('C', 'D')]

# 計算相鄰每日溫差
daily_temps = [25, 28, 26, 31]
diffs = [next_day - prev_day for prev_day, next_day in itertools.pairwise(daily_temps)]
print(diffs)  # 輸出: [3, -2, 5]
```

---

##### batched() 定長批次分組切塊

- **使用時機**：在 **Python 3.12+** 引入。將串流按照固定大小 `n` 拆分為不重疊的批次元組。常用於資料庫批次寫入、API 批次請求或分塊處理。
- **語法**：`itertools.batched(iterable, n)`
- **參數說明**：
  - `iterable`：目標可迭代物件。
  - `n`：每批大小正整數（若最後一批不足 `n` 個則包含所有剩餘元素）。
- **回傳值**：
  - 批次元組迭代器。

```python
import itertools

records = [1, 2, 3, 4, 5, 6, 7, 8]

# 每 3 筆切成一批
batches = list(itertools.batched(records, 3))
print(batches)  # 輸出: [(1, 2, 3), (4, 5, 6), (7, 8)]
```

---

# 4. 分組與分流 (Grouping & Splitting)

##### groupby() 連續相鄰元素鍵值分組

- **使用時機**：
  - 將資料流中具備相同特徵的連續相鄰元素打包分組。
  - 核心特徵：每當 Key 改變時就會產生一個新分組。
- **語法**：`itertools.groupby(iterable, key=None)`
- **參數說明**：
  - `iterable`：目標可迭代物件。
  - `key`：特徵提取計算函式（若未指定則以元素本身作為分組依據）。
- **回傳值**：
  - 產生 `(key, group_iterator)` 二元組的迭代器。

> **[核心天條]：groupby 僅對「連續相鄰」的相同元素分組！**  
> 若要對整體資料進行全域聚合分組，**呼叫前必須先使用相同的 key 函式進行 `sorted()` 排序**！否則相同的 Key 若分散在各處，會被切斷成多個彼此獨立的零散分組！

```python
import itertools

students = [
    {"name": "Alice", "city": "台北"},
    {"name": "Bob", "city": "台中"},
    {"name": "Charlie", "city": "台北"},
    {"name": "David", "city": "台中"},
]

# 1. 核心天條：先按分組 key 排序！
key_func = lambda s: s["city"]
sorted_students = sorted(students, key=key_func)

# 2. 進行分組
grouped_result = {}
for city, group in itertools.groupby(sorted_students, key=key_func):
    # group 是一個只能被迭代一次的生成器，務必立即轉為 list 保存
    grouped_result[city] = [s["name"] for s in group]

print(grouped_result)
# 輸出: {'台中': ['Bob', 'David'], '台北': ['Alice', 'Charlie']}
```

---

##### tee() 複製分裂獨立迭代器

- **使用時機**：
  - 需要對同一個只能遍歷一次的單向資料流（如產生器、網路 Socket 串流）進行多次不同邏輯的讀取時使用。
  - 將單一迭代器分裂為 `n` 個完全各自獨立的迭代器。
- **語法**：`itertools.tee(iterable, n=2)`
- **參數說明**：
  - `iterable`：原始可迭代物件。
  - `n`：分裂產生的獨立迭代器副本數量（預設為 `2`）。
- **回傳值**：
  - 包含 `n` 個獨立迭代器的元組。

> **[核心天條]：tee() 建立副本後，絕對嚴禁再直接讀取原始迭代器！**  
> 一旦原始迭代器在外部被讀取推進，`tee` 內部維護的暫存佇列會產生狀態脫節。此外，若多個分裂迭代器讀取的進度差距過大，底層會被迫在記憶體快取巨大落差區段，造成記憶體爆滿！

```python
import itertools

# 原始生成器
def data_stream():
    for i in range(1, 4):
        yield i

stream = data_stream()

# 1. 分裂為兩個完全獨立的迭代器
iter1, iter2 = itertools.tee(stream, 2)

# 2. 兩者可各自獨立前進，互不干擾
print(list(iter1))  # 輸出: [1, 2, 3]
print(list(iter2))  # 輸出: [1, 2, 3]
```

---

# 5. 排列組合迭代器 (Combinatoric Iterators)

排列組合函式能以數學維度窮舉出所有可能組合，且全程保持惰性生成，避免記憶體爆炸。

##### product() 笛卡兒積多重迴圈展開

- **使用時機**：
  - 計算多個集合的笛卡兒積 (Cartesian Product)。
  - **程式碼美化神器**：將三層以上巢狀的 `for x in A: for y in B: for z in C:` 扁平化展開為單一乾淨迴圈！
  - 機器學習超參數網格搜尋 (Grid Search)。
- **語法**：`itertools.product(*iterables, repeat=1)`
- **參數說明**：
  - `*iterables`：多個可迭代集合。
  - `repeat`：重複與自身進行笛卡兒積的次數（例如 `product(A, repeat=3)` 等價於 `product(A, A, A)`）。
- **回傳值**：
  - 笛卡兒積元組迭代器。

```python
import itertools

# 1. 多集合笛卡兒積：展開顏色與尺寸的所有搭配
colors = ["紅", "藍"]
sizes = ["S", "M"]
combinations = list(itertools.product(colors, sizes))
print(combinations)
# 輸出: [('紅', 'S'), ('紅', 'M'), ('藍', 'S'), ('藍', 'M')]

# 2. 模擬連續擲硬幣 2 次的所有可能結果 (自身 repeat)
coin_flips = list(itertools.product(["正面", "反面"], repeat=2))
print(coin_flips)
# 輸出: [('正面', '正面'), ('正面', '反面'), ('反面', '正面'), ('反面', '反面')]
```

---

##### permutations() 排列生成器

- **使用時機**：
  - 計算排列數 $P(n, r) = rac{n!}{(n-r)!}$。
  - **考慮順序**，元素不可重複選取。例如 `(A, B)` 與 `(B, A)` 被視為兩種不同排列。
- **語法**：`itertools.permutations(iterable, r=None)`
- **參數說明**：
  - `iterable`：目標元素池。
  - `r`：每次選取的元素個數。若未指定或為 `None`，預設為選取全部元素（即全排列 $n!$）。
- **回傳值**：
  - 排列元組迭代器。

```python
import itertools

items = ["A", "B", "C"]

# 從 3 個字母中選取 2 個進行排列 (3 * 2 = 6 種)
perms = list(itertools.permutations(items, 2))
print(perms)
# 輸出: [('A', 'B'), ('A', 'C'), ('B', 'A'), ('B', 'C'), ('C', 'A'), ('C', 'B')]
```

---

##### combinations() 組合生成器

- **使用時機**：
  - 計算組合數 $C(n, r) = rac{n!}{r!(n-r)!}$。
  - **不考慮順序**，元素不可重複選取。例如挑出 `('A', 'B')` 後，就不會再出現 `('B', 'A')`。
- **語法**：`itertools.combinations(iterable, r)`
- **參數說明**：
  - `iterable`：目標元素池。
  - `r`：每次選取的元素個數整數（必填引數）。
- **回傳值**：
  - 組合元組迭代器。

```python
import itertools

items = ["A", "B", "C", "D"]

# 從 4 位候選人中挑選 2 位搭檔 (4 * 3 / 2 = 6 種)
combs = list(itertools.combinations(items, 2))
print(combs)
# 輸出: [('A', 'B'), ('A', 'C'), ('A', 'D'), ('B', 'C'), ('B', 'D'), ('C', 'D')]
```

---

##### combinations_with_replacement() 可重複選取組合生成器

- **使用時機**：
  - **不考慮順序**，但**允許同一個元素被重複挑選多次**。
  - 例如買 2 球冰淇淋，口味允許重複點兩球巧克力 `('巧', '巧')`。
- **語法**：`itertools.combinations_with_replacement(iterable, r)`
- **參數說明**：
  - `iterable`：目標元素池。
  - `r`：每次選取的元素個數整數（必填引數）。
- **回傳值**：
  - 可重複組合元組迭代器。

```python
import itertools

items = ["A", "B", "C"]

# 從 3 種元素中允許重複挑選 2 個
rep_combs = list(itertools.combinations_with_replacement(items, 2))
print(rep_combs)
# 輸出: [('A', 'A'), ('A', 'B'), ('A', 'C'), ('B', 'B'), ('B', 'C'), ('C', 'C')]
```

---

#### 排列組合四大函式維度對照表

| 函式名稱 | 考慮先後順序？ | 允許自身重複挑選？ | 數學概念 | 範例輸入 `['A', 'B']` 挑 2 個結果 |
| :--- | :--- | :--- | :--- | :--- |
| **`product(p, repeat=r)`** | **是**（`AB != BA`） | **是**（允許 `AA`, `BB`） | 笛卡兒積 / 乘法原理 | `('A', 'A'), ('A', 'B'), ('B', 'A'), ('B', 'B')` |
| **`permutations(p, r)`** | **是**（`AB != BA`） | **否**（無重複元素） | 排列數 $P(n, r)$ | `('A', 'B'), ('B', 'A')` |
| **`combinations(p, r)`** | **否**（只看組合內容） | **否**（無重複元素） | 組合數 $C(n, r)$ | `('A', 'B')` |
| **`combinations_with_replacement(p, r)`** | **否**（只看組合內容） | **是**（允許 `AA`, `BB`） | 重複組合 $H(n, r)$ | `('A', 'A'), ('A', 'B'), ('B', 'B')` |

---

# 實戰避坑與核心天條

## 1. groupby 未預先排序導致分組破碎成多份零散組

> **[核心天條]：呼叫 `itertools.groupby()` 前，輸入序列必須已按相同的分組 Key 完成排序！**

- **錯誤症狀**：
  預期將清單分為 2 個類別，結果 `groupby` 跑出 5 個零碎分組，相同 Key 的資料分散在多個字典項目中。
- **背後原理**：
  `itertools.groupby` 底層採用串流掃描，只檢查「當前元素是否與前一個元素具備相同的 Key」。一旦 Key 發生改變就立刻結算並開新組，無法跨越不同區段自動歸納。

```python
import itertools

data = [
    {"type": "A", "val": 1},
    {"type": "B", "val": 2},
    {"type": "A", "val": 3},  # A 與前一個 A 隔著 B
]

# [錯誤寫法]：未排序直接分組，產生兩個零碎的 A 群組
# for k, g in itertools.groupby(data, key=lambda x: x["type"]):
#     print(k, list(g)) # A 出現兩次！

# [正確寫法]：先使用相同 key 函式排序再分組
sorted_data = sorted(data, key=lambda x: x["type"])
grouped = {k: list(g) for k, g in itertools.groupby(sorted_data, key=lambda x: x["type"])}
print(grouped.keys())  # 輸出: dict_keys(['A', 'B'])
```

---

## 2. 迭代器一次性消耗特性導致後續重讀為空

> **[核心天條]：itertools 產出的物件是一次性迭代器，遍歷完畢後指標停留在最末端，再次迭代直接為空！**

- **錯誤症狀**：
  第一次使用 `for` 迴圈或 `sum()` 有資料，第二次讀取時變數直接變成空清單或 `0`。
- **背後原理**：
  `itertools` 的物件實作了迭代器協定（`__next__`），內部只維護當前游標偏移量。一旦遍歷至觸發 `StopIteration`，游標無法自動倒帶重置。若需多次使用，必須在第一次遍歷時顯式轉換為 `list()`，或使用 `itertools.tee()` 分裂。

```python
import itertools

# [錯誤寫法]：重複消費同一個迭代器
acc = itertools.accumulate([1, 2, 3])
print(list(acc))  # 輸出: [1, 3, 6]
print(list(acc))  # 輸出: [] (已經被消耗殆盡！)

# [正確寫法 A]：若資料量不大，即刻轉為 list 保存
acc_list = list(itertools.accumulate([1, 2, 3]))

# [正確寫法 B]：若為串流且需雙重讀取，使用 tee 分流
a1, a2 = itertools.tee(itertools.accumulate([1, 2, 3]))
```

---

## 3. 無窮迭代器直接丟進 list() 引爆記憶體崩潰

> **[核心天條]：嚴禁對無窮迭代器（count、cycle、無上限 repeat）直接調用 `list()` 或 `tuple()`！**

- **錯誤症狀**：
  程式瞬間無回應、CPU 狂飆至 100%、記憶體迅速耗盡並觸發作業系統 OOM Killer 或 Python `MemoryError` 崩潰。
- **背後原理**：
  `list()` 會不斷呼叫 `next()` 直到拋出 `StopIteration`。但無窮迭代器永遠不會觸發終止條件，導致無限膨脹直到記憶體完全被塞爆。

```python
import itertools

# [錯誤寫法]：直接轉換無窮生成器，導致系統記憶體溢出
# bad = list(itertools.count(1))  # 永遠無法結束，直接記憶體爆炸！

# [正確寫法]：必須使用 islice 限制截取長度，或在迴圈中主動 break
safe_list = list(itertools.islice(itertools.count(1), 10))
print(safe_list)  # 輸出: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

---

## 4. groupby 內部 group 生成器未及時消費引發資料覆蓋

> **[核心天條]：在 groupby 迴圈中，子迭代器 `group` 必須在下一輪迴圈開始前完全消費完畢！**

- **錯誤症狀**：
  試圖把所有 `group` 生成器搜集到字典或列表中晚點處理，結果發現前面的組全部變成空清單或資料被覆蓋。
- **背後原理**：
  `itertools.groupby` 內部共用同一個底層游標。當外層推進到下一個 Key 時，前一個分組的子生成器會被底層自動快轉並失效。若未在該輪迴圈內呼叫 `list(group)`，資料將永久遺失。

```python
import itertools

data = [("A", 1), ("A", 2), ("B", 3)]

# [錯誤寫法]：保留子迭代器延後消費，導致第一組資料遺失
# groups = {k: g for k, g in itertools.groupby(data, key=lambda x: x[0])}
# print(list(groups["A"]))  # 輸出: [] (資料已被底層前進吞噬！)

# [正確寫法]：在每輪迴圈中當場以 list(g) 實體化保存
groups = {k: list(g) for k, g in itertools.groupby(data, key=lambda x: x[0])}
print(groups["A"])  # 輸出: [('A', 1), ('A', 2)]
```
