機器學習框架

適用於以下情況：

1. 處理文字、影像、聲音

2. 對架構高度客製化

3. 巨量資料，以 GPU 加速

4. 串接 AI 模型

---

# Tensor 張量

深度學習中，無論是圖片的**像素**、一段**文字**的數值化表示，還是神經網路裡面的**權重參數**，全部都是用 **Tensor 來儲存和運算**的

我們會把訓練資料裝在 Tensor，**相較 Numpy 的陣列形式**，Tensor 可以**運行在 GPU 上**，達到更好的效能

## 維度

- **0 維：**單一數字

- **1 維（向量）：**一排數字的串列

- **2 維（矩陣）：**表格

- **3 維（張量）：**立體數據塊

---

## 建立 Tensor

```Python
import torch

# 從一般的 Python 串列 (List) 建立
tensor_1d = torch.tensor([1, 2, 3, 4])
print(tensor_1d)

# 建立一個 2x3 (2列3行) 的隨機小數矩陣
tensor_rand = torch.rand(2, 3)
print(tensor_rand)

# 建立一個 3x3 全為 0 的矩陣
tensor_zeros = torch.zeros(3, 3) # 也可為 one
print(tensor_zeros)
```

---

### torch\.empty 使用空閒區塊

尋找一塊**閒置的記憶體空間**（**已經被使用過**，但被標幾位空閒），分配給此張量使用

可以把它視為**隨機生成出一個張量**（內容為上一個張量的元素），但較節省時間（**省去了重頭建立一個張量的時間**）

> 若都沒有找到合適記憶體空間，則全為 0

```Python
torch.empty(1 維大小, 2 維, ...)
```

---

#### \_like\(\) 複製型態

建立新張量時，**複製目標張量的形狀、型別、記憶體位置**

```Python
... torch.xxx_like(要複製的模板張量, ...)
```

---



## 屬性

```Python
t = torch.tensor([[1.0, 2.0, 3.0], [4.0, 5.0, 6.0]])

# 1. 形狀 (Shape)：這最重要！告訴你這個資料長什麼樣子
print("形狀 (Shape):", t.shape)  # 會輸出 torch.Size([2, 3])，代表 2 列 3 行

# 2. 資料型態 (Data Type)：整數還是浮點數？
print("資料型態 (dtype):", t.dtype) # 預設通常是 torch.float32 或 torch.int64

# 3. 所在設備 (Device)：在 CPU 還是 GPU 上？
print("所在設備 (device):", t.device) # 預設會在 cpu
```

---

## \.\.\. 省略

切片中使用，表示前面不管多少維度，通通都要

```Python
[:, :, :, 切片起點:切片終點]
```

```Python
[..., 切片起點:切片終點]
```

---

## C/GPU 運行

```Python
變數 = 變數.to("mps")
```

```Python
device = torch.device("cpu") 

device = torch.device("cuda") # NVIDA 顯卡

device = torch.device("mps") # Apple M 晶片
```

實際在專案上，我們會**對其設備進行檢查**，而不是直接指派到 GPU（如要是指定 cuda，M 系列晶片設備則無法使用）

```Python
import torch

if torch.cuda.is_available(): # 檢查有無 cuda
    device = torch.device("cuda")
elif torch.backends.mps.is_available(): # 檢查有無 M 晶片
    device = torch.device("mps")
else: # 都沒有則跑在 cpu
    device = torch.device("cpu")

t = torch.tensor([1, 2, 3])
t = t.to(device)
print(t.device) # 檢查結果跑在哪
```

但這樣要對每個 Tensor 獨立設定，以下方式改進

```Python
import torch

# 1. 直接設定全局預設設備為 mps
torch.set_default_device('mps')

# 2. 接下來建立的所有 Tensor，都會「自動」出生在 MPS 上，不用再寫 .to()
a = torch.tensor([1.0, 2.0])
b = torch.randn(3, 3)

print(a.device)  # 輸出會直接是 mps:0
print(b.device)  # 輸出會直接是 mps:0 --> 0 表示跑在第一張晶片上面
```

```Python
import torch
import torch.nn as nn

device = torch.device("mps" if torch.backends.mps.is_available() else "cpu")

# 這是一個包含大量隱藏層與 Tensor 的模型
class MyModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.layer1 = nn.Linear(100, 50) # 這裡面有超多 Tensor
        self.layer2 = nn.Linear(50, 10)  # 這裡面也有超多 Tensor

model = MyModel()

# 只要這一行！模型裡面所有的層級、成千上萬的權重 Tensor 就會一起搬到 GPU
model.to(device)
```

而若我們無中生有的生成張量，或是由串練轉換，除了第一個都需要注記統一，避免跑在不同運算上報錯

```Python
# 前面已經有見了一個 x 不需要標注
new_zeros = torch.zeros(10, device=x.device) 
new_tensor = torch.tensor([1, 2, 3], device=x.device)
```

---

## 型別轉換

現代大模型使用 **float16 或 bfloat16**（FP16/BF16半精度浮點數）\-\-\> 16 bit，2 位元

而傳統深度學習（如 PyTorch）使用** float32**（FP32單精度浮點數）\-\-\> 32 bit，4 位元

> 而 tokenizer 的結果會是整數（在 PyTorch 為 torch\.long）

運算時需用 PyTroch 的函數進行轉換（轉換的不是本體，而是一個副本）

在計算** softmax、Norm、Loss **時要將 **16 轉 32**，計算後再轉回

---

## \[ \] 索引操作

```Python
... = 目標張量[維度 1, 維度 2, ...]

... = 目標張量[維度 1] # 如果有多個維度，只寫了一個代表只有一維有限定取什麼，其他維度全拿
```

```Python
... = 目標張量[拿取索引]
```

```Python
... = x[[True, False, False, True]]
```

```Python
x[[True, False, False, True]] = ...
```

---

## \_ 原地覆蓋

寫在函數名後方，對於已經建立好的張量進行操作時，不需另外請求記憶體，而是直接付蓋

例如：

```Python
... scatter_add_
... init.kaiming_uniform_(...)
```

---

## 運算

### 乘法

#### \*、\.mul\(\) 元素級乘法 

**對應位子做乘法**，有**廣播**機制
後面可加上底線，直接在原張量記憶體中覆蓋

\.mul\(\) 用法是前面的張量直接跟括號內的做運算

---

#### @ 、 \.matmul\(\) 矩陣乘法

正常的**矩陣乘法**計算，可底線

---

### /、\.div 逐元素除法

可底線

---

### 降維運算

若只是簡單的運算可以直接使用符號運算，若是涉及維度、複雜數學、統計（平均、變異數）則使用函數

運算後該欄位會被壓扁，用 **keepdim=True 保留原有維度**，否則會**將原有維度 \-1**

---

#### 運算時的維度選擇

例如，如在 MoE 中要把多位專家計算出來的權重進行加權計算

```Python
y = (y.view(batch_size * seq_len, -1, hidden_size) * top_k_w.unsqueeze(-1)).sum(dim=1)
```

做簡單的假設，原始 y 的形狀為

```Python
[
    # ===== 第 0 個 Token =====
    [ # 對 dim=1 操作在這
        [1,  2,  3],   # 專家 A 的意見 (dim=1, index=0) 
        [10, 20, 30]   # 專家 B 的意見 (dim=1, index=1)，[] 裡面才是 dim=2
    ], 
    
    # ===== 第 1 個 Token =====
    [
        [4,  5,  6],   # 專家 A 的意見 (dim=1, index=0)
        [40, 50, 60]   # 專家 B 的意見 (dim=1, index=1)
    ]
]
```

