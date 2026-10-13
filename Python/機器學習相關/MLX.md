# 概念與原理

## 什麼是 Apple MLX？

MLX 是由 Apple 機器學習研究團隊（Apple Machine Learning Research）專為 Apple Silicon（M 系列晶片）深度客製化的開源機器學習框架。

在架構設計上，MLX 緊密結合了 Apple 晶片的硬體特性，具備以下四大核心技術優勢：

1. **統一記憶體架構 (Unified Memory Architecture, UMA)**：  
   在 Mac 電腦中，CPU 與 GPU 共享同一塊超高頻寬實體記憶體。  
   MLX 張量在 CPU 與 GPU 之間傳遞無需透過 PCIe 匯流排複製資料，達到真正的「零拷貝 (Zero-copy)」轉換。
2. **惰性求值機制 (Lazy Evaluation)**：  
   所有運算預設不會立即執行，而是先在記憶體中建立輕量化計算圖。  
   只有在顯式呼叫 `mx.eval()`、輸出字串或轉為外部格式時，才會驅動 Metal 著色核心計算，大幅節省記憶體消耗並實現動態算子融合。
3. **雙生態直覺 API 設計**：  
   `mlx.core` 語法設計高度對齊 NumPy，提供直觀的數值計算陣列介面。  
   `mlx.nn` 與 `mlx.optimizers` 則深度借鑑 PyTorch，降低深度學習模型遷移門檻。
4. **純函數式變換能力 (Function Transformations)**：  
   吸納了 JAX 框架的純函數設計哲學，原生支援 `mx.grad`、`mx.vmap` 與 `mx.compile` 等高階數學變換。

> **[生動比喻]**：  
> - **傳統分體式架構 (NVIDIA GPU + Intel CPU)**：如同住在隔壁棟大樓的兩位廚師。CPU 備好料後，必須打包封箱、叫貨車經由跨海大橋 (PCIe 總線) 運送給 GPU 廚師料理，耗時且消耗大量貨車容量 (顯存)。  
> - **Apple Silicon + MLX 統一架構**：如同身處同一個開放式中島廚房。CPU 切好的蔬菜放在中島檯面上 (統一記憶體)，GPU 伸手就能直接下鍋，完全不需要任何搬運與裝箱過程。

---

## 惰性求值 (Lazy Evaluation) 底層運作機制

理解惰性求值是掌握 MLX 效能調優與除錯的第一步。

當你在 Python 中撰寫 `c = a + b` 時，MLX 僅在後台記錄「c 是由 a 與 b 相加而成」的依賴節點，並不會立刻向 GPU 發送計算指令。

```text
代碼調用: c = a + b
   │
   ▼
1. 記錄計算圖 ➔ 在記憶體中建立計算節點 (此時尚未佔用 GPU 算力)
   │
   ▼
2. 算子融合優化 ➔ 等待後續運算累積，尋找可合併為單一 Metal Shader 的算子
   │
   ▼
3. 觸發求值 ➔ 遇到 mx.eval(c)、print(c) 或 c.tolist()
   │
   ▼
4. Metal 執行 ➔ 批次呼叫 GPU 高速運算核心，一次性產出具體數值結果
```

> **[核心天條]**：  
> 惰性求值雖然能提升運算效率，但如果在長期運行的訓練迴圈中忘記呼叫 `mx.eval()`，計算圖將無限堆疊，最終耗盡系統記憶體導致程式無預警崩潰！

---

## 統一記憶體 (Unified Memory) 與硬體裝置排程

MLX 預設將運算排程於 GPU 執行。若有特殊需求，亦可靈活指定 CPU：

- **GPU 運算 (預設)**：適合高度平行的張量相乘、卷積與神經網絡權重前向傳播。
- **CPU 運算**：適合高度分支判斷、小型控制流程或序列化任務。

因為記憶體完全共享，切換裝置時不會產生任何資料複製負擔。

---

# 跨框架設計與語法演進對照表

本表系統化對比傳統 PyTorch (CUDA / MPS) 與 Apple MLX 的架構與語法差異：

