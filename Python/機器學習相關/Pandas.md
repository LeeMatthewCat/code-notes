用於對表格型的數據進行處理，想像成是**程式語言版本的 Excel**

```Python
import numpy as np # 通常也會用到
import pandas as pd
```

主要由兩種型別組成：

- **series 一維陣列：**單一欄或單一列表格

- **DataFrame 二維陣列：**一個表格

# 資料結構

## **series**

由**值**跟**索引**（預設為 0、1\.\.\.，**可自訂**）組成

可以利用**串列、字典、NumPy 的陣列**來建立

```Python
print(pd.Series([1,2,3,4,5])) # 用串列建立
print(pd.Series([1,2,3,4,5], index=['a','b','c','d','e'])) # 自訂索引

print(pd.Series({'a':1,'b':2,'c':3,'d':4,'e':5})) # 用字典

print(pd.Series(np.array([1,2,3,4,5]), index=['a', 'b', 'c', 'd', 'e'])) # 用陣列建立
```

```Python
serise = pd.Series([1,2,3,4,5])

print(serise.values)
print(serise.index) # RangeIndex(start=0, stop=5, step=1)
```

---

## DataFrame

### DataFrame 基本

它的功用如同 **Excel** 試算表或 **SQL 資料表**，可對它進行篩選、排序、樞紐分析及統計、新增或刪減欄位等

其實， DataFrame 是一個擁有**列（row）**及**欄（column）**索引的 Series 所組成的 Python 字典

同樣可以**用 index 註明第一欄的內容**，

而**字典的鍵**會變成**欄位名稱（第一列內容）**，值會變成該欄位的資料，一樣有預設的索引值在最左列

> ```Python
> df = pd.DataFrame(data, index = ["a", "b", "c", "d"])
> ```

```Python
import numpy as np
import pandas as pd

data = {
    '姓名': ['Alice', 'Bob', 'Charlie', 'David'],
    '年齡': [25, 30, 35, 28],
    '城市': ['台北', '台中', '高雄', '台北'],
    '薪水': [50000, 65000, np.nan, 55000] # np.nan 空值
}

# 轉換成 DataFrame
df = pd.DataFrame(data)

print(df)
#         姓名  年齡  城市     薪水 --> 欄位名稱
# 0    Alice  25  台北  50000
# 1      Bob  30  台中  65000
# 2  Charlie  35  高雄  70000
# 3    David  28  台北  55000
# ↑
# 預設索引
```

而使用串列來生成時，可將一個串列當成一列，或一欄

```Python
data = [ 
        ['Alice', 25, '台北'], 
        ['Bob', 30, '台中'], 
        ['Charlie', 35, '高雄'] ]

df = pd.DataFrame(data, columns=['姓名', '年齡', '城市']) # 指定欄位標題
```

```Bash
# 每個串列代表「一欄 (Column)」
data = {
    '姓名': ['Alice', 'Bob', 'Charlie'],
    '年齡': [25, 30, 35],
    '城市': ['台北', '台中', '高雄']
}

df = pd.DataFrame(data)
```

---

### Axis 軸

是用動作的走向來判斷，**不是單純依照欄和列**

- **=0 表示垂直向下走**

- **=1 表示水平往右走**

進行**運算處理**是用**擠壓**的方式，也就是把一列一欄壓在一起計算（跟上面的方向邏輯同）

而**搜尋**則是依照世上述的方式，搜尋到**向垂直方向進行處理**（方向邏輯不同）

---

**一些常用的函數：**

### **資訊查詢**

```Python
df.info()   # 顯示表格的摘要（欄位名稱、資料型態、是否有缺失值），不用 print

print(df.describe()) # 數據統計（只針對數字）
print(df.describe(include='all')) # 針對非數字也進行數量統計

print(df.head()) # 查看前五項（預設）
```

> **關於 info\(\)：**
> 
> Index: 4 entries, a to d **\-\-\>  4 列，索引 a 開頭 d 結尾**
> 
> Data columns \(total 4 columns\): **\-\-\> 4 欄**
> 
> Column  Non\-Null Count  Dtype 
> 
> \-\- \-\- \-\- \-\- \-\- \-\- \-\- \-\- \-\- \-\- \-\- \-\- \-\- \-\-
> 
> 0   姓名      4 non\-null      object **\-\-\> 遇到無法分類為單純數字的資料（如字串）時，就會被當稱 object**
> 
> 1   年齡      4 non\-null      int64
> 
> 2   城市      4 non\-null      object
> 
> 3   薪水      3 non\-null      float64 **\-\-\> 非空值 3 個**
> 
> dtypes: float64\(1\), int64\(1\), object\(2\) **\-\-\> 欄位型別統計（若型別裡面有一個非數字（如字串），則整欄會作為 object**
> 
> memory usage: 160\.0\+ bytes **\-\-\> 記憶體統計**