在進行 \.sim\(\) 前，維度為 \(batch\_size \* seq\_len, 專家數, hidden\_size\) 

直覺上覺得我們要對特徵（hidden\_size 這個維度的元素）進行加權，我們應該選擇 \.sum\(dim=2, keepdim=True\)

但實際上會變成：

```Python
[
    # ===== 第 0 個 Token =====
    [
        [ 6],   # 專家 A: (1 + 2 + 3) -> 特徵被壓扁成一個數字
        [60]    # 專家 B: (10 + 20 + 30) -> 特徵被壓扁成一個數字
    ],

    # ===== 第 1 個 Token =====
    [
        [ 15],  # 專家 A: (4 + 5 + 6)
        [150]   # 專家 B: (40 + 50 + 60)
    ]
]
```

正確做法是 \.sum\(dim=1\)：

```Python
[
    # 第 0 個 Token (專家 A 和 B 互相對齊相加)
    [11, 22, 33],  # [1+10, 2+20, 3+30]

    # 第 1 個 Token (專家 A 和 B 互相對齊相加)
    [44, 55, 66]   # [4+40, 5+50, 6+60]
]
```

---

#### torch\.sum\(\) 加

沒指定維度則對張量內所有元素加總

#### torch\.mean\(\) 平均

```Python
import torch

x = torch.tensor([[1.0, 2.0, 3.0],
                  [4.0, 5.0, 6.0]]) # 形狀為 (2, 3)

# 情況一：完全不指定 dim（全體總平均）
print(torch.mean(x)) 
# 輸出：tensor(3.5000)  <- (1+2+3+4+5+6)/6

# 情況二：dim=0（垂直向下壓扁）
# 消除「列」，把每一行跨越不同列的數字拿來算平均
print(torch.mean(x, dim=0)) 
# 輸出：tensor([2.5000, 3.5000, 4.5000]) <- 形狀變成 (3,)，也就是 [(1+4)/2, (2+5)/2, (3+6)/2]

# 情況三：dim=1（水平向右壓扁）
# 消除「行」，把每一列裡面的所有行拿來算平均
print(torch.mean(x, dim=1)) 
# 輸出：tensor([2.0000, 5.0000]) <- 形狀變成 (2,)，也就是 [(1+2+3)/3, (4+5+6)/3]
```

---



#### torch\.max\(\) 、 torch\.min\(\) 最大最小值

#### torch\.argmax\(\) 最大索引值

---

### torch\.sqrt 開根號

---

### torch\.rsqur\(\) 倒數開根號

**Reciprocal Square Root**（倒數平方根）

效能較 1 / torch\.sqrt\(\) 好

```Python
import torch

# 準備一個 Tensor，內容是 4, 16, 100
x = torch.tensor([4.0, 16.0, 100.0])

# 使用 rsqrt 計算
y = torch.rsqrt(x)

print(y)
# 輸出: tensor([0.5000, 0.2500, 0.1000])
# 驗證：1/√4 = 1/2 = 0.5
#      1/√16 = 1/4 = 0.25
#      1/√100 = 1/10 = 0.1
```

---

### torch\.outer\(\) 一維外積

兩個一維陣列，長度為 N 和 M，產生 **N, M **的二維陣列

和高中的**向量外積（叉積，Cross Product）為不同概念**

```Python
import torch

# u 是一個長度為 3 的 1D 張量
u = torch.tensor([1, 2, 3])

# v 是一個長度為 4 的 1D 張量
v = torch.tensor([10, 20, 30, 40])

out = torch.outer(u, v)
print(out)
# 輸出：
# tensor([[ 10,  20,  30,  40],  # 1 * [10, 20, 30, 40]
#         [ 20,  40,  60,  80],  # 2 * [10, 20, 30, 40]
#         [ 30,  60,  90, 120]]) # 3 * [10, 20, 30, 40]
```

---

## torch\.where\(\) 替換

```Python
torch.where(條件, x, y)
```

條件**成立時就從 x 拿數字**，反之則從 y

```Python
x = torch.tensor([-12, 2, -3, -45, 8])

condition = x > 0

print(torch.where(condition, x, torch.tensor(0))) # ([0, 2, 0, 0, 8])
```

如果只放條件，則輸出條件的索引

```Python
x = torch.tensor([-12, 2, -3, -45, 8])

print(x > 0) # ([1, 4])
```

---

## torch\.clamp\(\) 限制範圍

將張量內的所有元素強制限制在你指定的數值之間

```Python
import torch

x = torch.tensor([-5, 5, 15])

# 直接依序傳入 0 (min) 與 10 (max)
out = torch.clamp(x, 0, 10) 

# 結果與 torch.clamp(x, min=0, max=10) 完全相同
# 輸出：tensor([ 0,  5, 10])
```

---

## torch\.arange\(\) 生成等差數列

生成等差數列，且**包頭不包尾**

```Python
torch.arange(起點, 終點, 步長)

torch,arange(終點) # 起點 0，步長 1
```

---

## \.scatter\_\(\) 指定索引覆蓋

```Python
目標張量..scatter_(目標維度, 分發索引, 數據源)
# 用底線直接覆蓋
```

- **分發來源**為一**索引值**張量，對應索引為**來源數據分別覆蓋目標張量的哪個索引**

- 此**分發索引**的維度必須跟**數據源張量形狀相同**

```Python
import torch

# 1. 準備一個長度為 5 的目標張量（裡面原本有些舊資料）
target = torch.tensor([1.0, 2.0, 3.0, 4.0, 5.0])

# 2. 準備來源數據
src = torch.tensor([99.0, 88.0, 77.0])

# 3. 指定分發位置
index = torch.tensor([0, 4, 2])

# 4. 執行蓋印章
target.scatter_(0, index, src)

print(target)
# 輸出：tensor([99.0,  2.0, 77.0,  4.0, 88.0])
```

---

### \.scatter\_add\_\(\) 指定索引加法

原理相同，但不是覆蓋，而是兩張量做**加法**

---

### \.scatter\_reduce\_\(\)  指定索引運算

- **include\_self**：**原始資料是否保留**（做**加法、乘法運算需保留**，否則只是**單純的覆蓋**，沒做到運算）

```Python
target.scatter_reduce_(目標維度, 分發索引, 數據源, reduc="運算方式", include_self=True)
```

---

## 廣播機制 Broadcasting

允許不同形狀的當量進行**算術運算（加減乘除）**，自動對**小的進行擴張**

從**最後一個**維度開始比對（向右對齊），兩個張量**必須在該維度大小一致**（元素數量同），否則其中一個**大小要為 1**

所以有時會**因張量沒有對齊而無法運算**

以下兩種解法（等效）：

### unsqueeze\(\) 升維 

將**指定索引維度新增一個元素**

```Python
# 假設我們有特徵 A 和權重 B
A = torch.randn(5, 10) # 5 個樣本，每個樣本 10 個特徵
B = torch.randn(5)     # 想給每個樣本乘上不同的權重

# 直接 A * B 會報錯，因為 (5, 10) 和 (5,) 無法從右對齊廣播（10 和 5 對不上）
# 我們需要把 B 的形狀改成 (5, 1)

# 方法一：使用 unsqueeze
result1 = A * B.unsqueeze(1)
```