| 功能操作 | PyTorch (MPS / CUDA) 寫法 | MLX 現代推薦標準寫法 | 核心差異與硬體優勢 |
| :--- | :--- | :--- | :--- |
| **陣列張量建立** | `torch.tensor([1, 2], device="mps")` | `mx.array([1, 2])` | 原生配置於統一記憶體，無需顯式傳遞至裝置 |
| **裝置切換與資料搬移** | `x.to("mps")` 或 `x.cpu()` | 零拷貝共享，動態指定 `mx.default_device()` | 消除 PCIe 記憶體複製開銷，CPU/GPU 共享記憶體池 |
| **計算圖評估求值** | 即時求值 (Eager Evaluation) | 惰性求值 (Lazy)，顯式 `mx.eval(x)` | 支援動態融合 Metal 計算核，減少記憶體中繼暫存 |
| **自動微分運算** | 隱式計算圖 `loss.backward()` | 函數式變換 `mx.grad(fn)` 或 `mx.value_and_grad(fn)` | 純函數式高階微分設計，無隱式狀態殘留 |
| **即時編譯加速 (JIT)** | `torch.compile(model)` | `mx.compile(fn)` | 生成並融合最佳化 Metal 著色核心，大幅提升吞吐量 |
| **模型權限與量化** | 第三方 bitsandbytes 或 GPTQ | `nn.QuantizedLinear` 原生 4-bit / 8-bit | Apple Silicon 硬體原生向量矩陣乘法加速量化 |
| **大型語言模型推論** | Hugging Face Transformers (`device_map="auto"`) | `mlx_lm` 生態套件 (`mlx_lm.generate`) | 專為 Mac 記憶體頻寬最佳化，推論速度極快 |

---

# 核心 API 類別與函式字典對照表

