# 概念與原理

## 什麼是 Pandas？

Pandas 是 Python 生態系中最核心、最普及的高效能結構化資料分析與清洗標準函式庫。

其底層基於 C 語言優化的 NumPy 陣列構建，專為處理二維表格型資料（Tabular Data）、時間序列資料以及異質關聯式資料而設計。

在資料科學、機器學習工程與商業分析中，原生 Python 的串列（List）與字典（Dict）在面對數十萬筆甚至數千萬筆資料時運算效能低落且缺乏標籤對齊機制。

Pandas 提供具備軸標籤（Index）的資料結構，將複雜的矩陣切片、分組聚合、時間重取樣與表格關聯簡化為直觀的單行 API。

> **[生動比喻]**：  
> - **原生 Python 串列與字典**：像「手動計算紙與活頁筆記本」，每一筆運算都要自己寫迴圈一筆一筆翻閱，容易出錯且耗時費力。  
> - **Pandas**：像「具備渦輪引擎與自動化巨集的現代程式化 Excel / SQL 引擎」！不僅能在一瞬間對數百萬列資料完成直向加總與橫向過濾，還能自帶座標標籤，讓數據處理快如閃電且條理分明。  
> - **Series**：像「Excel 試算表中的單一垂直直行」，每一格都有對應的行號（Index 標籤）。  
> - **DataFrame**：像「整張完整的 Excel 工作表（Worksheet）」，同時具備直向欄標籤（Columns）與橫向列標籤（Index）。

---

## 核心資料結構與軸向（Axis）哲學

Pandas 的世界由兩大核心基石構成：

1. **`Series`（一維標籤化數列）**：單一欄位，由同質的資料值（Values）與專屬軸標籤（Index）組成。
2. **`DataFrame`（二維結構化資料表）**：二維矩陣，本質上是由多個共用同一個 Index 的 Series 並排組合而成字典結構。

### 核心軸向 (Axis) 走向與計算口訣

在 Pandas 中，幾乎所有操作（如 `drop`、`sum`、`mean`、`apply`）都會遇到 `axis` 參數：

```text
               axis = 1  (水平向右跨欄 ──>)
               Column 0    Column 1    Column 2
axis = 0   ┌───────────┬───────────┬───────────┐
(垂直向下  │  Row 0    │  Row 0    │  Row 0    │
跨列走訪   ├───────────┼───────────┼───────────┤
   │       │  Row 1    │  Row 1    │  Row 1    │
   v)      └───────────┴───────────┴───────────┘
```

- **`axis=0`（或 `axis='index'`）**：**垂直向下走訪！**  
  沿著「列（Row）」的方向向下壓扁計算。例如 `df.sum(axis=0)` 會把每一列壓在一起，算出**每一欄（Column）的垂直總和**。
- **`axis=1`（或 `axis='columns'`）**：**水平向右走訪！**  
  沿著「欄（Column）」的方向向右壓扁計算。例如 `df.sum(axis=1)` 會把同一列的所有欄位加在一起，算出**每一列（Row）的橫向總分**。

---

## 現代語法升級與版本演進對照表

Pandas 在 2.0+ 與 2.1+ 經歷重大架構優化，請務必遵循現代標準語法：

| 功能操作 | 舊式語法 (已棄用 / 易踩坑) | 現代標準語法 (推薦) | 核心差異與說明 |
| :--- | :--- | :--- | :--- |
| **全表逐格元素映射** | `df.applymap(func)` | `df.map(func)` | Pandas 2.1+ 正式棄用 `applymap()`，統一由 `map()` 接管 |
| **數列沿用填補** | `s.fillna(method='ffill')` | `s.ffill()` | `fillna(method=...)` 參數已廢棄，改用專屬獨立語意函式 |
| **回傳底層純矩陣** | `df.values` | `df.to_numpy()` | `values` 回傳型別不可預測，`to_numpy()` 支援精確 dtype 轉換 |
| **字串欄位底層引擎** | `object` (Python 字串指標) | `string[pyarrow]` | Pandas 2.0+ 引入 PyArrow 後端字串，記憶體省數倍且運算極速 |
| **欄位刪除** | `df.drop(['A'], axis=1)` | `df.drop(columns=['A'])` | 顯式指定 `columns=` 參數，擺脫容易混淆的 `axis=1` |
| **向前填補方法名稱** | `df.fillna(method='pad')` | `df.ffill()` | 舊有別名廢棄，全面統一為 `ffill()` 與 `bfill()` |

---

# 核心 API 類別、函式與屬性字典對照表