```Python
# 方法二：使用 None (更直覺、Pythonic)
result2 = A * B[:, None] 

print(result1.shape) # 輸出: torch.Size([5, 10])
```

---

## squeeze\(\) 降維

把**只有一個元素的維度去除**，達成降維

- 也可用** dim= **來指定維度

- 若指定張量**沒有元素數量為 1 的維度**，直接跳過動作（**不會報錯**）

```Python
# 形狀是 [1, 5, 1, 4]
x = torch.zeros(1, 5, 1, 4)

y = x.squeeze() # 變為 [5, 4]
```

---

## 複製

### clone\(\) 整份複製

把**整份張量**完整做複製

```Python
... = xxx.clone
```

---

### \.repeat\(\) 分維度複製

針對**指定維度**去做複製

```Python
x = torch.tensor([1, 2])

result_1 = x.repeat(3) # [1, 2, 1, 2, 1, 2] 整組完整進行複製

result_2 = x.repeat(3, 2) # 增加維度
# [[1, 2, 1, 2],
# [1, 2, 1, 2],
# [1, 2, 1, 2]]
```

---

#### torch\.repeat\_interleave 同位子重複

直接在元素後方重複

```Python
torch.repeat_interleave(原張量, repeats=次數, dim=在哪個維度進行，預設 0)
```

```Python
x = torch.tensor([1, 2, 3])

# 每個元素都連續重複 3 次
result = torch.repeat_interleave(x, repeats=3)
# [1, 1, 1, 2, 2, 2, 3, 3, 3]
```

```Python
x = torch.tensor([10, 20, 30])
# 定義一個分配清單：第一個元素重複 1 次，第二個 3 次，第三個 0 次(直接消失)
counts = torch.tensor([1, 3, 0]) # 次數清單

result = torch.repeat_interleave(x, repeats=counts)

print(result)
# 輸出：tensor([10, 20, 20, 20])
# 10 出現 1 次，20 出現 3 次，30 消失了！
```

---

## 排序

### torch\.sort\(\) 排序

---

### torch\.argsort\(\) 索引排序

回傳元素排序後的索引

- 一次只能對一個維度的元素進行排序

- descending=True

```Python
torch.argsort(x, dim=排序維度, descending=True) # 預設 false，升冪排序
```

```Python
x = torch.tensor([80, 20, 50])

result = torch.argsort(x) # [1, 2, 0]
```

---

## 改變張量形狀

### 連續與不連續變形

簡待判斷要使用何種：

- 新建立、只做過數學運算的用 view\(\) \-\-\> 節省效能

- 維度被扭曲（如 \.transpose\(\) 轉置）用 reshape\(\)

#### view\(\) 連續型變形

改變形狀，但**保持元素數量一致**

- **\-1 表示自動計算剩餘**，直接幫你填寫（只能有一個 \-1）

> $列\times欄=列'\times欄'$

```Python
預變張量.view(列, 欄)

預變張量.view(列, -1) # 限制要幾列

預變張量.view(-1, 欄) # 限制要幾欄
```

view\(\) 為了追求極致的速度，它要求張量**在記憶體中必須是連續的**（Contiguous）。如果你之前對張量做過轉置（如 \.T 或 \.transpose\(\)），**資料在記憶體中的順序會被錯開**，**倒置 view\(\) 報錯**

> 解法：
> 
> 1. 寫成 **x\.contiguous\(\)\.view\(\.\.\.\)**，先把張量變連續
> 
> 2. **reshape\(\)**

---

#### reshape\(\) 不連續型變形

和 \.view\(\) 相同，差在在可接受不連續的記憶體

> 但很多資深的 PyTorch 工程師依然偏愛寫 view\(\)，原因在於：view\(\) 可以當作程式碼的「安全檢查哨」
> 因為 view\(\) 嚴格禁止複製記憶體，如果你在程式碼中寫 view\(\) 且順利執行，你就 100% 確定此處沒有發生任何記憶體複製，效能絕對是最高的 
> 
> 相反地，如果你完全依賴 reshape\(\)，它可能會在你沒注意到的地方（例如某個迴圈裡）默默地一直複製大張量，累積起來就會導致模型訓練變慢，甚至造成記憶體不足（OOM）的隱形 Bug 

---

### expand\(\) 拉伸

將大小為 **1**（維度哪只有一個元素）的維度**拉伸至指定尺寸**

拉升的方式是**複製該維度內原有的元素**

- **\-1 表不做變更**

```Python
預拉伸張量.expand(一維拉伸大小, 二維, ...)

預拉伸張量.expand(-1,-1, ...) # -1 表不做變更
```

---

### torch\.transpose\(\) 張量轉置

矩陣進行乘法運算（@、 \.matmul）時，只對最後兩個維度進行運算，須將要運算的部分轉置到後方維度

```Python
... = torch.transpose(預轉置張量, 維度 1, 維度 2) # 將維度 1 和 2 交換
# 維度用索引表示
```

```Python
x = torch.tensor([[1, 2, 3],
                  [4, 5, 6]]) # 形狀為 (2, 3)
                  
print(torch.treanspose(x, 0, 1)) # 變成 (3,2)
# ([[1, 4],
# [2, 5],
# [3, 6]])
```

---

### torch\.cat\(\) 拼接

concatenate 的縮寫 

```Python
... = torch.cat([預拼接張量, 預拼接張量], dim = 拼接維度)
```

```Python
import torch

# 建立兩個 3D 張量
A = torch.randn(2, 3, 4)
B = torch.randn(2, 3, 5)

# 沿著第三維 (dim=2) 拼接
out = torch.cat([A, B], dim=2)

print("拼接後的形狀:", out.shape)
# 輸出：torch.Size([2, 3, 9])
```

---

## 統計

### torch\.bincount\(\) 數量統計

統計**非負整數出現的次數**，按照**索引排序輸出次數**（輸出的索引代表當前統計的數字）

輸出的是排序後張量的此處應該是原張量的第幾索引

- 只接受一維張量

```Python
torch.bincount(輸入張量（限定一維）, minlength=總共統計數量)
```

```Python
torch.bincount(torch.tensor([50, 10, 30]))
# [1, 2, 0]
```

---

numel\(\) 元素總量

Number of Elements，統計所有元素總和，常用於**參數量計算**

```Python
... = xxx.numel()
```

```Python
total_params
```

---

## torch\.triu\(\) 掩碼

以**對角線進行切割並對其於部分做掩碼**，預設覆蓋為 **0**，常會設為**負無限大**（float\("\-inf"\) ）

- diagonal 設定**對角線位移**

    = 0 保留對角線及右上元素，大於 0 往右上位移，小於則往左下

- 只對**最後兩維作用 **

```Python
torch.triu(預掩碼張量, diagonal=0, 輸出張量)
```

```Python
x = torch.arange(1, 17).view(4, 4)
# tensor([[ 1,  2,  3,  4],
#         [ 5,  6,  7,  8],
#         [ 9, 10, 11, 12],
#         [13, 14, 15, 16]])
print(torch.triu(x, diagonal=0))
# tensor([[ 1,  2,  3,  4],
#         [ 0,  6,  7,  8],
#         [ 0,  0, 11, 12],
#         [ 0,  0,  0, 16]])
```

- 自己

    **diagonal=1** 時：

---

## torch\.topk 選前 k

- 取出的是兩個二維張量維度為 \(總 token 數, k\)