| 類別 / 函式名稱 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- |
| **[[#mx.array() 建立 MLX 多維陣列張量\|mx.array()]]** | **建立支援統一記憶體的多維陣列張量** | 資料前處理、張量建立、數值運算起手式 |
| **[[#mx.eval() 強制觸發惰性運算求值\|mx.eval()]]** | **顯式強制計算惰性求值圖譜並釋放中繼節點** | 批次訓練迴圈、釋放圖暫存、取得具體數值 |
| **[[#mx.default_device() 查詢或設定預設運算裝置\|mx.default_device()]]** | **查詢或切換預設執行硬體 (GPU / CPU)** | 效能調優、強制特定算子於 CPU 上執行 |
| **[[#mx.savez() 與 mx.load() 序列化與載入張量權重\|mx.savez() / mx.load()]]** | **將多個張量以 safetensors 或 npz 格式儲存與讀取** | 模型檢查點保存、權重匯出、跨平台參數交換 |
| **[[#mx.grad() 建立一階純函數微分器\|mx.grad()]]** | **對純函數建立自動微分高階轉換函數** | 純純量損失求導、客製化數學函數最佳化 |
| **[[#mx.value_and_grad() 同時計算函數值與梯度字典\|mx.value_and_grad()]]** | **單次前向傳播同時取得純量損失值與模型參數梯度** | 神經網路標準訓練步驟、自訂損失反向傳播 |
| **[[#mx.compile() JIT 即時編譯融合 Metal 計算核\|mx.compile()]]** | **將 Python 函數編譯並融合為最佳化 Metal 著色核心** | 高頻重複執行之訓練步驟加速、推論延遲降低 |
| **[[#mx.vmap() 自動向量化批次映射變換\|mx.vmap()]]** | **將針對單一樣本編寫的純函數自動提升為批次運算** | 單一樣本邏輯向量化、批次樣本並行計算 |
| **[[#nn.Module 神經網路模型抽象基底類別\|nn.Module]]** | **所有神經網路層與模型架構的抽象基底類別** | 自訂網路模型、模組參數生命週期管理 |
| **[[#nn.Linear 線性全連接神經層\|nn.Linear]]** | **執行仿射線性變換 (y = xW^T + b)** | 特徵降維、投影層、多層感知機 (MLP) 構建 |
| **[[#nn.QuantizedLinear 4-bit 與 8-bit 硬體量化線性層\|nn.QuantizedLinear]]** | **在 Metal 原生執行仿射低位元權重矩陣乘法** | 本地大語言模型高能效推論、顯存極致壓縮 |
| **[[#nn.Sequential 順序式神經網路容器\|nn.Sequential]]** | **將多個神經網路層按宣告順序串聯執行** | 快速搭建前饋神經網路、卷積特徵提取層 |
| **[[#nn.losses.cross_entropy() 交叉熵損失計算函式\|cross_entropy()]]** | **計算預測機率對數與真實標籤之交叉熵損失** | 多類別分類任務、語言模型下一個 Token 預測 |
| **[[#optim.AdamW 權重衰減 Adam 優化器\|optim.AdamW]]** | **結合動量、自適應學習率與獨立權重衰減的優化演算法** | Transformer 訓練、神經網路主流參數更新 |
| **[[#mlx_lm.load() 載入開源大語言模型與分詞器\|mlx_lm.load()]]** | **載入 Hugging Face 格式之開源 LLM 與專用 Tokenizer** | 本地大語言模型載入、量化權重初始化 |
| **[[#mlx_lm.generate() 執行大語言模型串流推論生成\|mlx_lm.generate()]]** | **調用模型與取樣策略生成文字並支援串流回傳** | 本機終端對話助手、智慧文本生成、本地 Agent 驅動 |

---

# 核心基礎運算與張量操作 (mlx.core)

##### mx.array() 建立 MLX 多維陣列張量

- **使用時機**：初始化多維資料、轉換 Python 序列或 NumPy 陣列至 MLX 統一記憶體張量時使用。
- **語法**：`mx.array(data, dtype=None)`
- **參數說明**：
  - `data`：純量、列表、元組或 NumPy ndarray 等資料來源。
  - `dtype`：可選參數，指定資料型別（如 `mx.float32`、`mx.int32`、`mx.bfloat16`）。
- **回傳值**：
  - `mx.array`：配置於統一記憶體之多維陣列實例。

```python
import mlx.core as mx

# 1. 自 Python 列表建立一維張量
a = mx.array([1.0, 2.0, 3.0])
print(a.shape)  # 假設印出：(3,)
print(a.dtype)  # 假設印出：float32

# 2. 建立二維常數張量矩陣
b = mx.zeros((2, 3), dtype=mx.int32)
print(b)  # 假設印出：array([[0, 0, 0], [0, 0, 0]], dtype=int32)
```

---

##### mx.eval() 強制觸發惰性運算求值

- **使用時機**：需要打破惰性求值計算圖、強制執行底層 Metal 著色核心運算並釋放中繼圖譜節點時使用。在訓練迴圈中必須定期呼叫以防記憶體外溢。
- **語法**：`mx.eval(*arrays)`
- **參數說明**：
  - `*arrays`：一或多個需要被評估的 `mx.array` 物件或模型參數樹。
- **回傳值**：
  - `None`：原位更新陣列狀態為已求值。

```python
import mlx.core as mx

x = mx.random.normal((1000, 1000))
y = mx.random.normal((1000, 1000))

# 此時僅建立計算圖，尚未真正進行矩陣乘法
z = x @ y

# 強制提交 GPU 佇列並完成數值計算
mx.eval(z)
print(z.shape)  # 假設印出：(1000, 1000)
```

---

##### mx.default_device() 查詢或設定預設運算裝置

- **使用時機**：需要明確指定運算運行於 Apple GPU 或 Apple CPU 時使用。
- **語法**：`mx.default_device()` 與 `mx.set_default_device(device)`
- **參數說明**：
  - `device`：傳入 `mx.gpu` 或 `mx.cpu` 裝置常數。
- **回傳值**：
  - `Device`：當前系統所使用的預設運算裝置物件。

```python
import mlx.core as mx

# 1. 查詢預設運算裝置
current_device = mx.default_device()
print(current_device)  # 假設印出：Device(gpu, 0)

# 2. 強制設定為 CPU 裝置
mx.set_default_device(mx.cpu)
```

---

##### mx.savez() 與 mx.load() 序列化與載入張量權重

- **使用時機**：儲存模型訓練檢查點、備份張量資料或從磁碟讀取 Safetensors / NPZ 檔案時使用。
- **語法**：
  - `mx.savez(file, **arrays)`
  - `mx.load(file)`
- **參數說明**：
  - `file`：檔案路徑字串或路徑物件（支援 `.npz` 或 `.safetensors`）。
  - `**arrays`：要儲存的具名張量關鍵字引數。
- **回傳值**：
  - `savez`：`None`。
  - `load`：包含權重張量的字典物件。

```python
import mlx.core as mx

weights = {"w": mx.array([1.0, 2.0]), "b": mx.array([0.5])}

# 1. 序列化儲存至本機檔案
mx.savez("checkpoint.npz", **weights)

# 2. 自本機檔案載入權重
loaded = mx.load("checkpoint.npz")
print(loaded["w"])  # 假設印出：array([1, 2], dtype=float32)
```

---

# 函數變換與自動微分 (Transformations)

##### mx.grad() 建立一階純函數微分器

- **使用時機**：針對傳回純量（純單一數值）的數學函數計算一階梯度導函數時使用。
- **語法**：`mx.grad(fn, argnums=0)`
- **參數說明**：
  - `fn`：接受張量輸入並回傳單一純量張量的純函數 (Pure Function)。
  - `argnums`：整數或整數元組，指定要對第幾個輸入參數進行求導（預設為第 0 個參數）。
- **回傳值**：
  - `Callable`：輸入參數相同但回傳對應梯度的轉換函數。

```python
import mlx.core as mx

# 定義目標函數: f(x) = x^2 + 3x + 5
def f(x: mx.array) -> mx.array:
    return x ** 2 + 3 * x + 5

# 取得導函數: f'(x) = 2x + 3
df = mx.grad(f)

x_val = mx.array(4.0)
grad_val = df(x_val)
print(grad_val.item())  # 假設印出：11.0 (2*4 + 3 = 11)
```

---

##### mx.value_and_grad() 同時計算函數值與梯度字典

- **使用時機**：神經網路標準訓練迴圈中最常用的核心微分器。一次前向計算同時傳回損失數值與模型參數梯度，效率最高。
- **語法**：`mx.value_and_grad(fn, argnums=0)`
- **參數說明**：
  - `fn`：損失評估純函數，接受模型與資料輸入，回傳單一純量損失。
  - `argnums`：指定求導參數目標位置。
- **回傳值**：
  - `Callable`：呼叫時傳回雙元素元組 `(loss, grads)` 的轉換函數。

```python
import mlx.core as mx
import mlx.nn as nn

model = nn.Linear(2, 1)

def loss_fn(model, x, y):
    pred = model(x)
    return mx.mean((pred - y) ** 2)

# 同時取得 loss 與模型權重樹狀結構梯度
loss_and_grad_fn = nn.value_and_grad(model, loss_fn)

x_batch = mx.array([[1.0, 2.0]])
y_batch = mx.array([[5.0]])

loss, grads = loss_and_grad_fn(model, x_batch, y_batch)
print(loss)  # 假設印出：array(12.4, dtype=float32)
```

---

##### mx.compile() JIT 即時編譯融合 Metal 計算核

- **使用時機**：加速執行頻率極高的計算密集型函數（如整個訓練步驟 `step` 或自訂注意力機制），消除 Python 解譯器負擔並自動融合多個 Metal 著色核心。
- **語法**：`mx.compile(fn, inputs=None, outputs=None)`
- **參數說明**：
  - `fn`：需要即時編譯的純函數。
- **回傳值**：
  - `Callable`：已編譯為 Metal 核心之高效執行函數。

```python
import mlx.core as mx

def gelu_approx(x: mx.array) -> mx.array:
    return 0.5 * x * (1.0 + mx.tanh(0.79788456 * (x + 0.044715 * x ** 3)))

# JIT 即時編譯加速
fast_gelu = mx.compile(gelu_approx)

x = mx.random.normal((512, 512))
out = fast_gelu(x)
mx.eval(out)
```

---

##### mx.vmap() 自動向量化批次映射變換

- **使用時機**：將原本針對「單一樣本」編寫的純函數，自動轉換為支援「多樣本批次並行計算」的高階函數，免去手動處理 Batch 維度與廣播 (Broadcasting) 的心智負擔。
- **語法**：`mx.vmap(fn, in_axes=0, out_axes=0)`
- **參數說明**：
  - `fn`：針對單一向量或張量撰寫的函數。
  - `in_axes`：指定輸入引數中哪一個維度代表批次軸（預設為 0）。
  - `out_axes`：指定輸出結果中哪一個維度作為批次軸（預設為 0）。
- **回傳值**：
  - `Callable`：自動批次向量化之新函數。

```python
import mlx.core as mx

# 僅針對單個向量計算內積
def dot_product(v1, v2):
    return mx.sum(v1 * v2)

# 自動提升為批次矩陣計算
batch_dot = mx.vmap(dot_product)

batch_a = mx.ones((10, 5))
batch_b = mx.ones((10, 5))
result = batch_dot(batch_a, batch_b)
print(result.shape)  # 假設印出：(10,)
```

---

# 神經網路構建與訓練體系 (mlx.nn & mlx.optimizers)

##### nn.Module 神經網路模型抽象基底類別

- **使用時機**：自訂深度學習架構、卷積網路或 Transformer 子模組時的繼承起點。
- **語法**：`class MyModel(nn.Module): ...`
- **核心方法**：
  - `__init__()`：宣告內部神經網路層與參數。
  - `__call__()`：定義前向傳播 (Forward Pass) 計算邏輯。
  - `parameters()`：取得當前模組所有可學習權重的巢狀字典。
- **回傳值**：
  - 模型實例物件。

```python
import mlx.core as mx
import mlx.nn as nn

class MLP(nn.Module):
    def __init__(self, in_dim: int, out_dim: int):
        super().__init__()
        self.fc1 = nn.Linear(in_dim, 32)
        self.relu = nn.ReLU()
        self.fc2 = nn.Linear(32, out_dim)

    def __call__(self, x: mx.array) -> mx.array:
        return self.fc2(self.relu(self.fc1(x)))

model = MLP(10, 2)
out = model(mx.zeros((1, 10)))
print(out.shape)  # 假設印出：(1, 2)
```

---

##### nn.Linear 線性全連接神經層

- **使用時機**：執行標準的特徵維度投影與線性轉換運算。
- **語法**：`nn.Linear(input_dims, output_dims, bias=True)`
- **參數說明**：
  - `input_dims`：輸入特徵維度大小。
  - `output_dims`：輸出特徵維度大小。
  - `bias`：布林值，是否包含可學習偏差向量（預設為 `True`）。
- **回傳值**：
  - 線性層實例物件。

```python
import mlx.core as mx
import mlx.nn as nn

layer = nn.Linear(64, 128)
x = mx.random.normal((8, 64))
y = layer(x)
print(y.shape)  # 假設印出：(8, 128)
```

---

##### nn.QuantizedLinear 4-bit 與 8-bit 硬體量化線性層

- **使用時機**：在 Mac 本地低記憶體環境下運行或推論開源大型語言模型（如 Llama 3、Mistral、Qwen）。直接在 Metal 上執行低位元仿射反量化矩陣乘法。
- **語法**：`nn.QuantizedLinear(input_dims, output_dims, bits=4, group_size=64, bias=False)`
- **參數說明**：
  - `bits`：量化位元寬度（支援 `4` 或 `8`）。
  - `group_size`：量化分組大小（常見為 `32`、`64` 或 `128`）。
- **回傳值**：
  - 硬體量化線性層實例。

```python
import mlx.core as mx
import mlx.nn as nn

# 建立 4-bit 量化層 (佔用記憶體僅原先 float16 的 25%)
qlinear = nn.QuantizedLinear.from_linear(
    nn.Linear(1024, 1024),
    bits=4,
    group_size=64
)
x = mx.random.normal((1, 1024))
out = qlinear(x)
print(out.shape)  # 假設印出：(1, 1024)
```

---

##### nn.Sequential 順序式神經網路容器

- **使用時機**：模型結構為線性前向串聯結構時，免除繁瑣的手動 forward 宣告。
- **語法**：`nn.Sequential(*layers)`
- **參數說明**：
  - `*layers`：依序執行的神經網路層序列。
- **回傳值**：
  - 順序神經網路容器實例。

```python
import mlx.nn as nn

model = nn.Sequential(
    nn.Linear(10, 20),
    nn.GELU(),
    nn.Linear(20, 2)
)
```

---

##### nn.losses.cross_entropy() 交叉熵損失計算函式

- **使用時機**：多類別分類任務或語言模型自回歸預測訓練時評估損失。
- **語法**：`nn.losses.cross_entropy(logits, targets, reduction="mean")`
- **參數說明**：
  - `logits`：未經過 Softmax 的原始模型輸出分數 (Unnormalized Logits)。
  - `targets`：真實類別標籤索引整數陣列。
  - `reduction`：縮減模式（`"none"`、`"mean"` 或 `"sum"`）。
- **回傳值**：
  - `mx.array`：純量損失張量。

```python
import mlx.core as mx
import mlx.nn as nn

logits = mx.array([[2.0, 1.0, 0.1], [0.5, 3.0, 0.2]])
targets = mx.array([0, 1])

loss = nn.losses.cross_entropy(logits, targets)
print(loss.item())  # 假設印出：0.3541
```

---

##### optim.AdamW 權重衰減 Adam 優化器

- **使用時機**：訓練現代神經網路（特別是 Transformer）的標準首選優化器。
- **語法**：`optim.AdamW(learning_rate, betas=[0.9, 0.999], eps=1e-8, weight_decay=0.01)`
- **參數說明**：
  - `learning_rate`：學習率數值或學習率調度器實例。
  - `weight_decay`：解耦權重衰減係數。
- **核心更新方法**：
  - `optimizer.update(model, grads)`：根據計算所得梯度更新模型參數樹。

```python
import mlx.core as mx
import mlx.nn as nn
import mlx.optimizers as optim

model = nn.Linear(4, 2)
optimizer = optim.AdamW(learning_rate=1e-3, weight_decay=1e-2)

def loss_fn(model, x, y):
    return mx.mean((model(x) - y) ** 2)

loss_and_grad = nn.value_and_grad(model, loss_fn)

x = mx.random.normal((8, 4))
y = mx.random.normal((8, 2))

# 1. 計算梯度
loss, grads = loss_and_grad(model, x, y)

# 2. 應用梯度更新模型參數
optimizer.update(model, grads)

# 3. 推進優化器與模型求值 (防止計算圖累積)
mx.eval(model.parameters(), optimizer.state)
```

---

# 大語言模型生態與量化推論 (mlx-lm)

`mlx-lm` 是建構於 MLX 之上的官方生態套件，支援 Hugging Face 社群中絕大多數主流開源 LLM（如 Llama 3、Qwen 2.5、Mistral、DeepSeek-R1-Distill、Gemma 等），並原生支援 4-bit 量化極速推論與 LoRA 微調。

##### mlx_lm.load() 載入開源大語言模型與分詞器

- **使用時機**：在 Mac 上載入本機或遠端 Hugging Face Hub 開源大語言模型與對應 Tokenizer。
- **語法**：`mlx_lm.load(path_or_hf_repo, tokenizer_config=None, model_config=None)`
- **參數說明**：
  - `path_or_hf_repo`：Hugging Face 倉庫名稱（如 `"mlx-community/Qwen2.5-7B-Instruct-4bit"`）或本機路徑。
- **回傳值**：
  - `tuple[model, tokenizer]`：模型實例與分詞器元組。

```python
from mlx_lm import load

# 一鍵自 mlx-community 載入 4-bit 量化大模型
model, tokenizer = load("mlx-community/Qwen2.5-7B-Instruct-4bit")
```

---

##### mlx_lm.generate() 執行大語言模型串流推論生成

- **使用時機**：調用載入的大語言模型生成對話或文字內容。
- **語法**：`mlx_lm.generate(model, tokenizer, prompt, max_tokens=512, verbose=False)`
- **參數說明**：
  - `model`：已載入的模型實例。
  - `tokenizer`：對應的 Tokenizer 物件。
  - `prompt`：輸入提示詞字串。
  - `max_tokens`：最大生成長度。
  - `verbose`：布林值，是否直接在終端機串流輸出生成進度與速度報告。
- **回傳值**：
  - `str`：模型生成的文字結果。

```python
from mlx_lm import load, generate

model, tokenizer = load("mlx-community/Qwen2.5-7B-Instruct-4bit")

prompt = "請用三句話簡介 Apple Silicon 的統一記憶體優勢："
response = generate(
    model,
    tokenizer,
    prompt=prompt,
    max_tokens=200,
    verbose=True
)
print("最終結果:", response)
```

---

# 實戰除錯與核心天條

## 1. 訓練迴圈忘記呼叫 mx.eval() 導致記憶體暴增洩漏

- **錯誤症狀**：模型訓練幾十個 Step 後，系統記憶體佔用持續狂飆，最終被 macOS 系統強行以 `OOM / Killed: 9` 終止程序。
- **背後原理**：MLX 為惰性求值機制。若未顯式呼叫 `mx.eval()`，每一次迭代產生的張量依賴圖譜會永久懸掛在記憶體中，計算圖幾何級數膨脹。
- **記憶口訣**：每輪迭代必求值，參數狀態一同算，及時釋放計算圖。

```python
# [錯誤寫法]：迴圈中只做更新，未顯式求值
for epoch in range(100):
    loss, grads = loss_fn(model, x, y)
    optimizer.update(model, grads)
    # 致命地雷：缺少 mx.eval，計算圖無限堆疊至當機！

# [正確寫法]：更新後立即呼叫 mx.eval 原位求值並清空圖譜
for epoch in range(100):
    loss, grads = loss_fn(model, x, y)
    optimizer.update(model, grads)
    mx.eval(model.parameters(), optimizer.state)
```

---

## 2. 誤用 PyTorch 隱式狀態 loss.backward() 語法

- **錯誤症狀**：拋出 `AttributeError: 'array' object has no attribute 'backward'`。
- **背後原理**：MLX 沒有 PyTorch 式的隱式計算圖物件與 `.backward()` 機制。MLX 採用純函數式求導，必須透過 `mx.grad` 或 `nn.value_and_grad` 進行函數轉換。
- **記憶口訣**：無 backward 物件導向，純函數求導取梯度，value_and_grad 是標準。

```python
# [錯誤寫法]：試圖調用 PyTorch 的 backward 語法
loss = loss_fn(model, x, y)
loss.backward()  # 報錯：MLX array 無 backward 方法！

# [正確寫法]：定義損失函數並透過 value_and_grad 取得梯度
loss_and_grad = nn.value_and_grad(model, loss_fn)
loss, grads = loss_and_grad(model, x, y)
```

---

## 3. 張量嘗試就地修改 (In-place Mutation) 引發未定義行為

- **錯誤症狀**：在微分變換中修改張量，拋出異常或導致反向傳播梯度錯誤。
- **背後原理**：為支援純函數式變換與高階圖優化，MLX 張量具備函數式不可變 (Immutable) 特徵，不支援就地修改底層記憶體陣列。
- **記憶口訣**：張量本質不可變，賦值重組建新圖，嚴禁就地切片改。

```python
# [錯誤寫法]：試圖對張量內部元素進行就地修改
x = mx.zeros((3,))
x[0] = 5.0  # 雖然支援特定切片語法，但在自動微分作用域內可能破壞計算圖

# [正確寫法]：使用純函數運算重構張量
x = mx.concatenate([mx.array([5.0]), x[1:]])
```

---

## 4. 混淆 Python 原生數值與 MLX 延遲陣列純量

- **錯誤症狀**：在條件判斷式 `if loss < 0.01:` 中直接使用未求值的 MLX 純量，造成不必要的隱式同步延遲或圖構建中斷。
- **背後原理**：當 Python 的 `if` 判斷式評估 `mx.array` 純量時，會被強制同步阻塞等待 GPU 完成計算，打破了批次執行的流水線效能。
- **記憶口訣**：流程控制勿卡關，先取 item 再判斷，批次計算走平行。

```python
# [錯誤寫法]：高頻迴圈中每步直接對未 eval 的 array 做條件判斷
if loss < 0.001:  # 強制每一步同步等待，GPU 流水線被拆碎
    break

# [正確寫法]：明確顯式取值並限制同步頻率
if epoch % 10 == 0:
    mx.eval(loss)
    if loss.item() < 0.001:
        break
```