| 類別 / 函式 / 屬性名稱 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- |
| **[[#pd.Series() 建立一維標籤化陣列\|pd.Series()]]** | **建立帶有軸標籤（Index）的一維同質陣列實例** | 單一特徵欄位處理、時間序列、字典轉換為標籤數列 |
| **[[#pd.DataFrame() 建立二維表格資料結構\|pd.DataFrame()]]** | **建立具備列標籤（Index）與欄標籤（Columns）的二維異質資料表** | 整合多來源資料、巢狀字典結構化、表格初始化 |
| **[[#pd.date_range() 產生連續時間戳記索引數列\|pd.date_range()]]** | **產生固定頻率的 DatetimeIndex 時間軸序列** | 金融量化時間序列對齊、歷史日誌索引補齊、週期性任務排程 |
| **[[#pd.read_csv() 讀取 CSV 與分隔文字檔\|pd.read_csv()]]** | **將逗號分隔值（CSV）或自訂分隔符文字檔載入為 DataFrame** | 讀取外部資料集、匯入日誌檔、大型數據批次載入 |
| **[[#DataFrame.to_csv() 匯出資料表為 CSV 檔案\|df.to_csv()]]** | **將 DataFrame 二維資料寫入本機 CSV 格式檔案或字串緩衝區** | 模型預測結果存檔、報表匯出、管道處理中繼保存 |
| **[[#pd.read_excel() 與 DataFrame.to_excel() 讀寫 Excel 工作表\|pd.read_excel() / df.to_excel()]]** | **跨平台讀取與匯出微軟 Excel（.xlsx / .xls）試算表活頁簿** | 辦公室自動化報表、財務模型數據對接、多 Sheet 工作表解析 |
| **[[#pd.read_json() 與 DataFrame.to_json() 讀寫 JSON 結構化檔案\|pd.read_json() / df.to_json()]]** | **解析或序列化 JSON 字串與文字檔案為 DataFrame** | RESTful API 資料對接、NoSQL 資料流載入、前端視覺化資料交換 |
| **[[#pd.read_sql() 與 DataFrame.to_sql() 關聯式資料庫讀寫\|pd.read_sql() / df.to_sql()]]** | **透過 SQLAlchemy 引擎執行 SQL 查詢並將結果轉為 DataFrame 或反向入庫** | 從 SQLite/PostgreSQL/MySQL 提取訓練集、批次入庫分析報表 |
| **[[#DataFrame.shape 表格形狀維度元組\|df.shape]]** | **以二元元組 (列數, 欄數) 快速取得資料表幾何維度** | 資料載入後的首要檢查、維度變更斷言（Assert）、特徵工程維度追蹤 |
| **[[#DataFrame.dtypes 欄位資料型別數列\|df.dtypes]]** | **檢視 DataFrame 各欄位的底層資料型別（Dtype）** | 檢查是否有數值欄位被誤判為 object 字串、型別異常排查 |
| **[[#DataFrame.columns 與 DataFrame.index 欄列標籤索引物件\|df.columns / df.index]]** | **讀取、檢查或批次覆寫資料表的欄位名稱清單與列索引清單** | 欄位重命名標準化、全表欄名清洗、行索引重設 |
| **[[#DataFrame.values 與 DataFrame.to_numpy() 轉為底層 NumPy 陣列\|df.values / df.to_numpy()]]** | **提取 DataFrame 的底層數據矩陣，剝除標籤轉為純 NumPy ndarray** | 對接 scikit-learn 特徵矩陣、深度學習張量轉換、底層線性代數運算 |
| **[[#DataFrame.size 與 DataFrame.ndim 元素總數與維度數\|df.size / df.ndim]]** | **獲取資料表內含的純量元素總數與維度階數** | 防禦性檢查資料表是否為空、記憶體開銷估計 |
| **[[#DataFrame.head() 與 DataFrame.tail() 預覽首尾資料列\|df.head() / df.tail()]]** | **快速抽樣檢視資料表前 N 列或後 N 列資料** | 載入資料後的初次視覺檢查、驗證轉換後的資料外觀 |
| **[[#DataFrame.info() 列印資料表記憶體與欄位摘要\|df.info()]]** | **在標準輸出中印出資料表結構摘要：索引範圍、欄位型別、非空計數與記憶體用量** | 評估資料表缺失狀況、識別不合適的資料型態、記憶體優化診斷 |
| **[[#DataFrame.describe() 生成數值與類別統計摘要\|df.describe()]]** | **計算各欄位的描述性統計量（計數、均值、標準差、分位數、極值）** | 探索性資料分析（EDA）、辨識數值異常偏態、離群值探查 |
| **[[#Series.value_counts() 類別值頻率計數\|s.value_counts()]]** | **計算單一欄位中各相異值出現的次數分配，依頻率降冪排序** | 類別型特徵分佈檢查、不平衡資料集診斷、調查問卷次數統計 |
| **[[#Series.unique() 與 Series.nunique() 相異值清單與計數\|s.unique() / s.nunique()]]** | **獲取單一數列中所有不重複值的陣列，或統計不重複值的數量** | 檢查特徵基數（Cardinality）、枚舉標籤清單、主鍵唯一性校驗 |
| **[[#DataFrame.loc[] 基於標籤的切片與選取\|df.loc[]]]** | **透過列標籤名稱與欄標籤名稱精確切片選取或賦值** | 根據日期索引查值、特定名稱欄位選取、包含首尾之區間切片 |
| **[[#DataFrame.iloc[] 基於整數位置的切片與選取\|df.iloc[]]]** | **透過純整數位置索引（從 0 起算）進行列與欄的切片或賦值** | 提取前 N 列特定欄、陣列式矩陣取值、排除最後一欄特徵分割 |
| **[[#DataFrame.at[] 與 DataFrame.iat[] 超高效純量存取\|df.at[] / df.iat[]]]** | **針對單一儲存格進行極速讀取或寫入（略過 Series 構建開銷）** | 高頻迴圈中讀寫單一數值、單點數值即時覆寫修正 |
| **[[#DataFrame.query() 基於布林表達式字串快速篩選\|df.query()]]** | **使用類似 SQL 的直觀字串表達式過濾資料列** | 複雜多條件過濾、程式化動態組合過濾條件、提升篩選代碼可讀性 |
| **[[#Series.isin() 集合成員資格條件遮罩\|s.isin() / df.isin()]]** | **檢查數列各元素是否存在於指定清單或集合中，返回布林遮罩** | 類別篩選白名單、過濾多個特定城市或編號、多選條件比對 |
| **[[#DataFrame.filter() 依標籤名稱篩選欄或列\|df.filter()]]** | **根據特定子字串、正規表達式或清單選取欄位或索引標籤** | 從數百個欄位中挑選特定前綴欄位（如 'feature_*'）、特徵名稱批次提取 |
| **[[#DataFrame.isna() 與 DataFrame.notna() 缺失值偵測布林遮罩\|df.isna() / df.notna()]]** | **逐格檢查純量是否為缺失值（NaN、None），產生等維度布林遮罩** | 統計各欄缺失數量、資料清洗防禦性驗證、篩選空值列 |
| **[[#DataFrame.dropna() 剔除包含空值的列或欄\|df.dropna()]]** | **根據門檻規則過濾並刪除帶有缺失值的資料列（Row）或資料欄（Column）** | 去除無效問卷列、移除缺失率過高之欄位、模型訓練前資料完整化 |
| **[[#DataFrame.fillna() 填補缺失值\|df.fillna()]]** | **使用指定常數、欄位均值、中位數或向前向後推移填補 NaN 空缺** | 特徵工程缺失補齊、金融數據沿用前一日報價、分類缺值標記為 Unknown |
| **[[#DataFrame.ffill() 與 DataFrame.bfill() 向前向後沿用填補\|df.ffill() / df.bfill()]]** | **以最後一個有效觀測值向前推進（Forward fill）或向後回溯填補缺失值** | 時間序列缺漏填補、感測器信號斷訊沿用、狀態維護 |
| **[[#DataFrame.astype() 明確轉換欄位資料型別\|df.astype()]]** | **強制將欄位轉換為指定型別（如 int32, float64, string, category）** | 記憶體最佳化（Downcasting）、字串轉類別型態、浮點數轉整數 |
| **[[#pd.to_datetime() 強大時間戳記解析器\|pd.to_datetime()]]** | **將多樣化格式的字串、整數 Unix 時間戳轉換為標準 DatetimeIndex / Timestamp** | 清洗雜亂日期字串、解析 ISO 8601 時間格式、建立時間序列分析基準 |
| **[[#pd.to_numeric() 強制數值型別轉換\|pd.to_numeric()]]** | **將夾雜無效符號或文字的數列安全轉換為整數或浮點數** | 清洗髒數據（如夾雜 '-' 或 'N/A' 的價格欄位）、異常值無害化轉換 |
| **[[#DataFrame.drop() 刪除指定欄位或索引列\|df.drop()]]** | **依據欄位名稱或列索引標籤自資料表中移除目標軸向之元素** | 剔除不必要特徵欄位、刪除特定異常樣本列 |
| **[[#DataFrame.rename() 重新命名欄位或索引標籤\|df.rename()]]** | **透過字典映射或函式規則對欄位名稱或列索引名稱進行替換** | 將中文欄名轉為英文程式碼標準名、統一第三方欄位命名風格 |
| **[[#DataFrame.drop_duplicates() 剔除重複列\|df.drop_duplicates()]]** | **根據全表或指定子欄位識別重複出現的資料列並進行剔除** | 消除爬蟲重複抓取之網頁資料、防止交易記錄重複計入 |
| **[[#DataFrame.sort_values() 依數值排序資料列\|df.sort_values()]]** | **根據一或多個指定欄位的數值大小對資料列進行升冪或降冪重新排序** | 排行榜製作、時間排序對齊、探索最大/最小值記錄 |
| **[[#DataFrame.set_index() 與 DataFrame.reset_index() 索引結構重構\|df.set_index() / df.reset_index()]]** | **將現有欄位提升為索引，或將現有索引降級還原為普通欄位並重建預設整數索引** | 時間序列前置設定、groupby 彙總後還原扁平資料表 |
| **[[#DataFrame.melt() 寬表格逆樞紐轉為長表格\|df.melt()]]** | **將資料表的多個量測欄位旋轉壓縮為 (變數, 數值) 兩欄長表格形式（Unpivot）** | 整理時間橫跨多欄之寬表、為 Seaborn/Plotly 繪圖做資料長型重塑 |
| **[[#DataFrame.pivot() 與 DataFrame.pivot_table() 樞紐分析表轉換\|df.pivot() / df.pivot_table()]]** | **長格式轉寬格式，並可自訂聚合運算函式（如求和、均值）生成多維交叉分析表** | 財務多維度交叉報表、維度展開矩陣、探索不同類別交叉統計量 |
| **[[#DataFrame.groupby() 拆分-應用-合併分組引擎\|df.groupby()]]** | **根據指定類別欄位將資料拆分為多個子組群，以便接續進行聚合或轉換操作** | 分部門計算平均薪資、各商品品類銷售額加總、分組特徵工程 |
| **[[#GroupBy.agg() 多元分組聚合運算\|groupby.agg()]]** | **對分組後的資料表同時套用單一或多個自訂聚合函式** | 各欄計算不同統計量（如薪資算平均、年齡取最大值）、客製化複合指標計算 |
| **[[#GroupBy.transform() 分組廣播轉換保持原形狀\|groupby.transform()]]** | **在分組內部進行運算後，將結果廣播回傳為與原始資料表同長度之數列** | 計算組內標準化 Z-Score、組內佔比、計算每筆資料與組內均值之差距 |
| **[[#DataFrame.apply() 逐欄或逐列自訂函式映射\|df.apply()]]** | **對資料表的每一欄（axis=0）或每一列（axis=1）呼叫自訂 Python 函式** | 跨欄複合運算、自訂演算法套用、非向量化邏輯批次處理 |
| **[[#DataFrame.map() 全表元素逐格對映轉換\|df.map() / s.map()]]** | **對資料表內的每一個儲存格元素進行一對一函式映射或字典替換** | 全表數值四捨五入、單位符號格式化、類別代碼字典替換 |
| **[[#pd.concat() 軸向拼接合併多個資料表\|pd.concat()]]** | **沿著垂直方向（軸 0，增長列）或水平方向（軸 1，增長欄）直接拼接多個表格** | 批次合併每月銷售報表、橫向拼接特徵矩陣與標籤數列 |
| **[[#pd.merge() 與 DataFrame.merge() 關聯式資料庫鍵值連接\|pd.merge() / df.merge()]]** | **基於一或多個共同鍵值欄位執行資料庫風格的 JOIN（Inner/Left/Right/Outer）** | 多資料表主外鍵關聯、訂單資料與客戶資料依 ID 串聯 |
| **[[#DataFrame.join() 基於索引快速水平拼接\|df.join()]]** | **主要基於列索引（Index）進行高效的左右資料表水平連接** | 兩個具有相同 Index 標籤之資料表依索引合併 |
| **[[#Series.str 向量化字串存取器\|s.str]]** | **提供大量針對字串欄位的元素級向量化文字處理方法（如 strip, split, contains, replace）** | 清洗髒字串、提取 Email 網域名稱、電話號碼格式正規化、字串比對過濾 |
| **[[#Series.dt 向量化時間存取器\|s.dt]]** | **提取日期時間型別（datetime64）的年月日、時分秒、星期與時區等屬性** | 按月份分組統計趨勢、特徵工程提取是否為週末 (is_weekend)、時間維度衍生 |
| **[[#DataFrame.resample() 時間序列頻率重取樣\|df.resample()]]** | **針對具備 DatetimeIndex 的資料表進行時間維度頻率轉換（降取樣或升取樣）與彙總** | 將每分鐘交易降取樣為日 K 線、將每日流量聚合成每月報表 |
| **[[#DataFrame.sum() 與 DataFrame.mean() 加總與算術平均值\|df.sum() / df.mean()]]** | **沿著指定軸向計算數值欄位的元素總和或算術平均值** | 全表指標統計、直向欄位加總、橫向總分平均計算 |
| **[[#DataFrame.max() 與 DataFrame.min() 極值統計\|df.max() / df.min()]]** | **沿著指定軸向尋找數值或字串欄位的最大值與最小值** | 尋找價格最高最低點、探索數值範圍區間、異常峰值監控 |
| **[[#DataFrame.idxmax() 與 DataFrame.idxmin() 極值所在索引定位\|df.idxmax() / df.idxmin()]]** | **尋找最大值或最小值首次出現時所對應的行標籤（Index）或列標籤** | 快速找出銷售冠軍的員工姓名、尋找歷史最低價發生的日期 |
| **[[#DataFrame.std() 與 DataFrame.var() 標準差與變異數\|df.std() / df.var()]]** | **計算樣本標準差（Sample Standard Deviation）與樣本變異數（Sample Variance）** | 評估股價波動度、品質管制離散度分析、常態分佈特徵檢驗 |
| **[[#DataFrame.cumsum() 與 DataFrame.cumprod() 累加與累乘計算\|df.cumsum() / df.cumprod()]]** | **沿著時間或樣本方向持續累加（累積加總）或累乘數值** | 計算年至今累計營收（YTD）、複利資產淨值曲線計算、累計用戶數 |
| **[[#DataFrame.rolling() 移動視窗滑動計算\|df.rolling()]]** | **建立固定長度的滑動視窗（Moving Window），進行平滑化或滾動指標統計** | 計算 5 日移動平均線 (5MA)、滾動波動率、異常值平滑濾波 |
| **[[#Series.shift() 與 Series.diff() 時間落後與差分計算\|s.shift() / s.diff()]]** | **將數列沿著索引向前或向後推移 N 步，或直接計算相鄰元素間的數值差分** | 計算日漲跌幅、前後期數值落差比對、特徵工程構造 Lag Features |
| **[[#Series.pct_change() 百分比增長率計算\|s.pct_change()]]** | **計算當前元素與前一期元素之間的百分比變動率（Percentage Change）** | 計算股票日收益率、營收月增率 (MoM)、年增率 (YoY) |
| **[[#DataFrame.copy() 顯式深拷貝避免鏈式賦值警告\|df.copy()]]** | **建立資料表及其底層數據與索引的完整獨立拷貝（Deep Copy）** | 防禦 SettingWithCopyWarning、切片後獨立修改不污染原始資料表 |
| **[[#pd.set_option() 與 pd.get_option() 全域顯示與行為配置\|pd.set_option() / pd.get_option()]]** | **設定或查詢 Pandas 終端列印列數、欄數、字串長度與浮點數精確度等全域行為** | 防止大表格被省略號截斷 (display.max_columns)、控制浮點數格式化輸出 |

---

# 資料結構核心與工廠方法 (Core Structures & Factory)

建立一維 Series、二維 DataFrame 與連續時間序列的核心入口。

---

##### pd.Series() 建立一維標籤化陣列

- **使用時機**：需要將 Python 串列、字典或 NumPy 一維陣列包裝為具備標籤索引的 Series 結構時使用。
- **語法**：`pd.Series(data=None, index=None, dtype=None, name=None, copy=None)`
- **參數說明**：
  - `data`：輸入數據（可為 array-like、Iterable、dict 或純量值）。
  - `index`：自訂軸標籤索引清單。若未傳入，預設為 `RangeIndex(0, 1, 2, ...)`。
  - `dtype`：強制指定資料型別（如 `float64`、`string` 等）。
  - `name`：給予該數列的名稱字串，轉成 DataFrame 時會直接作為欄位名稱。
- **回傳值**：
  - `Series`：一維標籤化數列實例。

```python
import numpy as np
import pandas as pd

# 1. 透過字典建立（Key 自動轉為 Index 標籤）
s_dict = pd.Series({"apple": 30, "banana": 20, "cherry": 50}, name="水果價格")

# 2. 透過串列搭配自訂索引
s_list = pd.Series([10, 20, 30], index=["a", "b", "c"])
print(s_list["b"])  # 輸出: 20
```

---

##### pd.DataFrame() 建立二維表格資料結構

- **使用時機**：將巢狀字典、二維陣列、記錄串列或 Series 組合打包為標準二維資料表時使用。
- **語法**：`pd.DataFrame(data=None, index=None, columns=None, dtype=None, copy=None)`
- **參數說明**：
  - `data`：輸入資料（dict of ndarrays/lists、二維 ndarray、Series 串列等）。
  - `index`：自訂列（Row）索引標籤。
  - `columns`：自訂欄（Column）欄位名稱清單。
  - `dtype`：強制套用的全表或單一欄位資料型別。
- **回傳值**：
  - `DataFrame`：二維結構化資料表格。

```python
import pandas as pd

# 1. 以「字典套串列」建立（每個 Key 代表一欄 Column）
data = {
    "姓名": ["Alice", "Bob", "Charlie"],
    "年齡": [25, 30, 35],
    "城市": ["台北", "台中", "高雄"]
}
df = pd.DataFrame(data)
print(df.shape)  # 輸出: (3, 3)
```

**`df` 表格結構**：

| Index | 姓名 | 年齡 | 城市 |
| :---: | :--- | :---: | :--- |
| 0 | Alice | 25 | 台北 |
| 1 | Bob | 30 | 台中 |
| 2 | Charlie | 35 | 高雄 |

---

##### pd.date_range() 產生連續時間戳記索引數列

- **使用時機**：需要為時間序列分析建立一段起訖分明、頻率固定的時間索引時使用。
- **語法**：`pd.date_range(start=None, end=None, periods=None, freq=None, tz=None, normalize=False, name=None, inclusive="both")`
- **參數說明**：
  - `start`：時間起始點（字串或 datetime 物件）。
  - `end`：時間結束點。
  - `periods`：欲產生的時間區段總筆數（與 start/end 擇二組合即可推算）。
  - `freq`：時間間隔頻率（如 `'D'` 為日、`'B'` 為工作日、`'h'` 為小時、`'ME'` 為月末）。
- **回傳值**：
  - `DatetimeIndex`：時間戳記索引物件。

```python
import pandas as pd

# 建立 2026 年 10 月前 5 天的每日時間索引
dates = pd.date_range(start="2026-10-01", periods=5, freq="D")
ts_df = pd.DataFrame({"流量": [100, 120, 150, 130, 180]}, index=dates)
print(ts_df.index.day_name())  # 輸出星期名稱清單
```

**`ts_df` 表格結構**：

| Index | 流量 |
| :---: | :---: |
| 2026-10-01 | 100 |
| 2026-10-02 | 120 |
| 2026-10-03 | 150 |
| 2026-10-04 | 130 |
| 2026-10-05 | 180 |

---

---

# 外部資料讀寫與 I/O 操作 (Input / Output)

涵蓋 CSV、Excel、JSON 與 SQL 資料庫之雙向資料傳輸標準函式。

---

##### pd.read_csv() 讀取 CSV 與分隔文字檔

- **使用時機**：從硬碟或網路 URL 讀取本機 CSV、TSV 或文字檔案為 DataFrame 時使用。
- **語法**：`pd.read_csv(filepath_or_buffer, sep=",", header="infer", names=None, index_col=None, usecols=None, dtype=None, parse_dates=None, na_values=None, chunksize=None, encoding=None)`
- **參數說明**：
  - `filepath_or_buffer`：檔案路徑、URL 或類檔案物件。
  - `sep`：欄位分隔符號，預設為逗號 `','`（TSV 檔設為 `'	'`）。
  - `usecols`：只載入特定欄位清單（極致節省記憶體關鍵參數）。
  - `parse_dates`：指定需要自動解析為 datetime 型別的欄位名稱串列。
  - `chunksize`：批次載入的區塊列數（回傳 TextFileReader 迭代器，處理超大型檔案必備）。
- **回傳值**：
  - `DataFrame` 或 `TextFileReader`（設定 chunksize 時）。

```python
import pandas as pd

# 僅讀取指定欄位並自動解析時間欄位
df = pd.read_csv("orders.csv", usecols=["order_id", "date", "amount"], parse_dates=["date"])
```

---

##### DataFrame.to_csv() 匯出資料表為 CSV 檔案

- **使用時機**：完成資料清洗與運算後，需要將成果持久化保存為 CSV 格式時使用。
- **語法**：`df.to_csv(path_or_buf=None, sep=",", index=True, columns=None, header=True, encoding=None, mode="w")`
- **參數說明**：
  - `path_or_buf`：寫入的檔案路徑字串或檔案串流（若傳入 `None` 則直接回傳 CSV 字串）。
  - `index`：是否將列索引寫入第一欄（預設 `True`；多數情況建議顯式設為 `index=False` 以免多出無意義的數字欄）。
  - `encoding`：編碼格式（含有繁體中文在 Windows Excel 開啟時常設為 `'utf-8-sig'`）。
- **回傳值**：
  - `None` 或 `str`（若未提供路徑）。

```python
import pandas as pd

df = pd.DataFrame({"姓名": ["Alice", "Bob"], "成績": [90, 85]})
# 匯出時排除預設數字索引，避免生成 Unnamed: 0 欄位
df.to_csv("students.csv", index=False, encoding="utf-8-sig")
```

**`df` 表格結構**：

| Index | 姓名 | 成績 |
| :---: | :--- | :---: |
| 0 | Alice | 90 |
| 1 | Bob | 85 |

> **[常見踩坑：為何要加 `index=False`？避免 `Unnamed: 0` 幽靈欄位]**：  
> - **問題成因**：DataFrame 預設帶有從 0 起算的流水號索引（Index）。若以預設 `index=True` 匯出，最左側的索引數字會被寫入 CSV，且該欄位在首行沒有標頭名稱（CSV 首行呈現 `,姓名,成績`）。  
> - **踩坑現象**：下次使用 `pd.read_csv("students.csv")` 重新載入時，Pandas 因第一欄缺少欄位名稱，會自動將其命名為 `Unnamed: 0`。若反覆存取，還會滾雪球產生 `Unnamed: 0.1` 等無用重複欄位。  
> - **最佳實踐**：  
>   - **匯出時預防**：若索引無特殊業務意義，一律顯式加上 `index=False`。  
>   - **讀取時補救**：若拿到已包含索引欄位的 CSV，可用 `pd.read_csv("students.csv", index_col=0)` 將第一欄還原為索引，或用 `df.drop(columns=["Unnamed: 0"])` 予以剔除。

---

##### pd.read_excel() 與 DataFrame.to_excel() 讀寫 Excel 工作表

- **使用時機**：需要與商務分析師、財務人員的 Excel 活頁簿進行雙向資料交換時使用。
- **語法**：
  - 讀取：`pd.read_excel(io, sheet_name=0, header=0, names=None, index_col=None, usecols=None)`
  - 匯出：`df.to_excel(excel_writer, sheet_name="Sheet1", index=True)`
- **參數說明**：
  - `io` / `excel_writer`：檔案路徑或 `ExcelWriter` 物件。
  - `sheet_name`：工作表名稱字串或整數索引（設為 `None` 可一次性讀入所有 Sheet 字典）。
- **回傳值**：
  - 讀取回傳 `DataFrame` 或 `dict[str, DataFrame]`；匯出回傳 `None`。

```python
import pandas as pd

# 讀取名為 '2026_Q3' 的工作表
df = pd.read_excel("finance.xlsx", sheet_name="2026_Q3")
# 匯出為新 Excel 工作表
df.to_excel("report.xlsx", sheet_name="清洗結果", index=False)
```

---

##### pd.read_json() 與 DataFrame.to_json() 讀寫 JSON 結構化檔案

- **使用時機**：處理 Web API 回傳的 JSON 陣列或微服務之間的資料傳遞時使用。
- **語法**：
  - 讀取：`pd.read_json(path_or_buf, orient=None, typ="frame", dtype=None)`
  - 匯出：`df.to_json(path_or_buf=None, orient=None, force_ascii=True)`
- **參數說明**：
  - `orient`：資料組織架構（常見有 `'records'` 字典串列、`'split'`、`'index'`、`'columns'`）。
- **回傳值**：
  - 讀取回傳 `DataFrame`；匯出回傳 `None` 或 `str`。

```python
import pandas as pd

json_str = '[{"id": 1, "name": "Apple"}, {"id": 2, "name": "Banana"}]'
df = pd.read_json(json_str, orient="records")
print(df.iloc[0]["name"])  # 輸出: Apple
```

---

##### pd.read_sql() 與 DataFrame.to_sql() 關聯式資料庫讀寫

- **使用時機**：需要直接對關聯式資料庫執行 SQL 查詢並載入記憶體，或將運算完的資料表寫回資料庫表格時使用。
- **語法**：
  - 讀取：`pd.read_sql(sql, con, index_col=None, parse_dates=None, columns=None, chunksize=None)`
  - 寫入：`df.to_sql(name, con, schema=None, if_exists="fail", index=True, dtype=None)`
- **參數說明**：
  - `sql`：SQL 查詢字串或 SQLAlchemy Selectable。
  - `con`：SQLAlchemy Engine 資料庫連線引擎或連線物件。
  - `if_exists`：寫入時若資料庫資料表已存在的處理策略（`'fail'` 拋錯、`'replace'` 刪除重建、`'append'` 追加寫入）。
- **回傳值**：
  - 讀取回傳 `DataFrame`；寫入回傳 `int`（受影響列數）或 `None`。

```python
import pandas as pd
from sqlalchemy import create_engine

# 建立 SQLite 記憶體連線引擎
engine = create_engine("sqlite:///:memory:")
df = pd.DataFrame({"id": [1, 2], "score": [95, 88]})

# 將 DataFrame 寫入資料庫表 'grades'
df.to_sql("grades", con=engine, if_exists="replace", index=False)

# 透過 SQL 查詢讀回
df_read = pd.read_sql("SELECT * FROM grades WHERE score >= 90", con=engine)
print(len(df_read))  # 輸出: 1
```

**`df` 表格結構**：

| Index | id | score |
| :---: | :---: | :---: |
| 0 | 1 | 95 |
| 1 | 2 | 88 |

---

---

# 表格結構元數據與基礎屬性 (Metadata & Properties)

檢視與反射 DataFrame 的形狀維度、型別、標籤與底層資料矩陣。

---

##### DataFrame.shape 表格形狀維度元組

- **使用時機**：資料載入或經過篩選後，第一時間確認資料表的總列數與總欄數。
- **語法**：`df.shape`（唯讀屬性）
- **屬性型別**：`tuple[int, int]`，格式為 `(num_rows, num_columns)`。

```python
import pandas as pd

df = pd.DataFrame({"A": [1, 2, 3], "B": [4, 5, 6]})
rows, cols = df.shape
print(f"總列數: {rows}, 總欄數: {cols}")  # 輸出: 總列數: 3, 總欄數: 2
```

**`df` 表格結構**：

| Index | A | B |
| :---: | :---: | :---: |
| 0 | 1 | 4 |
| 1 | 2 | 5 |
| 2 | 3 | 6 |

---

##### DataFrame.dtypes 欄位資料型別數列

- **使用時機**：排查欄位型別是否符合預期（例如日期是否被當成字串、數字欄位是否夾雜符號變成 object）。
- **語法**：`df.dtypes`（唯讀屬性）
- **屬性型別**：`Series`（Index 為欄位名，Value 為 dtype 物件）。

```python
import pandas as pd

df = pd.DataFrame({"年齡": [25, 30], "薪資": [50000.5, 65000.0], "姓名": ["Alice", "Bob"]})
print(df.dtypes)
# 年齡       int64
# 薪資     float64
# 姓名      object
```

**`df` 表格結構**：

| Index | 年齡 | 薪資 | 姓名 |
| :---: | :---: | :---: | :--- |
| 0 | 25 | 50000.5 | Alice |
| 1 | 30 | 65000.0 | Bob |

---

##### DataFrame.columns 與 DataFrame.index 欄列標籤索引物件

- **使用時機**：需要遍歷欄位名稱、檢查特定欄位是否存在，或批次為全表欄名去除空白與轉為小寫時使用。
- **語法**：
  - 欄位索引：`df.columns`
  - 列索引：`df.index`
- **屬性型別**：`Index` 物件實例。

```python
import pandas as pd

df = pd.DataFrame({" User ": [1, 2], " Age ": [20, 25]})

# 1. 批次清洗欄位名稱：去除首尾空格並轉為小寫
df.columns = df.columns.str.strip().str.lower()
print(df.columns.tolist())  # 輸出: ['user', 'age']

# 2. 判斷特定欄位是否存在
if "user" in df.columns:
    print("欄位存在！")
```

**`df` 表格結構**：

| Index | ` User ` | ` Age ` |
| :---: | :---: | :---: |
| 0 | 1 | 20 |
| 1 | 2 | 25 |

---

##### DataFrame.values 與 DataFrame.to_numpy() 轉為底層 NumPy 陣列

- **使用時機**：將清洗好的特徵資料表送入機器學習模型（如 scikit-learn、PyTorch）進行訓練時使用。
- **語法**：`df.to_numpy(dtype=None, copy=False)`（現代推薦）或 `df.values`（舊式屬性）
- **回傳值**：
  - `numpy.ndarray`：二維陣列矩陣。

```python
import pandas as pd

df = pd.DataFrame({"x1": [1.0, 2.0], "x2": [3.0, 4.0]})
# 現代標準推薦寫法: to_numpy()
X = df.to_numpy()
print(type(X))  # 輸出: <class 'numpy.ndarray'>
```

**`df` 表格結構**：

| Index | x1 | x2 |
| :---: | :---: | :---: |
| 0 | 1.0 | 3.0 |
| 1 | 2.0 | 4.0 |

---

##### DataFrame.size 與 DataFrame.ndim 元素總數與維度數

- **使用時機**：需要快速計算全表包含多少個資料格子（列數乘以欄數），或確認維度是否為 2 時使用。
- **語法**：`df.size` 與 `df.ndim`
- **屬性型別**：`int` 整數（DataFrame 的 ndim 固定為 2，Series 固定為 1）。

```python
import pandas as pd

df = pd.DataFrame({"A": [1, 2], "B": [3, 4], "C": [5, 6]})
print(df.size)  # 輸出: 6 (2 列 * 3 欄)
print(df.ndim)  # 輸出: 2
```

**`df` 表格結構**：

| Index | A | B | C |
| :---: | :---: | :---: | :---: |
| 0 | 1 | 3 | 5 |
| 1 | 2 | 4 | 6 |

---

---

# 探查、檢視與統計摘要 (Inspection & Summary)

初次載入資料集的結構診斷、記憶體用量、數值分佈與頻率分析。

---

##### DataFrame.head() 與 DataFrame.tail() 預覽首尾資料列

- **使用時機**：資料載入或轉換後，快速抽樣確認前幾列或最後幾列的數值與結構是否正確。
- **語法**：`df.head(n=5)` 與 `df.tail(n=5)`
- **參數說明**：
  - `n`：欲檢視的列數（整數，預設為 5）。
- **回傳值**：
  - `DataFrame`：子資料表。

```python
import pandas as pd

df = pd.DataFrame({"ID": range(100)})
print(df.head(3))  # 檢視前 3 筆 (0, 1, 2)
print(df.tail(2))  # 檢視後 2 筆 (98, 99)
```

**`df` 表格結構**：

| Index | ID |
| :---: | :---: |
| 0 | 0 |
| 1 | 1 |
| 2 | 2 |
| ... | ... |
| 98 | 98 |
| 99 | 99 |

---

##### DataFrame.info() 列印資料表記憶體與欄位摘要

- **使用時機**：拿到陌生資料集時必執行的診斷工具，一眼看清所有欄位的非空值數量與佔用記憶體。
- **語法**：`df.info(verbose=None, memory_usage=None, show_counts=None)`
- **回傳值**：
  - `None`（直接印出文字摘要至 stdout）。

```python
import pandas as pd

df = pd.DataFrame({"姓名": ["Alice", None], "年齡": [25, 30]})
df.info()
# 輸出: 包含 RangeIndex 2 entries, 2 columns, Dtype 統計, memory usage 等
```

**`df` 表格結構**：

| Index | 姓名 | 年齡 |
| :---: | :--- | :---: |
| 0 | Alice | 25 |
| 1 | `NaN` | 30 |

---

##### DataFrame.describe() 生成數值與類別統計摘要

- **使用時機**：進行探索性資料分析（EDA），評估數值特徵的集中趨勢與離散程度時使用。
- **語法**：`df.describe(percentiles=None, include=None, exclude=None)`
- **參數說明**：
  - `percentiles`：自訂欲檢視的百分位數串列（預設為 `[0.25, 0.5, 0.75]`）。
  - `include`：包含的資料型別（例如傳入 `'all'` 則字串類別欄位也會統計頻率與唯一值；或傳入 `'number'`、`'object'`）。
- **回傳值**：
  - `DataFrame`：統計結果矩陣。

```python
import pandas as pd

df = pd.DataFrame({"成績": [60, 70, 80, 90, 100], "班級": ["A", "A", "B", "B", "B"]})
print(df.describe())  # 預設僅針對數值型欄位計算 count, mean, std, min, 25%, 50%, 75%, max
print(df.describe(include="all"))  # 針對字串欄位補充 unique, top, freq
```

**`df` 表格結構**：

| Index | 成績 | 班級 |
| :---: | :---: | :--- |
| 0 | 60 | A |
| 1 | 70 | A |
| 2 | 80 | B |
| 3 | 90 | B |
| 4 | 100 | B |

---

##### Series.value_counts() 類別值頻率計數

- **使用時機**：統計類別欄位（Categorical Feature）中每一種數值各自出現了幾次。
- **語法**：`s.value_counts(normalize=False, sort=True, ascending=False, bins=None, dropna=True)`
- **參數說明**：
  - `normalize`：若設為 `True`，回傳比例（佔比，總和為 1.0）而非絕對次數。
  - `dropna`：是否忽略缺失值 NaN（預設 `True`；若需檢查缺失佔比應設為 `False`）。
- **回傳值**：
  - `Series`：Index 為相異值，Value 為出現次數或比例。

```python
import pandas as pd

s = pd.Series(["台北", "台中", "台北", "高雄", "台北"])
print(s.value_counts())
# 台北    3
# 台中    1
# 高雄    1

# 計算百分比比例
print(s.value_counts(normalize=True))
# 台北    0.6
```

---

##### Series.unique() 與 Series.nunique() 相異值清單與計數

- **使用時機**：想要列出一個欄位究竟有哪些相異選項，或者統計相異種類總共有幾種時使用。
- **語法**：
  - 清單：`s.unique()`
  - 計數：`s.nunique(dropna=True)`（DataFrame 亦可整表呼叫 `df.nunique()`）
- **回傳值**：
  - `unique()` 回傳 `numpy.ndarray`；`nunique()` 回傳 `int`（或 DataFrame 的 Series）。

```python
import pandas as pd

s = pd.Series(["A", "B", "A", "C", None])
print(s.unique())   # 輸出: array(['A', 'B', 'C', None], dtype=object)
print(s.nunique())  # 輸出: 3 (預設排除空值)
```

---

---

# 索引選取、切片與條件篩選 (Indexing & Selection)

標籤切片、整數位置選取、超高效純量存取與複雜 SQL 風格布林表達式查詢。

---

##### DataFrame.loc[] 基於標籤的切片與選取

- **使用時機**：已知明確的列標籤（Index 名稱）與欄位名稱時，進行精準資料提取或條件寫入。
- **語法**：`df.loc[row_indexer, col_indexer]`
- **核心特點**：
  - **包頭也包尾（Inclusive）**：與 Python 傳統切片不同，`df.loc['a':'c']` 會完整包含 `'c'`！
  - 支援布林遮罩（Boolean Mask）篩選。
- **回傳值**：
  - 純量值、`Series` 或 `DataFrame`（依傳入維度而定）。

```python
import pandas as pd

df = pd.DataFrame({"成績": [80, 90, 85]}, index=["Alice", "Bob", "Charlie"])
# 1. 單一標籤提取
print(df.loc["Bob", "成績"])  # 輸出: 90

# 2. 標籤切片 (包含 Charlie！)
print(df.loc["Alice":"Charlie"])
```

**`df` 表格結構**：

| Index | 成績 |
| :---: | :---: |
| Alice | 80 |
| Bob | 90 |
| Charlie | 85 |

---

##### DataFrame.iloc[] 基於整數位置的切片與選取

- **使用時機**：不關心標籤叫什麼名字，只根據純粹的物理位置（第幾列、第幾欄）提取資料時使用。
- **語法**：`df.iloc[row_position, col_position]`
- **核心特點**：
  - **包頭不包尾（Exclusive）**：嚴格遵循 Python 標準切片規則，`df.iloc[0:2]` 僅包含索引 0 與 1。
- **回傳值**：
  - 純量值、`Series` 或 `DataFrame`。

```python
import pandas as pd

df = pd.DataFrame({"A": [10, 20, 30], "B": [40, 50, 60], "C": [70, 80, 90]})
# 提取前兩列、前兩欄（特徵矩陣與標籤分離常用）
sub_df = df.iloc[0:2, 0:2]
print(sub_df.shape)  # 輸出: (2, 2)
```

**`df` 表格結構**：

| Index | A | B | C |
| :---: | :---: | :---: | :---: |
| 0 | 10 | 40 | 70 |
| 1 | 20 | 50 | 80 |
| 2 | 30 | 60 | 90 |

---

##### DataFrame.at[] 與 DataFrame.iat[] 超高效純量存取

- **使用時機**：只需要精確讀取或覆寫單一個格子的數值時使用。底層避開了建立 Series 的開銷，速度比 `loc` / `iloc` 快上數倍至數十倍。
- **語法**：
  - 依標籤：`df.at[row_label, col_label]`
  - 依位置：`df.iat[row_pos, col_pos]`
- **回傳值**：
  - 單一純量值（int, float, str 等）。

```python
import pandas as pd

df = pd.DataFrame({"A": [1, 2], "B": [3, 4]}, index=["r1", "r2"])
# 快速覆寫單一數值
df.at["r1", "B"] = 99
print(df.iat[0, 1])  # 輸出: 99
```

**`df` 表格結構**：

| Index | A | B |
| :---: | :---: | :---: |
| r1 | 1 | 3 |
| r2 | 2 | 4 |

---

##### DataFrame.query() 基於布林表達式字串快速篩選

- **使用時機**：當過濾條件繁多時，使用傳統中括號 `df[(df['a']>1) & (df['b']<2)]` 容易夾雜大量括號與重複變數名，`query()` 能以純文字表達式大幅提高可讀性。
- **語法**：`df.query(expr, inplace=False)`
- **核心特色**：
  - 支援使用 `@變數名` 直接引用當前 Python 區域變數！
- **回傳值**：
  - `DataFrame`：篩選後的資料表。

```python
import pandas as pd

df = pd.DataFrame({"年齡": [20, 30, 40], "薪資": [30000, 60000, 80000]})
min_salary = 50000

# 使用 @ 符號引入外部區域變數，並以 and / or 串接
result = df.query("年齡 >= 30 and 薪資 >= @min_salary")
print(len(result))  # 輸出: 2
```

**`df` 表格結構**：

| Index | 年齡 | 薪資 |
| :---: | :---: | :---: |
| 0 | 20 | 30000 |
| 1 | 30 | 60000 |
| 2 | 40 | 80000 |

---

##### Series.isin() 集合成員資格條件遮罩

- **使用時機**：需要判斷某一欄的數值是否存在於給定的多個候選選項中（類似 SQL 中的 `IN` 語法）。
- **語法**：`s.isin(values)` 或 `df.isin(values)`
- **參數說明**：
  - `values`：集合、串列、Series 或字典。
- **回傳值**：
  - 同維度的布林遮罩（Boolean Series/DataFrame）。

```python
import pandas as pd

df = pd.DataFrame({"城市": ["台北", "新竹", "台中", "台南"]})
target_cities = ["台北", "台中"]

# 挑出目標城市清單內的資料列
filtered_df = df[df["城市"].isin(target_cities)]
print(filtered_df["城市"].tolist())  # 輸出: ['台北', '台中']
```

**`df` 表格結構**：

| Index | 城市 |
| :---: | :--- |
| 0 | 台北 |
| 1 | 新竹 |
| 2 | 台中 |
| 3 | 台南 |

---

##### DataFrame.filter() 依標籤名稱篩選欄或列

- **使用時機**：並非根據內容值過濾，而是根據「欄名」或「列名」本身的名字規則進行挑選時使用。
- **語法**：`df.filter(items=None, like=None, regex=None, axis=None)`
- **參數說明**：
  - `items`：明確要保留的標籤清單。
  - `like`：模糊包含特定子字串的標籤（如 `like="cost"`）。
  - `regex`：正規表達式比對（如 `regex="^score_"`）。
  - `axis`：搜尋維度（預設 `axis=1` 搜尋欄名，`axis=0` 搜尋列名）。
- **回傳值**：
  - `DataFrame`。

```python
import pandas as pd

df = pd.DataFrame({"user_id": [1], "test_score_1": [80], "test_score_2": [90], "final_eval": [85]})
# 只挑出以 'test_' 開頭的欄位
test_cols = df.filter(regex="^test_")
print(test_cols.columns.tolist())  # 輸出: ['test_score_1', 'test_score_2']
```

**`df` 表格結構**：

| Index | user_id | test_score_1 | test_score_2 | final_eval |
| :---: | :---: | :---: | :---: | :---: |
| 0 | 1 | 80 | 90 | 85 |

---

---

# 缺失值處理與型別轉換 (Missing Data & Casting)

缺失值盤點、剔除、填補、向前沿用與字串/時間/數值型態強制轉換安全函式。

---

##### DataFrame.isna() 與 DataFrame.notna() 缺失值偵測布林遮罩

- **使用時機**：全表盤點哪裡存在空值，常搭配 `.sum()` 計算各欄缺失數量。
- **語法**：`df.isna()`（等價於 `df.isnull()`）與 `df.notna()`（等價於 `df.notnull()`）
- **回傳值**：
  - 同形狀的布林值 DataFrame。

```python
import numpy as np
import pandas as pd

df = pd.DataFrame({"A": [1, np.nan, 3], "B": [np.nan, np.nan, "ok"]})
# 統計各欄位的空值總數
print(df.isna().sum())
# A    1
# B    2
```

**`df` 表格結構**：

| Index | A | B |
| :---: | :---: | :--- |
| 0 | 1.0 | `NaN` |
| 1 | `NaN` | `NaN` |
| 2 | 3.0 | ok |

---

##### DataFrame.dropna() 剔除包含空值的列或欄

- **使用時機**：當樣本數充足或缺失值無法合理填補時，直接丟棄帶有空值的列或欄。
- **語法**：`df.dropna(axis=0, how="any", thresh=None, subset=None, inplace=False, ignore_index=False)`
- **參數說明**：
  - `axis`：`0` 或 `'index'` 刪除列；`1` 或 `'columns'` 刪除欄。
  - `how`：`'any'`（只要有一個空值就刪除）或 `'all'`（整列/整欄全為空值才刪除）。
  - `thresh`：整數，至少要保留 N 個「非空值」才不被刪除。
  - `subset`：指定僅檢查特定關鍵欄位（如 `subset=['user_id', 'email']`）。
- **回傳值**：
  - 剔除後的 `DataFrame`。

```python
import numpy as np
import pandas as pd

df = pd.DataFrame({"姓名": ["Alice", "Bob", "Charlie"], "電話": ["123", np.nan, "789"]})
# 僅針對 '電話' 欄位檢查，若電話為空則刪除該列
clean_df = df.dropna(subset=["電話"])
print(len(clean_df))  # 輸出: 2
```

**`df` 表格結構**：

| Index | 姓名 | 電話 |
| :---: | :--- | :--- |
| 0 | Alice | 123 |
| 1 | Bob | `NaN` |
| 2 | Charlie | 789 |

---

##### DataFrame.fillna() 填補缺失值

- **使用時機**：保留資料列，並使用合理的估計值（如 0、平均值、前筆資料）填補空缺。
- **語法**：`df.fillna(value=None, method=None, axis=None, inplace=False, limit=None)`
- **參數說明**：
  - `value`：填補數值、字典（各欄指定不同填補值）或 Series。
  - `method`：（舊版參數，現代 Pandas 推薦改用專屬的 `ffill()` 或 `bfill()`）。
- **回傳值**：
  - 填補後的 `DataFrame`。

```python
import numpy as np
import pandas as pd

df = pd.DataFrame({"成績": [80, np.nan, 90], "狀態": ["已繳費", np.nan, "已繳費"]})
# 1. 字典分欄精細填補：數值欄填平均，類別欄填預設字串
mean_score = df["成績"].mean()
df_filled = df.fillna({"成績": mean_score, "狀態": "待確認"})
print(df_filled.loc[1, "成績"])  # 輸出: 85.0
```

**`df` 表格結構**：

| Index | 成績 | 狀態 |
| :---: | :---: | :--- |
| 0 | 80.0 | 已繳費 |
| 1 | `NaN` | `NaN` |
| 2 | 90.0 | 已繳費 |

---

##### DataFrame.ffill() 與 DataFrame.bfill() 向前向後沿用填補

- **使用時機**：處理時間序列或連續事件數據時，若某時刻缺值，通常沿用上一時刻的狀態（Forward Fill）。
- **語法**：`df.ffill(axis=None, inplace=False, limit=None)` 與 `df.bfill(...)`
- **參數說明**：
  - `limit`：連續最多向前/向後填補的次數上限。
- **回傳值**：
  - `DataFrame`。

```python
import numpy as np
import pandas as pd

s = pd.Series([100, np.nan, np.nan, 105])
# 向前沿用上一筆非空值
print(s.ffill().tolist())  # 輸出: [100.0, 100.0, 100.0, 105.0]
```

---

##### DataFrame.astype() 明確轉換欄位資料型別

- **使用時機**：將記憶體龐大的 `float64` 降級為 `float32`，或將重現率極高的文字轉為 `category` 以換取數倍效能提升。
- **語法**：`df.astype(dtype, copy=None, errors="raise")`
- **參數說明**：
  - `dtype`：目標型別名稱或字典 `{欄位名: 目標型別}`。
  - `errors`：轉換失敗時策略（`'raise'` 拋錯、`'ignore'` 忽略保持原樣）。
- **回傳值**：
  - 轉型後的 `DataFrame` 或 `Series`。

```python
import pandas as pd

df = pd.DataFrame({"ID": [1.0, 2.0], "類別": ["A", "B"]})
# 字典批次轉換多欄型別
df = df.astype({"ID": "int64", "類別": "category"})
print(df["ID"].dtype)  # 輸出: int64
```

**`df` 表格結構**：

| Index | ID | 類別 |
| :---: | :---: | :--- |
| 0 | 1.0 | A |
| 1 | 2.0 | B |

---

##### pd.to_datetime() 強大時間戳記解析器

- **使用時機**：外部載入的日期往往是純文字（object 型態），必須轉為 pandas Timestamp 才能進行時間差運算或年月提取。
- **語法**：`pd.to_datetime(arg, errors="raise", dayfirst=False, yearfirst=False, utc=False, format=None, unit=None)`
- **參數說明**：
  - `arg`：字串、串列或 Series。
  - `format`：顯式指定解析格式（如 `'%Y-%m-%d %H:%M:%S'`，明確指定可加速解析數十倍）。
  - `errors`：無法解析時策略（`'coerce'` 最常用：解析失敗強制轉為 `NaT` 缺失值而非崩潰）。
- **回傳值**：
  - `Timestamp` 或 `DatetimeIndex` 或 `Series`。

```python
import pandas as pd

dates = ["2026-10-01", "2026/10/02", "無效日期"]
# errors='coerce' 可防呆將無效日期轉為 NaT
s_time = pd.to_datetime(dates, errors="coerce")
print(s_time[0].year)  # 輸出: 2026
print(pd.isna(s_time[2]))  # 輸出: True
```

---

##### pd.to_numeric() 強制數值型別轉換

- **使用時機**：數值欄位中夾雜了 `"?"`、`"N/A"` 等髒字元導致整個欄位被判斷為字串時，安全轉為 float 並將髒字元轉為 NaN。
- **語法**：`pd.to_numeric(arg, errors="raise", downcast=None)`
- **參數說明**：
  - `errors`：設為 `'coerce'` 時，無法解析的字元自動替換為 `np.nan`。
  - `downcast`：自動壓縮為最小相容型別（`'integer'`、`'float'`）以節省記憶體。
- **回傳值**：
  - 數值型 `Series`。

```python
import pandas as pd

raw_prices = pd.Series(["100", "250.5", "缺貨", "300"])
clean_prices = pd.to_numeric(raw_prices, errors="coerce")
print(clean_prices.tolist())  # 輸出: [100.0, 250.5, nan, 300.0]
```

---

---

# 資料清洗、整理與軸向旋轉 (Data Wrangling & Reshaping)

欄位刪除與重命名、去重、排序、索引結構切換以及長寬表格雙向旋轉樞紐分析。

---

##### DataFrame.drop() 刪除指定欄位或索引列

- **使用時機**：去除資料表中不再需要的特徵欄位（Columns）或不符資格的樣本列（Rows）。
- **語法**：`df.drop(labels=None, axis=0, index=None, columns=None, level=None, inplace=False, errors="raise")`
- **參數說明**：
  - `columns`：直接傳入欲刪除的欄名單一字串或串列（語意清晰，現代推薦）。
  - `index`：傳入欲刪除的列索引標籤串列。
  - `axis`：舊式寫法中 `axis=1` 代表欄，`axis=0` 代表列。
- **回傳值**：
  - 刪除後的 `DataFrame`。

```python
import pandas as pd

df = pd.DataFrame({"A": [1, 2], "B": [3, 4], "C": [5, 6]})
# 現代推薦明確寫法：使用 columns 參數
df_dropped = df.drop(columns=["B", "C"])
print(df_dropped.columns.tolist())  # 輸出: ['A']
```

**`df` 表格結構**：

| Index | A | B | C |
| :---: | :---: | :---: | :---: |
| 0 | 1 | 3 | 5 |
| 1 | 2 | 4 | 6 |

---

##### DataFrame.rename() 重新命名欄位或索引標籤

- **使用時機**：不破壞其他欄位，僅精準更改某些特定欄位或列的名稱時使用。
- **語法**：`df.rename(mapper=None, index=None, columns=None, axis=None, inplace=False, errors="ignore")`
- **參數說明**：
  - `columns`：字典 `{舊欄名: 新欄名}` 或字串轉換函式。
  - `index`：字典 `{舊索引: 新索引}`。
- **回傳值**：
  - 重命名後的 `DataFrame`。

```python
import pandas as pd

df = pd.DataFrame({"old_a": [1], "old_b": [2]})
df = df.rename(columns={"old_a": "new_a", "old_b": "new_b"})
print(df.columns.tolist())  # 輸出: ['new_a', 'new_b']
```

**`df` 表格結構**：

| Index | old_a | old_b |
| :---: | :---: | :---: |
| 0 | 1 | 2 |

---

##### DataFrame.drop_duplicates() 剔除重複列

- **使用時機**：資料收集過程中產生重複記錄時，根據唯一鍵（如 ID）進行去重。
- **語法**：`df.drop_duplicates(subset=None, keep="first", inplace=False, ignore_index=False)`
- **參數說明**：
  - `subset`：指定評估重複的欄位名稱串列（若為 `None` 則全欄位完全相同才算重複）。
  - `keep`：保留策略（`'first'` 保留首筆、`'last'` 保留末筆、`False` 只要有重複全部刪光）。
- **回傳值**：
  - 去重後的 `DataFrame`。

```python
import pandas as pd

df = pd.DataFrame({"ID": [101, 102, 101], "姓名": ["Alice", "Bob", "Alice_New"]})
# 依據 ID 去重，保留最新（最後出現）的一筆
df_unique = df.drop_duplicates(subset=["ID"], keep="last")
print(df_unique.iloc[0]["姓名"])  # 輸出: Bob
```

**`df` 表格結構**：

| Index | ID | 姓名 |
| :---: | :---: | :--- |
| 0 | 101 | Alice |
| 1 | 102 | Bob |
| 2 | 101 | Alice_New |

---

##### DataFrame.sort_values() 依數值排序資料列

- **使用時機**：依據關鍵數值（如銷售額、評分、日期）對全表進行升冪或降冪排列。
- **語法**：`df.sort_values(by, axis=0, ascending=True, inplace=False, kind="quicksort", na_position="last", ignore_index=False)`
- **參數說明**：
  - `by`：排序依據的欄位名稱（字串或串列）。
  - `ascending`：布林值或布林串列（`True` 升冪小到大；`False` 降冪大到小）。
  - `na_position`：缺失值 NaN 置放位置（`'last'` 擺最末端、`'first'` 擺最頂部）。
- **回傳值**：
  - 排序後的 `DataFrame`。

```python
import pandas as pd

df = pd.DataFrame({"部門": ["A", "B", "A"], "考績": [85, 95, 90]})
# 多欄排序：部門升冪，考績降冪
df_sorted = df.sort_values(by=["部門", "考績"], ascending=[True, False])
print(df_sorted.iloc[0]["考績"])  # 輸出: 90
```

**`df` 表格結構**：

| Index | 部門 | 考績 |
| :---: | :--- | :---: |
| 0 | A | 85 |
| 1 | B | 95 |
| 2 | A | 90 |

---

##### DataFrame.set_index() 與 DataFrame.reset_index() 索引結構重構

- **使用時機**：將具有識別意義的欄位（如 Date、UserID）設為 Index 以加速檢索；或在分組運算後將多層索引壓平成一般欄位。
- **語法**：
  - 提升：`df.set_index(keys, drop=True, append=False, inplace=False)`
  - 還原：`df.reset_index(level=None, drop=False, inplace=False)`
- **參數說明**：
  - `drop`：在 `reset_index` 中若設為 `True`，則直接拋棄舊索引而不將其還原為新欄位。
- **回傳值**：
  - `DataFrame`。

```python
import pandas as pd

df = pd.DataFrame({"ID": [101, 102], "分數": [80, 90]})
# 提升 ID 為 Index
df_indexed = df.set_index("ID")
print(df_indexed.loc[101, "分數"])  # 輸出: 80

# 還原為普通整數索引
df_flat = df_indexed.reset_index()
print(df_flat.columns.tolist())  # 輸出: ['ID', '分數']
```

**`df` 表格結構**：

| Index | ID | 分數 |
| :---: | :---: | :---: |
| 0 | 101 | 80 |
| 1 | 102 | 90 |

---

##### DataFrame.melt() 寬表格逆樞紐轉為長表格

- **使用時機**：資料欄位呈現「姓名、2024銷量、2025銷量、2026銷量」等寬表結構時，將年份壓成「年份」與「銷量」兩欄長表。
- **語法**：`df.melt(id_vars=None, value_vars=None, var_name=None, value_name="value", ignore_index=True)`
- **參數說明**：
  - `id_vars`：保持不變的識別依據欄位（如姓名、ID）。
  - `value_vars`：欲旋轉壓縮的欄位清單（若未指定則包含所有非 id_vars 欄位）。
  - `var_name`：新變數欄位名稱（如 `'年份'`）。
  - `value_name`：新數值欄位名稱（如 `'銷售額'`）。
- **回傳值**：
  - 長格式 `DataFrame`。

```python
import pandas as pd

df = pd.DataFrame({"姓名": ["Alice"], "2025": [100], "2026": [150]})
df_long = df.melt(id_vars=["姓名"], var_name="年份", value_name="業績")
print(df_long.shape)  # 輸出: (2, 3) 包含 [姓名, 年份, 業績]
```

**`df` 表格結構**：

| Index | 姓名 | 2025 | 2026 |
| :---: | :--- | :---: | :---: |
| 0 | Alice | 100 | 150 |

---

##### DataFrame.pivot() 與 DataFrame.pivot_table() 樞紐分析表轉換

- **使用時機**：類似 Excel 的「樞紐分析表」，將長型紀錄依照列索引（index）與欄標籤（columns）展開為二維交叉矩陣，並對交會點數值進行聚合。
- **語法**：
  - 純展開（無聚合）：`df.pivot(columns=None, index=None, values=None)`
  - 樞紐分析（含聚合）：`df.pivot_table(values=None, index=None, columns=None, aggfunc="mean", fill_value=None, margins=False)`
- **參數說明**：
  - `aggfunc`：聚合函式（如 `'mean'`、`'sum'`、`'count'`、`np.std` 等）。
  - `margins`：布林值，是否在最末端自動新增「總計（All）」列與欄。
- **回傳值**：
  - 樞紐分析後的 `DataFrame`。

```python
import pandas as pd

df = pd.DataFrame({
    "部門": ["研發", "研發", "業務", "業務"],
    "性別": ["男", "女", "男", "女"],
    "薪資": [70000, 75000, 50000, 52000]
})
pt = df.pivot_table(index="部門", columns="性別", values="薪資", aggfunc="mean")
print(pt.loc["研發", "女"])  # 輸出: 75000.0
```

**`df` 表格結構**：

| Index | 部門 | 性別 | 薪資 |
| :---: | :--- | :--- | :---: |
| 0 | 研發 | 男 | 70000 |
| 1 | 研發 | 女 | 75000 |
| 2 | 業務 | 男 | 50000 |
| 3 | 業務 | 女 | 52000 |

---

---

# 分組聚合與轉換計算 (GroupBy & Operations)

拆分-應用-合併機制、多元聚合、組內廣播轉換與逐欄/逐格對映處理。

---

##### DataFrame.groupby() 拆分-應用-合併分組引擎

- **使用時機**：SQL `GROUP BY` 的強大對應實作。將大資料表按特定類別（如地區、店家、性別）拆成多個獨立組別。
- **語法**：`df.groupby(by=None, axis=0, level=None, as_index=True, sort=True, group_keys=True, observed=False, dropna=True)`
- **參數說明**：
  - `by`：分組依據欄名（字串或串列）。
  - `as_index`：是否將分組鍵作為結果的 Index（設為 `False` 則直接保持為一般欄位，極為常用）。
- **回傳值**：
  - `DataFrameGroupBy` 物件。

```python
import pandas as pd

df = pd.DataFrame({"部門": ["IT", "HR", "IT"], "薪資": [60000, 45000, 70000]})
# 分組後求平均
print(df.groupby("部門", as_index=False)["薪資"].mean())
```

**`df` 表格結構**：

| Index | 部門 | 薪資 |
| :---: | :--- | :---: |
| 0 | IT | 60000 |
| 1 | HR | 45000 |
| 2 | IT | 70000 |

---

##### GroupBy.agg() 多元分組聚合運算

- **使用時機**：在 `groupby` 之後，需要對不同欄位同時施加不同的統計指標時使用。
- **語法**：`grouped.agg(func=None, *args, **kwargs)`（或縮寫 `grouped.aggregate()`）
- **參數說明**：
  - `func`：函式名稱字串、函式物件、清單或字典 `{欄位名: [統計指標清單]}`。
- **回傳值**：
  - 聚合運算後的 `DataFrame`。

```python
import pandas as pd

df = pd.DataFrame({
    "部門": ["業務", "業務", "研發", "研發"],
    "業績": [100, 150, 80, 120],
    "客戶數": [10, 12, 5, 8]
})

# 字典指定：業績算總和與平均，客戶數算最大值
summary = df.groupby("部門").agg({
    "業績": ["sum", "mean"],
    "客戶數": "max"
})
print(summary)
```

**`df` 表格結構**：

| Index | 部門 | 業績 | 客戶數 |
| :---: | :--- | :---: | :---: |
| 0 | 業務 | 100 | 10 |
| 1 | 業務 | 150 | 12 |
| 2 | 研發 | 80 | 5 |
| 3 | 研發 | 120 | 8 |

---

##### GroupBy.transform() 分組廣播轉換保持原形狀

- **使用時機**：不想壓縮資料表的列數，而是希望將分組的統計量（如各部門平均值）直接貼回每一列原始資料旁。
- **語法**：`grouped.transform(func, *args, **kwargs)`
- **核心特點**：
  - 回傳的數列長度與原始資料表完全一致，可直接作為新欄位賦值。
- **回傳值**：
  - 與原始輸入長度相同的 `Series` 或 `DataFrame`。

```python
import pandas as pd

df = pd.DataFrame({"部門": ["A", "A", "B"], "薪資": [50000, 70000, 60000]})
# 計算各員工相對於自己部門平均薪資的差異
dept_mean = df.groupby("部門")["薪資"].transform("mean")
df["薪資差額"] = df["薪資"] - dept_mean
print(df["薪資差額"].tolist())  # 輸出: [-10000.0, 10000.0, 0.0]
```

**`df` 表格結構**：

| Index | 部門 | 薪資 |
| :---: | :--- | :---: |
| 0 | A | 50000 |
| 1 | A | 70000 |
| 2 | B | 60000 |

---

##### DataFrame.apply() 逐欄或逐列自訂函式映射

- **使用時機**：無法透過內建向量化函式完成的複雜自訂邏輯，逐欄（Series）或逐列（Row）執行。
- **語法**：`df.apply(func, axis=0, raw=False, result_type=None, args=(), **kwargs)`
- **參數說明**：
  - `func`：作用於每一欄/列的 Callable 函式。
  - `axis`：`0` 或 `'index'` 垂直向下對每欄運算；`1` 或 `'columns'` 水平向右對每列運算。
- **回傳值**：
  - `Series` 或 `DataFrame`。

```python
import pandas as pd

df = pd.DataFrame({"底薪": [30000, 40000], "獎金": [5000, 8000]})
# 水平橫向計算總年薪 (axis=1)
def calc_total(row):
    return (row["底薪"] + row["獎金"]) * 14

df["總年薪"] = df.apply(calc_total, axis=1)
print(df.loc[0, "總年薪"])  # 輸出: 490000
```

**`df` 表格結構**：

| Index | 底薪 | 獎金 |
| :---: | :---: | :---: |
| 0 | 30000 | 5000 |
| 1 | 40000 | 8000 |

---

##### DataFrame.map() 全表元素逐格對映轉換

- **使用時機**：對數列中的值進行字典代換，或對 DataFrame 的「每一個格子」套用同一純量轉換函式。
- **語法**：
  - DataFrame：`df.map(func, na_action=None)`（Pandas 2.1+ 取代舊版 applymap）
  - Series：`s.map(arg, na_action=None)`（支援傳入字典 `{舊值: 新值}`）
- **回傳值**：
  - 轉換後的 `DataFrame` 或 `Series`。

```python
import pandas as pd

# 1. Series 透過字典直接代換類別代碼
s = pd.Series(["M", "F", "M"])
s_mapped = s.map({"M": "男性", "F": "女性"})
print(s_mapped.tolist())  # 輸出: ['男性', '女性', '男性']

# 2. DataFrame 逐格套用純量函式 (Pandas 2.1+ 推薦寫法)
df = pd.DataFrame({"A": [1.234, 5.678], "B": [9.876, 3.456]})
print(df.map(lambda x: round(x, 1)))
```

**`df` 表格結構**：

| Index | A | B |
| :---: | :---: | :---: |
| 0 | 1.234 | 9.876 |
| 1 | 5.678 | 3.456 |

---

---

# 多表合併與關聯對齊 (Combining & Merging)

垂直/水平拼接、關聯式資料庫 SQL JOIN 鍵值連接與索引快速合併。

---

##### pd.concat() 軸向拼接合併多個資料表

- **使用時機**：結構相同（或不同）的多個 DataFrame 進行單純的「頭接尾堆疊」或「左右並排對齊」。
- **語法**：`pd.concat(objs, axis=0, join="outer", ignore_index=False, keys=None, verify_integrity=False, copy=None)`
- **參數說明**：
  - `objs`：包含多個 Series 或 DataFrame 的序列（Sequence）。
  - `axis`：`0`（垂直堆疊增長列數，預設）；`1`（水平拼接增長欄數）。
  - `ignore_index`：是否丟棄舊索引並重建從 0 起算的連續整數索引（極常用，避免垂直拼接時索引重複）。
- **回傳值**：
  - 拼接後的 `DataFrame` 或 `Series`。

```python
import pandas as pd

df1 = pd.DataFrame({"ID": [1, 2], "名稱": ["A", "B"]})
df2 = pd.DataFrame({"ID": [3, 4], "名稱": ["C", "D"]})

# 垂直向下拼接並重設索引 (ignore_index=True)
all_df = pd.concat([df1, df2], ignore_index=True)
print(len(all_df))  # 輸出: 4
```

**`df1` 表格結構**：

| Index | ID | 名稱 |
| :---: | :---: | :--- |
| 0 | 1 | A |
| 1 | 2 | B |

**`df2` 表格結構**：

| Index | ID | 名稱 |
| :---: | :---: | :--- |
| 0 | 3 | C |
| 1 | 4 | D |

---

##### pd.merge() 與 DataFrame.merge() 關聯式資料庫鍵值連接

- **使用時機**：SQL 關聯查詢的主力實作。當兩張資料表有共同的欄位（如 `user_id`），需要將它們橫向串聯在一起時使用。
- **語法**：`pd.merge(left, right, how="inner", on=None, left_on=None, right_on=None, left_index=False, right_index=False, suffixes=("_x", "_y"))`
- **參數說明**：
  - `how`：連接類型（`'inner'` 兩表皆有才保留、`'left'` 保留左表全部、`'right'` 保留右表全部、`'outer'` 聯集保留全部）。
  - `on`：兩表共同的主鍵欄名。
  - `left_on` / `right_on`：當兩表主鍵名稱不同時分別指定。
  - `suffixes`：當兩表存在非主鍵同名欄位時，自動追加的後綴字串元組。
- **回傳值**：
  - 關聯連接後的 `DataFrame`。

```python
import pandas as pd

users = pd.DataFrame({"uid": [1, 2], "name": ["Alice", "Bob"]})
orders = pd.DataFrame({"uid": [1, 1], "product": ["Book", "Pen"]})

# Left Join: 確保所有訂單皆對齊用戶名稱
merged = pd.merge(orders, users, on="uid", how="left")
print(merged.columns.tolist())  # 輸出: ['uid', 'product', 'name']
```

**`users` 表格結構**：

| Index | uid | name |
| :---: | :---: | :--- |
| 0 | 1 | Alice |
| 1 | 2 | Bob |

**`orders` 表格結構**：

| Index | uid | product |
| :---: | :---: | :--- |
| 0 | 1 | Book |
| 1 | 1 | Pen |

---

##### DataFrame.join() 基於索引快速水平拼接

- **使用時機**：兩張表的關聯鍵本身就是它們的 Index 時，呼叫 `df.join()` 比 `merge()` 更精簡快速。
- **語法**：`df.join(other, on=None, how="left", lsuffix="", rsuffix="", sort=False)`
- **參數說明**：
  - `other`：另一個 DataFrame 或 Series。
  - `how`：連接方式（預設為 `'left'`）。
- **回傳值**：
  - `DataFrame`。

```python
import pandas as pd

df1 = pd.DataFrame({"A": [1, 2]}, index=["x", "y"])
df2 = pd.DataFrame({"B": [3, 4]}, index=["x", "y"])
print(df1.join(df2))
```

**`df1` 表格結構**：

| Index | A |
| :---: | :---: |
| x | 1 |
| y | 2 |

**`df2` 表格結構**：

| Index | B |
| :---: | :---: |
| x | 3 |
| y | 4 |

---

---

# 字串向量化與時間序列專用屬性 (Vectorized String & Datetime)

免迴圈向量化文字清理、正規表達式與時間日期年月日提取與頻率重取樣。

---

##### Series.str 向量化字串存取器

- **使用時機**：當某一欄為字串資料時，透過 `.str.` 存取器可以像處理純字串一樣批次處理整欄資料，且原生具備空值傳播安全保護。
- **常用方法**：
  - `s.str.strip()`：去除首尾空白。
  - `s.str.lower()` / `s.str.upper()`：大小寫轉換。
  - `s.str.contains(pat, regex=True, na=False)`：是否包含特定子字串或正規表達式（回傳布林遮罩，極常用於條件篩選）。
  - `s.str.split(pat, expand=True)`：字串切割（`expand=True` 可直接將切割結果拆成多個獨立欄位）。
  - `s.str.replace(pat, repl, regex=True)`：文字替換。
- **回傳值**：
  - 運算後的 `Series` 或 `DataFrame`。

```python
import pandas as pd

s = pd.Series([" alice@mail.com ", "BOB@google.com", None])

# 1. 去除空白並轉小寫
s_clean = s.str.strip().str.lower()

# 2. 篩選包含 'google' 的記錄 (na=False 確保空值不報錯)
is_google = s_clean.str.contains("google", na=False)
print(is_google.tolist())  # 輸出: [False, True, False]

# 3. 切割出使用者帳號與網域 (expand=True 轉為二維 DataFrame)
domains = s_clean.str.split("@", expand=True)
print(domains.shape)  # 輸出: (3, 2)
```

---

##### Series.dt 向量化時間存取器

- **使用時機**：當欄位已轉為 `datetime64` 型態時，透過 `.dt.` 存取器可直接批次提取時間日期子維度。
- **常用屬性與方法**：
  - `s.dt.year` / `s.dt.month` / `s.dt.day`：提取年、月、日整數。
  - `s.dt.hour` / `s.dt.minute` / `s.dt.second`：提取時、分、秒。
  - `s.dt.dayofweek` / `s.dt.day_name()`：星期幾（0 為週一）與英文星期名稱。
  - `s.dt.strftime(format)`：將時間格式化回指定字串格式。
- **回傳值**：
  - 提取後的數值或字串 `Series`。

```python
import pandas as pd

df = pd.DataFrame({"交易時間": pd.to_datetime(["2026-10-01 09:30:00", "2026-10-04 18:20:00"])})

# 批次提取月份、星期與小時
df["月份"] = df["交易時間"].dt.month
df["小時"] = df["交易時間"].dt.hour
df["是否為週末"] = df["交易時間"].dt.dayofweek >= 5
print(df["是否為週末"].tolist())  # 輸出: [False, True]
```

**`df` 表格結構**：

| Index | 交易時間 |
| :---: | :--- |
| 0 | 2026-10-01 09:30:00 |
| 1 | 2026-10-04 18:20:00 |

---

##### DataFrame.resample() 時間序列頻率重取樣

- **使用時機**：類似時間維度的 `groupby`。當資料庫以高頻率記錄時，將其重新取樣彙總為低頻率（如分轉日、日轉月）。
- **語法**：`df.resample(rule, axis=0, closed=None, label=None, convention="start", kind=None)`
- **參數說明**：
  - `rule`：目標頻率字串（如 `'D'` 天、`'W'` 週、`'ME'` 月末、`'h'` 小時）。
- **回傳值**：
  - `Resampler` 物件，後續接續呼叫 `.sum()`、`.mean()`、`.ohlc()` 等聚合方法。

```python
import pandas as pd

dates = pd.date_range("2026-10-01", periods=10, freq="D")
df = pd.DataFrame({"點擊數": range(10)}, index=dates)

# 將每日數據重取樣彙總為每週總點擊數
weekly = df.resample("W").sum()
print(len(weekly))  # 輸出聚合後的週數
```

**`df` 表格結構**：

| Index | 點擊數 |
| :---: | :---: |
| 2026-10-01 | 0 |
| 2026-10-02 | 1 |
| 2026-10-03 | 2 |
| 2026-10-04 | 3 |
| 2026-10-05 | 4 |
| 2026-10-06 | 5 |
| 2026-10-07 | 6 |
| 2026-10-08 | 7 |
| 2026-10-09 | 8 |
| 2026-10-10 | 9 |

---

---

# 統計計算與累計視窗 (Statistics & Windowing)

描述統計、極值索引定位、累積計算、滑動視窗平滑化與差分增長率。

---

##### DataFrame.sum() 與 DataFrame.mean() 加總與算術平均值

- **使用時機**：計算各欄或各列的數值總和或平均值。
- **語法**：`df.sum(axis=0, skipna=True)` 與 `df.mean(axis=0, skipna=True)`
- **參數說明**：
  - `axis`：`0`（垂直向下，計算各欄統計值，預設）；`1`（水平向右，計算各列之和/平均）。
  - `skipna`：布林值，計算時是否自動略過空值（預設 `True`）。
- **回傳值**：
  - `Series`。

```python
import pandas as pd

df = pd.DataFrame({"國文": [80, 90], "英文": [70, 85]})
# 1. 直向計算各科全班平均 (axis=0)
print(df.mean(axis=0)["國文"])  # 輸出: 85.0

# 2. 橫向計算每位學生的總分 (axis=1)
df["總分"] = df.sum(axis=1)
print(df["總分"].tolist())  # 輸出: [150, 175]
```

**`df` 表格結構**：

| Index | 國文 | 英文 |
| :---: | :---: | :---: |
| 0 | 80 | 70 |
| 1 | 90 | 85 |

---

##### DataFrame.max() 與 DataFrame.min() 極值統計

- **使用時機**：找出欄位或樣本中的極大值與極小值。
- **語法**：`df.max(axis=0, skipna=True)` 與 `df.min(axis=0, skipna=True)`
- **回傳值**：
  - `Series`。

```python
import pandas as pd

s = pd.Series([10, 45, 99, 23])
print(f"最大值: {s.max()}, 最小值: {s.min()}")  # 輸出: 最大值: 99, 最小值: 10
```

---

##### DataFrame.idxmax() 與 DataFrame.idxmin() 極值所在索引定位

- **使用時機**：不只知道最高分是多少，更需要知道「是誰」拿到最高分，直接回傳對應的 Index 標籤。
- **語法**：`df.idxmax(axis=0, skipna=True)` 與 `df.idxmin(axis=0, skipna=True)`
- **回傳值**：
  - 索引標籤純量或 `Series`。

```python
import pandas as pd

s = pd.Series([100, 250, 80], index=["Alice", "Bob", "Charlie"])
# 找出業績最高的員工姓名 (Index)
top_sales = s.idxmax()
print(top_sales)  # 輸出: Bob
```

---

##### DataFrame.std() 與 DataFrame.var() 標準差與變異數

- **使用時機**：評估數據的離散程度與波動性。Pandas 預設除以 $N-1$（無偏樣本估計，`ddof=1`）。
- **語法**：`df.std(axis=0, skipna=True, ddof=1)` 與 `df.var(axis=0, skipna=True, ddof=1)`
- **回傳值**：
  - `Series`。

```python
import pandas as pd

s = pd.Series([10, 20, 30, 40, 50])
print(round(s.std(), 2))  # 輸出: 15.81
```

---

##### DataFrame.cumsum() 與 DataFrame.cumprod() 累加與累乘計算

- **使用時機**：計算隨著時間推移不斷累積的統計量（如每日累計營收、資產累積回報率）。
- **語法**：`df.cumsum(axis=0, skipna=True)` 與 `df.cumprod(axis=0, skipna=True)`
- **回傳值**：
  - 同維度的 `DataFrame` 或 `Series`。

```python
import pandas as pd

s = pd.Series([100, 150, 200], index=["1月", "2月", "3月"])
print(s.cumsum().tolist())  # 輸出: [100, 250, 450]
```

---

##### DataFrame.rolling() 移動視窗滑動計算

- **使用時機**：技術分析與時間序列分析核心工具，計算過去 N 天的移動平均、移動標準差等。
- **語法**：`df.rolling(window, min_periods=None, center=False, win_type=None, on=None, axis=0)`
- **參數說明**：
  - `window`：視窗大小（如整數 5 代表 5 筆資料）。
  - `min_periods`：視窗內至少需要多少非空值才能計算（若不足則回傳 NaN）。
- **回傳值**：
  - `Rolling` 物件，後續呼叫 `.mean()`、`.std()` 等聚合方法。

```python
import pandas as pd

prices = pd.Series([10, 11, 12, 13, 14, 15])
# 計算 3 筆資料的移動平均線 (MA3)
ma3 = prices.rolling(window=3).mean()
print(ma3.tolist())  # 輸出: [nan, nan, 11.0, 12.0, 13.0, 14.0]
```

---

##### Series.shift() 與 Series.diff() 時間落後與差分計算

- **使用時機**：需要計算「今天跟昨天差多少」或需要構造時間序列前期的延遲特徵（Lag Feature）。
- **語法**：
  - 推移：`s.shift(periods=1, freq=None, axis=0, fill_value=None)`
  - 差分：`s.diff(periods=1, axis=0)`（等價於 `s - s.shift(periods)`）
- **回傳值**：
  - 同長度 `Series`。

```python
import pandas as pd

s = pd.Series([100, 105, 102, 110])
# 1. 取得上一期的數值 (shift)
prev = s.shift(1)
print(prev.tolist())  # 輸出: [nan, 100.0, 105.0, 102.0]

# 2. 計算本期與上期的差額 (diff)
diff = s.diff()
print(diff.tolist())  # 輸出: [nan, 5.0, -3.0, 8.0]
```

---

##### Series.pct_change() 百分比增長率計算

- **使用時機**：金融與經濟數據分析必備，直接計算增長率 $rac{x_t - x_{t-1}}{x_{t-1}}$。
- **語法**：`s.pct_change(periods=1, fill_method=None, limit=None, freq=None)`
- **回傳值**：
  - 百分比浮點數 `Series`。

```python
import pandas as pd

prices = pd.Series([100.0, 110.0, 99.0])
# 計算各期收益率
returns = prices.pct_change()
print(returns.tolist())  # 輸出: [nan, 0.1, -0.1] (漲 10%，跌 10%)
```

---

---

# 效能最佳化、深淺拷貝與設定 (Optimization & Settings)

記憶體複製機制防禦警告，以及全域顯示列寬與小數位數配置。

---

##### DataFrame.copy() 顯式深拷貝避免鏈式賦值警告

- **使用時機**：從大型資料表切片篩選出一組子資料表時，顯式深拷貝以確保子表與母表完全斷開記憶體共享，防止修改時跳出 `SettingWithCopyWarning`。
- **語法**：`df.copy(deep=True)`
- **參數說明**：
  - `deep`：預設為 `True`，複製資料與索引結構本體；若為 `False` 則為淺拷貝（Shallow Copy）。
- **回傳值**：
  - 獨立的 `DataFrame` 實例。

```python
import pandas as pd

df = pd.DataFrame({"A": [1, 2, 3], "B": [4, 5, 6]})
# [正確寫法]：顯式呼叫 copy() 切斷記憶體共享
sub_df = df[df["A"] > 1].copy()
sub_df["B"] = 999  # 安全賦值，完全不報警告，亦不污染原始 df
```

**`df` 表格結構**：

| Index | A | B |
| :---: | :---: | :---: |
| 0 | 1 | 4 |
| 1 | 2 | 5 |
| 2 | 3 | 6 |

---

##### pd.set_option() 與 pd.get_option() 全域顯示與行為配置

- **使用時機**：在終端機或 Jupyter Notebook 中，避免大資料表的欄位或列數被自動以省略號 `...` 隱藏，或統一設定浮點數小數位數。
- **語法**：`pd.set_option(pat, value)` 與 `pd.get_option(pat)`
- **常用設定鍵值**：
  - `'display.max_columns'`：最大顯示欄數（設為 `None` 代表全部展開無截斷）。
  - `'display.max_rows'`：最大顯示列數。
  - `'display.float_format'`：浮點數格式化字串（如 `'{:.2f}'.format`）。
  - `'display.max_colwidth'`：單一欄位字串最大展示寬度。
- **回傳值**：
  - `None` 或查詢到的配置值。

```python
import pandas as pd

# 完整展開所有欄位，防止印出時出現省略號截斷
pd.set_option("display.max_columns", None)
# 設定浮點數預設印出兩位小數
pd.set_option("display.float_format", "{:.2f}".format)
```

---

---

# 實戰避坑與核心天條

## 1. 鏈式賦值引發 SettingWithCopyWarning 警告

> **[核心天條]：嚴禁使用鏈式切片賦值！請一律使用 `.loc[]` 或顯式 `.copy()`！**  
> 當你寫下 `df[df['A'] > 2]['B'] = 99` 時，Python 先執行了第一層篩選產生一個臨時視圖或拷貝，接著嘗試對這個臨時對象賦值。Pandas 無法保證此賦值是否會寫回原始母表，因而拋出著名的 `SettingWithCopyWarning`！

```python
import pandas as pd

df = pd.DataFrame({"A": [1, 2, 3], "B": [10, 20, 30]})

# [錯誤寫法]：鏈式切片賦值 (Chained Assignment)
# df[df["A"] > 1]["B"] = 999  # 引發 SettingWithCopyWarning，且修改可能失敗！

# [正確寫法 A]：使用 loc 單步精確定位並賦值
df.loc[df["A"] > 1, "B"] = 999

# [正確寫法 B]：若是要抽取出獨立子資料表，顯式加上 .copy()
sub_df = df[df["A"] > 1].copy()
sub_df["B"] = 888  # 完全安全，與母表互不干擾
```

**`df` 表格結構**：

| Index | A | B |
| :---: | :---: | :---: |
| 0 | 1 | 10 |
| 1 | 2 | 20 |
| 2 | 3 | 30 |

---

## 2. 條件篩選漏加小括號引爆運算子優先級災難

> **[核心天條]：多條件篩選時，每一個獨立條件運算式都必須用小括號 `()` 包裹！**  
> 在 Python 中，位元運算子 `&`（AND）與 `|`（OR）的運算優先級高於比較運算子 `>=`、`<=`、`==`。  
> 若寫成 `df[df['A'] > 10 & df['B'] < 20]`，Python 會優先運算 `10 & df['B']`，立即引發 `TypeError` 或產生完全錯誤的布林邏輯！

```python
import pandas as pd

df = pd.DataFrame({"A": [5, 15, 25], "B": [10, 15, 20]})

# [錯誤寫法]：未加括號
# result = df[df["A"] > 10 & df["B"] < 20]  # 引爆 TypeError！

# [正確寫法]：每個條件皆以括號嚴格包覆
result = df[(df["A"] > 10) & (df["B"] < 20)]
```

**`df` 表格結構**：

| Index | A | B |
| :---: | :---: | :---: |
| 0 | 5 | 10 |
| 1 | 15 | 15 |
| 2 | 25 | 20 |

---

## 3. 在 DataFrame 上使用 for 迴圈逐列迭代造成效能雪崩

> **[核心天條]：永遠優先選擇向量化操作 (Vectorization) 或 `apply`，嚴禁使用 `for index, row in df.iterrows():` 處理大量數據！**  
> `iterrows()` 在每一列迴圈中都會將資料封裝為一個全新的 Series 物件，產生巨大的物件生成開銷。處理 100 萬筆資料時，`iterrows()` 可能耗時數分鐘，而向量化操作僅需數毫秒（快上數百倍至千倍）！

```python
import pandas as pd

df = pd.DataFrame({"單價": [100, 200, 300], "數量": [2, 3, 4]})

# [錯誤寫法]：低效 Python 迴圈
# total = []
# for idx, row in df.iterrows():
#     total.append(row["單價"] * row["數量"])
# df["總額"] = total

# [正確寫法]：極速向量化運算 (C 語言底層並行)
df["總額"] = df["單價"] * df["數量"]
```

**`df` 表格結構**：

| Index | 單價 | 數量 |
| :---: | :---: | :---: |
| 0 | 100 | 2 |
| 1 | 200 | 3 |
| 2 | 300 | 4 |

---

## 4. inplace=True 的記憶體陷阱與方法鏈條斷裂

> **[核心天條]：避免過度迷信 `inplace=True`，現代 Pandas 推薦以重新賦值維護函數式鏈條！**  
> `inplace=True` 不僅不能保證真正原地零拷貝，還會導致方法回傳 `None`，徹底阻斷了流暢的方法鏈條（Method Chaining，如 `df.dropna().sort_values().head()`）。Pandas 官方核心團隊已在未來的 Pandas 3.0 路線圖中計畫逐步淡出 `inplace` 模式。

```python
import pandas as pd

df = pd.DataFrame({"A": [3, 1, 2], "B": [None, 4, 5]})

# [不推薦寫法]：阻斷方法鏈，且回傳 None
# df.dropna(inplace=True)
# df.sort_values(by="A", inplace=True)

# [推薦寫法]：乾淨明確的重新賦值或優雅的方法鏈接
df_clean = df.dropna().sort_values(by="A").reset_index(drop=True)
```

**`df` 表格結構**：

| Index | A | B |
| :---: | :---: | :---: |
| 0 | 3 | `NaN` |
| 1 | 1 | 4.0 |
| 2 | 2 | 5.0 |

---