- sorted=True 會花額外的時間去做排序，設為 false 則會隨機排序，節省效能

```Python
前 k 的數值, 前 k 的索引 = torch.topk(
    輸入張量,
    選 k 個,
    dim=-1, # 所選維度
    largest=True, # 預設選最大的
    sorted=True # 預設從大排到小
)
```

---

## F\.one\_hot\(\) 獨熱編碼

將一維張量變為一個分類邊編碼

兩個作用：

1. 把類別標籤（如名稱、編號）等轉換成完全公平、互不干擾的數字

2. 作為遮罩（如 MoE）

```Python
F.one_hot(目標索引張量, num_classes=類別數量)
```

---

## torch\.compile 編譯加速

原本在計算時是看到一行運算，再把需要的元素從記憶體搬出，算完再放回去

而加速後會在模型開始運行前，先看完一次整個神經網路進行算自融合（等等要用的東西先不要放回去）

用這樣的方式來加快計算速度，但目前**不支援 mac 上的運行**（只對 NVIDIA 支援）

```Python
加速後的模型物件 = torch.compile(原模型)
```

---

## item\(\) 取出

以下作用：

1. 把值**轉為 float（FP64）**

2. 回到** CPU** 進行

3. **切斷和 grad\_fn** 的連結

- 且只能對**總元素量為 1 **的張量作用

---

# nn 神經網路

Neural Network，

裡面有**模型基底、Layer、激活函數、Loss 函數**等功能

在 PyTorch 裡，**所有**的神經網路模型，底層都是**繼承自 nn\.Module 這個類別，**

當類別裡面有**包含需要被 AI 訓練或更新的權重參數**，就需要繼承

```Python
class ...(nn.Module):
    ...
```

---

## forward\(\) 前向傳播函數

前向傳播就是資料從輸入層進入，**經過中間許多隱藏層的權重（Weights）與偏差（Biases）加權計算**，最後在輸出層得到一個預測結果的過程

而所有自定義的神經網路**類別**（只要是**繼承自 nn\.Module**），都**必須**強制實作 forward 函數

當繼承 **nn\.Module **的類別在** __init__ 裡被呼叫時**，會進行被呼叫類別裡的** __init__ 函數**（進行**初始化**）

會用一個物件去接收他

```Python
...
def __init__(self, ...):
    self.gate = MoEGate(...) # 執行 MoEGate 類別裡的 __init__
```

**在 forward 函數裡**會去呼叫剛才**在 \_\_init\_\_ 裡建立的物件**，會進行被呼叫類別的** forward 函數**

```Python
...
def forward(...):
    ... = self.gate(...) # 執行 MoEGate 類別裡的 forward 函數
```

而**直接呼叫類別會進行 \_\_init\_\_**，**建立好的類別物件**才會執行 **forward** 函數

```Python
model = MeowMindForCausalLM(lm_config) # 執行 __init__

output = model(...) # 執行 forward
```

---

## 梯度

### backward\(\) 反向傳播求導引擎

從輸出層「由後往前」計算，找出每一個神經元之間的權重，分別對最終的誤差貢獻了多少

只要是 **requires\_grad=True** 的張量或和為此狀態的張量做過運算，都會有 **grad\_fn **屬性，記錄了是經過神麼運算產生的

而 **損失\.backward\(\)** 順著每個張量的 **grad\_fn 從尾部開始向頭部走**，並利用**連鎖率求出梯度**，並**放入每個參數的 \.grad 屬性**

```Python
損失.backward()
```

---

#### requires\_grad 梯度

每個**神經網路內**的張量（權重和偏置）都會自帶此參數並設為 **Ture**（非神經網路內的張量也可自行設定此參數）

而這樣的張量會自帶 **\.grad 屬性**，一開始為 **None**，只要執行 **\.backward\(\) 算出的梯度**就會放於此屬性

而此張量**跟他人運算產生新張量**，**新張量的梯度也會被開啟**（requires\_grad=Ture），他的 **\.grad\_fn **屬性會記錄是**透過什麼運算生出來的**（用於反向傳播）

#### 關閉梯度

```Python
x = torch.tensor([1.0, 2.0]) # 一開始是 False
x.requires_grad_(True)       # 注意有底線！直接把 x 的開關掰到 True
```

```Python
x = torch.tensor([1.0, 2.0], requires_grad=True) # 正在錄影

y = x.detach() # y 拿到了 x 的數值，但它的 requires_grad 自動變成了 False！
```

---

#### @torch\.no\_grad\(\) 關閉函數梯度

當模型在推理、驗證階段，關閉梯度紀錄可節省記憶體，大幅提升速度

> 訓練時，GPU 為了記住怎麼倒回去算梯度，必須把每一層的中間特徵死死存在顯存裡

```Python
@torch.no_grad
def ...(...):
    ...
```

---

#### 關於梯度累積

**每次都先除以梯度累積次數**，然後**反向傳播進到 \.grad 被累加**，最後**累積次數達到設定的梯度累積次數才開始做參數更新**，以達到**模仿大批次訓練**的效果

> 假設累積次數為 3 \-\> **\(第一次 loss\) / 3 \+ \(第二次 loss\) / 3 \+ \(第三次 loss\) **/ 3，效果**等同三次 loss 先加起來再除 3**

```Python
for ...: # 一輪迴圈讀取一批次內容
    ......
    
    loss = loss / 梯度累積次數
    
    ...
    
    loss.backward() # 把 loss 轉梯度並存進 .grad
    
    ...
    
    if 目前步數（進行到第幾個批次） % 梯度累積次數 == 0 or step == 本輪總訓練批次:
        
        進行梯度更新
```

---

### torch\.amp\.autocast 混合精度計算

> Automatic Mixed Precision \-\> 自動 混合 精度
> 
> Auto Cast \-\> 自動 轉型
> 
> Ctx \-\> context（上下文管理器）

模型在做矩陣乘法（Linear 層、Attention 層）時，其實不需要那麼高的精準度，用 **FP16** 或 **BF16**（16位元）就綽綽有餘了，但有些關鍵計算（如損失值 Loss、Softmax 歸一化）如果精度太低會直接崩潰

他會**自動判斷此運算是否可以降級成 FP16/BF16 來執行**（從 **FP 32 降**來省效能），不行降級的計算則會被升回 FP32

> BF16 支援數值範圍較 FP 16 大

要先**建立物件**，然後**搭配 with 來使用**（因為為上下文管理器）

```Python
autocast_ctx = torch.amp.autocast(device_type="mps", dtype=torch.float16)

with autocast_ctx:
    ...
```

在**縮排內進行的運算，包含調用函數的計算，都會進行自動的轉型判斷**

而若無法運行於 mps 或 cuda 則無法使用混合精度

一般來說會做判斷是否有 mps 或 cuda 可用，若無才不用，所以需要設定條件不成立時物件為 nullcontext\(\)，才不會在 with 呼叫時出現問題

```Python
if torch.cuda.is_available():
    autocast_ctx = torch.amp.autocast(device_type="cuda", dtype=torch.float16)
elif torch.mps.is_available():
    autocast_ctx = torch.amp.autocast(device_type="mps", dtype=torch.float16)
else:
    autocast_ctx = nullcontext()
    
with autocast_ctx:
    ...
```

---

### torch\.amp\.GradScaler 梯度縮放

若使用 **FP16 來進行混合精度的訓練**，**必須強制綁定梯度縮放**（BF16 可以忽略）