> 關於 describe\(\)：
> 
> 年齡            薪水
> 
> count   4\.000000      3\.000000 \-\-\> 非空值數量
> 
> mean   29\.500000  56666\.666667 \-\-\>平個
> 
> std     4\.203173   7637\.626158 \-\-\> 標準差
> 
> min    25\.000000  50000\.000000 \-\-\> 最小值
> 
> 25%    27\.250000  52500\.000000 \-\-\> 第一四分位
> 
> 50%    29\.000000  55000\.000000 \-\-\> 第二
> 
> 75%    31\.250000  60000\.000000 \-\-\> 第三
> 
> max    35\.000000  65000\.000000 \-\-\> 最大值

---

### **資料選取**

```Bash
print(df['姓名']) # 只拿出姓名這一欄（回傳一個 Series）

print(df[['姓名', '薪水']]) # 同時拿出姓名和薪水兩欄（回傳一個DataFrame）

print(df[df["年齡"] >= 30]) #篩選後挑出
# 裡面的 df[] 為條件判斷，外面的為篩選

print(df.loc[0:2]) # 選取索引 0~2，和傳統切片不同，包頭也包尾
df.loc[0:2, 'Name'] # 只要特定欄

print(df.loc[0:2]) # 透過索引尋找
```

> loc 和 iloc 差異：
> 
> 前者透過索引的名字搜尋，後者則是真正根據索引（包頭不包尾）

---

### **新增**

```Python
df.loc[4] = ['Eve', 22, '台南', 48000] # 選取後修改

df['年薪'] = df['薪水'] * 12 # 新增一欄
```

---

### **contact\(\) 合併**

```Python
import numpy as np
import pandas as pd

df_1 = pd.DataFrame({
    '姓名': ['Alice', 'Bob', 'Charlie', 'David'],
    '年齡': [25, 30, 35, 28],
    '城市': ['台北', '台中', '高雄', '台北'],
    '薪水': [50000, 65000, np.nan, 55000]
})

df_2 = pd.DataFrame({
    '姓名': ['Frank', 'Grace'],
    '年齡': [40, 29],
    '城市': ['台北', '高雄'],
    '薪水': [80000, 52000]
})

print(pd.concat([df_1, df_2], ignore_index=True)) # 忽略索引，預設為 false
```

> **在索引為預設下，沒有忽略索引：**
> 
> 姓名  年齡  城市       薪水
> 
> 0    Alice  25  台北  50000\.0
> 
> 1      Bob  30  台中  65000\.0
> 
> 2  Charlie  35  高雄      NaN
> 
> 3    David  28  台北  55000\.0
> 
> 0    Frank  40  台北  80000\.0 \-\-\> 索引重複
> 
> 1    Grace  29  高雄  52000\.0 

---

### **drop\(\) 刪除**

在未註明是欄位的情況下刪除欄時一定是著名 **axis=1（直的）**，而列則不用（直接標注索引）

```Python
df = df.drop("城市", axis=1) # 刪除欄
df = df.drop(columns=['城市']) # 註明為欄位

df = df.drop(index=[1]) # 刪除列（使用索引）
df = df.drop(index=[0, 1]) # 一次刪多列
df = df.drop("a") # 用自訂的
```

---

### **sort\_values\(\) **排序資料

```Python
df = df.sort_values(by='年齡') # 預設為上升

df = df.sort_values(by='年齡', ascending=False) #大到小
```

> **字串**也可被排序

如果遇到**同樣的**，可設定其他條件排序

```Python
df = df.sort_values(by=['年齡', '薪水'], ascending=[True, False]) # 用串列包起
```

遇到**缺值**時，**預設會被放到最下面**，也可自訂於最頂

```Python
df = df.sort_values(by='薪水', na_position='first')
```

---

### map\(\) 遍歷處理

對表格裡的每一個數值做同一件事（如全部轉成字串、全部加上錢字號）時使用

```Python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    '商品A': [100.5, 200.1],
    '商品B': [300.9, 400.2]
})

print(df.map(round))
```

---

### apply\(\) 逐欄／列處理

```Python
df_scores = pd.DataFrame({
    '國文': [80, 90, 70],
    '數學': [60, 85, 95]
}, index=['Alice', 'Bob', 'Charlie'])

# 自訂函式：計算一整組數字的最高分與最低分的差距
def calc_range(series):
    return series.max() - series.min()

# 範例 A (直向，axis=0)：算出「國文」的最高最低分差，以及「數學」的分差
print(df_scores.apply(calc_range)) 

# 範例 B (橫向，axis=1)：算出「每個人」自己兩科成績的落差
print(df_scores.apply(calc_range, axis=1))
```

---