原因是反向傳播後得到的 Loss 往往非常小，由 FP32 轉 FP16 會有**下溢**（數字太接近 0）發生（最後會被四捨五入變 0），

梯度縮放的做法是將 Loss 在 FP32 時先**乘以一個很大的數字**（預設通常是 65536），**等要交給優化器更新權重時再除回來**

1. **建立梯度縮放物件**

    - **enabled 可設定條件判斷**，FP16 時為 True，BF16 為 False（**scaler 物件還是存在，只是下面的動作會變成沒有實質動作**）

    ```Python
    梯度縮放物件 = torch.amp.GradScaler(device="mps", enable=True) # 建立梯度縮放物件物件
    ```

2. 把** loss 放大**，並**進行反向傳播轉為梯度**

    - enabled 為 **False** 時，**梯度縮放物件\.scale\(loss\) 會還原為 loss 本身**

    ```Python
    梯度縮放物件.scale(loss).backward() # 把 loss 放大，並進行反向傳播（算梯度）
    ```

> - 爲**梯度裁剪**做準備：
> 
>     將被放大的梯度**先除回去**，否則梯度裁剪會失效（梯度被放這麼大，會被視為每步的梯度都爆炸了，每步都進行裁剪，導致錯誤學習）
> 
>     - **反向傳播後的階段**，不需保留紀錄，直接**加底線覆蓋**
> 
>     - 上面 \.**scale\(\) 反向傳播還沒結束**，不能直接覆蓋
> 
>     - 經過此步後， **梯度縮放物件\.step\(優化器物件\)**  動作則只剩**檢查 NaN 即觸發優化器的作用**
> 
>     ```Python
>     梯度縮放物件.unscale_(優化器物件)
>     
>     torch.nn.utils.clip_grad_norm_(模型物件.parameters(), max_norm=1.0) # 進行梯度裁剪
>     ```

3. 把**被放大的梯度除回去**，並**檢查是否有問題**（出現 **NaN**），沒問題則觸發 **優化器\.step\(\)** 來**更新參數**

    > **有問題則則拒絕參數更新**

    ```Python
    梯度縮放物件.step(優化器物件)
    ```

4. 根據上一步的檢查做縮放係數的調整

    > 如果連續 2000 步（預設）順利通過檢查（沒出現 NaN）\-\> 縮放係數翻倍
    > 
    > 如出現了 \-\> 縮放係數砍半

    ```Python
    梯度縮放物件.update()
    ```

---

### torch\.nn\.utils\.clip\_grad\_norm\_\(\) 梯度裁剪

設定一個梯度長度的最大值（max\_norm），當梯度長度超過限制時，對其做縮放

> **反向傳播後的階段**，不需保留紀錄，直接**加底線覆蓋**

```Python
torch.nn.utils.clip_grad_norm_(模型物件.parameters(), max_norm=1.0)
```

---

### Optimizer 優化器

模型透過像傳播得到**梯度**後，需要**用優化器對模型權重（參數）進行修改**

```Python
from torch import optim
```

#### optim\.AdamW 梯度優化

> **SGD（隨機梯度下降）**：像一個盲人下山，哪裡陡就往哪裡走。遇到複雜、崎嶇的「損失地貌（Loss Landscape）」時，極容易卡在死胡同（局部最佳解或鞍點）動彈不得
> 
> **Adam（自適應矩陣估計）**：大腦升級。它不僅會看當下的陡峭度，還會**記錄過去走過的路（動量 Momentum）**，並且會**針對不同的參數給予不同的步伐大小（自適應學習率），**那些經常劇烈變動的參數步伐會變小，冷門的參數步伐會變大
> 
> **AdamW（權重衰減解耦的 Adam）**：大模型的救星。在原本的 Adam 中，工程師為了防止模型死記硬背（過擬合 Overfitting），會加上一種叫「權重衰減（Weight Decay）」的懲罰機制。但科學家發現 Adam 在底層把這項懲罰與自適應學習率混在一起算，導致懲罰失效。**AdamW 把這兩者強行解耦（Decoupled），讓懲罰獨立運作，從此大模型在微調時變得極其穩定**

1. 建立優化器物件

    初始化一個優化器個體，並**把模型參數（ \.parameters\(\) ）綁定**

    ```Python
    優化器物件 = optim.AdamW(
        model.parameters(),     # 1. 必填：告訴優化器你要調整模型的哪些參數
        lr=1e-5,                # 2. 核心：學習率 (Learning Rate)，即每一步的步伐大小
        betas=(0.9, 0.999),     # 3. 進階：動量追蹤係數 (一階與二階)
        eps=1e-8,               # 4. 安全：防止分母為 0 的極小值
        weight_decay=0.01       # 5. 防過擬合：權重衰減的懲罰強度，越大代表發生過擬合懲罰月中
    )
    ```

    ```Python
    優化器物件 = optim.AdamW([
        {"params": regular_params, "weight_decay": 0.01}, # 字典 1：普通參數
        {"params": no_decay_params, "weight_decay": 0.0}  # 字典 2：偏置與歸一化層
    ], lr=1e-5)
    ```

2. **參數更新**

    依照 **\.grad** 屬性，對權重和偏置進行修改

    > 若進行了梯度縮，梯度縮放物件\.step\(優化器物件\) 會取代此步驟

    ```Python
    優化器物件.step()
    ```

3. **梯度歸零**

    **把 \.grad 屬性清空**，給下一次的梯度更新流出空間（預設變為 Noen）

    ```Python
    優化器物件.zero_grad()
    ```

---

#### param\_groups 參數組

為一個**串列**，裡面**裝著字典**

```Python
... = optimizer.param_groups
```

預設情況下（如剛被 optim\.AdamW 建立），串列內只有一個字典，此字典管轄了所有的參數，**全體共享學習率及權重衰減（weight\_decay）**

> ```Shell
> [
>    { 'params': [張量1, 張量2], 'lr': 0.0001, 'weight_decay': 0.01 },  <- param_group 0
>    { 'params': [張量3, 張量4], 'lr': 0.0001, 'weight_decay': 0.00 }   <- param_group 1
>  ]
> ```
> 
> 所以更改學習率要這樣：
> 
> ```Python
> for param_group in optimizer.param_groups:
>     param_group["lr"] = lr
> ```

若建立優化器時做了**分組**，每組就會是一個字典

> ```Python
> [
>     { 'params': [普通參數們...], 'lr': 1e-5, 'weight_decay': 0.01 },
>     { 'params': [偏置與歸一化層...], 'lr': 1e-5, 'weight_decay': 0.0 }
> ]
> ```

---

## 權重

### nn\.Parameter 標為權重

告訴模型此 tensor 是模型的權重，自動出現在 model\.parameters\(\) 的清單裡

```Python
def __init__(self, ...):
    self. ... = nn.Parameter(張量的建置...)
```

---

#### parameters\(\) 顯示權重

把這個神經網路模型裡，所有需**要參與訓練、計算梯度、被優化器更新的權重（W）和偏置（B）打包**

需要用**迴圈迭代**才能顯示出完整內容

```Python
# 建立一个極簡模型：一層線性層
model = nn.Linear(in_features=3, out_features=2)

# 清點所有參數
for param in model.parameters():
    print(param)
```

```Python
# 1. 權重矩陣 (Shape: [2, 3])
tensor([[ 0.2311, -0.4121,  0.0891],
        [-0.1012,  0.3142, -0.2211]], requires_grad=True)

# 2. 偏置向量 (Shape: [2])
tensor([0.1123, 0.5432], requires_grad=True)
```

- **requires\_grad=True** 表示梯度記錄器為開啟狀態，通常這時模型處於訓練階段

---

#### named\_parameters\(\) 名稱和權重

返回**元組 \(名稱, 權重張量\)**

---

### init\.kaiming\_uniform\(\)\_ 愷明均勻分佈 

從一個區間**均勻分布填入張量**

```Python
from torch.nn import init # 否則：torch.nn.init.kaiming_uniform_()

init.kaiming_uniform_(
    # 用底線直接覆蓋
    目標張量,
    a=0, # 激活函數接受的負斜率（浮點數，預設 0）
    mode="fan_in", # 前向傳播使用（預設），反向傳播設為 "fan_out"
    nonlinearity="leaky_relu", # 使用的激活函數，預設 leaky_relu，不代表真的會使用，沒用到可以不管（如 MoE 時）
    generator=None # 隨機亂數生產器，預設 None
)
```

- **Kaiming Uniform 凱明均勻分布**：

    愷明均勻分佈（Kaiming Uniform）會從一個對稱的均勻分佈區間 $[- \text{bound}, \text{bound}]$  中隨機抽取數字來填滿矩陣

    其計算 $\text{bound}$‬（邊界）的公式為：

    $\text{bound} = \sqrt{\frac{6}{(1 + a^2) \times \text{fan\_in}}}$， a 通常設為 $\sqrt5$
    其中，‭$\text{fan\_in}$ 代表的就是輸入維度 

---

### F\.linear 線性計算

進行  $xW^T + b$ 的計算

接受維度為 **\(任意, in\_features\)，\(out\_features, in\_features\)，\(out\_features\)**，故通常要先對張量降維

```Python
F.linear(w, x, bias=None) # 如果有偏差值 bias=b（張量）
```

---

### nn\.Linear\(\) 權重實作

對輸入矩陣進行線性轉換，本質上就是使用 nn\.Parameter

**隨機建立 w 和 b，並進入到前向傳播階段**（放進 $y = xW^T + b$ 輸出 y），存放在 \.weight 屬性

- **in\_features：**輸入維度

- **out\_features：**輸出維度

- **bisas：**有無偏差值（預設為 True）

```Python
... = nn.Linear(
    in_features=輸入維度,
    out_features=輸出維度,
    bias=True
) # 在這裡 w 和 b 就建立好了

... = linear_layer(x)

print(linear_layer.weight.data) # 檢查 w
print(linear_layer.bias.data) # 檢查 b
```

> 【輸入矩陣】 形狀是 \(3, 4\)
> 
> \(有 3 個 Token，每個長度是 4\)
> 
> 「我」: \[ 0\.1,  0\.2,  0\.3,  0\.4 \]
> 
> 「愛」: \[ 0\.5,  0\.6,  0\.7,  0\.8 \]
> 
> 「AI」: \[ 0\.9,  1\.0,  1\.1,  1\.2 \]
> 
> ---
> 
> ▼ 經過 nn\.Linear\(4, 2\) 轉換
> 
> ---
> 
> 【輸出矩陣】 形狀變成 \(3, 2\)
> 
> \(依然是 3 個 Token，但每個長度縮減成 2\)
> 
> 「我」: \[ 2\.5, \-1\.2 \]
> 
> 「愛」: \[ 4\.1,  0\.3 \]
> 
> 「AI」: \[ 5\.8,  1\.9 \]

---

#### 如何選擇

**第一步：問自己「這是不是標準的特徵變換層？」**
深度學習中 90% 的場景都是標準的特徵維度轉換（例如從 512 維映射到 256 維）

- **如果是** 

$\rightarrow$ 毫不猶豫選擇 **nn\.Linear**
**原因**：它是神經網路的「標準積木」。PyTorch 官方對 nn\.Linear 的前向傳播在 CPU/GPU 軟硬體層面都做了極致優化，且內建了科學的權重初始化（Kaiming / Uniform 初始化），你不需要手動去搞亂數

- **如果不是** 

$\rightarrow$ 進入第二步。
**第二步：問自己「後續有沒有複雜的 Tensor 操作（如 Scatter / Top\-K）？」**
這就回到了我們剛剛討論的 MoEGate 或是圖神經網路（GNN）的場景

- **如果是** 

$\rightarrow$ 強烈建議選擇 **nn\.Parameter**
•        **原因**：當這塊權重需要頻繁與 index、scatter\_reduce\_ 進行形狀對齊時，nn\.Linear 肚子裡反向儲存的 \(out\_features, in\_features\) 權重形狀會逼你寫出大量的 \.t\(\) 或 \.view\(\)，這極易引發 Bug。直接用 nn\.Parameter 自訂形狀，程式碼最直覺。
**第三步：考慮「多卡分散式訓練」與「底層優化」**

- 如果你的模型需要做**大模型切片（Tensor Parallelism / Expert Parallelism）**，或者需要對權重套用自訂的 **Triton / CUDA Kernel** 

$\rightarrow$ 選擇 **nn\.Parameter**。因為它赤裸裸的 Tensor 屬性給了分散式框架最大的操作自由度

---

## 模型物件

不同部分的組件會被分散在不同類別，並繼承 nn\.module，

最後會有一個類別做出統整，對於因果語言模型來說通常叫做：模型名ForCausalLM，他同樣繼承 nn\.module

而此類別最終會成為呼叫出模型物件的類別

被呼叫出來的模型物件擁有以下屬性／方法：

---

### 模式

分為**訓練模式和評估模式**

> 現代方案中 BatchNorm 由 **RmsNorm** 取代，且評估模式不會被鎖定

```Python
模型物件.train()

模型物件.eval()
```

---

## 層

### nn\.ModuleList 多層打包

把**多層神經網路放入串列**（利用串列生成式迭代創建）**打包**

接受的參數必為**繼承自 nn\.Module 的類別**，也可為空（**只先建立容器**）

```Python
... = nn.ModuleList(
    [構建層的類別() for n in range(層數)] # 串列生成式
)
```

---

### nn\.Dropout\(\) 正則化

- 防止過擬合 

- 要求為**浮點數**

- **僅在訓練模式（\.train\(\)，預設）有作用**，評估模式（\.eval\(\) ）不受影響

```Python
dropout_layer = nn.Dropout(p=0.5) # 建立一個物件，並設定機率
# 有 50% 機率元素會歸零
x = torch.tensor([1.0, 2.0, 3.0, 4.0, 5.0, 6.0])

dropout_layer.train() # 訓練模式，預設
print(dropout_layer(x)) # [ 0.,  4.,  6.,  0., 10.,  0.]
# ----------------------------------------------------------
dropout_layer.eval() # 評估模式
print(dropout_layer(x)) # [1., 2., 3., 4., 5., 6.] 沒反應
```

---

### nn\.act\_name / F\.act\_anme 激活函數調用

```Python
class ...(nn.Module):
    def __init__(self, args):
        ...
        
    def forward(self, x):
        ...
        return ...F.ReLU(x)... 
```

```Python
class ...(nn.Module):
    def __init__(self, args):
        ...
        self.act_fn = nn.ReLU()
    
    def forward(self, x):
        ...
        return ...self.act_fn(x)... 
```

```Python
class ...(nn.Module):
    def __init__(self, args):
        ...
        self.act_fn = ACT2FN[args.hidden_act] # 在字典中得到 nn.ReLU() 這個函數
    
    def forward(self, x):
        ...
        return ...self.act_fn(x)... 
```

---

## Attention

### F\.scaled\_dot\_product\_attention 快注意力計算

FlashAttention 的計算函數

使用前先檢查：

1. 是否為**支援架構**（Nvidia 顯卡等）

2. 資料型態是否為** float16 或 bfloat16**

3. 是否啟用 **dropout** 或**特殊遮罩**

若不符合條件不會直接報錯，而是直接**降級成其他的加速方案**

```Python
... = F.scaled_dot_product_attention(
    q, k, v,
    attn_mask=None, # 或其他遮罩
    dropout_p=00, # 正則化
    is_causal=True # 自動實作因果遮罩
)
```

- **因果遮罩**讓訓練時模型可以**從頭一字一字的看**，**看後面內容時前面也可看到**

- 函數包含計算 $\frac{Q \times K^T}{\sqrt{d_k}
‬}$，因果遮罩實作，softmax 套用，以及最後乘上 V

---

### F\.softmax 轉為機率分布

將張量進行 softmax 轉換為機率分布

```Python
... = F.softmax(
    輸入張量,
    dim=-1, # 對哪一維度進行操作
    dtype=None # 指定輸出型別，預設為 None，表照舊
)
```

---

### F\.cross\_entropy 交叉熵損失

用來衡量**模型預測的機率分布和真實答案的差**，也就是** Loss**

- 函數內**包含 softmax 過程**，不需自行實作

- 大部分情況預測值維度為**（總 token, 詞彙數）**答案為 **（總 token），回傳零維的單一數**

    > 也支援更高維度的，例如圖像處理時

- **ignore\_index** 出現在**正確答案**裡

```Python
... = F.cross_entropy(
    模型預測值 logits,
    正確答案 labels
    ignore_index=-100 # 
)
```

---

## 狀態

### register\_buffer 註冊工具張量

工具張量（buffer）指的是必須要進到 GPU 計算，但不需要去做學習、微調的張量，

也就是能被計算出一個固定、不需修改的值，例如 RoPE 的正餘弦張量

> RoPE 的正餘弦張量被註冊為工具張量最主要是為了可以跟隨模型到 GPU 中，他本身就不被計算梯度

當備注冊為 buffer 後，此張量會：

1. 隨模型被一起打包、存檔

2. 不計算梯度

3. 不被優化器更新

```Python
self.register_buffer("buffer 名", 目標張量, persistent=False)
```

- persistent：是否要被存放進權重檔

    > RoPE 通常會每次都計算，不做存檔來浪費空間

---

## nn\.Embedding\(\) token 轉換

用於**查表把 token 轉換為向量**

- 輸入維度為 \(2, 3\)，輸出變為 \(2, 3, **embedding\_size**\)，前面不變，最後多一個 **embedding\_size **大小的維度

```Python
... = nn.Embedding(
    num_embeddings=6400, # 詞彙量
    embedding_dim=512, # 特徵維度大小（hidden_size）
    padding_idx=None, # 選填，填充來對其，接受 int
    max_norm=None, # 選填，最大值限制，接受 float
    scale_grad_by_freq=False, # 選填，**依字頻縮放梯度**
    sparse=False # 選填，**稀疏梯度開關**
)
```

---

# 資料採集

1. 由採樣器物件決定順序

2. 由批次採樣器打包成批次

3. 由載入器取出資料（依照批次採樣器的打包順序）

---

## 自建採樣器或批次採樣器物件

採樣器決定一個資料集中讀取的順序，達成**對資料集的抽籤**

```Python
from torch.utils.data import Sampler
```

要**對抽籤動作進行修改的類別**需要**繼承自 Sampler**

並且要實作以下函數：

1. **\_\_iter\_\_\(\)：**決定**如何抽籤**

2. **\_\_len\_\_\(\)：**決定**還有多少籤**可以抽抽

---

---

## 採樣器

負責決定**資料的訓練順序**，做到**每個 epoch 不同**

為了防止模型**死背**資料出現的順序，認為什麼資料後面一定就要出現什麼

---

### 對於單卡的採樣器

#### RandomSample\(\) 隨機生成順序

- **無法斷點續訓**

    > 因為沒辦法設定滾動式（隨 epoch 調整）的種子碼

---

#### torch\.randperm\(\) 隨機生成順序

> permutation \-\> 排列

給定數量，會把 0 到 n\-1 （當**索引**用）打亂打包成**一維張量**

- 搭配 **torch\.manual\_seed\(\) 設定全局隨機碼**或使用 generator 來設定

- **支持斷點續訓**

```Python
... = torch.randperm(數量, generator=None)
```

將生成出來的順序**再轉為串列（\.tolist\(\) ）**作為批次採樣器的順序來源

> 不直接隨機生成串列，因為 torch\.randperm **速度較快**

---

### DistributedSample** **建立分佈式採樣器迭代物件

把資料集平均分配給不同張卡做 **DDP**，本質上就是對數據集做**切片**

> 先用**隨機種子碼做隨機打亂**，在用 **\(rank: : world\_size\) 的方式切片給每一個顯卡**

得到的是一個**迭代物件**，在回圈中**一次得到一個索引**

```Python
分佈式採樣器迭代物件 = DistributedSample(
    資料集,
    num_replicas=總卡數,
    rank=當前誰被的全局卡數,
    shuffle=True, # -> 依賴 set_epoch() 手動在每一 epoch 更新隨機種子
    drop_last=False # 是否捨棄多餘
)
```

> **dist\.init\_process\_group\(\) 已建立**的情況下，**num\_replicas 和 rank 參數可以不填**

> **drop\_last** 除不盡的情況下（**資料無法平均分配**），若設為 True，則會把**多餘的捨棄**，反之則是去**複製重複的資料來舔補**

---

#### set\_epoch\(\) 更新隨機種子

在每一輪 epoch 前必須呼叫，來重設基礎種子

```Python
分佈式採樣器迭代物件.set_epoch(epoch )
```

更新方式為：

$隨機種子碼=基礎種子碼+當前 Epoch$

---

## 批次採樣器

### BatchSampler\(\) 打包成批次

根據“採樣器物件”給的索引順序，把資料依照 batch\_size **打包成串列**

```Python
批次串列 = BatchSampler(
    sampler=RandomSampler(train_ds), # 1. 底層抽樣器類別
    batch_size=32,                   # 2. 批次大小（負責數到 32 根就捆成一捆）
    drop_last=True                   # 3. 尾數處理（最後一捆不滿 32 根要不要丟掉）
)
```

---

### 自建斷點續訓的批次採樣器

取代了原本的 BatchSampler\(\)

1. **繼承自 Sampler（**要實作 \_\_iter\_\_\(\)、\_\_len\_\_\(\)）

- 在 \_\_iter\_\_\(\) 裡實作：

2. 傳入** torch\.randperm\(\)\.tolist\(\) 產出的串列或 DistributedSample\(\) 做原始批次採樣器**

3. 保證斷點後產生的**資料順序和原先一致**

    > 傳入的採樣器採用固定邏輯的隨機種子碼

4. 採樣器給出的**每一個索引都要被遍歷**，包含要被跳過的步驟

- 在 \_\_len\_\_ 裡實作：

5. **總步數的更改**

---

## DataLoader\(\) 載入器

- **batch\_size** 可替換為 **batch\_sampler **來搭配採樣器進行數據打包（**傳入採樣器物件**）

```Python
載入器物件 = DataLoader(
    dataset=my_dataset,       # 1. 必填！你的數據倉庫（繼承自 Dataset 的物件）
    batch_size=32,            # 2. 一批要有幾筆數據
    shuffle=False,            # 3. 要不要把數據亂數打亂（防模型死記順序）
    num_workers=4,            # 4. 僱用幾個司機（執行緒）同時幫忙搬數據
    pin_memory=True,          # 5. 特權通道：加速數據從記憶體（RAM）搬到顯存（VRAM）
    collate_fn=collate_fn,    # 6. 大模型靈魂：打包規則（如何把不同長度的句子對齊）
    batch_sampler=my_sampler, # 7. 多卡 DDP 標配：決定哪張卡分到哪些數據
    pin_memory=False          # 8. Pinned Memory，開啟時資料進到 GPU 不需經過 CPU
    drop_last=False           # 9. 是否捨棄
)
```

> **shuffle **設定**為 True 時**，會**自動實作 RandomSampler 和 BatchSampler**
> 
> ```Python
> self.sampler = RandomSampler(dataset, generator=generator)
> 
> self.sampler = RandomSampler(dataset, generator=generator)
> ```
> 
> **而在 DDP 時不可使用**，會導致讀取到重複的數據
> 
> 因為自動實作的 **RandomSampler** 會和自建的 **DistributedSample 衝突**

> **drop\_last** 除不盡的情況下（**資料無法平均分配**），若設為 True，則會把**多餘的捨棄**，反之則是去**複製重複的資料來舔補**

- 載入器物件以**迭代器的方式取出一批次資料**

- 對載入器物件** len\(\) 可得到批次大小**

---

# 儲存

## torch\.save\(\) 儲存檔案

儲存的檔案本質上就是**字典**，擁有**鍵與值**

```Python
torch.save(
   存檔物件, # 絕大部分用於存模型權重檔：模型.state_dict()
   f="地址"
)
```

---

## torch\.load\(\) 讀檔

讀取先前用 torch\.save 存檔的**權重（參數）、張量、模型**

```Python
... = torch.load(
    f="model.pt", # 檔案路徑
    map_location=None, # 映射路徑， mac 用 "mps"、"cpu"
    weights_only=True # 是否只允許載入權重資料（tensor、dist、list、數字）
)
```

---

### state\_dict\(\) 存取參數

將模型所有權重存為字典（鍵對值）

- 對象可以是**模型物件（參數）、優化器物件（一二階動量、學習率）、梯度縮放物件（縮放倍率、連續安全次數）**

```Python
... = 模型.state_dict()
```

---

### load\_state\_dict\(\) 將參數載入到物件

- 返回值為**鍵**（**可選**）：第一個返回的為模型裡有此層，但權重檔內沒有的；第二個為模型裡沒有，但權重檔有的

- 對象可以是**模型物件（參數）、優化器物件（一二階動量、學習率）、梯度縮放物件（縮放倍率、連續安全次數）**

```Python
missing_keys, unexpected_keys = 模型.load_state_dict(
    state_dict, # 用 torch.load() 讀取的權重
    strict=True # 權重與模型是否需要完全吻合（每層都要對應到權重）
)
```

---

## empty\_cache\(\) 清除快取

當

1. **儲存大型檢查點**（Checkpoint）之後

2. 程式內部的階段切換（例如：**訓練完立刻要做測試**）

```Python
torch.cuda.empty_cache()

torch.mps.empty_cache()
```

---

# DDP 分散式訓練

```Python
import torch.distributed as dist
```

控制多張顯示卡或多台機器並行運算，也就是 **DDP（Distributed Data Parallel）**

> **Distributed Data Parallel \-\> 分散式 數據 平行**

四個重要名詞：

- **World Size** **總卡數**：參與訓練的** GPU 總數**

- **Rank 全局編號**：每一個 GPU 都擁有一個**索引**，其中** Rank 0 為主節點（Master）**，負責**列印日誌和儲存權重**

- **Local Rank 本地編號**：**單一設備內**多卡的索引

    

    > **本地（Local）** 處理的是 **「硬體與作業系統層面」** 的事（跟實體插槽、單機作業系統有關）
    > 
    > **全局（Global）** 處理的是 **「演算法與邏輯協調層面」** 的事（跟整個訓練任務、網路通訊、身份唯一性有關）

- **Backend 通訊後端**：顯卡間彼此**溝通**的底層**協議**

---

## dist\.init\_process\_group\(\) 建立多卡環境

1. 開通多張顯示卡（或多台機器）之間的網絡通訊通道，組建出一個**通訊群組（Process Group）**

2. **確認身份**（確認每個進程並記錄下 world\_size、全局 rank）

```Python
import torch.distributed as dist

dist.init_process_group(
    backend="nccl", # 通訊協議
    init_method="env://", # 聯絡方式
    timeout=datetime.timedelta(seconds=1800), # 有設備超過多久沒回應報錯
    world_size=-1,
    rank=-1
)
```

- backend：nccl 為 NVIDIA 顯卡協議，Mac 上用跑在 CPU 上的 **gloo **

---

## dist\.destroy\_process\_group\(\) 清理分佈式訓練環境

有使用 DDP 時，訓練後要將環境刪除

---

## DistributedDataParallel\(\) 開啟 DDP

開啟分散式訓練

```Python
分散式訓練模型物件 = DistributedDataParallel(
    原模型物件,
    device_ids=[local_rank], # 目前進程中管轄的 GPU 本地編號，接受串列
    output_device=None # 輸出的張量要放在哪，None -> 跟隨模型最後一層的指定去向，通常就是上面傳入的 local_rank
)    
```

---

## no\_sync 關閉梯度同步

> synchronize \-\> 同步

若進行了**梯度累積**，使用此上下文管理器達到“**在不進行梯度更新的時候，不和其他卡做通訊**” \-\> 節省效能

```Python
with 模型物件.no_sync():
    ... 前向傳播及反向傳播
```

> 用 **nullcontext\(\)** 來簡化程式：
> 
> ```Python
> grad_accu_ctx = model.no_sync() if ... else nullcontext()
>         
> with grad_accu_ctx:
>     with autocast_ctx:
>         result = model(...) # 前向傳播
>         
>     scaler.scale(loss).backward() # 反向完播
> ```

---

## dist\.barrier\(\) 等待完成

等待直到所有 GPU 都跑到此處才繼續下面程式

例如在要清理 DDP 環境前需要等待所有 GPU 都完成了才開始清除

---

## 資訊

### dist\.is\_initialized\(\) 是否已經建立分散訓練環境

判斷的是**現在是不是已經成功處在多卡運作的狀態中了**？

> os\.environ\.get\("RANK"\) 判斷是否要求開啟分散式訓練

回傳布林值

---

### is\_main\_process\(\) 是否為主線程

判斷是否處在主線程（Global Rank 為 0）回傳布林值

---

### 取得 rank

取得目前執行指令的 GPU 的**全域編號**（為索引）：

```Python
... = dist.get_rank() 

# 或：
... = int(os.environ.get("RANK"))
```

取得本地編號則是：

```Python
... = int(os.environ.get("LOCAL_RANK"))
```

---

### dist\.get\_world\_size 取得總卡數

---







---

