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
   吸納了 JAX 框架的純函數設計哲學，原生支援 `mx.grad`、`mx.vmap`、`mx.checkpoint` 與 `mx.compile` 等高階數學變換。

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

## 計算圖即時編譯 (JIT) 與 Metal 核心融合原理

MLX 的 `mx.compile()` 採用即時編譯（Just-In-Time Compilation）技術，將 Python 算子運算圖深度優化為原生 Metal 著色核心。

1. **圖形追蹤 (Tracing)**：  
   初次執行時，MLX 記錄純函數輸入與輸出之間的有向無環圖 (DAG)。
2. **算子融合 (Kernel Fusion)**：  
   將多個連續的逐元素運算（如加法、乘法、GELU 激活）與維度約簡合併至單一 Metal Compute Kernel。
3. **消除全域顯存讀寫 (Bandwidth Optimization)**：  
   中繼張量直接於晶片內部快取與暫存器完成傳遞，不再重複讀寫全域統一記憶體，顯著提高計算吞吐量。

---

## 張量不可變性 (Tensor Immutability) 與殘差連接 (Residual Connection)

在 MLX 中，多維陣列張量（`mx.array`）被設計為**嚴格不可變 (Immutable)** 的純函數式資料結構。

### 1. 為什麼張量設計為不可變？

- **純函數式自動微分**：自動微分引擎（`mx.grad`、`mx.value_and_grad`）需要明確追蹤每個計算節點的輸入與輸出。若允許張量就地修改，反向傳播時難以保證中間激活值未被篡改。
- **無副作用與安全引用**：在 Python 中進行賦值（如 `residual = x`）時，僅是增加物件參照（Reference）。因為張量不可變，你永遠不需要擔心後續模組或分支運算在背後悄悄修改底層記憶體陣列。
- **零成本「複製一份」**：在不可變資料結構下，保留一份資料供後續使用（例如 `residual = x`）是完全**零開銷 (Zero-overhead)** 的，既不會觸發實體記憶體搬移或深拷貝 (Deep Copy)，又能獲得如同獨立副本的安全保證。

### 2. 核心實戰場景：殘差連接 (Residual Connection)

在現代深度學習架構（如 Transformer 的 Pre-LN 結構、ResNet 或 DiT 擴散模型）中，**殘差連接 (Residual Connection / Skip Connection)** 是不可或缺的核心組件：
$$y = x + \mathrm{SubLayer}(\mathrm{Norm}(x))$$

- **在 PyTorch 中的潛在陷阱**：若程式碼中不慎使用了就地修改運算（例如 `torch.relu_()`、`x += delta` 或切片就地賦值），先前保留下來作為殘差的張量可能在不知不覺中被覆蓋，導致反向傳播報錯（*RuntimeError: a view of a leaf Variable that requires grad is being used in an in-place operation*）。
- **在 MLX 中的絕對安全與直覺**：直接將輸入參照賦值給殘差變數 `residual = x`。後續即使對 `x` 進行層歸一化、注意力機制或前饋網路變換，所有算子都會傳回全新的張量，原始的 `residual` 永遠維持原貌，最終直接相加 `x = residual + branch_output` 即可。

```python
import mlx.core as mx
import mlx.nn as nn

class ResidualTransformerBlock(nn.Module):
    def __init__(self, dims: int, num_heads: int):
        super().__init__()
        self.norm = nn.RMSNorm(dims)
        self.attn = nn.MultiHeadAttention(dims, num_heads)
        self.ffn = nn.Sequential(
            nn.Linear(dims, dims * 4),
            nn.GELU(),
            nn.Linear(dims * 4, dims)
        )

    def __call__(self, x: mx.array, mask: mx.array = None) -> mx.array:
        # 1. 殘差連接應用：直接賦值引用「複製一份」殘差
        # 由於 MLX 張量不可變，此操作為零記憶體複製開銷，且保證後續運算絕不污染原始輸入
        residual = x
        
        # 2. 主分支計算 (norm 與 attn 均產生新張量，residual 保持不變)
        x = self.norm(x)
        x = self.attn(queries=x, keys=x, values=x, mask=mask)
        
        # 3. 殘差相加融合
        x = residual + x
        
        # 4. 第二段 FFN 殘差連接
        residual = x
        x = residual + self.ffn(self.norm(x))
        return x

# 驗證殘差張量不可變性
x = mx.ones((2, 4, 64))
block = ResidualTransformerBlock(dims=64, num_heads=4)
out = block(x)
mx.eval(out)
print(f"殘差模組輸出形狀: {out.shape}")  # 印出：(2, 4, 64)
```

---

# 跨框架設計與語法演進對照表

本表系統化對比傳統 PyTorch (CUDA / MPS) 與 Apple MLX 的架構與語法差異：

| 功能操作 | PyTorch (MPS / CUDA) 寫法 | MLX 現代推薦標準寫法 | 核心差異與硬體優勢 |
| :--- | :--- | :--- | :--- |
| **陣列張量建立** | `torch.tensor([1, 2], device="mps")` | `mx.array([1, 2])` | 原生配置於統一記憶體，無需顯式傳遞至裝置 |
| **裝置切換與資料搬移** | `x.to("mps")` 或 `x.cpu()` | 零拷貝共享，動態指定 `mx.default_device()` | 消除 PCIe 記憶體複製開銷，CPU/GPU 共享記憶體池 |
| **隨機數生成模式** | 僅支援全域隱式狀態 `torch.manual_seed(42)` | 原生雙軌支援：全域狀態 (`mx.random.seed`) 與 JAX 風格無狀態金鑰 (`mx.random.key` / `split`) | 消除多執行緒並行計算的隨機衝突，原生相容純函數式編譯與變換 |
| **矩陣乘法運算** | `torch.matmul(A, B)` 或 `A @ B` | `mx.matmul(A, B)` 或 `A @ B` | 自動調用 Apple Silicon 內部 AMX 矩陣協處理器加速 |
| **計算圖評估求值** | 即時求值 (Eager Evaluation) | 惰性求值 (Lazy)，顯式 `mx.eval(x)` | 支援動態融合 Metal 計算核，減少記憶體中繼暫存 |
| **自動微分運算** | 隱式計算圖 `loss.backward()` | 函數式變換 `mx.grad(fn)` 或 `mx.value_and_grad(fn)` | 純函數式高階微分設計，無隱式狀態殘留 |
| **即時編譯加速 (JIT)** | `torch.compile(model)` | `mx.compile(fn)` | 生成並融合最佳化 Metal 著色核心，大幅提升吞吐量 |
| **梯度檢查點 (顯存節約)** | `torch.utils.checkpoint.checkpoint` | `mx.checkpoint(fn)` | 以前向傳播重計算代替記憶體留存，大幅降低峰值顯存 |
| **注意力機制加速** | `F.scaled_dot_product_attention` | `mlx.core.fast.scaled_dot_product_attention` | Metal 原生硬體融合注意力核心，大幅降低顯存頻寬消耗 |
| **旋轉位置編碼加速** | 手寫複數旋轉矩陣乘法 | `mlx.core.fast.rope` | 核心層硬體深度優化，極速計算 RoPE 嵌入 |
| **模型權限與量化** | 第三方 bitsandbytes 或 GPTQ | `nn.QuantizedLinear` 原生 4-bit / 8-bit | Apple Silicon 硬體原生向量矩陣乘法加速量化 |
| **大型語言模型推論** | Hugging Face Transformers (`device_map="auto"`) | `mlx_lm` 生態套件 (`mlx_lm.generate`) | 專為 Mac 記憶體頻寬最佳化，推論速度極快 |

---

# 核心 API 類別與函式字典對照表

| 類別 / 函式名稱 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- |
| **[[#MLX 數值資料型別 (Data Types / dtypes) 全景指南\|mx.bfloat16 / mx.float16 / mx.float32]]** | **張量數值位元精度與型別定義 (BF16, FP16, FP32 等)** | 模型微調推論精度配置、省顯存半精度計算、數值溢位防範 |
| **[[#mx.array() 建立 MLX 多維陣列張量\|mx.array()]]** | **建立支援統一記憶體的多維陣列張量** | 資料前處理、張量建立、數值運算起手式 |
| **[[#mx.eval() 強制觸發惰性運算求值\|mx.eval()]]** | **顯式強制計算惰性求值圖譜並釋放中繼節點** | 批次訓練迴圈、釋放圖暫存、取得具體數值 |
| **[[#mx.default_device() 查詢或設定預設運算裝置\|mx.default_device()]]** | **查詢或切換預設執行硬體 (GPU / CPU)** | 效能調優、強制特定算子於 CPU 上執行 |
| **[[#mx.savez() 與 mx.load() 序列化與載入張量權重\|mx.savez() / mx.load()]]** | **將多個張量以 safetensors 或 npz 格式儲存與讀取** | 模型檢查點保存、權重匯出、跨平台參數交換 |
| **[[#mx.reshape() 與 array.reshape() 重構張量維度形狀\|mx.reshape() / .reshape()]]** | **重組張量維度結構並支援 -1 自動維度推斷** | Transformer 隱藏層投影轉換、批次序列打散與拍平 |
| **[[#mx.transpose() 與 array.T / array.mT 軸向轉置與批次矩陣轉置\|mx.transpose() / .T / .mT]]** | **任意多維軸向重新排列 (Permute) 與批次矩陣轉置** | 多頭注意力 (B, S, H, D) 轉置為 (B, H, S, D)、矩陣相乘對齊 |
| **[[#mx.swapaxes() 與 mx.moveaxis() 指定維度軸向交換與移動\|mx.swapaxes() / mx.moveaxis()]]** | **精確交換兩維度或將特定軸移動至目標位置** | 視覺特徵圖 NCHW 與 NHWC 通道佈局轉換、注意力維度微調 |
| **[[#mx.squeeze() 與 mx.expand_dims() 壓縮單元維度與擴展虛擬軸\|mx.squeeze() / mx.expand_dims()]]** | **消除大小為 1 的多餘單元軸或在指定位置插入新維度** | 移除單一樣本批次維度、擴展廣播軸向以執行張量運算 |
| **[[#mx.flatten() 與 mx.unflatten() 展平張量維度與指定軸還原結構\|mx.flatten() / mx.unflatten()]]** | **拍平多維張量為向量或將指定軸拆解還原為子維度** | 卷積特徵展平送入分類頭、多頭輸出維度拆解還原 |
| **[[#mx.concatenate() 沿指定軸拼接張量序列\|mx.concatenate()]]** | **將多個相同維度張量沿指定軸向進行無縫拼接** | 特徵拼接、多模態表示融合、批次資料整合 |
| **[[#mx.stack() 與 mx.unstack() 沿新維度堆疊張量與反向解包\|mx.stack() / mx.unstack()]]** | **沿全新維度軸向堆疊多個張量或解包切片為列表** | 批次資料建構、多特徵向量組合為矩陣、切片解包 |
| **[[#mx.split() 沿指定軸切分張量為子陣列清單\|mx.split()]]** | **將單一張量沿指定維度均等或按索引切分為子張量** | 多頭注意力 QKV 投影切分、特徵分塊處理 |
| **[[#mx.broadcast_to() 零拷貝廣播擴充目標形狀\|mx.broadcast_to()]]** | **以虛擬步長零拷貝方式將張量廣播延伸為目標尺寸** | 偏置向量擴展相加、因果注意力遮罩廣播至多頭與批次 |
| **[[#mx.repeat() 與 mx.tile() 元素級重複與多維平鋪擴展\|mx.repeat() / mx.tile()]]** | **沿指定軸進行元素就地重複或將張量整體磁磚化平鋪** | GQA (Grouped-Query Attention) 頭數複製、批次維度擴展 |
| **[[#mx.pad() 多維張量邊界填充\|mx.pad()]]** | **沿多維度邊界填充常數、反射或邊緣數值** | CNN 特徵圖邊界填充、長度不一文本序列補齊 (Padding) |
| **[[#mx.where() 條件三元選擇算子\|mx.where()]]** | **依據布林條件在兩張量或純量間挑選元素 (c ? x : y)** | 自訂非線性啟動函數、因果注意力遮罩生成、邊界截斷 |
| **[[#mx.take() 與 mx.take_along_axis() 多維陣列索引抽取\|mx.take() / mx.take_along_axis()]]** | **依離散索引或沿特定軸向依多維索引矩陣抽取數值** | 詞嵌入離散 Token 查表、提取預測最大機率標籤分數 |
| **[[#array.at[...] 函數式不可變局部更新\|array.at[...]]]** | **JAX 風格純函數式局部元素與切片數值運算** | 不可變張量安全局部累加、保持 JIT 編譯圖純函數性 |
| **[[#array.view() 底層二進位記憶體型別重釋\|array.view()]]** | **保持底層 Byte 記憶體不變重新以新型別解讀資料** | 浮點與無符號整數二進位位元重釋、底層編碼操作 |
| **[[#mx.flip() 與 mx.roll() 維度翻轉與循環滾動位移\|mx.flip() / mx.roll()]]** | **沿指定軸反轉元素排列順序或進行循環滾動位移** | 時間序列逆序、Swin Transformer 移動視窗注意力 (Shifted Window) |
| **[[#mx.metal.get_active_memory() 查詢當前 Metal 活躍使用記憶體\|mx.metal.get_active_memory()]]** | **獲取當前配置於 GPU 且尚在使用的記憶體位元組數** | 顯存佔用監控、OOM 異常前置診斷、批次大小調優 |
| **[[#mx.metal.clear_cache() 清理 Metal 釋放快取池釋放實體記憶體\|mx.metal.clear_cache()]]** | **釋放底層 Metal 記憶體配置器快取池中的閒置記憶體** | 大模型載入前後釋放記憶體、批次訓練間隙記憶體重置 |
| **[[#mx.random.seed() 設定全域隨機種子\|mx.random.seed()]]** | **設定全域 PRNG 隨機狀態種子數值** | 快速原型開發、腳本執行全域隨機性重現 |
| **[[#mx.random.key() 建立純函數式 PRNG 密鑰\|mx.random.key()]]** | **建立無狀態純函數式亂數生成密鑰** | 純函數式隨機計算、多執行緒並行隨機採樣 |
| **[[#mx.random.split() 分割無狀態 PRNG 子金鑰\|mx.random.split()]]** | **自現有金鑰衍生分割產生多個獨立子金鑰** | 分支隨機計算、避免平行迭代隨機序列相關性 |
| **[[#mx.random.normal() 常態分佈高斯取樣\|mx.random.normal()]]** | **生成符合常態高斯分佈之多維隨機張量** | 權重矩陣初始化、擴散模型前向加噪、高斯雜訊擾動 |
| **[[#mx.random.multivariate_normal() 多元常態高斯分佈取樣\|mx.random.multivariate_normal()]]** | **依指定均值向量與協方差矩陣抽取多維相關常態分佈樣本** | VAE 潛在空間採樣、高斯混合模型 (GMM)、貝氏最佳化 |
| **[[#mx.random.truncated_normal() 截斷常態分佈取樣\|mx.random.truncated_normal()]]** | **生成去除極端離群值之截斷常態分佈張量** | Transformer / ViT 權重初始化、早期訓練穩定度維護 |
| **[[#mx.random.uniform() 均勻分佈隨機取樣\|mx.random.uniform()]]** | **在指定連續區間內生成均勻分佈隨機數** | 權重均勻初始化 (Xavier)、超參數隨機搜尋 |
| **[[#mx.random.bernoulli() 伯努利二項分佈遮罩\|mx.random.bernoulli()]]** | **依指定機率生成布林二元隨機遮罩張量** | 自訂實現 Dropout 層、隨機深度 (Stochastic Depth) |
| **[[#mx.random.randint() 離散均勻整數取樣\|mx.random.randint()]]** | **在指定整數區間內均勻抽取離散整數張量** | 隨機空間裁切座標、抽樣類別標籤、雜湊索引生成 |
| **[[#mx.random.categorical() 類別分佈對數機率採樣\|mx.random.categorical()]]** | **依據未歸一化對數機率分佈進行隨機離散抽樣** | LLM 文本生成 Temperature 採樣、RL 動作選取 |
| **[[#mx.random.permutation() 隨機置換與陣列洗牌\|mx.random.permutation()]]** | **對整數序列或多維陣列沿指定軸隨機洗牌** | 訓練週期 (Epoch) 資料集隨機洗牌、樣本亂序打亂 |
| **[[#mx.random.gumbel() 耿貝爾分佈取樣\|mx.random.gumbel()]]** | **生成標準耿貝爾分佈隨機變數張量** | Gumbel-Max 技巧、Gumbel-Softmax 可微分重參數化 |
| **[[#mx.random.laplace() 拉普拉斯雙指數分佈取樣\|mx.random.laplace()]]** | **生成尖峰重尾之拉普拉斯分佈隨機張量** | 差分隱私 (DP) 雜訊機制、L1 稀疏先驗建模、魯棒回歸 |
| **[[#mx.matmul() 與 @ 批次矩陣相乘運算\|mx.matmul() / @]]** | **執行二維或高維張量廣播矩陣乘法運算** | 全連接神經層前向計算、多頭注意力 QK 乘積 |
| **[[#mx.linalg.norm() 矩陣與向量範數計算\|mx.linalg.norm()]]** | **計算張量之 L1、L2 範數或 Frobenius 矩陣範數** | 權重正則化懲罰、梯度裁剪 (Gradient Clipping) |
| **[[#mx.linalg.inv() 方陣求逆與線性方程求解\|mx.linalg.inv()]]** | **求解非奇異方陣之逆矩陣** | 線性回歸封閉式解、協方差矩陣求逆、卡爾曼濾波 |
| **[[#mx.einsum() 愛因斯坦求和約定張量運算\|mx.einsum()]]** | **使用愛因斯坦求和標記法精簡表達複雜多維張量縮併** | 自訂多維注意力權重縮併、雙線性特徵融合 |
| **[[#mx.softmax() 沿指定軸機率分佈歸一化\|mx.softmax()]]** | **將原始分數轉換為平滑歸一化機率分佈** | 注意力權重生成、分類模型輸出機率評估 |
| **[[#mx.logsumexp() 數值穩定對數指數和運算\|mx.logsumexp()]]** | **以數值穩定演算法計算 log(sum(exp(x))) 防止溢位** | 自訂交叉熵損失函數、計算配分函數與對數似然 |
| **[[#mx.clip() 數值截斷與梯度防爆裁剪\|mx.clip()]]** | **將張量所有元素限制在指定上下界區間內** | 防止梯度爆炸、強化學習價值函數截斷 |
| **[[#mx.topk() 提取前 K 大元素數值與索引\|mx.topk()]]** | **沿指定維度快速提取前 K 個最大數值及其索引位置** | LLM 自回歸生成 Top-K 採樣、推薦系統候選召回 |
| **[[#fast.scaled_dot_product_attention() Metal 融合縮放點積注意力\|fast.scaled_dot_product_attention()]]** | **硬體融合版縮放點積注意力 (FlashAttention 等級優化)** | Transformer 訓練與推論、大幅降低顯存頻寬消耗 |
| **[[#fast.rope() 旋轉位置編碼硬體融合加速\|fast.rope()]]** | **原生底層加速算子計算旋轉位置編碼 (RoPE)** | 現代大語言模型 (Llama 3, Qwen) 查詢與鍵向量編碼 |
| **[[#fast.rms_norm() 硬體融合均方根層歸一化\|fast.rms_norm()]]** | **底層融合計算均方根歸一化 (RMSNorm)** | 現代 Transformer 前向層歸一化極速推論 |
| **[[#fast.metal_kernel() 定義客製化 Metal 著色核心\|fast.metal_kernel()]]** | **以 Metal 著色語言 (MSL) 定義底層硬體客製化算子** | 特殊非線性啟動函數、自訂硬體融合算子加速 |
| **[[#mx.grad() 建立一階純函數微分器\|mx.grad()]]** | **對純函數建立自動微分高階轉換函數** | 純純量損失求導、客製化數學函數最佳化 |
| **[[#mx.value_and_grad() 同時計算函數值與梯度字典\|mx.value_and_grad()]]** | **單次前向傳播同時取得純量損失值與模型參數梯度** | 神經網路標準訓練步驟、自訂損失反向傳播 |
| **[[#mx.compile() JIT 即時編譯融合 Metal 計算核\|mx.compile()]]** | **將 Python 函數編譯並融合為最佳化 Metal 著色核心** | 高頻重複執行之訓練步驟加速、推論延遲降低 |
| **[[#mx.vmap() 自動向量化批次映射變換\|mx.vmap()]]** | **將針對單一樣本編寫的純函數自動提升為批次運算** | 單一樣本邏輯向量化、批次樣本並行計算 |
| **[[#mx.checkpoint() 梯度檢查點重計算節省顯存\|mx.checkpoint()]]** | **以前向重計算換取顯存節省，在反向傳播時動態重構中繼激活值** | 訓練超長上下文 Transformer、超大 Batch Size 顯存防爆 |
| **[[#nn.Module 神經網路模型抽象基底類別\|nn.Module]]** | **所有神經網路層與模型架構的抽象基底類別** | 自訂網路模型、模組參數生命週期管理 |
| **[[#nn.Linear 線性全連接神經層\|nn.Linear]]** | **執行仿射線性變換 (y = xW^T + b)** | 特徵降維、投影層、多層感知機 (MLP) 構建 |
| **[[#nn.QuantizedLinear 4-bit 與 8-bit 硬體量化線性層\|nn.QuantizedLinear]]** | **在 Metal 原生執行仿射低位元權重矩陣乘法** | 本地大語言模型高能效推論、顯存極致壓縮 |
| **[[#nn.Embedding 詞嵌入向量查表層\|nn.Embedding]]** | **將離散 Token ID 映射為連續密集實數向量表示** | NLP 模型輸入表示、Token 嵌入層、位置編碼表 |
| **[[#nn.QuantizedEmbedding 硬體量化詞嵌入查表層\|nn.QuantizedEmbedding]]** | **4-bit / 8-bit 量化詞嵌入矩陣查表與線性投影** | 大語言模型超大詞彙表顯存極致壓縮 |
| **[[#nn.Dropout 隨機神經元丟棄正則化層\|nn.Dropout]]** | **在訓練期依指定機率隨機將激活值置零並自動按比例放大** | 抑制神經網路過擬合、增強模型特徵泛化能力 |
| **[[#nn.LayerNorm 標準層歸一化神經層\|nn.LayerNorm]]** | **沿特徵維度計算均值與方差並套用可學習仿射參數歸一化** | 經典 Transformer、BERT 與 ViT 的特徵穩定歸一化 |
| **[[#nn.RMSNorm 均方根層歸一化層\|nn.RMSNorm]]** | **現代大語言模型必備之無偏置均方根正規化模組** | 構建 Llama 3、Mistral 等架構之 Pre-Norm 層 |
| **[[#nn.RoPE 旋轉位置編碼神經模組\|nn.RoPE]]** | **大語言模型必備之相對旋轉位置編碼模組** | Llama 3、Qwen、Mistral 等架構 Query 與 Key 位置編碼與 KV Cache 增量推論 |
| **[[#nn.MultiHeadAttention 多頭自注意力神經層\|nn.MultiHeadAttention]]** | **經典 Transformer 多頭注意力機制神經網路模組** | 序列特徵建模、長文字自注意力表示提取 |
| **[[#nn.Conv2d 二維卷積神經網路層\|nn.Conv2d]]** | **對影像或網格狀資料執行二維空間特徵卷積提取** | 電腦視覺影像分類、卷積神經網絡 (CNN) 骨幹 |
| **[[#nn.Sequential 順序式神經網路容器\|nn.Sequential]]** | **將多個神經網路層按宣告順序串聯執行** | 快速搭建前饋神經網路、卷積特徵提取層 |
| **[[#nn.losses.cross_entropy() 交叉熵損失計算函式\|cross_entropy()]]** | **計算預測機率對數與真實標籤之交叉熵損失** | 多類別分類任務、語言模型下一個 Token 預測 |
| **[[#nn.losses.mse_loss() 均方誤差回歸損失函式\|mse_loss()]]** | **計算預測值與真實連續目標值之間的均方誤差 (MSE)** | 數值回歸任務、圖像重構、自編碼器 (Autoencoder) 訓練 |
| **[[#nn.losses.binary_cross_entropy() 二元交叉熵損失函式\|binary_cross_entropy()]]** | **計算二元分類任務中預測機率與真實標籤的二元交叉熵** | 點擊率預估 (CTR)、多標籤分類、二元決策判定 |
| **[[#model.parameters 模型可學習權重樹狀字典\|model.parameters]]** | **獲取模組中所有可學習權重張量之嵌套結構字典** | 提供優化器更新、檢查點保存、參數梯度更新 |
| **[[#model.freeze() 凍結模型模組權重參數\|model.freeze()]]** | **遞迴將模型或指定子層的參數設為不可訓練狀態** | 遷移學習特徵提取器凍結、LoRA 微調僅訓練適配層 |
| **[[#optim.AdamW 權重衰減 Adam 優化器\|optim.AdamW]]** | **結合動量、自適應學習率與獨立權重衰減的優化演算法** | Transformer 訓練、神經網路主流參數更新 |
| **[[#optim.cosine_decay() 餘弦退火學習率排程器\|optim.cosine_decay()]]** | **建立依餘弦函數平滑衰減至指定終點值的學習率排程函式** | 現代 LLM 預訓練與微調學習率衰減、平滑收斂控制 |
| **[[#mlx.utils.tree_map() 遞迴對樹狀結構張量套用變換\|mlx.utils.tree_map()]]** | **對嵌套字典或元組中的所有張量節點遞迴套用指定純函數** | 全域梯度裁剪、參數權重平滑、樹狀結構批次運算 |
| **[[#mlx_lm.load() 載入開源大語言模型與分詞器\|mlx_lm.load()]]** | **載入 Hugging Face 格式之開源 LLM 與專用 Tokenizer** | 本地大語言模型載入、量化權重初始化 |
| **[[#mlx_lm.generate() 執行大語言模型串流推論生成\|mlx_lm.generate()]]** | **調用模型與取樣策略生成文字並支援串流回傳** | 本機終端對話助手、智慧文本生成、本地 Agent 驅動 |
| **[[#mlx_lm.convert() 將 Hugging Face 模型權重轉為 MLX 與量化格式\|mlx_lm.convert()]]** | **將原始 PyTorch/Safetensors 模型轉換並壓縮為 MLX 專用 4-bit/8-bit 權重** | 私有模型本地部署、模型顯存壓縮優化 |
| **[[#initialize_rope() 動態旋轉位置編碼工廠函式\|initialize_rope()]]** | **解析模型組態動態建立標準 RoPE 或 YaRN 等長上下文位置編碼模組** | 現代長上下文 LLM (YaRN 16K/32K/128K、Llama 3、Qwen) 位置編碼初始化 |
---

# 核心基礎運算與張量操作 (mlx.core)

## MLX 數值資料型別 (Data Types / dtypes) 全景指南

在 Apple MLX 中，所有張量（`mx.array`）都具備明確的資料型別（`dtype`），用於決定其在 Apple Silicon 統一記憶體中佔用的位元組大小、數值表示範圍與底層硬體運算單元（如 AMX 矩陣乘法加速器或 Metal 浮點向量單元）。

### 1. MLX 支援的完整資料型別矩陣

| 分類 | 型別常數 | 位元組大小 (Bytes) | 說明與典型應用場景 |
| :--- | :--- | :--- | :--- |
| **大模型半精度浮點** | `mx.bfloat16` | 2 bytes (16-bit) | **現代 LLM 標準型別**（Llama 3、Mistral、DeepSeek）。指數位與 FP32 相同，徹底杜絕溢位，比 FP32 省 50% 顯存與頻寬。 |
| **傳統半精度浮點** | `mx.float16` | 2 bytes (16-bit) | 傳統電腦視覺與音訊模型常用。精度高於 BF16，但最大值僅 65,504，訓練中較易溢位。 |
| **標準單精度浮點** | `mx.float32` | 4 bytes (32-bit) | **MLX 預設浮點型別**。科學計算、損失累積、評估指標與數值敏感層（如 Softmax / RMSNorm 累加）的基準精度。 |
| **雙精度浮點** | `mx.float64` | 8 bytes (64-bit) | 高精度物理模擬或統計求逆。**注意：Apple Silicon GPU 不支援硬體 FP64 向量單元，僅限 CPU 運算。** |
| **有符號整數** | `mx.int8` / `mx.int16` / `mx.int32` / `mx.int64` | 1 / 2 / 4 / 8 bytes | `int32` 為 Token ID 查表與預設整數；`int8` 常用於對稱量化權重；`int64` 用於超大索引。 |
| **無符號整數** | `mx.uint8` / `mx.uint16` / `mx.uint32` / `mx.uint64` | 1 / 2 / 4 / 8 bytes | `uint8` 常用於影像像素 (0~255) 與量化縮放；`uint32` 為 MLX 隨機金鑰 (`mx.random.key`) 的底層規格。 |
| **布林型別** | `mx.bool_` | 1 byte (8-bit) | 注意力因果遮罩 (Mask)、條件選取 (`mx.where`) 與伯努利丟棄遮罩。 |
| **複數型別** | `mx.complex64` | 8 bytes (2x FP32) | 傅立葉變換 (FFT)、信號處理與頻域特徵分析。 |

---

### 2. 深度剖析：為什麼現代 LLM 全面採用 `mx.bfloat16`？

一般程式語言中的「普通浮點數」通常為 **FP32 (單精度 32-bit)** 或 **FP64 (雙精度 64-bit)**。而 `mx.bfloat16` (Brain Floating Point 16) 是專門為深度學習打造的 16 位元浮點數。

#### 底層位元分配與數值範圍對照表

| 浮點型別 | 符號位 (Sign) | 指數位 (Exponent) | 尾數位 (Fraction) | 動態範圍 (Max / Min) | 機器精度 $\epsilon$ (Epsilon) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **FP32** (`mx.float32`) | 1 bit | **8 bits** | 23 bits | $\approx \pm 3.4 \times 10^{38}$ | $\approx 1.19 \times 10^{-7}$ |
| **FP16** (`mx.float16`) | 1 bit | 5 bits | 10 bits | $\approx \pm 65,504$ (極易溢位) | $\approx 9.77 \times 10^{-4}$ |
| **BF16** (`mx.bfloat16`) | 1 bit | **8 bits** | 7 bits | $\approx \pm 3.39 \times 10^{38}$ (與 FP32 一致) | $\approx 7.81 \times 10^{-3}$ |

#### `bfloat16` 的三大不可替代優勢

1. **動態範圍完全等同於 FP32**：  
   `bfloat16` 的指數位達到 8 bits，其數值上限高達 $3.39 \times 10^{38}$。傳統 FP16 超過 65,504 就會變成 `inf`，在反向傳播時經常發生梯度爆炸與 NaN；而 `bfloat16` 徹底消除了溢位風險，免除了繁瑣的動態損失縮放（Loss Scaling）。
2. **與 FP32 轉換零電路開銷**：  
   `bfloat16` 的高 16 位元與 FP32 的高 16 位元（1 符號 + 8 指數 + 7 高位尾數）完全重合。硬體將 FP32 轉為 `bfloat16` 時只需截斷低 16 位尾數，轉換速度極快且能效極高。
3. **Apple Silicon 硬體原生加速**：  
   M 系列晶片的神經網路引擎 (Neural Engine) 與 Metal GPU 具備完整的 BF16 原生向量乘加指令。在統一記憶體中，將權重由 FP32 轉為 `mx.bfloat16` 直接省下 **50% 記憶體空間與傳輸頻寬**，推論吞吐量大幅暴增。

---

### 3. 型別檢查、轉換與極限查詢常用操作

```python
import mlx.core as mx
import mlx.nn as nn

# 1. 建立指定型別的張量
x_bf16 = mx.array([1.5, 2.5, 3.5], dtype=mx.bfloat16)
print(f"張量型別: {x_bf16.dtype}")           # 印出：mlx.core.bfloat16
print(f"單一元素位元組數: {x_bf16.itemsize}")  # 印出：2 bytes

# 2. astype() 顯式型別轉換
x_fp32 = x_bf16.astype(mx.float32)
print(f"轉換後型別: {x_fp32.dtype}")         # 印出：mlx.core.float32
print(f"單一元素位元組數: {x_fp32.itemsize}")  # 印出：4 bytes

# 3. 查詢浮點數極限範圍 (mx.finfo) 與整數範圍 (mx.iinfo)
bf16_info = mx.finfo(mx.bfloat16)
print(f"bfloat16 最大值: {bf16_info.max:.4e}, 最小值: {bf16_info.min:.4e}")
# 印出：bfloat16 最大值: 3.3895e+38, 最小值: -3.3895e+38

fp16_info = mx.finfo(mx.float16)
print(f"float16 最大值: {fp16_info.max}")  # 印出：65504.0 (容易溢位)

int_info = mx.iinfo(mx.int32)
print(f"int32 範圍: [{int_info.min}, {int_info.max}]")
# 印出：int32 範圍: [-2147483648, 2147483647]

# 4. 神經網路模型全域轉為 bfloat16
model = nn.Linear(1024, 1024)
print(f"預設權重型別: {model.weight.dtype}")  # 印出：float32
model.set_dtype(mx.bfloat16)
print(f"轉型後權重型別: {model.weight.dtype}")  # 印出：bfloat16
```

---

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

##### mx.reshape() 與 array.reshape() 重構張量維度形狀

- **使用時機**：改變張量的維度幾何結構而不改變元素總數與底層記憶體資料順序（行優先 C-order）。常見於 Transformer 隱藏層投影轉換、批次序列拍平為矩陣 `(Batch, Seq, Dim) -> (Batch*Seq, Dim)`，或將單一維度展開為多頭維度 `(Batch, Seq, Dim) -> (Batch, Seq, Heads, Head_Dim)`。
- **語法**：
  - `array.reshape(*shape)` 或 `array.reshape(shape)`
  - `mx.reshape(array, shape)`
- **參數說明**：
  - `shape`：整數元組或連續展開的整數。支援傳入 `-1` 代表該維度由系統自動推斷計算（單次調用僅允許一個 `-1`）。
- **回傳值**：
  - `mx.array`：具備新幾何維度結構的張量視圖。

```python
import mlx.core as mx

x = mx.arange(12)  # 一維張量: [0, 1, 2, ..., 11]

# 1. 顯式指定目標形狀 (3 列 4 行)
r1 = x.reshape(3, 4)
print(f"r1 形狀: {r1.shape}")  # 印出：(3, 4)

# 2. 使用 -1 自動推斷未知維度 (設定 2 列，自動推算為 6 行)
r2 = x.reshape(2, -1)
print(f"r2 形狀: {r2.shape}")  # 印出：(2, 6)

# 3. 模擬 Transformer 多頭注意力維度拆解: (Batch=2, Seq=16, Dim=512) -> (2, 16, 8, 64)
hidden_states = mx.random.normal((2, 16, 512))
heads = hidden_states.reshape(2, 16, 8, 64)
print(f"拆分多頭後形狀: {heads.shape}")  # 印出：(2, 16, 8, 64)
```

---

##### mx.transpose() 與 array.T / array.mT 軸向轉置與批次矩陣轉置

- **使用時機**：矩陣轉置與任意多維軸向重新排列（等同 PyTorch 中的 `permute`）。最經典應用是在 Multi-Head Attention 中將序列維度與注意力頭維度對調 `(Batch, Seq, Heads, Dim) -> (Batch, Heads, Seq, Dim)`，以便對各頭獨立進行並行矩陣乘法。
- **語法**：
  - `array.T`：轉置屬性。對一維張量無影響，對二維矩陣反轉行列，對多維張量完全顛倒所有軸向順序。
  - `array.mT`：批次矩陣轉置（Matrix Transpose）屬性。僅交換最後兩維 `(*, M, N) -> (*, N, M)`，完美保留所有前置批次與注意力頭維度。
  - `mx.transpose(a, axes=None)` 或 `array.transpose(*axes)`：按指定軸序列進行任意多維置換。
- **參數說明**：
  - `axes`：指定新維度順序的整數元組或序列。若為 `None` 則完全顛倒所有維度。
- **回傳值**：
  - `mx.array`：軸向重新排列後的張量。

```python
import mlx.core as mx

# 1. 二維矩陣標準轉置
mat = mx.array([[1, 2, 3], [4, 5, 6]])
print(f"原始形狀: {mat.shape}, 轉置後形狀: {mat.T.shape}")  # 印出：(2, 3) 與 (3, 2)

# 2. 多頭注意力關鍵轉置: (Batch=2, Seq=16, Heads=8, Head_Dim=64)
# 交換 Seq 與 Heads 軸向 -> (Batch=2, Heads=8, Seq=16, Head_Dim=64)
q = mx.random.normal((2, 16, 8, 64))
q_transposed = q.transpose(0, 2, 1, 3)
print(f"多頭獨立前向形狀: {q_transposed.shape}")  # 印出：(2, 8, 16, 64)

# 3. 批次矩陣轉置 .mT: 僅交換末尾兩維，用於 Attention 點積 Q @ K.mT
k_transposed = q_transposed  # 假設 K 與 Q 形狀相同
attn_scores = q_transposed @ k_transposed.mT
print(f"注意力分數矩陣形狀: {attn_scores.shape}")  # 印出：(2, 8, 16, 16)
```

---

##### mx.swapaxes() 與 mx.moveaxis() 指定維度軸向交換與移動

- **使用時機**：
  - `swapaxes`：精確對調兩個指定的軸向，簡潔易讀。
  - `moveaxis`：將指定維度從來源位置移動至目標位置（如在電腦視覺中將影像通道軸由末尾移動至前置 `NHWC -> NCHW`，或反向調整）。
- **語法**：
  - `mx.swapaxes(a, axis1, axis2)` 或 `array.swapaxes(axis1, axis2)`
  - `mx.moveaxis(a, source, destination)` 或 `array.moveaxis(source, destination)`
- **參數說明**：
  - `axis1`, `axis2`：待互換的兩軸整數索引。
  - `source`, `destination`：來源軸索引與目標軸索引。
- **回傳值**：
  - `mx.array`：軸向調整後的張量。

```python
import mlx.core as mx

x = mx.zeros((2, 16, 8, 64))

# 1. 使用 swapaxes 交換第 1 軸 (Seq) 與第 2 軸 (Heads)
s = x.swapaxes(1, 2)
print(f"swapaxes 後形狀: {s.shape}")  # 印出：(2, 8, 16, 64)

# 2. 使用 moveaxis 將通道軸從最後移動至第 1 軸: (B, H, W, C) -> (B, C, H, W)
img_nhwc = mx.zeros((4, 32, 32, 3))
img_nchw = img_nhwc.moveaxis(-1, 1)
print(f"moveaxis 轉換形狀: {img_nchw.shape}")  # 印出：(4, 3, 32, 32)
```

---

##### mx.squeeze() 與 mx.expand_dims() 壓縮單元維度與擴展虛擬軸

- **使用時機**：
  - `squeeze`：去除張量中大小為 1 的多餘維度（如批次處理完成後將 `(1, 128)` 壓縮為 `(128,)`，或將單通道灰階影像 `(28, 28, 1)` 轉為 `(28, 28)`）。
  - `expand_dims` 或 `None` / `mx.newaxis`：在指定軸位置插入一個長度為 1 的新維度，常用於使形狀符合廣播運算或批次輸入需求。
- **語法**：
  - `mx.squeeze(a, axis=None)` 或 `array.squeeze(axis=None)`
  - `mx.expand_dims(a, axis)`
  - 切片語法糖：`array[None, :]`、`array[:, None]` 或 `array[:, mx.newaxis, :]`
- **參數說明**：
  - `axis`：欲壓縮或擴展的目標軸整數或元組。若 `squeeze` 未指定 `axis` 則壓縮所有為 1 的維度。
- **回傳值**：
  - `mx.array`：維度增減後的張量。

```python
import mlx.core as mx

feat = mx.zeros((1, 32, 1, 128))

# 1. 壓縮所有大小為 1 的單元軸
sq_all = feat.squeeze()
print(f"全壓縮形狀: {sq_all.shape}")  # 印出：(32, 128)

# 2. 僅壓縮第 0 軸 (保留第 2 軸)
sq_0 = feat.squeeze(axis=0)
print(f"指定軸壓縮形狀: {sq_0.shape}")  # 印出：(32, 1, 128)

# 3. expand_dims 擴展維度: 插入注意力頭軸 (32, 128) -> (32, 1, 128)
expanded = mx.expand_dims(sq_all, axis=1)
print(f"expand_dims 形狀: {expanded.shape}")  # 印出：(32, 1, 128)

# 4. 使用 None / mx.newaxis 切片語法糖 (等價於 expand_dims)
syntax_exp = sq_all[:, None, :]
print(f"切片擴展形狀: {syntax_exp.shape}")  # 印出：(32, 1, 128)
```

---

##### mx.flatten() 與 mx.unflatten() 展平張量維度與指定軸還原結構

- **使用時機**：
  - `flatten`：將多個維度壓平成單一維度。常用於全連接分類頭前拍平卷積特徵圖 `(Batch, H, W, C) -> (Batch, H*W*C)`，或在自注意力完成後將多頭維度合併回隱藏維度。
  - `unflatten`：`flatten` 的逆向操作。將張量中某個指定軸拆解還原為多維幾何結構，免去手動計算整數 tuple 的繁瑣過程。
- **語法**：
  - `mx.flatten(a, start_axis=0, end_axis=-1)` 或 `array.flatten(start_axis=0, end_axis=-1)`
  - `mx.unflatten(a, axis, shape)`
- **參數說明**：
  - `start_axis`, `end_axis`：展平的起始與終止軸索引（包含該軸）。
  - `axis`：`unflatten` 中欲拆解的目標軸索引。
  - `shape`：目標軸拆解後的子形狀元組。
- **回傳值**：
  - `mx.array`：展平或結構還原後的張量。

```python
import mlx.core as mx

# 1. 多頭注意力輸出: (Batch=2, Heads=8, Seq=16, Dim=64)
attn_out = mx.random.normal((2, 8, 16, 64))

# 轉置回 (Batch=2, Seq=16, Heads=8, Dim=64)
attn_trans = attn_out.transpose(0, 2, 1, 3)

# 2. 將最後兩維 (Heads, Dim) 展平合併為單一隱藏特徵維度 512
flattened = attn_trans.flatten(start_axis=2, end_axis=3)
print(f"合併多頭後形狀: {flattened.shape}")  # 印出：(2, 16, 512)

# 3. 使用 unflatten 逆向將最後一維 (512) 重新拆解回 (Heads=8, Dim=64)
unflattened = mx.unflatten(flattened, axis=-1, shape=(8, 64))
print(f"還原結構後形狀: {unflattened.shape}")  # 印出：(2, 16, 8, 64)
```

---

##### mx.concatenate() 沿指定軸拼接張量序列

- **使用時機**：在多特徵融合、RNN/LSTM 狀態串聯、多模態表示串接或沿着現有維度組合多個張量時使用。
- **語法**：`mx.concatenate(arrays, axis=0)`
- **參數說明**：
  - `arrays`：由待拼接張量組成的序列清單。所有張量除了拼接軸外，其餘維度尺寸必須嚴格相等。
  - `axis`：沿著進行拼接的目標維度軸向（預設為 0）。
- **回傳值**：
  - `mx.array`：沿指定軸完成拼接合併後的新張量。

```python
import mlx.core as mx

x1 = mx.ones((2, 3))
x2 = mx.zeros((2, 3))

# 沿特徵欄位軸 (axis=1) 橫向拼接
combined = mx.concatenate([x1, x2], axis=1)
mx.eval(combined)
print(f"拼接後形狀: {combined.shape}")  # 印出：(2, 6)
```

---

##### mx.stack() 與 mx.unstack() 沿新維度堆疊張量與反向解包

- **使用時機**：
  - `stack`：將多個形狀完全相同的張量沿著「一個全新創建的維度」堆疊起來（與 `concatenate` 在既有軸擴充不同）。例如將多筆獨立樣本特徵向量堆疊成一個批次矩陣。
  - `unstack`：沿指定軸將張量切片拆解回子張量清單。
- **語法**：
  - `mx.stack(arrays, axis=0)`
  - `mx.unstack(array, axis=0)`
- **參數說明**：
  - `arrays`：待堆疊的張量清單（形狀必須完全一致）。
  - `axis`：新維度插入的位置索引。
- **回傳值**：
  - `stack`：維度增加 1 的新張量。
  - `unstack`：子張量組成的 Python 清單。

```python
import mlx.core as mx

v1 = mx.array([1, 2, 3])
v2 = mx.array([4, 5, 6])

# 1. 沿第 0 軸堆疊 (形狀由 (3,) 提升至 (2, 3))
stacked_0 = mx.stack([v1, v2], axis=0)
print(f"stack axis=0 形狀: {stacked_0.shape}")  # 印出：(2, 3)

# 2. 沿第 1 軸堆疊 (形狀提升至 (3, 2))
stacked_1 = mx.stack([v1, v2], axis=1)
print(f"stack axis=1 形狀: {stacked_1.shape}")  # 印出：(3, 2)

# 3. unstack 反向解包為清單
unstacked_list = mx.unstack(stacked_0, axis=0)
print(f"解包數量: {len(unstacked_list)}, 單個形狀: {unstacked_list[0].shape}")  # 印出：2 與 (3,)
```

---

##### mx.split() 沿指定軸切分張量為子陣列清單

- **使用時機**：多頭注意力機制中將大線性投影矩陣切分為 Query、Key、Value，或將資料集沿樣本軸切分為多塊時使用。
- **語法**：`mx.split(array, indices_or_sections, axis=0)` 或 `array.split(indices_or_sections, axis=0)`
- **參數說明**：
  - `array`：待切分的原始多維張量。
  - `indices_or_sections`：整數（均等切分成 N 等份），或整數序列（切分點索引列表）。
  - `axis`：執行切分的軸向（預設為 0）。
- **回傳值**：
  - `list[mx.array]`：由切分出的子張量組成的 Python 清單。

```python
import mlx.core as mx

# 模擬融合 QKV 投影矩陣輸出: (Batch=2, Seq=16, 3*Dim=768)
qkv = mx.random.normal((2, 16, 768))

# 沿最後一維均等切分為 3 個 256 維度的獨立張量
q, k, v = mx.split(qkv, indices_or_sections=3, axis=-1)
print(f"Q 形狀: {q.shape}, K 形狀: {k.shape}, V 形狀: {v.shape}")
# 印出：Q 形狀: (2, 16, 256), K 形狀: (2, 16, 256), V 形狀: (2, 16, 256)
```

---

##### mx.broadcast_to() 零拷貝廣播擴充目標形狀

- **使用時機**：顯式將張量廣播到更大的形狀（例如將一維偏置向量 `(Dim,)` 擴充為 `(Batch, Seq, Dim)`，或將二維因果遮罩 `(1, 1, Seq, Seq)` 廣播至所有批次與注意力頭）。
- **底層架構優勢**：在 Apple Silicon 統一記憶體中，廣播運算僅透過步長（Strides）設為 0 來達成視圖擴展，**完全不會複製任何底層記憶體資料**，開銷為零。
- **語法**：`mx.broadcast_to(array, shape)`
- **參數說明**：
  - `array`：待廣播的原張量。
  - `shape`：目標幾何形狀（必須相容於 NumPy 廣播規則）。
- **回傳值**：
  - `mx.array`：廣播擴展後的新張量視圖。

```python
import mlx.core as mx

# 1. 將一維特徵偏置 (3,) 廣播至 (4, 3) 矩陣
bias = mx.array([10.0, 20.0, 30.0])
broadcasted = mx.broadcast_to(bias, shape=(4, 3))
print(f"廣播後形狀: {broadcasted.shape}")  # 印出：(4, 3)

# 2. 因果注意力遮罩廣播: (1, 1, 16, 16) 廣播至 (Batch=2, Heads=8, 16, 16)
mask = mx.zeros((1, 1, 16, 16))
full_mask = mx.broadcast_to(mask, shape=(2, 8, 16, 16))
print(f"多頭因果遮罩廣播形狀: {full_mask.shape}")  # 印出：(2, 8, 16, 16)
```

---

##### mx.repeat() 與 mx.tile() 元素級重複與多維平鋪擴展

- **使用時機**：
  - `repeat`：沿指定軸將每個元素就地重複複製 N 次（如 `[1, 2]` 重複 2 次得到 `[1, 1, 2, 2]`）。在現代大模型 Grouped-Query Attention (GQA，如 Llama 3、Mistral) 中，用於將數量較少的 Key/Value 頭按比例重複複製擴充，以對齊 Query 頭數量。
  - `tile`：將整個張量視為地磚進行多維連續平鋪（如 `[1, 2]` 平鋪 2 次得到 `[1, 2, 1, 2]`）。
- **語法**：
  - `mx.repeat(a, repeats, axis=None)`
  - `mx.tile(a, reps)`
- **參數說明**：
  - `repeats`：各元素重複複製的次數。
  - `reps`：各軸向平鋪的倍數整數或整數元組。
- **回傳值**：
  - `mx.array`：重複複製擴展後的張量。

```python
import mlx.core as mx

a = mx.array([1, 2, 3])

# 1. repeat: 元素級相鄰重複
rep = mx.repeat(a, repeats=2, axis=0)
print(f"repeat 結果: {rep}")  # 印出：array([1, 1, 2, 2, 3, 3], dtype=int32)

# 2. tile: 整塊平鋪複製
til = mx.tile(a, 2)
print(f"tile 結果: {til}")    # 印出：array([1, 2, 3, 1, 2, 3], dtype=int32)

# 3. 模擬 GQA 中將 2 個 KV Heads 複製擴增為 8 個以對齊 Query Heads
# 形狀: (Batch=1, KV_Heads=2, Seq=4, Dim=64) -> 重複 4 次 -> (1, 8, 4, 64)
kv_heads = mx.random.normal((1, 2, 4, 64))
expanded_kv = mx.repeat(kv_heads, repeats=4, axis=1)
print(f"GQA 擴展後 KV Heads 形狀: {expanded_kv.shape}")  # 印出：(1, 8, 4, 64)
```

---

##### mx.pad() 多維張量邊界填充

- **使用時機**：自回歸文本序列長度補齊 (Batch Padding)、卷積神經網路中客製化特徵圖外圍填補、邊界反射填充以維持空間維度。
- **語法**：`mx.pad(array, pad_width, mode="constant", constant_values=0)`
- **參數說明**：
  - `array`：待填充的輸入張量。
  - `pad_width`：每個維度前後填充數量的元組序列 `[(前0, 後0), (前1, 後1), ...]`。
  - `mode`：填充演算法（預設 `"constant"` 常數填充，亦支援 `"edge"` 邊緣延伸）。
  - `constant_values`：常數填充數值（預設為 0）。
- **回傳值**：
  - `mx.array`：完成邊界填充的外擴張量。

```python
import mlx.core as mx

# 1. 2D 矩陣邊界常數填充 (上下各填 1 行，左右各填 2 行)
img = mx.ones((2, 2))
padded = mx.pad(img, pad_width=[(1, 1), (2, 2)], mode="constant", constant_values=0)
print(f"填充後形狀: {padded.shape}")  # 印出：(4, 6)

# 2. NLP 句子長度批次補齊 (將長度為 3 的序列補齊至 6)
token_ids = mx.array([101, 2054, 102])
padded_tokens = mx.pad(token_ids, pad_width=(0, 3), constant_values=0)
print(f"補齊後 Token ID: {padded_tokens}")  # 印出：array([101, 2054, 102, 0, 0, 0], dtype=int32)
```

---

##### mx.where() 條件三元選擇算子

- **使用時機**：依布林條件遮罩在兩個張量或純量間挑選元素（等價於三元運算子 `condition ? x : y`）。常用於自訂 ReLU/GELU 截斷、因果注意力遮罩（將 False 未來位置替換為 $-\infty$）、強化學習數值截斷。
- **語法**：`mx.where(condition, x, y)`
- **參數說明**：
  - `condition`：布林型別張量 (`mx.bool_`)。
  - `x`：條件為 `True` 時選取的數值或張量。
  - `y`：條件為 `False` 時選取的數值或張量。
- **回傳值**：
  - `mx.array`：依條件選取組合後的新張量。

```python
import mlx.core as mx

# 1. 基礎布林三元選取
cond = mx.array([True, False, True])
x = mx.array([1, 2, 3])
y = mx.array([9, 8, 7])
out = mx.where(cond, x, y)
print(f"where 輸出: {out}")  # 印出：array([1, 8, 3], dtype=int32)

# 2. 自訂 ReLU 啟動函數 (小於 0 置為 0.0)
feats = mx.array([-2.5, 0.0, 3.2, -1.0])
relu_out = mx.where(feats > 0, feats, 0.0)
print(f"ReLU 效果: {relu_out}")  # 印出：array([0, 0, 3.2, 0], dtype=float32)

# 3. 因果注意力遮罩防窺未來 Token (將未來位置設為負無窮大)
seq_len = 4
i = mx.arange(seq_len)[:, None]
j = mx.arange(seq_len)[None, :]
causal_condition = i >= j  # 下三角為 True (過去與現在)，上三角為 False (未來)
causal_mask = mx.where(causal_condition, 0.0, -1e9)
print("因果遮罩矩陣:")
print(causal_mask)
```

---

##### mx.take() 與 mx.take_along_axis() 多維陣列索引抽取

- **使用時機**：
  - `take`：沿指定軸或扁平索引直接抽取目標元素，是離散 Token 查表與資料集隨機抽樣的底層核心。
  - `take_along_axis`：依據與資料維度對應的多維索引張量抽取數值。常用於分類任務中提取最高預測機率的 Logits、強化學習從 Q-Table 中提取選定動作的價值。
- **語法**：
  - `mx.take(a, indices, axis=None)`
  - `mx.take_along_axis(a, indices, axis)`
- **參數說明**：
  - `a`：被抽取的來源多維張量。
  - `indices`：整數索引張量。
  - `axis`：抽取目標軸向。
- **回傳值**：
  - `mx.array`：抽取出的目標張量。

```python
import mlx.core as mx

# 1. mx.take: 沿指定軸抽取特定離散索引
embedding_table = mx.array([[10, 11], [20, 21], [30, 31], [40, 41]])
token_ids = mx.array([0, 2, 3])
extracted = mx.take(embedding_table, token_ids, axis=0)
print("抽取特徵向量:")
print(extracted)  # 印出索引 0, 2, 3 的列

# 2. mx.take_along_axis: 沿分類軸依據 argmax 索引提取 Top-1 預測分數
logits = mx.array([[0.1, 0.9, 0.3], [0.8, 0.0, 0.2]])
best_classes = mx.argmax(logits, axis=1, keepdims=True)
top_scores = mx.take_along_axis(logits, best_classes, axis=1)
print(f"最佳預測類別: {best_classes.squeeze()}")  # 印出：array([1, 0], dtype=uint32)
print(f"對應預測機率: {top_scores.squeeze()}")   # 印出：array([0.9, 0.8], dtype=float32)
```

---

##### array.at[...] 函數式不可變局部更新

- **使用時機**：以 JAX 風格的純函數式運算對張量特定切片或離散索引位置進行局部累加、替換或運算（支援 `.add`、`.subtract`、`.multiply`、`.divide`、`.maximum`、`.minimum`）。在保證張量不可變性（Immutability）與 JIT 編譯圖純函數性的前提下，安全地產生局部更新後的新張量，杜絕記憶體就地污染。
- **語法**：
  - `array.at[idx].add(value)`
  - `array.at[idx].subtract(value)`
  - `array.at[idx].multiply(value)`
  - `array.at[idx].divide(value)`
  - `array.at[idx].maximum(value)`
  - `array.at[idx].minimum(value)`
- **參數說明**：
  - `idx`：切片索引、單一整數或整數索引張量。
  - `value`：欲進行運算的標量或張量數值。
- **回傳值**：
  - `mx.array`：套用運算後生成的**全新張量**（原始張量保持完全不變）。

```python
import mlx.core as mx

base = mx.zeros((5,))

# 1. 局部純函數式累加 (原始 base 保持全 0，返回全新 added 張量)
added = base.at[1].add(10.0)
print(f"局部累加後新張量: {added}")  # 印出：array([0, 10, 0, 0, 0], dtype=float32)
print(f"原始張量保持不可變: {base}")   # 印出：array([0, 0, 0, 0, 0], dtype=float32)

# 2. 局部切片數值運算
multi = added.at[1].multiply(2.5)
print(f"局部乘法後新張量: {multi}")  # 印出：array([0, 25, 0, 0, 0], dtype=float32)

# 3. 散列聚合 (Scatter-Add 累加重複索引)
scatter_idx = mx.array([2, 2, 4])
scattered = base.at[scatter_idx].add(1.0)
print(f"重複索引累加結果: {scattered}")  # 印出：array([0, 0, 2, 0, 1], dtype=float32)
```

---

##### array.view() 底層二進位記憶體型別重釋

- **使用時機**：在完全不改變記憶體底層二進位 Byte 位元的前提下，將張量以另一種資料型別重新解讀（Bitcast）。常見於底層演算法、硬體驅動優化、快速位元操作（例如將 `float32` 重釋為 `uint32` 進行二進位浮點遮罩編碼）。
- **語法**：`array.view(dtype)` 或 `mx.view(array, dtype)`
- **參數說明**：
  - `dtype`：目標解讀資料型別（例如 `mx.uint32`、`mx.int32`）。
- **回傳值**：
  - `mx.array`：以新型別解讀底層位元組後的張量。

```python
import mlx.core as mx

# 將 32-bit 浮點數 1.0 以 32-bit 無符號整數解讀 (IEEE 754 二進位表示為 0x3F800000 = 1065353216)
f_val = mx.array([1.0], dtype=mx.float32)
u_val = f_val.view(mx.uint32)
print(f"float32 數值: {f_val.item()}")  # 印出：1.0
print(f"uint32 位元值: {u_val.item()}")  # 印出：1065353216
```

---

##### mx.flip() 與 mx.roll() 維度翻轉與循環滾動位移

- **使用時機**：
  - `flip`：沿指定軸反轉元素排列順序。用於時間序列逆向分析（如雙向 RNN 反向特徵提取）、卷積濾波器空間鏡射、電腦視覺資料擴增（水平翻轉）。
  - `roll`：沿指定軸將元素循環滾動位移。用於 Swin Transformer 中的移動視窗注意力機制 (Shifted Window Multi-Head Attention, SW-MSA)、周期性邊界條件模擬。
- **語法**：
  - `mx.flip(a, axis)` 或 `array.flip(axis)`
  - `mx.roll(a, shift, axis=None)` 或 `array.roll(shift, axis=None)`
- **參數說明**：
  - `axis`：翻轉或滾動的目標軸。
  - `shift`：滾動位移的元素數量（正數表示向右/向下滾動，負數表示向左/向上滾動）。
- **回傳值**：
  - `mx.array`：翻轉或循環位移後的張量。

```python
import mlx.core as mx

seq = mx.arange(5)  # [0, 1, 2, 3, 4]

# 1. flip: 沿第 0 軸倒轉序列
flipped = mx.flip(seq, axis=0)
print(f"倒轉後序列: {flipped}")  # 印出：array([4, 3, 2, 1, 0], dtype=int32)

# 2. roll: 向右循環位移 1 格 (末尾 4 繞回首位)
rolled = mx.roll(seq, shift=1, axis=0)
print(f"循環位移後序列: {rolled}")  # 印出：array([4, 0, 1, 2, 3], dtype=int32)

# 3. 2D 特徵圖循環滾動 (模擬 Swin Transformer 視窗位移)
feature_map = mx.arange(16).reshape(4, 4)
shifted_map = mx.roll(feature_map, shift=(-1, -1), axis=(0, 1))
print("視窗循環位移:")
print(shifted_map)
```

---

##### mx.metal.get_active_memory() 查詢當前 Metal 活躍使用記憶體

- **使用時機**：監控 Apple Silicon GPU 當前配置且尚處於活躍使用中的實體記憶體位元組數。用於動態調整 Batch Size 或預防 OOM 崩潰。
- **語法**：`mx.metal.get_active_memory()`
- **回傳值**：
  - `int`：以位元組 (Bytes) 為單位的活躍 GPU 記憶體使用量。

```python
import mlx.core as mx

# 查詢當前 Metal 配置顯存
bytes_used = mx.metal.get_active_memory()
mb_used = bytes_used / (1024 ** 2)
print(f"當前 Metal 記憶體佔用: {mb_used:.2f} MB")
```

---

##### mx.metal.clear_cache() 清理 Metal 釋放快取池釋放實體記憶體

- **使用時機**：在完成大型模型微調、載入全新權重或單次巨大推論後，顯式清理底層配置器快取池中的閒置記憶體區塊，歸還系統實體 RAM。
- **語法**：`mx.metal.clear_cache()`
- **回傳值**：
  - `None`：原位清理記憶體快取。

```python
import mlx.core as mx

# 執行記憶體密集計算後清理暫存池
mx.metal.clear_cache()
```

---

# 隨機數生成與機率取樣 (mlx.core.random)

在機器學習與深度學習中，隨機數廣泛應用於模型權重初始化、資料隨機增強、Dropout 遮罩、大語言模型取樣生成以及強化學習探索。

MLX 提供極具特色的**雙軌隨機數生成架構 (Dual PRNG Design)**：
- **全域有狀態模式 (NumPy / PyTorch 風格)**：透過 `mx.random.seed()` 設定全域狀態，後續呼叫隨機函式時無需手動傳遞密鑰，簡潔直觀。
- **純函數無狀態金鑰模式 (JAX 風格)**：透過 `mx.random.key()` 顯式建立金鑰張量，每次隨機運算必須傳入金鑰並透過 `mx.random.split()` 衍生新金鑰。這種設計徹底消除全域隱式副作用，確保多執行緒平行計算、分散式訓練與 `mx.compile()` JIT 編譯下的 100% 確定性可重現。

> **[生動比喻]**：  
> - **全域有狀態模式**：像放在公共大廳的自動發牌機，任何人去抽牌都會推進機器的同一個內部指針。若兩位廚師 (多執行緒) 同時伸手，拿到的牌序會相互干擾。  
> - **純函數無狀態金鑰模式**：像每位廚師各自領取一本嚴密編號的「密碼金鑰本」。每次取樣前撕下當前頁並衍生出下一頁密碼，彼此完全獨立、絕不打架，即便時空重來也能發出完全一致的牌序！

---

##### mx.random.seed() 設定全域隨機種子

- **使用時機**：在一般腳本或快速原型開發中設定全域偽隨機數生成器 (PRNG) 狀態，確保後續隨機操作之實驗可重現性。
- **語法**：`mx.random.seed(seed)`
- **參數說明**：
  - `seed`：非負整數種子數值。
- **回傳值**：
  - `None`：原位更新框架內部全域 PRNG 狀態。

```python
import mlx.core as mx

# 1. 設定全域隨機種子
mx.random.seed(42)

# 2. 生成隨機數 (兩次執行產出相同數值)
x = mx.random.normal((2, 2))
print(x)  # 假設印出：array([[0.102, -0.428], [0.891, 0.145]], dtype=float32)
```

---

##### mx.random.key() 建立純函數式 PRNG 密鑰

- **使用時機**：在需要嚴格無副作用、平行資料處理或函數式變換 (`mx.vmap`、`mx.compile`) 的場景下，初始化無狀態隨機金鑰。
- **語法**：`mx.random.key(seed)`
- **參數說明**：
  - `seed`：非負整數種子數值。
- **回傳值**：
  - `mx.array`：長度為 2、型別為 `uint32` 的不可變隨機金鑰張量。

```python
import mlx.core as mx

# 建立無狀態 PRNG 初始金鑰
key = mx.random.key(2026)
print(key)  # 假設印出：array([0, 2026], dtype=uint32)
```

---

##### mx.random.split() 分割無狀態 PRNG 子金鑰

- **使用時機**：從現有 PRNG 金鑰中衍生分割出多個相互獨立的新金鑰，供多個神經網路分支、不同批次樣本或後續步驟使用。
- **語法**：`mx.random.split(key, num=2)`
- **參數說明**：
  - `key`：現有的 PRNG 密鑰張量。
  - `num`：整數，需要分割生成的子金鑰數量（預設為 2）。
- **回傳值**：
  - `tuple[mx.array, ...]` 或 `mx.array`：由獨立新金鑰組成的元組或堆疊張量。

```python
import mlx.core as mx

key = mx.random.key(1234)

# 1. 分割為兩個獨立金鑰: 一個供當前操作，一個留給下一步
key, subkey = mx.random.split(key)

# 2. 使用子金鑰進行取樣
sample = mx.random.normal((3,), key=subkey)
print(sample.shape)  # 假設印出：(3,)
```

---

##### mx.random.normal() 常態分佈高斯取樣

- **使用時機**：深度學習模型權重隨機初始化、向輸入特徵注入高斯雜訊擾動、擴散模型 (Diffusion Models) 前向加噪過程時使用。
- **語法**：`mx.random.normal(shape, loc=0.0, scale=1.0, dtype=mx.float32, key=None)`
- **參數說明**：
  - `shape`：輸出多維張量維度大小序列（如 `(32, 128)`）。
  - `loc`：高斯分佈之平均值 $\mu$（預設為 0.0）。
  - `scale`：高斯分佈之標準差 $\sigma$（預設為 1.0）。
  - `dtype`：可選浮點數型別（預設為 `mx.float32`）。
  - `key`：可選無狀態 PRNG 金鑰；若未提供則自動調用全域 PRNG 狀態。
- **回傳值**：
  - `mx.array`：符合常態分佈 $\mathcal{N}(\mu, \sigma^2)$ 之多維隨機張量。

```python
import mlx.core as mx

# 1. 有狀態全域取樣
w = mx.random.normal((64, 32), loc=0.0, scale=0.02)
print(w.shape)  # 假設印出：(64, 32)

# 2. 無狀態金鑰取樣 (完全可重現)
key = mx.random.key(99)
noise = mx.random.normal((10,), key=key)
print(noise.dtype)  # 假設印出：float32
```

---

##### mx.random.multivariate_normal() 多元常態高斯分佈取樣

- **使用時機**：自多元高斯分佈中採樣多維隨機向量。在變分自編碼器 (VAE) 潛在空間潛變數取樣、高斯混合模型 (GMM)、貝氏神經網絡權重採樣，以及模擬多維具相關性特徵時使用。
- **語法**：`mx.random.multivariate_normal(mean, cov, shape=None, dtype=mx.float32, key=None)`
- **參數說明**：
  - `mean`：均值向量張量，形狀為 `(..., D)`。
  - `cov`：對稱半正定協方差矩陣張量，形狀為 `(..., D, D)`。
  - `shape`：可選輸出前綴批次維度元組。輸出總體形狀為 `(*shape, *mean.shape[:-1], D)`。
  - `dtype`：可選浮點數資料型別（預設為 `mx.float32`）。
  - `key`：可選無狀態 PRNG 密鑰張量。
- **回傳值**：
  - `mx.array`：符合多元常態分佈 $\mathcal{N}(oldsymbol{\mu}, oldsymbol{\Sigma})$ 之隨機張量。

```python
import mlx.core as mx

# 1. 定義 2 維均值向量與協方差矩陣 (兩特徵呈現正相關 0.8)
mean = mx.array([0.0, 1.0])
cov = mx.array([[1.0, 0.8], [0.8, 2.0]])

# 2. 批次採樣 100 筆 2 維相關特徵樣本
samples = mx.random.multivariate_normal(mean, cov, shape=(100,))
mx.eval(samples)
print(samples.shape)  # 假設印出：(100, 2)

# 3. 模擬 VAE 潛在空間採樣: (Batch=4, Latent_dim=8)
mu = mx.zeros((4, 8))
sigma_cov = mx.broadcast_to(mx.eye(8), (4, 8, 8))
z = mx.random.multivariate_normal(mu, sigma_cov)
print(z.shape)  # 假設印出：(4, 8)
```

---

##### mx.random.truncated_normal() 截斷常態分佈取樣

- **使用時機**：用於神經網路（特別是 Transformer、BERT 與 ViT）權重矩陣初始化。能剔除超過指定標準差閾值的極端離群值，避免早期梯度爆炸並維持訓練穩定度。
- **語法**：`mx.random.truncated_normal(lower, upper, shape=None, dtype=mx.float32, key=None)`
- **參數說明**：
  - `lower`：截斷下界純量或張量（例如 $-2.0$）。
  - `upper`：截斷上界純量或張量（例如 $+2.0$）。
  - `shape`：輸出張量維度形狀。
  - `dtype`：可選浮點數資料型別（預設為 `mx.float32`）。
  - `key`：可選無狀態 PRNG 金鑰。
- **回傳值**：
  - `mx.array`：數值嚴格限定於 $[lower, upper]$ 區間內之常態分佈張量。

```python
import mlx.core as mx

# 初始化 Linear 層權重，數值截斷在 [-2.0, 2.0] 標準差區間
weights = mx.random.truncated_normal(lower=-2.0, upper=2.0, shape=(128, 64))
mx.eval(weights)
print(mx.min(weights) >= -2.0)  # 假設印出：array(True, dtype=bool)
print(mx.max(weights) <= 2.0)   # 假設印出：array(True, dtype=bool)
```

---

##### mx.random.uniform() 均勻分佈隨機取樣

- **使用時機**：隨機超參數網格搜尋、權重均勻初始化（如 Xavier Uniform 或 He Uniform）、資料擴增隨機縮放因子生成時使用。
- **語法**：`mx.random.uniform(low=0.0, high=1.0, shape=None, dtype=mx.float32, key=None)`
- **參數說明**：
  - `low`：區間下界數值（包含，預設 0.0）。
  - `high`：區間上界數值（不包含，預設 1.0）。
  - `shape`：輸出張量維度形狀。
  - `dtype`：可選浮點數型別。
  - `key`：可選無狀態 PRNG 金鑰。
- **回傳值**：
  - `mx.array`：數值均勻落在 $[low, high)$ 區間之隨機張量。

```python
import mlx.core as mx

# 1. 生成 [0, 1) 區間連續隨機數
u = mx.random.uniform(shape=(4, 4))
print(u.shape)  # 假設印出：(4, 4)

# 2. 模擬 Xavier 均勻初始化: U(-a, a)
limit = (6.0 / (128 + 64)) ** 0.5
xavier_w = mx.random.uniform(low=-limit, high=limit, shape=(128, 64))
```

---

##### mx.random.bernoulli() 伯努利二項分佈遮罩

- **使用時機**：自訂神經網路 Dropout 正規化層、隨機深度 (Stochastic Depth) 以及強化學習二元探索動作生成時使用。
- **語法**：`mx.random.bernoulli(p=0.5, shape=None, key=None)`
- **參數說明**：
  - `p`：成功輸出為 `True` (1) 的機率值（介於 0.0 至 1.0 之間，預設 0.5）。
  - `shape`：輸出張量維度大小。
  - `key`：可選無狀態 PRNG 金鑰。
- **回傳值**：
  - `mx.array`：型別為 `bool` 的二元隨機遮罩張量。

```python
import mlx.core as mx

# 模擬 Dropout 率為 0.2 (保留機率 p = 0.8)
keep_prob = 0.8
mask = mx.random.bernoulli(p=keep_prob, shape=(8, 16))

# 應用遮罩並依倒數比例縮放維持期望值
x = mx.ones((8, 16))
x_dropped = (x * mask) / keep_prob
print(x_dropped.shape)  # 假設印出：(8, 16)
```

---

##### mx.random.randint() 離散均勻整數取樣

- **使用時機**：自資料集中隨機抽樣索引、影像處理中隨機裁切空間起始座標生成、隨機類別標籤生成時使用。
- **語法**：`mx.random.randint(low, high, shape=None, dtype=mx.int32, key=None)`
- **參數說明**：
  - `low`：取樣整數下界（包含）。
  - `high`：取樣整數上界（不包含）。
  - `shape`：輸出張量形狀。
  - `dtype`：可選整數型別（如 `mx.int32`、`mx.uint32`）。
  - `key`：可選無狀態 PRNG 金鑰。
- **回傳值**：
  - `mx.array`：數值落在 $[low, high)$ 區間的離散整數隨機張量。

```python
import mlx.core as mx

# 隨機抽取 5 個介於 0 到 999 之間的樣本索引
sample_indices = mx.random.randint(0, 1000, shape=(5,))
print(sample_indices)  # 假設印出：array([384, 12, 905, 417, 62], dtype=int32)
```

---

##### mx.random.categorical() 類別分佈對數機率採樣

- **使用時機**：大語言模型文本自回歸生成中的**核心取樣函式**（配合 Temperature 溫度係數對 Logits 進行多項式隨機採樣）、強化學習離散 Policy 動作選取時使用。
- **語法**：`mx.random.categorical(logits, axis=-1, num_samples=None, key=None)`
- **參數說明**：
  - `logits`：未歸一化對數機率分數張量（內部會自動以 Softmax 換算機率進行無偏取樣）。
  - `axis`：執行類別抽樣的特徵軸向（預設為最後一軸 `-1`）。
  - `num_samples`：每個分佈獨立抽取的樣本個數。
  - `key`：可選無狀態 PRNG 金鑰。
- **回傳值**：
  - `mx.array`：型別為 `uint32` 的抽樣類別索引張量。

```python
import mlx.core as mx

# 1. 模型對 4 個詞彙預測的原始 Logits
logits = mx.array([1.5, 3.8, 0.2, 5.1])

# 2. 引入溫度係數調整多樣性
temperature = 0.7
scaled_logits = logits / temperature

# 3. 依據對數機率進行隨機採樣
token_id = mx.random.categorical(scaled_logits)
print(token_id.item())  # 假設印出：3 (高機率選中分數最高的第 3 號 Token)
```

---

##### mx.random.permutation() 隨機置換與陣列洗牌

- **使用時機**：在機器學習訓練流程中，每個訓練週期 (Epoch) 開始前隨機打亂樣本順序以消除樣本相關性，或建立隨機投影矩陣時使用。
- **語法**：`mx.random.permutation(x, axis=0, key=None)`
- **參數說明**：
  - `x`：整數 $N$（自動生成 $0$ 到 $N-1$ 的隨機排列），或輸入之多維張量。
  - `axis`：沿著哪一個維度執行隨機置換（預設為 0）。
  - `key`：可選無狀態 PRNG 金鑰。
- **回傳值**：
  - `mx.array`：重排後的整數排列向量或洗牌後的多維張量。

```python
import mlx.core as mx

# 1. 產生 0 至 9 的隨機打亂索引
indices = mx.random.permutation(10)
print(indices)  # 假設印出：array([7, 2, 9, 0, 4, 1, 8, 3, 6, 5], dtype=uint32)

# 2. 沿第 0 軸打亂特徵矩陣
features = mx.array([[1, 2], [3, 4], [5, 6]])
shuffled = mx.random.permutation(features, axis=0)
mx.eval(shuffled)
```

---

##### mx.random.gumbel() 耿貝爾分佈取樣

- **使用時機**：實作 Gumbel-Max 採樣技巧或 Gumbel-Softmax 可微分重參數化 (Reparameterization Trick)。能將離散的 Argmax 取樣轉化為平滑連續且可進行梯度反向傳播的 Softmax 形式。
- **語法**：`mx.random.gumbel(shape=None, dtype=mx.float32, key=None)`
- **參數說明**：
  - `shape`：輸出張量維度大小。
  - `dtype`：可選浮點數資料型別（預設為 `mx.float32`）。
  - `key`：可選無狀態 PRNG 金鑰。
- **回傳值**：
  - `mx.array`：符合標準耿貝爾分佈 $G \sim 	ext{Gumbel}(0, 1)$ 之隨機張量。

```python
import mlx.core as mx

logits = mx.array([2.0, 1.0, 0.5])

# 1. 抽取耿貝爾雜訊
gumbel_noise = mx.random.gumbel(shape=logits.shape)

# 2. Gumbel-Softmax 可微分鬆弛採樣
tau = 0.5  # 溫度退火超參數
soft_sample = mx.softmax((logits + gumbel_noise) / tau)
print(soft_sample)  # 假設印出：array([0.884, 0.089, 0.027], dtype=float32)
```

---

##### mx.random.laplace() 拉普拉斯雙指數分佈取樣

- **使用時機**：生成拉普拉斯（雙指數）分佈隨機數。在機器學習中廣泛應用於差分隱私 (Differential Privacy, DP-SGD) 的拉普拉斯機制梯度加噪、貝氏 Lasso（L1 稀疏正則化的機率對偶先驗），以及對極端離群值具備高抗擾能力的重尾分佈建模。
- **語法**：`mx.random.laplace(shape=None, dtype=mx.float32, loc=0.0, scale=1.0, key=None)`
- **參數說明**：
  - `shape`：輸出張量維度大小序列（如 `(32, 64)`）。
  - `dtype`：可選浮點數資料型別（預設為 `mx.float32`）。
  - `loc`：分佈之位置中心參數 $\mu$（預設為 0.0）。
  - `scale`：分佈之尺度多樣性參數 $b > 0$（預設為 1.0）。
  - `key`：可選無狀態 PRNG 密鑰張量。
- **回傳值**：
  - `mx.array`：符合拉普拉斯分佈 $	ext{Laplace}(\mu, b)$ 之隨機張量。

```python
import mlx.core as mx

# 1. 抽取標準拉普拉斯隨機雜訊
noise = mx.random.laplace(shape=(4, 4), loc=0.0, scale=1.0)
print(noise.shape)  # 假設印出：(4, 4)

# 2. 機器學習差分隱私 (DP-SGD) 梯度加噪實作
grad = mx.ones((10, 5))
sensitivity = 0.1  # 梯度 L1 敏感度
epsilon = 1.0      # 隱私預算參數
dp_scale = sensitivity / epsilon

# 注入拉普拉斯噪聲保護訓練樣本隱私
dp_grad = grad + mx.random.laplace(shape=grad.shape, scale=dp_scale)
mx.eval(dp_grad)
```

---

# 機器學習線性代數與張量運算 (mlx.core & linalg)

##### mx.matmul() 與 @ 批次矩陣相乘運算

- **使用時機**：計算神經網路權重前向線性投影、特徵變換或注意力矩陣相乘時使用。
- **語法**：`mx.matmul(a, b)` 或使用運算子 `a @ b`
- **參數說明**：
  - `a`：形狀為 `(..., M, K)` 的多維張量。
  - `b`：形狀為 `(..., K, N)` 的多維張量。
- **回傳值**：
  - `mx.array`：形狀為 `(..., M, N)` 的相乘結果張量。

```python
import mlx.core as mx

# 1. 定義特徵矩陣與權重矩陣
x = mx.random.normal((32, 128))
w = mx.random.normal((128, 64))

# 2. 使用 @ 運算子調用 AMX 矩陣乘法加速
y = x @ w
mx.eval(y)
print(y.shape)  # 假設印出：(32, 64)
```

---

##### mx.linalg.norm() 矩陣與向量範數計算

- **使用時機**：計算向量長度、正則化損失懲罰項或在訓練迴圈中執行全域梯度裁剪 (Gradient Clipping) 時使用。
- **語法**：`mx.linalg.norm(a, ord=None, axis=None, keepdims=False)`
- **參數說明**：
  - `a`：輸入多維張量。
  - `ord`：範數階數（預設為 2 即 Euclidean L2 範數；可指定 `1` 或 `"fro"`）。
  - `axis`：計算軸向（整數或元組；若為 `None` 則展平計算全域範數）。
  - `keepdims`：是否保留原始維度大小。
- **回傳值**：
  - `mx.array`：指定軸向上的範數張量。

```python
import mlx.core as mx

# 1. 計算單一特徵向量的 L2 範數 (長度)
v = mx.array([3.0, 4.0])
l2_norm = mx.linalg.norm(v)
print(l2_norm.item())  # 假設印出：5.0

# 2. 模擬多梯度張量之全域梯度範數計算
grad = mx.random.normal((100, 50))
total_norm = mx.linalg.norm(grad)
max_norm = 1.0

# 3. 梯度裁剪: 若超過閾值則按比例縮小
clipped_grad = mx.where(total_norm > max_norm, grad * (max_norm / total_norm), grad)
mx.eval(clipped_grad)
```

---

##### mx.linalg.inv() 方陣求逆與線性方程求解

- **使用時機**：計算常態線性回歸閉合解、協方差矩陣求逆或多元高斯分佈機率密度計算時使用。
- **語法**：`mx.linalg.inv(a)`
- **參數說明**：
  - `a`：形狀為 `(..., N, N)` 的非奇異二維方陣或批次方陣。
- **回傳值**：
  - `mx.array`：對應的逆矩陣張量，滿足 `a @ inv(a) = I`。

```python
import mlx.core as mx

# 1. 建立 2x2 可逆方陣
mat = mx.array([[1.0, 2.0], [3.0, 4.0]])

# 2. 求解逆矩陣
inv_mat = mx.linalg.inv(mat)

# 3. 驗證矩陣乘積為單位矩陣
identity = mat @ inv_mat
mx.eval(identity)
print(identity)  # 假設印出：array([[1, 0], [0, 1]], dtype=float32)
```

---

##### mx.einsum() 愛因斯坦求和約定張量運算

- **使用時機**：需要以一行優雅語法處理跨維度轉置、矩陣批量相乘、對角線提取或多頭注意力張量縮併時使用。
- **語法**：`mx.einsum(equation, *operands)`
- **參數說明**：
  - `equation`：求和約定字串（例如 `"bth,bsh->bts"` 代表多頭注意力分數矩陣乘法）。
  - `*operands`：參與縮併的張量運算元清單。
- **回傳值**：
  - `mx.array`：依據約定縮併後的新張量。

```python
import mlx.core as mx

# 批次為 4，查詢序列長度 8，鍵長度 16，特徵維度 32
q = mx.random.normal((4, 8, 32))
k = mx.random.normal((4, 16, 32))

# 利用 einsum 計算點積注意力矩陣: bth, bsh -> bts
attn_scores = mx.einsum("bth,bsh->bts", q, k)
mx.eval(attn_scores)
print(attn_scores.shape)  # 假設印出：(4, 8, 16)
```

---

# 機器學習數值聚合與機率函數 (Math & Statistics)

##### mx.softmax() 沿指定軸機率分佈歸一化

- **使用時機**：將分類模型輸出的 Logits 或自注意力矩陣分數轉換為數值總和為 1.0 的平滑機率分佈時使用。
- **語法**：`mx.softmax(a, axis=-1)`
- **參數說明**：
  - `a`：輸入任意維度之實數張量。
  - `axis`：執行 Softmax 歸一化的軸向（預設為最後一軸 `-1`）。
- **回傳值**：
  - `mx.array`：與輸入形狀完全相同且元素皆介於 (0, 1) 的機率張量。

```python
import mlx.core as mx

# 1. 分類模型原始未歸一化預測分數
logits = mx.array([2.0, 1.0, 0.1])

# 2. 轉換為機率分佈
probs = mx.softmax(logits)
print(probs)  # 假設印出：array([0.659, 0.242, 0.098], dtype=float32)
print(mx.sum(probs).item())  # 假設印出：1.0
```

---

##### mx.logsumexp() 數值穩定對數指數和運算

- **使用時機**：自訂數值穩定的交叉熵損失函數、計算機率圖模型邊際分佈或對數配分函數時使用。底層內部自動減去最大值，徹底防止 $e^x$ 浮點數溢位 (Overflow)。
- **語法**：`mx.logsumexp(a, axis=None, keepdims=False)`
- **參數說明**：
  - `a`：輸入張量。
  - `axis`：求和軸向（預設為全域）。
  - `keepdims`：是否保持原始維度階數。
- **回傳值**：
  - `mx.array`：數值穩定後的對數指數和純量或張量。

```python
import mlx.core as mx

# 即使輸入數值極大 (傳統 exp(1000) 會直接噴 inf 溢位)
large_logits = mx.array([1000.0, 1001.0, 1002.0])

# logsumexp 原生具備防溢位優化
result = mx.logsumexp(large_logits)
print(result.item())  # 假設印出：1002.4076 (計算精準，絕不溢位崩潰)
```

---

##### mx.clip() 數值截斷與梯度防爆裁剪

- **使用時機**：限制資料數值上下限邊界、對數運算前防止輸入 0（取極小值 epsilon）、或針對單一梯度進行數值截斷時使用。
- **語法**：`mx.clip(a, a_min=None, a_max=None)`
- **參數說明**：
  - `a`：輸入多維張量。
  - `a_min`：允許的下界數值（低於此值會被替換為 `a_min`）。
  - `a_max`：允許的上界數值（高於此值會被替換為 `a_max`）。
- **回傳值**：
  - `mx.array`：截斷後數值範圍落在指定區間內的張量。

```python
import mlx.core as mx

x = mx.array([-5.0, 0.5, 10.0])

# 限制在 [0.0, 1.0] 區間
clipped = mx.clip(x, a_min=0.0, a_max=1.0)
print(clipped)  # 假設印出：array([0, 0.5, 1], dtype=float32)
```

---

##### mx.topk() 提取前 K 大元素數值與索引

- **使用時機**：大語言模型自回歸生成時進行 Top-K 隨機採樣、分類任務計算 Top-K 準確率或推薦系統取最高推薦分數候選時使用。
- **語法**：`mx.topk(a, k, axis=-1)`
- **參數說明**：
  - `a`：輸入多維張量。
  - `k`：整數，需要保留的最大元素數量。
  - `axis`：操作維度軸向（預設為 `-1`）。
- **回傳值**：
  - `tuple[values, indices]`：由前 K 大數值張量與其對應原始索引張量組成的元組。

```python
import mlx.core as mx

# 詞彙表中各 Token 的未歸一化 Logits
logits = mx.array([0.1, 4.5, 1.2, 8.9, 3.2])

# 提取前 2 大最可能的候選 Token 數值與對應索引
top_values, top_indices = mx.topk(logits, k=2)
print(top_indices)  # 假設印出：array([3, 1], dtype=uint32)
print(top_values)   # 假設印出：array([8.9, 4.5], dtype=float32)
```

---

# Metal 硬體級深度加速算子 (mlx.core.fast)

Apple 團隊在 `mlx.core.fast` 模組中實作了數個專門針對 Apple Silicon 架構最佳化的 Metal 底層融合著色核。這些算子將記憶體頻寬需求降至理論極限，是 MLX 運行現代大模型速度能比肩甚至超越專用加速卡的技術核心。

##### fast.scaled_dot_product_attention() Metal 融合縮放點積注意力

- **使用時機**：訓練或推論 Transformer 架構時計算自注意力機制 (Self-Attention) 的**最高效核心算子**。底層將矩陣乘法、縮放因子、遮罩掩碼與 Softmax 合併在單一 GPU 核心循環中，不將完整的 $N 	imes N$ 注意力矩陣實體寫入記憶體，顯存消耗降為 $O(N)$。
- **語法**：`fast.scaled_dot_product_attention(queries, keys, values, scale=None, mask=None)`
- **參數說明**：
  - `queries`：查詢張量，形狀為 `(B, n_q_heads, L, D)`。
  - `keys`：鍵張量，形狀為 `(B, n_k_heads, S, D)`。
  - `values`：值張量，形狀為 `(B, n_k_heads, S, D)`。
  - `scale`：縮放因子純量（預設為 `1.0 / sqrt(D)`）。
  - `mask`：可選因果注意力遮罩 (Causal Mask) 或填充掩碼張量。
- **回傳值**：
  - `mx.array`：形狀為 `(B, n_q_heads, L, D)` 的注意力輸出張量。

```python
import mlx.core as mx
import mlx.core.fast as fast

# 批次 2，頭數 8，序列長度 128，頭維度 64
q = mx.random.normal((2, 8, 128, 64))
k = mx.random.normal((2, 8, 128, 64))
v = mx.random.normal((2, 8, 128, 64))

# 調用硬體融合加速注意力
out = fast.scaled_dot_product_attention(q, k, v)
mx.eval(out)
print(out.shape)  # 假設印出：(2, 8, 128, 64)
```

---

##### fast.rope() 旋轉位置編碼硬體融合加速

- **使用時機**：現代大語言模型（如 Llama 3、Mistral、Qwen 2.5、Gemma）在計算注意力前，對 Query 與 Key 向量施加旋轉位置編碼 (Rotary Positional Embedding) 時使用。底層直接以融合著色核完成交錯複數旋轉運算。
- **語法**：`fast.rope(x, dims, traditional=False, base=10000.0, scale=1.0, offset=0)`
- **參數說明**：
  - `x`：輸入張量，形狀為 `(B, n_heads, L, D)`。
  - `dims`：需要套用旋轉位置編碼的維度數量（通常等於頭維度 `D`）。
  - `traditional`：布林值，設定為 `False` 代表採用現代交錯旋轉模式。
  - `base`：RoPE 頻率基底（現代模型常用 `10000.0` 或 `500000.0`）。
  - `offset`：當前 Token 在上下文中的起始絕對位置索引。
- **回傳值**：
  - `mx.array`：注入旋轉位置編碼後之張量。

```python
import mlx.core as mx
import mlx.core.fast as fast

# 模擬單一 Query 向量，形狀: (批次 1, 頭數 4, 序列長度 16, 頭維度 64)
q = mx.random.normal((1, 4, 16, 64))

# 套用硬體融合旋轉位置編碼
q_roped = fast.rope(q, dims=64, base=10000.0, offset=0)
mx.eval(q_roped)
print(q_roped.shape)  # 假設印出：(1, 4, 16, 64)
```

---

##### fast.rms_norm() 硬體融合均方根層歸一化

- **使用時機**：現代大型語言模型大多捨棄了傳統帶有均值漂移與偏置的 LayerNorm，全面轉向運算更輕快的 RMSNorm。`fast.rms_norm` 在單個 Metal 核心內同步完成平方和統計、開根號與權重縮放，比 Python 手寫層提速數倍。
- **語法**：`fast.rms_norm(x, weight, eps=1e-5)`
- **參數說明**：
  - `x`：輸入張量，形狀為 `(..., D)`。
  - `weight`：形狀為 `(D,)` 的可學習縮放係數向量。
  - `eps`：極小常數數值，防止分母除以零。
- **精度轉換特性（就算是 32 也會轉到 16 再轉回）**：在底層 Metal 著色核心中，即使張量以 32-bit (FP32) 傳入，硬體管線為了記憶體頻寬與吞吐最佳化，也會先轉到 16-bit 進行高效傳輸與縮放；但平方和統計累積 $\sum x_i^2$ 則自動提升至 32-bit (float32) 進行高精度累積以防溢位，最後再轉回相應精度。
- **回傳值**：
  - `mx.array`：經過均方根正規化後之張量。

```python
import mlx.core as mx
import mlx.core.fast as fast

hidden_dim = 1024
x = mx.random.normal((2, 64, hidden_dim))
w = mx.ones((hidden_dim,))

# 硬體融合極速歸一化
normed = fast.rms_norm(x, weight=w, eps=1e-6)
mx.eval(normed)
print(normed.shape)  # 假設印出：(2, 64, 1024)
```

---

##### fast.metal_kernel() 定義客製化 Metal 著色核心

- **使用時機**：當現有高階算子無法滿足特定數值運算需求，需要直接以 C++ 語法編寫 Metal Shading Language (MSL) 著色核心進行硬體級加速時使用。
- **語法**：`fast.metal_kernel(name, input_names, output_names, source, header="")`
- **參數說明**：
  - `name`：著色核心函式名稱。
  - `input_names`：輸入參數名稱字串列表。
  - `output_names`：輸出參數名稱字串列表。
  - `source`：MSL 核心運算主體 C++ 字串。
  - `header`：可選巨集或自訂輔助函式標頭字串。
- **回傳值**：
  - `Callable`：可直接接收 MLX 張量並提交至 Metal 佇列執行的自訂算子函式。

```python
import mlx.core as mx
import mlx.core.fast as fast

# 定義自訂 ReLU 客製化 Metal 著色核心
custom_relu = fast.metal_kernel(
    name="my_relu",
    input_names=["x"],
    output_names=["out"],
    source="out = max(x, (float)0.0);"
)

inp = mx.array([-2.0, 0.0, 3.0])
out = custom_relu(inputs=[inp], template=[("T", mx.float32)])
mx.eval(out[0])
print(out[0])  # 假設印出：array([0, 0, 3], dtype=float32)
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
  - `inputs`：可選參數樹，指定在快取檢查時追蹤的有狀態輸入。
  - `outputs`：可選參數樹，指定函數執行完畢後更新的狀態輸出。
- **回傳值**：
  - `Callable`：已編譯為 Metal 核心之高效執行函數。

```python
import mlx.core as mx
import mlx.nn as nn
import mlx.optimizers as optim

model = nn.Linear(4, 2)
optimizer = optim.AdamW(learning_rate=1e-3)

def loss_fn(model, x, y):
    return mx.mean((model(x) - y) ** 2)

# 編譯整個訓練步階，自動融合神經前向傳播、反向求導與權重更新
@mx.compile
def step(x, y):
    loss_and_grad = nn.value_and_grad(model, loss_fn)
    loss, grads = loss_and_grad(model, x, y)
    optimizer.update(model, grads)
    return loss

x = mx.random.normal((16, 4))
y = mx.random.normal((16, 2))
loss = step(x, y)
mx.eval(model.parameters(), optimizer.state)
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

##### mx.checkpoint() 梯度檢查點重計算節省顯存

- **使用時機**：在訓練深層 Transformer 或超長上下文序列時使用。以前向傳播重計算代替記憶體留存，在反向傳播計算梯度時動態重構中間層激活值，將顯存峰值需求降低數倍。
- **語法**：`mx.checkpoint(fn)`
- **參數說明**：
  - `fn`：需要套用梯度檢查點技術的純函數或前向傳播邏輯。
- **回傳值**：
  - `Callable`：經過記憶體節約包裝後的等價函數。

```python
import mlx.core as mx
import mlx.nn as nn

# 將深層多頭注意力層包裝為梯度檢查點
large_layer = nn.MultiHeadAttention(dims=512, num_heads=8)
checkpointed_forward = mx.checkpoint(large_layer)

x = mx.random.normal((2, 128, 512))
out = checkpointed_forward(queries=x, keys=x, values=x)
mx.eval(out)
print(out.shape)  # 假設印出：(2, 128, 512)
```

---

# 神經網路構建與訓練體系 (mlx.nn & mlx.optimizers)

## 什麼是「神經模組實例 (Neural Module Instance)」？

在 MLX 深度學習程式設計中，「神經模組實例」是指透過神經網路類別（如 `nn.Linear`、`nn.Embedding`、`nn.Dropout` 或自訂繼承 `nn.Module` 的架構）實際呼叫建立出來的**具體 Python 物件**。

### 1. 類別 (Class) 與實例 (Instance) 的本質差異

- **類別 (Class / 藍圖)**：定義網路層的前向計算邏輯、參數宣告規則與運算行為。例如 `nn.Linear` 或 `nn.Embedding` 本身只是 Python 類別，尚未分配具體的權重數值與輸入輸出通道維度。
- **實例 (Instance / 具體物件)**：依據類別定義傳入具體尺寸（如 `input_dims=4, output_dims=2`）後，在系統記憶體中所生成的具體物件。例如 `layer = nn.Linear(4, 2)`，此時 `layer` 就是一個線性神經模組實例。

### 2. MLX 模組實例的底層架構：字典特化子類別 (Dict Subclass)

在 Apple MLX 的架構實作中，所有神經模組的基底類別 `nn.Module` 直接繼承自 Python 原生的 `dict` 字典。

這項設計意味著：**一個神經模組實例在底層本質上就是一個支援屬性存取與計算圖綁定的「參數字典」**。當你在實例內部定義 `self.weight = mx.array(...)` 時，該張量不僅以屬性存在，也會自動被註冊為字典鍵值，使得優化器能以純字典方式遍歷與更新權重。

### 3. 神經模組實例封裝的三大核心要素

在記憶體中，一個具體的神經模組實例主要封裝了以下內容：

1. **實體張量權重 (Tensor Weights)**：  
   直接配置於 Apple Silicon 統一記憶體（Unified Memory）中的可學習多維陣列（`mx.array`），例如線性層的 `weight`、`bias`，或是歸一化層的縮放係數向量 `weight`。
2. **結構與超參數配置 (Hyperparameters)**：  
   實例建立時固定的計算規格與常數，例如 `nn.Dropout` 的丟棄機率 `p`，或是 `nn.RMSNorm` 的數值防爆量 `eps`。
3. **動態執行狀態 (Runtime State)**：  
   控制模組行為的布林標記，例如表示模型處於訓練或評估推論模式的 `training` 屬性（在 `Dropout` 中決定是否進行隨機置零，或在 `BatchNorm` 中決定是否累積統計量）。

### 4. 模組實例的兩大關鍵運作特性

- **可調用性 (Callable / Forward Pass)**：  
  所有模組實例皆實作了 `__call__` 魔法方法。因此實例可以像原生 Python 函數一樣被直接調用（例如 `y = layer(x)`），內部會自動提取該實例持有的權重與超參數，並調用底層 Metal 著色核心執行硬體加速運算。
- **樹狀巢狀結構 (Composite / Tree Pattern)**：  
  父模組實例內部可以容納多個子模組實例（例如一個包含多個 `nn.Linear` 與 `nn.Dropout` 的 `nn.Sequential` 實例）。對父實例呼叫生命週期方法（如 `model.eval()`、`model.parameters()`、`model.freeze()` 或 `model.set_dtype()`）時，操作會自動沿著整個樹狀節點遞迴傳遞至底層所有子實例。

```python
import mlx.core as mx
import mlx.nn as nn

# 1. 類別 vs 實例：呼叫類別建立具體實例
layer_instance = nn.Linear(input_dims=4, output_dims=2)

# 2. 檢驗底層本質：實例既是 nn.Module 也是 Python dict
print(type(layer_instance))                    # <class 'mlx.nn.layers.linear.Linear'>
print(isinstance(layer_instance, nn.Module))   # True
print(isinstance(layer_instance, dict))        # True (底層為 dict 特化子類別)

# 3. 實例內部持有的具體權重張量 (實體分配在 Apple Silicon 統一記憶體中)
print(f"權重形狀: {layer_instance.weight.shape}")  # (2, 4)
print(f"偏置形狀: {layer_instance.bias.shape}")    # (2,)

# 4. 實例可直接作為函數執行前向計算 (呼叫內部 __call__)
x = mx.array([[1.0, 2.0, 3.0, 4.0]])
output = layer_instance(x)
print(f"輸出張量維度: {output.shape}")  # (1, 2)

# 5. 實例持有的動態生命週期狀態
print(f"初始訓練模式: {layer_instance.training}")  # True
layer_instance.eval()
print(f"切換後訓練模式: {layer_instance.training}")  # False
```

---

##### nn.Module 神經網路模型抽象基底類別

- **使用時機**：自訂深度學習架構、卷積網路或 Transformer 子模組時的繼承起點。MLX 中所有的神經網路層（如 `nn.Linear`、`nn.Embedding`、`nn.Dropout` 等）均繼承自 `nn.Module`（底層繼承自 Python `dict`）。
- **語法**：`class MyModel(nn.Module): ...`
- **物件核心方法與成員 (Methods & Attributes)**：
  - **前向計算與生命週期**：
    - `__init__()`：宣告內部神經網路層、子模組與初始張量參數。
    - `__call__(*args, **kwargs)`：定義前向傳播 (Forward Pass) 計算邏輯，直接以 `model(x)` （用**物件**）調用。
    - `train(mode=True)`：切換訓練模式（遞迴作用於所有子模組，啟用 `Dropout` 隨機遮罩與動態統計），回傳 `self`（支援鏈式呼叫）。
    - `eval()`：切換評估推論模式（等價於 `train(mode=False)`），關閉隨機丟棄行為，回傳 `self`。
    - `training`（屬性）：布林值，查詢模組當前是否處於訓練模式。
  - **參數與狀態管理**：
    - `parameters()`：遞迴回傳當前模組所有可學習權重張量（`mx.array`）的巢狀結構字典。
    - `trainable_parameters()`：遞迴回傳僅包含未凍結（可更新）權重的巢狀字典。
    - `freeze(recurse=True, keys=None, strict=False)`：凍結模組權重，使其不參與自動微分梯度計算（支援以 `keys` 指定特定名稱如 `"bias"`）。
    - `unfreeze(recurse=True, keys=None, strict=False)`：解凍模組權重，恢復梯度計算。
    - `state`（屬性）：模組狀態字典的即時參照（reference），對其內容進行修改會直接反映在模組自身。
    - `update(parameters, strict=True)`：以給定參數結構字典原位替換更新模組參數（供優化器步階更新或追蹤器設定）。
    - `apply(map_fn, filter_fn=None)`：對所有符合條件的權重張量套用自訂映射轉換函式並立即更新（例如全域權重裁剪）。
    - `set_dtype(dtype, predicate=None)`：將模組參數（預設針對所有浮點型張量）轉換為指定精度型別（如 `mx.float16`、`mx.bfloat16`）。
  - **模組架構巡訪與操作**：
    - `modules()`：回傳此模組及其內部所有巢狀子模組的扁平清單（`list[nn.Module]`）。
    - `named_modules()`：回傳子模組點記法名稱路徑與實例的元組清單（`list[tuple[str, nn.Module]]`，如 `("encoder.fc", Linear)`）。
    - `children()`：回傳此模組直接包含的一級子模組字典（鍵名為屬性名，值為子模組物件）。
    - `leaf_modules()`：回傳不包含其他子模組的最底層葉節點模組字典。
    - `update_modules(modules, strict=True)`：以字典方式動態替換子模組（熱插拔更換層或掛載 LoRA 適配器）。
    - `apply_to_modules(apply_fn)`：對自身及所有子模組遞迴執行自訂操作函式 `apply_fn(path, module)`。
  - **權重持久化與序列化**：
    - `save_weights(file)`：將模組所有權重保存至本機檔案（支援 `.safetensors` 與 `.npz`）。
    - `load_weights(file_or_weights, strict=True)`：自 `.safetensors`、`.npz` 檔案或 `(name, array)` 列表還原載入權重。
- **回傳值**：
  - 模型實例物件。

```python
import mlx.core as mx
import mlx.nn as nn

class CustomTransformerBlock(nn.Module):
    def __init__(self, dims: int, num_heads: int):
        super().__init__()
        self.attn = nn.MultiHeadAttention(dims, num_heads)
        self.ln1 = nn.LayerNorm(dims)
        self.ffn = nn.Sequential(
            nn.Linear(dims, dims * 4),
            nn.GELU(),
            nn.Linear(dims * 4, dims)
        )
        self.ln2 = nn.LayerNorm(dims)
        self.drop = nn.Dropout(0.1)

    def __call__(self, x: mx.array, mask: mx.array = None) -> mx.array:
        x = x + self.drop(self.attn(self.ln1(x), self.ln1(x), self.ln1(x), mask))
        x = x + self.drop(self.ffn(self.ln2(x)))
        return x

block = CustomTransformerBlock(dims=64, num_heads=4)

# 1. 訓練模式與評估模式切換
block.eval()
print(f"是否處於訓練模式: {block.training}")  # 印出：False
block.train()

# 2. 檢視子模組與葉節點模組
print("子模組清單:", [name for name, _ in block.named_modules()])
# 印出：[, attn, attn.key_proj, attn.out_proj, attn.query_proj, attn.value_proj, drop, ffn, ffn.layers.0, ffn.layers.1, ffn.layers.2, ln1, ln2]

# 3. 凍結注意力投影權重 (僅訓練 FFN 與 LayerNorm)
block.attn.freeze()
trainable = block.trainable_parameters()
print("注意力權重是否在訓練列表中:", "attn" in trainable)  # 印出：False

# 4. 全域精度轉換為 float16
block.set_dtype(mx.float16)
print(f"FFN 第一層權重精度: {block.ffn.layers[0].weight.dtype}")  # 印出：float16
```

---

##### nn.Linear 線性全連接神經層

- **使用時機**：執行標準的特徵維度投影與仿射線性轉換運算 $y = x W^\top + b$。
- **語法**：`nn.Linear(input_dims, output_dims, bias=True)`
- **參數說明**：
  - `input_dims`：輸入特徵維度大小。
  - `output_dims`：輸出特徵維度大小（**只改變最後一個維度**）。
  - `bias`：布林值，是否包含可學習偏差向量（預設為 `True`）。
- **物件方法與成員 (Methods & Attributes)**：
  - `__call__(x)`：前向傳播。接收末尾維度為 `input_dims` 的張量，回傳投影後維度為 `output_dims` 的張量。內部若有偏置會自動調用底層硬體融合算子 `mx.addmm`。
  - `to_quantized(group_size=64, bits=4, mode="affine", quantize_input=False)`：直接將此線性層轉換為量化線性層。若 `quantize_input=False` 轉為 `QuantizedLinear`（僅權重量化）；若 `quantize_input=True`（支援 `"nvfp4"`、`"mxfp8"`）轉為 `QQLinear`（同時量化權重與輸入激活值）。
  - `weight`（屬性）：形狀為 `(output_dims, input_dims)` 的可學習權重矩陣。
  - `bias`（屬性，選用）：形狀為 `(output_dims,)` 的可學習偏置向量（若初始化設定 `bias=True`）。
  - 關鍵繼承方法：支援 `freeze(keys="bias")`、`set_dtype(mx.float16)`、`parameters()`、`save_weights()`。
- **回傳值**：
  - 線性層實例物件。

```python
import mlx.core as mx
import mlx.nn as nn

layer = nn.Linear(64, 128)
x = mx.random.normal((8, 64))

# 1. 前向線性投影運算
y = layer(x)
print(f"輸出維度: {y.shape}")  # 印出：(8, 128)

# 2. 存取權重矩陣形狀
print(f"權重形狀: {layer.weight.shape}")  # 印出：(128, 64)

# 3. 凍結偏置參數 (僅更新權重矩陣)
layer.freeze(keys="bias")

# 4. 轉換為 4-bit 硬體量化線性層 (權重極致壓縮)
q_layer = layer.to_quantized(group_size=64, bits=4)
print(f"量化層型別: {type(q_layer).__name__}")  # 印出：QuantizedLinear
```

---

##### nn.QuantizedLinear 4-bit 與 8-bit 硬體量化線性層

- **使用時機**：在 Mac 本地低記憶體環境下運行或推論開源大型語言模型（如 Llama 3、Mistral、Qwen）。直接在 Metal 上執行低位元仿射反量化矩陣乘法。
- **語法**：`nn.QuantizedLinear(input_dims, output_dims, bias=False, group_size=64, bits=4, mode="affine")`
- **參數說明**：
  - `input_dims`：輸入特徵維度大小。
  - `output_dims`：輸出特徵維度大小。
  - `bias`：布林值，是否包含偏置向量（預設為 `True`）。
  - `group_size`：量化分組大小（支援 `32`、`64` 或 `128`，預設為 `64`）。
  - `bits`：量化位元寬度（支援 `4` 或 `8`，預設為 `4`）。
  - `mode`：量化演算法模式（支援 `"affine"`、`"mxfp4"`、`"nvfp4"`、`"mxfp8"`）。
- **物件方法與成員 (Methods & Attributes)**：
  - `__call__(x)`：前向傳播。調用底層極速硬體核心 `mx.quantized_matmul`，以打包的整數權重、刻度（scales）與偏置進行反量化矩陣相乘。
  - `from_linear(linear_layer, group_size=64, bits=4, mode="affine")`（類別方法 `classmethod`）：自現有的未量化 `nn.Linear` 實例就地提取權重並量化為 `QuantizedLinear` 物件。
  - `weight`（屬性）：打包儲存的整數量化權重陣列。
  - `scales`（屬性）：分組量化縮放係數張量。
  - `biases`（屬性，選用）：仿射量化分組偏移張量。
  - 繼承特性說明：量化層在初始化時即會自動呼叫 `self.freeze()`，權重不會被列入可訓練參數（`trainable_parameters()` 為空），專為推論加速設計。
- **回傳值**：
  - 硬體量化線性層實例。

```python
import mlx.core as mx
import mlx.nn as nn

# 1. 自既有線性層透過類別方法直接構建 4-bit 量化層
base_linear = nn.Linear(1024, 1024)
qlinear = nn.QuantizedLinear.from_linear(
    base_linear,
    group_size=64,
    bits=4,
    mode="affine"
)

# 2. 前向高速推論
x = mx.random.normal((1, 1024))
out = qlinear(x)
print(f"推論輸出維度: {out.shape}")  # 印出：(1, 1024)

# 3. 檢視量化權重與刻度屬性
print(f"量化權重形狀: {qlinear.weight.shape}, 刻度形狀: {qlinear.scales.shape}")
```

---

##### nn.Embedding 詞嵌入向量查表層

- **使用時機**：自然語言處理 (NLP) 或大語言模型中，將離散 Token ID 映射為連續密集實數特徵向量。
> 保留**輸入張量原本的所有維度**，並在最後面接上一個特徵維度（ `dims` 參數）。
- **語法**：`nn.Embedding(num_embeddings, dims)`
- **參數說明**：
  - `num_embeddings`：詞彙庫大小 (Vocabulary Size)。
  - `dims`：每個詞嵌入向量之特徵維度 (Embedding Dimension)。
- **計算位元精度說明 (預設算出來是 FP32)**：
  - MLX 的 `nn.Embedding` 內部可學習權重矩陣 `self.weight` 預設初始化為 **FP32 (`mx.float32`)** 精度。
  - 因此在前向傳播查表 `embedding(tokens)` 時，無論輸入的整數 Token ID 張量是 `int32` 還是其他整數型別，**算出來的輸出特徵向量位元精度一律為 FP32 (`mx.float32`)**。
  - 若模型整體採用半精度（如 `mx.float16` 或 `mx.bfloat16`）進行推論或訓練，務必顯式對嵌入層呼叫 `embedding.set_dtype(mx.float16)` 或在模型頂層呼叫 `model.set_dtype(...)`，否則嵌入特徵張量會持續以 32-bit 佔用雙倍的快取與記憶體頻寬。
- **物件方法與成員 (Methods & Attributes)**：
  - `__call__(x)`：前向傳播。接收任意形狀的整數索引張量，回傳查表特徵向量陣列 `self.weight[x]`（型別預設為 `float32`）。
  - `as_linear(x)`：將詞嵌入矩陣當作線性層調用，執行 `x @ self.weight.T` 矩陣乘法。在自回歸 Transformer 語言模型中，非常適合用於**權重綁定 (Weight Tying / Tied Embeddings)**，以相同的詞嵌入矩陣直接作為 LM Head 的分類預測投影層，大幅節省顯存。
  - `to_quantized(group_size=64, bits=4, mode="affine")`：回傳近似此嵌入層的高效量化版本 `QuantizedEmbedding`，將龐大的詞彙表嵌入矩陣壓縮為 4-bit 或 8-bit。
  - `weight`（屬性）：形狀為 `(num_embeddings, dims)` 的可學習詞嵌入權重矩陣（預設為 `float32`）。
  - 關鍵繼承方法：`set_dtype(mx.bfloat16)`、`parameters()`、`freeze()`。
- **回傳值**：
  - 詞嵌入層神經模組實例。

```python
import mlx.core as mx
import mlx.nn as nn

# 詞彙表大小 32000，嵌入維度 512
embedding = nn.Embedding(num_embeddings=32000, dims=512)

# 1. 標準輸入 Token ID 前向查表: (Batch=2, Seq=4)
tokens = mx.array([[101, 2054, 2003, 102], [101, 1045, 2066, 102]])
vectors = embedding(tokens)
print(f"嵌入特徵向量維度: {vectors.shape}")  # 印出：(2, 4, 512)

# 2. 驗證查表算出來的位元精度 (預設為 float32 / FP32)
print(f"查表輸出張量精度: {vectors.dtype}")  # 印出：float32

# 3. 調用 as_linear() 執行權重綁定反向投影 (LM Head)
# 將隱藏層表示投影回詞彙表維度的未歸一化對數機率 (Logits)
hidden_states = mx.random.normal((2, 4, 512))
logits = embedding.as_linear(hidden_states)
print(f"LM Head 輸出 Logits 維度: {logits.shape}")  # 印出：(2, 4, 32000)

# 4. 轉換為 4-bit 量化詞嵌入層
q_embedding = embedding.to_quantized(group_size=64, bits=4)
print(f"量化詞嵌入層型別: {type(q_embedding).__name__}")  # 印出：QuantizedEmbedding
```

---

##### nn.QuantizedEmbedding 硬體量化詞嵌入查表層

- **使用時機**：在大語言模型中詞彙量龐大（如 32,000 ~ 150,000+）時，詞嵌入權重往往佔用數百 MB 至數 GB 顯存。使用量化詞嵌入可在幾乎不損推論品質的前提下將嵌入層顯存縮減為 25%，大幅降低整體記憶體負擔。
- **語法**：`nn.QuantizedEmbedding(num_embeddings, dims, group_size=64, bits=4, mode="affine")`
- **參數說明**：
  - `num_embeddings`：詞彙庫大小 (Vocabulary Size)。
  - `dims`：每個詞嵌入向量特徵維度。
  - `group_size`：分組量化大小（常見為 `64` 或 `128`）。
  - `bits`：量化位元寬度（`4` 或 `8`）。
  - `mode`：量化模式（預設 `"affine"`）。
- **物件方法與成員 (Methods & Attributes)**：
  - `__call__(x)`：前向傳播。接收整數 Token ID，自量化權重中即時反量化提取特徵向量。
  - `as_linear(x)`：以量化矩陣乘法將詞嵌入當作線性層輸出投影（計算 `mx.quantized_matmul(x, self.weight.T, ...)`）。
  - `from_embedding(embedding_layer, group_size=64, bits=4, mode="affine")`（類別方法 `classmethod`）：自現有的 `nn.Embedding` 實例轉換並量化為 `QuantizedEmbedding`。
  - `weight`, `scales`, `biases`（屬性）：打包儲存的量化權重陣列與量化刻度。
- **回傳值**：
  - 量化詞嵌入模組實例。

```python
import mlx.core as mx
import mlx.nn as nn

# 1. 建立標準嵌入層並轉為 4-bit 量化詞嵌入
base_emb = nn.Embedding(num_embeddings=10000, dims=256)
q_emb = nn.QuantizedEmbedding.from_embedding(base_emb, group_size=64, bits=4)

# 2. 量化查表推論
tokens = mx.array([12, 45, 999])
feats = q_emb(tokens)
print(f"量化查表維度: {feats.shape}")  # 印出：(3, 256)

# 3. 量化權重綁定輸出投影
logits = q_emb.as_linear(feats)
print(f"量化投影 Logits 維度: {logits.shape}")  # 印出：(3, 10000)
```

---

##### nn.Dropout 隨機神經元丟棄正則化層

- **使用時機**：訓練期隨機將特徵元素以機率 $p$ 置零，防止神經網路過擬合。在評估推論期透過 `model.eval()` 自動直通不丟棄。
- **語法**：`nn.Dropout(p=0.5)`
- **參數說明**：
  - `p`：丟棄機率值（介於 0.0 到 1.0 之間，預設為 0.5）。
- **物件方法與成員 (Methods & Attributes)**：
  - `__call__(x)`：前向傳播。當處於訓練模式（`self.training == True`）時，以伯努利分佈生成隨機二元遮罩，將元素置零並依 $\frac{1}{1-p}$ 縮放維持期望值；當處於評估模式（`self.training == False`）時，直接返回輸入 `x`（零額外運算開銷）。
  - `train(mode=True)`：將 Dropout 層切換至訓練狀態（啟用隨機丟棄），回傳 `self`。
  - `eval()`：將 Dropout 層切換至評估/推論狀態（關閉隨機丟棄，輸入直通），回傳 `self`。
  - `training`（屬性）：布林值，查詢當前是否處於訓練啟用模式。
  - 同系列模組成員：
    - `nn.Dropout2d(p=0.5)`：2D 空間通道級丟棄。輸入形狀為 `(NHWC)` 或 `(HWC)`，隨機將整張特徵圖的某些特徵通道完全置零，特別適合淺層卷積神經網路。
    - `nn.Dropout3d(p=0.5)`：3D 空間通道級丟棄。輸入形狀為 `(NDHWC)` 或 `(DHWC)`，針對 3D 體積資料整塊通道置零。
- **回傳值**：
  - 隨機丟棄神經模組實例。

```python
import mlx.core as mx
import mlx.nn as nn

dropout = nn.Dropout(p=0.2)
x = mx.ones((2, 5))

# 1. 訓練模式: 啟用隨機丟棄並按 1/(1-p) 縮放
dropout.train()
print(f"當前模式狀態: training={dropout.training}")  # 印出：True
y_train = dropout(x)
mx.eval(y_train)

# 2. 評估模式: 關閉 Dropout，輸入完全直通
dropout.eval()
print(f"當前模式狀態: training={dropout.training}")  # 印出：False
y_eval = dropout(x)
print(y_eval)  # 印出：array([[1, 1, 1, 1, 1], [1, 1, 1, 1, 1]], dtype=float32)

# 3. 卷積特徵圖通道級丟棄 (Dropout2d，通道在後 NHWC)
drop2d = nn.Dropout2d(p=0.5)
drop2d.train()
feature_map = mx.ones((2, 8, 8, 16))
out_2d = drop2d(feature_map)
print(f"2D 空間通道丟棄後維度: {out_2d.shape}")  # 印出：(2, 8, 8, 16)
```

---

##### nn.LayerNorm 標準層歸一化神經層

- **使用時機**：經典 Transformer、BERT、ViT 模型中穩定深層特徵分佈。沿特徵維度計算均值與方差並套用可學習仿射縮放與偏置。
- **語法**：`nn.LayerNorm(dims, eps=1e-5, affine=True)`
- **參數說明**：
  - `dims`：需要歸一化的特徵維度大小。
  - `eps`：極小常數，防止除以零。
  - `affine`：布林值，是否包含可學習縮放係數與偏置向量（預設 `True`）。
- **物件方法與成員 (Methods & Attributes)**：
  - `__call__(x)`：前向傳播。沿最後特徵維度計算均值與方差進行歸一化 $y = \frac{x - \mathrm{E}[x]}{\sqrt{\mathrm{Var}[x] + \epsilon}} \gamma + \beta$。
  - `weight`（屬性）：可學習縮放係數向量 $\gamma$，形狀為 `(dims,)`（若 `affine=True`）。
  - `bias`（屬性）：可學習偏移向量 $\beta$，形狀為 `(dims,)`（若 `affine=True`）。
  - 關鍵繼承方法：`freeze(keys="bias")`、`set_dtype(mx.float16)`、`parameters()`。
- **回傳值**：
  - 層歸一化模組實例。

```python
import mlx.core as mx
import mlx.nn as nn

ln = nn.LayerNorm(dims=64)
x = mx.random.normal((4, 16, 64))

# 1. 前向計算歸一化
out = ln(x)
mx.eval(out)
print(f"歸一化輸出維度: {out.shape}")  # 印出：(4, 16, 64)

# 2. 檢視與操作可學習參數
print(f"縮放權重形狀: {ln.weight.shape}, 偏置形狀: {ln.bias.shape}")

# 3. 凍結偏置參數 (僅微調仿射縮放)
ln.freeze(keys="bias")
```

---

##### nn.RMSNorm 均方根層歸一化層

- **使用時機**：構建現代大型語言模型（Llama 3、Mistral、Qwen、DeepSeek）的核心正規化層。相較於傳統 `LayerNorm`，捨棄了均值計算 $\mathrm{E}[x]$ 與偏置參數 $\beta$，僅依據特徵的均方根 (Root Mean Square) 進行縮放：
  $$y = \frac{x}{\sqrt{\frac{1}{d} \sum_{i=1}^d x_i^2 + \epsilon}} \odot \gamma$$
  在 Apple Silicon 統一記憶體架構下，能減少約 30% ~ 50% 的中繼讀寫頻寬消耗，具備更極致的推論吞吐量與數值穩定性。
- **語法**：`nn.RMSNorm(dims, eps=1e-5)`
- **參數說明**：
  - `dims`：需要進行均方根歸一化的特徵維度大小（通常對應模型的隱藏層維度 `hidden_size`）。
  - `eps`：極小常數浮點數，防止除以零（必須為大於 0 的正數，預設為 `1e-5`，現代大模型常用 `1e-5` 或 `1e-6`）。
- **物件方法與成員 (Methods & Attributes)**：
  - **前向計算與硬體調度（就算是 32 也會轉到 16 再轉回）**：
    - `__call__(x)`：前向傳播計算。接收最後一維為 `dims` 的任意多維張量（如 `(Batch, Seq_Len, Hidden_Dim)`）。
    - **Metal 底層精度轉換機制（就算輸入為 32 也會轉到 16 再轉回）**：
      - 在 Apple Silicon 的 Metal 硬體計算管線中，為了達到最高記憶體頻寬吞吐量，**就算是傳入 32-bit (FP32) 輸入張量，Metal 底層核心在管線傳輸與標準化除法時也會先轉到 16-bit (FP16/BF16) 進行高效吞吐與縮放**。
      - 但在計算均方統計量 $\frac{1}{d} \sum_{i=1}^d x_i^2$ 時，為了防止 16-bit 浮點數發生嚴重的數值溢位（Overflow），Metal 核心會在 GPU 內部**自動提升至 32-bit (float32) 高精度進行累積與開根號**。
      - 完成均方根計算後，再轉回 16-bit 進行後續層的高速傳遞（若原始要求輸出 32-bit 則再轉回 32-bit）。這種「32 $\to$ 16 $\to$ 內部 32 累加 $\to$ 16/32 轉回」的混合精度流動機制，兼顧了硬體極致吞吐與數值防爆穩定性。
  - **內部參數與超參數屬性**：
    - `weight`（屬性）：形狀為 `(dims,)` 的可學習仿射縮放係數張量 $\gamma$。初始化時預設為全 1 向量（`mx.ones((dims,))`）。
    - `eps`（屬性）：初始化時設定的數值穩定常數。
  - **參數管理與微調控制 (繼承自 nn.Module)**：
    - `parameters()`：取得包含 `{"weight": self.weight}` 的巢狀結構字典。
    - `trainable_parameters()`：若模組未凍結，回傳包含 `weight` 的字典；若已凍結則回傳空字典。
    - `freeze(keys=None)`：將此層的可學習參數凍結（不參與梯度計算）。在 LoRA 或部分固定微調中，通常將 RMSNorm 固定不更新。
    - `unfreeze(keys=None)`：解除凍結，恢復參與自動微分求導。
  - **精度轉換與序列化 (繼承自 nn.Module)**：
    - `set_dtype(dtype)`：就地將縮放權重 `weight` 轉換為指定精度（如 `mx.float16` 或 `mx.bfloat16`），適配大模型推論環境。
    - `save_weights(file)`：將此層權重獨立儲存至 `.safetensors` 或 `.npz` 格式檔案。
    - `load_weights(file_or_weights, strict=True)`：自權重檔案或權重列表還原載入。
    - `update(parameters, strict=True)`：以字典原位替換更新 `weight` 張量。
    - `state`（屬性）：取得該模組即時狀態字典參照。
  - **模式切換**：
    - `train(mode=True)` / `eval()` / `training`：RMSNorm 屬於完全確定性無狀態計算（無 Dropout 遮罩，亦無 BatchNorm 的動態移動統計量），在訓練與評估模式下前向邏輯完全相同。
- **回傳值**：
  - 均方根層歸一化神經模組實例。

```python
import mlx.core as mx
import mlx.nn as nn

# 1. 初始化 RMSNorm 層 (隱藏維度 512, eps=1e-6)
rms = nn.RMSNorm(dims=512, eps=1e-6)

# 2. 檢視初始權重與超參數屬性
print(f"縮放係數初始形狀: {rms.weight.shape}")  # 印出：(512,)
print(f"初始值全為 1.0: {mx.all(rms.weight == 1.0).item()}")  # 印出：True
print(f"數值穩定常數 eps: {rms.eps}")  # 印出：1e-06

# 3. 前向計算 (多維張量自動沿最後維度歸一化)
x = mx.random.normal((2, 16, 512))
out = rms(x)
mx.eval(out)
print(f"歸一化輸出維度: {out.shape}")  # 印出：(2, 16, 512)

# 4. LoRA 微調場景：凍結 RMSNorm 權重
rms.freeze()
print(f"凍結後可訓練參數數量: {len(rms.trainable_parameters())}")  # 印出：0
rms.unfreeze()

# 5. 半精度模型推論轉換 (bfloat16)
rms.set_dtype(mx.bfloat16)
print(f"權重精度轉換後: {rms.weight.dtype}")  # 印出：bfloat16

# 6. 權重獨立序列化儲存與載入
rms.save_weights("rms_layer.safetensors")
new_rms = nn.RMSNorm(dims=512)
new_rms.load_weights("rms_layer.safetensors")
print(f"載入後權重精度一致: {new_rms.weight.dtype == mx.bfloat16}")  # 印出：True
```

---

##### nn.RoPE 旋轉位置編碼神經模組

- **使用時機**：構建現代主流大語言模型（如 Llama 3、Mistral、Qwen、DeepSeek 等）的標準位置編碼層。相較於早期直接將絕對位置向量與詞嵌入相加的傳統做法，RoPE (Rotary Position Embedding) 將位置資訊透過複數平面旋轉矩陣直接注入至 Query 與 Key 向量中。在計算注意力點積內積時天然具備**相對距離衰減特性**，並原生支援大模型自回歸生成時的 **KV Cache 增量解碼**（透過傳入 `offset` 偏移量）。
- **語法**：`nn.RoPE(dims, traditional=False, base=10000, scale=1.0)`
- **參數說明**：
  - `dims`：需要進行旋轉編碼的特徵維度大小（通常對應單個注意力頭的維度 `head_dim`，或部分旋轉時的指定維度；若輸入張量的特徵維度大於 `dims`，超出維度的部分會保持原樣不變）。
  - `traditional`：布林值。
    - `False`（預設，MLX 與現代開源模型推薦）：將特徵向量前半部分與後半部分配對旋轉（Stride 為半維度），更利於 Apple Silicon GPU 平行向量化執行。
    - `True`（原始論文經典實作）：將相鄰兩兩元素 $[x_0, x_1], [x_2, x_3]$ 配對旋轉。
  - `base`：計算角頻率的基數底數 $\theta$（預設為 `10000`；在現代長上下文模型如 Llama 3 中常設定為 `500000`，Qwen 中設定為 `1000000`）。
  - `scale`：位置縮放因子（預設為 `1.0`，可用於線性位置內插擴展上下文長度）。
- **物件方法與成員 (Methods & Attributes)**：
  - **前向計算與增量解碼**：
    - `__call__(x, offset: int = 0)`：前向旋轉編碼計算。
      - `x`：待旋轉的 Query 或 Key 多維張量，末尾維度必須大於或等於 `dims`。支援 `(Batch, Heads, Seq, Head_Dim)` 或 `(Batch, Seq, Heads, Head_Dim)`。
      - `offset`：整數偏移量，**在 KV Cache 自回歸流式生成時至關重要**！在 Prompt Prefill 階段輸入整段序列（`offset=0`）；在後續逐字 Token 生成階段，每次僅輸入當前新產生的單個 Token（長度為 1），此時設定 `offset = current_seq_len`，確保新 Token 能正確套用其在全局上下文中的絕對旋轉角度。
    - **底層硬體加速**：內部無縫調用 Apple Metal 原生硬體加速算子 `mx.fast.rope(...)`，以硬體融合指令在 Metal 著色核心中極速完成旋轉運算，零中繼暫存與顯存頻寬浪費。
  - **超參數屬性**：
    - `dims`、`traditional`、`base`、`scale`：初始化時儲存的超參數屬性。
  - **參數狀態說明**：
    - RoPE 屬於無可學習參數的純數學確定性變換模組，`parameters()` 回傳為空字典 `{}`，不會佔用模型權重顯存，亦不參與梯度更新。
- **回傳值**：
  - `nn.RoPE` 神經模組實例。

```python
import mlx.core as mx
import mlx.nn as nn

# 1. 初始化 RoPE 模組 (頭維度 64, Llama 3 常用 base=500000)
rope = nn.RoPE(dims=64, base=500000.0)

# 2. Prefill 階段：對完整的 Prompt 序列編碼 (Batch=1, Heads=8, Seq=16, Head_Dim=64)
q_prefill = mx.random.normal((1, 8, 16, 64))
k_prefill = mx.random.normal((1, 8, 16, 64))
q_prefill_rot = rope(q_prefill, offset=0)
k_prefill_rot = rope(k_prefill, offset=0)
print(f"Prefill 輸出形狀: {q_prefill_rot.shape}")  # 印出：(1, 8, 16, 64)

# 3. Decode 增量解碼階段：生成第 17 個 Token，傳入 offset=16
q_step = mx.random.normal((1, 8, 1, 64))
q_step_rot = rope(q_step, offset=16)
print(f"Decode 輸出形狀: {q_step_rot.shape}")     # 印出：(1, 8, 1, 64)

# 4. 部分維度旋轉 (Partial RoPE，例如僅旋轉前 32 維，後 32 維保持不變)
partial_rope = nn.RoPE(dims=32)
q_partial = mx.random.normal((1, 8, 4, 64))
q_partial_rot = partial_rope(q_partial)
assert mx.all(q_partial_rot[..., 32:] == q_partial[..., 32:])
print("部分旋轉驗證成功: 超出 dims 部分保持原樣不變！")
```

---

##### nn.MultiHeadAttention 多頭自注意力神經層

- **使用時機**：搭建 Transformer 編碼器、解碼器或各類序列特徵提取模型時使用。內部自動封裝 Query、Key、Value 線性投影、縮放點積注意力與最終輸出線性映射。
- **語法**：`nn.MultiHeadAttention(dims, num_heads, bias=False)`
- **參數說明**：
  - `dims`：輸入與輸出特徵總維度。
  - `num_heads`：注意力頭數量（必須能被 `dims` 整除）。
  - `query_input_dims` / `key_input_dims` / `value_input_dims`：可選的輸入維度（預設為 `dims`）。
  - `bias`：布林值，是否在線性投影中使用偏置（預設為 `False`）。
- **物件方法與成員 (Methods & Attributes)**：
  - `__call__(queries, keys, values, mask=None)`：前向傳播。執行 Q、K、V 線性投影，重塑維度為多頭格式後調用 `mx.fast.scaled_dot_product_attention` 融合硬體核心，最後經輸出投影回傳特徵。
  - `create_additive_causal_mask(N, dtype=mx.float32)`（靜態方法 `staticmethod`）：生成大小為 $(N, N)$ 的下三角加性因果注意力遮罩矩陣（未來位置填入 $-\infty$ 或該 dtype 之極小負數，過去與當前位置填入 0）。自回歸大模型生成時傳入 `mask` 防止注意力看見後續 Token。
  - 內部子模組屬性：
    - `query_proj`：`nn.Linear` 查詢投影層。
    - `key_proj`：`nn.Linear` 鍵投影層。
    - `value_proj`：`nn.Linear` 值投影層。
    - `out_proj`：`nn.Linear` 輸出投影層。
  - 關鍵繼承方法：`freeze()`、`set_dtype()`、`parameters()`。
- **回傳值**：
  - 多頭注意力模組實例。

```python
import mlx.core as mx
import mlx.nn as nn

# 512 維度，8 個注意力頭 (每個頭 64 維)
mha = nn.MultiHeadAttention(dims=512, num_heads=8)

# 1. 調用靜態方法生成加性因果遮罩 (預防自回歸推論看到未來 Token)
seq_len = 16
causal_mask = nn.MultiHeadAttention.create_additive_causal_mask(seq_len)
print(f"因果遮罩形狀: {causal_mask.shape}")  # 印出：(16, 16)

# 2. 帶因果遮罩的前向自注意力計算
x = mx.random.normal((2, seq_len, 512))
out = mha(queries=x, keys=x, values=x, mask=causal_mask)
mx.eval(out)
print(f"多頭注意力輸出維度: {out.shape}")  # 印出：(2, 16, 512)

# 3. 檢視內部子模組投影層
print("投影子模組:", list(mha.children().keys()))
# 印出：[query_proj, key_proj, value_proj, out_proj]
```

---

##### nn.Conv2d 二維卷積神經網路層

- **使用時機**：處理圖像識別、卷積神經網路骨幹 (CNN) 或多模態模型視覺編碼器 (Vision Transformer Patch 投影) 時使用。
- **語法**：`nn.Conv2d(in_channels, out_channels, kernel_size, stride=1, padding=0, dilation=1, groups=1, bias=True)`
- **參數說明**：
  - `in_channels`：輸入通道數。
  - `out_channels`：輸出通道數（卷積核數量）。
  - `kernel_size`：卷積核尺寸（整數或雙整數元組）。
  - `stride`：步長大小（預設 1）。
  - `padding`：邊界填充大小（預設 0）。
  - `bias`：是否使用偏差向量（預設 `True`）。
- **物件方法與成員 (Methods & Attributes)**：
  - `__call__(x)`：接收通道在後（`NHWC`：批次, 高, 寬, 通道）的四維影像特徵張量執行二維卷積運算。
  - `weight`（屬性）：卷積核權重張量，形狀為 `(out_channels, kH, kW, in_channels)`。
  - `bias`（屬性，選用）：偏置向量，形狀為 `(out_channels,)`（若 `bias=True`）。
  - 關鍵繼承方法：`freeze(keys="bias")`、`set_dtype(mx.float16)`、`parameters()`。
- **回傳值**：
  - 二維卷積層模組實例。

```python
import mlx.core as mx
import mlx.nn as nn

# 輸入 3 通道 (RGB)，輸出 16 通道，3x3 卷積核
conv = nn.Conv2d(in_channels=3, out_channels=16, kernel_size=3, padding=1)

# 注意：MLX 預設使用通道在後的維度佈局 (Batch, Height, Width, Channels)
img = mx.random.normal((1, 32, 32, 3))

# 1. 前向卷積特徵提取
feat = conv(img)
mx.eval(feat)
print(f"卷積輸出形狀: {feat.shape}")  # 印出：(1, 32, 32, 16)

# 2. 檢視卷積核張量形狀
print(f"卷積權重矩陣形狀: {conv.weight.shape}")  # 印出：(16, 3, 3, 3)

# 3. 凍結偏置參數
conv.freeze(keys="bias")
```

---

##### nn.Sequential 順序式神經網路容器

- **使用時機**：模型結構為線性前向串聯結構時，免除繁瑣的手動 forward 宣告。
- **語法**：`nn.Sequential(*layers)`
- **參數說明**：
  - `*layers`：依序執行的神經網路層或可調用物件序列。
- **物件方法與容器操作 (Methods & Container Operations)**：
  - `__call__(x)`：依順序逐層調用內部註冊的模組或函數。
  - `layers`（屬性）：內部子模組串列（`list[nn.Module]`）。
  - 容器特有操作：
    - 索引存取：`model.layers[0]` 可直接存取特定層。
    - 切片與長度：`len(model.layers)` 取得層數，支援列表切片。
    - 動態替換：支援原位修改 `model.layers[i] = new_layer` 或呼叫 `model.update_modules({"layers": {"0": new_layer}})`。
    - 遞迴繼承方法：呼叫 `model.train()`、`model.eval()`、`model.freeze()`、`model.set_dtype()` 時會自動遞迴作用於內部所有子層。
- **回傳值**：
  - 順序神經網路容器實例。

```python
import mlx.core as mx
import mlx.nn as nn

# 1. 構建前向順序網路容器
model = nn.Sequential(
    nn.Linear(64, 128),
    nn.ReLU(),
    nn.Dropout(0.2),
    nn.Linear(128, 10)
)

# 2. 容器層索引存取與層數檢視
print(f"網路容器總層數: {len(model.layers)}")  # 印出：4
print(f"首層型別: {type(model.layers[0]).__name__}")  # 印出：Linear

# 3. 前向計算
x = mx.random.normal((2, 64))
out = model(x)
print(f"輸出形狀: {out.shape}")  # 印出：(2, 10)

# 4. 全域遞迴評估模式切換 (自動傳遞至內部的 Dropout)
model.eval()
print(f"內部 Dropout 是否為訓練模式: {model.layers[2].training}")  # 印出：False

# 5. 動態熱插拔替換第 0 層
model.layers[0] = nn.Linear(64, 128, bias=False)
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

##### nn.losses.mse_loss() 均方誤差回歸損失函式

- **使用時機**：數值型目標預測回歸任務、自編碼器影像特徵重建或擴散模型去噪損失評估時使用。
- **語法**：`nn.losses.mse_loss(predictions, targets, reduction="mean")`
- **參數說明**：
  - `predictions`：模型前向輸出預測連續值張量。
  - `targets`：真實目標連續標籤張量。
  - `reduction`：縮減模式（`"none"`、`"mean"` 或 `"sum"`）。
- **回傳值**：
  - `mx.array`：均方誤差損失張量。

```python
import mlx.core as mx
import mlx.nn as nn

preds = mx.array([2.5, 0.0, 1.2])
targets = mx.array([3.0, -0.5, 1.0])

mse = nn.losses.mse_loss(preds, targets)
print(mse.item())  # 假設印出：0.1800
```

---

##### nn.losses.binary_cross_entropy() 二元交叉熵損失函式

- **使用時機**：二元決策分類、點擊率預估 (CTR) 或多標籤分類 (Multi-label Classification) 任務時使用。
- **語法**：`nn.losses.binary_cross_entropy(inputs, targets, reduction="mean")`
- **參數說明**：
  - `inputs`：介於 0 與 1 之間的預測機率值張量（通常經 Sigmoid 映射）。
  - `targets`：真實二元目標標籤張量（元素值為 0 或 1）。
  - `reduction`：縮減模式（`"none"`、`"mean"` 或 `"sum"`）。
- **回傳值**：
  - `mx.array`：二元交叉熵損失張量。

```python
import mlx.core as mx
import mlx.nn as nn

probs = mx.array([0.8, 0.1, 0.9])
labels = mx.array([1.0, 0.0, 1.0])

bce = nn.losses.binary_cross_entropy(probs, labels)
print(bce.item())  # 假設印出：0.1446
```

---

##### model.parameters 模型可學習權重樹狀字典

- **使用時機**：獲取神經網路中所有可學習權重之巢狀字典，傳遞給優化器進行梯度更新、檢查點保存或權重分析。
- **語法**：`model.parameters()`
- **回傳值**：
  - `dict`：包含模組權重與偏置張量的嵌套樹狀字典。

```python
import mlx.nn as nn

model = nn.Linear(4, 2)

# 獲取權重字典
params = model.parameters()
print(list(params.keys()))  # 假設印出：['weight', 'bias']
print(params["weight"].shape)  # 假設印出：(2, 4)
```

---

##### model.freeze() 凍結模型模組權重參數

- **使用時機**：在遷移學習 (Transfer Learning)、骨幹特徵提取器固定或 LoRA 微調時，將基礎模型權重設定為不可更新狀態。
- **語法**：`model.freeze(recurse=True, keys=None)`
- **參數說明**：
  - `recurse`：布林值，是否遞迴凍結所有子模組（預設為 `True`）。
  - `keys`：指定凍結的特定權重鍵名列表（若為 `None` 則凍結全部）。
- **回傳值**：
  - `None`：原位標記參數為非訓練狀態。

```python
import mlx.nn as nn

model = nn.Sequential(nn.Linear(10, 20), nn.Linear(20, 2))

# 凍結第一層線性權重
model.layers[0].freeze()

# 僅有第二層處於可訓練狀態
trainable_params = model.trainable_parameters()
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

##### optim.cosine_decay() 餘弦退火學習率排程器

- **使用時機**：訓練深度學習模型與大語言模型時動態平滑衰減學習率。初期維持較大學習率快速收斂，後期隨餘弦曲線平緩遞減至極小值。
- **語法**：`optim.cosine_decay(init, decay_steps, end=0.0)`
- **參數說明**：
  - `init`：起始學習率數值。
  - `decay_steps`：排程衰減總步數。
  - `end`：學習率衰減終止數值（預設為 0.0）。
- **回傳值**：
  - `Callable`：接收步數整數並傳回當前學習率浮點數的排程函式。

```python
import mlx.optimizers as optim

# 宣告餘弦退火排程：從 1e-3 平滑衰減至 1e-5，共 1000 步
lr_schedule = optim.cosine_decay(init=1e-3, decay_steps=1000, end=1e-5)

# 於各訓練 Step 取得動態學習率
current_lr = lr_schedule(500)
print(f"第 500 步學習率: {current_lr:.6f}")
```

---

##### mlx.utils.tree_map() 遞迴對樹狀結構張量套用變換

- **使用時機**：對神經網路權重字典或梯度巢狀樹狀結構中的每一個張量葉節點，統一且遞迴地套用指定純函數（如全域梯度裁剪、權重移動平均 EMA）。
- **語法**：`mlx.utils.tree_map(fn, *trees)`
- **參數說明**：
  - `fn`：接受張量輸入並回傳新張量的純函數。
  - `*trees`：一或多個具有相同鍵名嵌套結構的字典或元組。
- **回傳值**：
  - `dict` 或 `tuple`：結構完全相同但葉節點皆經過轉換的新樹狀結構。

```python
import mlx.core as mx
import mlx.utils as utils

grads = {
    "layer1": {"w": mx.array([10.0, -5.0]), "b": mx.array([2.0])},
    "layer2": {"w": mx.array([-8.0, 15.0])}
}

# 遞迴將所有梯度截斷在 [-3.0, 3.0] 區間內
clipped_grads = utils.tree_map(lambda x: mx.clip(x, -3.0, 3.0), grads)
print(clipped_grads["layer1"]["w"])  # 假設印出：array([3, -3], dtype=float32)
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

##### mlx_lm.convert() 將 Hugging Face 模型權重轉為 MLX 與量化格式

- **使用時機**：將社群原始 PyTorch 或 Safetensors 模型轉換為 MLX 原生權重格式，並可選擇一併執行 4-bit 或 8-bit 量化，以利在本地 Mac 高效運行。
- **語法**：`mlx_lm.convert(hf_path, mlx_path=None, quantize=False, q_bits=4, q_group_size=64)`
- **參數說明**：
  - `hf_path`：原始模型之 Hugging Face 倉庫 ID 或本機權重目錄。
  - `mlx_path`：轉換後 MLX 模型輸出目錄（預設為 `mlx_model`）。
  - `quantize`：布林值，是否啟用權重量化。
  - `q_bits`：量化位元寬度（常見為 4 或 8）。
  - `q_group_size`：量化分組大小（預設為 64）。
- **回傳值**：
  - `None`：原位輸出轉換與量化後的模型權重目錄。

```python
# 可於 Python 腳本調用或直接使用 CLI 命令列轉換
from mlx_lm import convert

# 將原始 16-bit 模型量化為 4-bit MLX 專用模型
convert(
    hf_path="meta-llama/Meta-Llama-3-8B-Instruct",
    mlx_path="Llama-3-8B-Instruct-4bit",
    quantize=True,
    q_bits=4
)
```

---

##### initialize_rope() 動態旋轉位置編碼工廠函式

- **使用時機**：建構或載入現代開源大語言模型時，根據模型的架構配置（特別是 Hugging Face `config.json` 中的 `rope_scaling` 欄位）自動實例化適配的位置編碼層。當模型需要擴展超長上下文視窗並採用 YaRN、Llama 3 頻率縮放或長上下文內插演算法時，使用 `initialize_rope()` 能自動解析字典超參數並組裝對應的 RoPE 模組，避免手動計算頻率分段與內插權重。
- **語法**：`initialize_rope(dims: int, base: float = 10000.0, traditional: bool = False, scaling_config: dict = None, max_position_embeddings: int = None, **kwargs)`
- **參數說明**：

| 參數名稱 | 資料型別 | 預設值 | 說明 |
| :--- | :--- | :--- | :--- |
| **`dims`** | `int` | *(必填)* | 每個注意力頭需要進行旋轉編碼的特徵維度大小 (`head_dim`)。 |
| **`base`** | `float` | `10000.0` | RoPE 角頻率基底常數 \(\theta\)（如 Llama 3 常用 `500000.0`，Qwen 常用 `1000000.0`）。 |
| **`traditional`** | `bool` | `False` | 旋轉實作模式。`False` 為 MLX 推薦的前後半部向量配對；`True` 為相鄰兩兩元素配對。 |
| **`scaling_config`** | `Optional[dict]` | `None` | 位置縮放配置字典（即 Hugging Face 模型組態中的 `rope_scaling`）。若為 `None`，自動回退建立標準 `nn.RoPE`。 |
| **`max_position_embeddings`** | `Optional[int]` | `None` | 模型支援的最大上下文長度上限。 |
| **`kwargs`** | `Any` | `-` | 接收其他非核心模型自訂引數。 |

- **回傳值**：
  - `nn.Module` 神經模組實例：當 `scaling_config` 為 `None` 時回傳標準 `mlx.nn.RoPE`；當設定為 `"yarn"` 時回傳 `YarnRoPE` 實例；若指定其他縮放類型則回傳對應的衍生模組（如 `Llama3RoPE`、`LinearScaledRoPE`）。

### 核心差異：initialize_rope() 與 nn.RoPE 對照

| 比較維度 | `mlx.nn.RoPE` | `mlx_lm.models.rope_utils.initialize_rope()` |
| :--- | :--- | :--- |
| **所屬層次** | `mlx.nn`（MLX 核心基礎神經網路庫） | `mlx_lm.models.rope_utils`（`mlx-lm` 高階 LLM 生態庫） |
| **物件本質** | 具體的神經模組類別 (`class RoPE(nn.Module)`) | 動態工廠函式 (Factory Function) |
| **配置方式** | 僅接收純量數值參數 (`dims`, `base`, `scale`) | 接收完整 `scaling_config` 字典，依組態自動派發 |
| **演算法支援** | 僅支援標準幾何旋轉與純線性縮放 (`scale` 乘數) | 原生支援 **YaRN**、**Llama 3 混合頻率縮放**、**Dynamic NTK** 等進階技術 |
| **回傳結果** | 固定回傳 `nn.RoPE` 模組實例 | 依組態動態回傳 `nn.RoPE`、`YarnRoPE` 或 `Llama3RoPE` 實例 |
| **典型應用** | 自訂小型模型、固定長度傳統注意力網路 | 載入社群長文本開源大模型 (32K / 128K)、長上下文推論與微調 |

### 深入原理：什麼是 YaRN (Yet another RoPE extensioN)

當大語言模型需要將上下文視窗由預訓練長度（如 4,096）外推擴展至更長範圍（如 16,384 或 32,768）時，傳統做法存在明顯缺陷：

1. **直接線性內插 (Linear Interpolation)**：將位置索引均勻壓縮 \(s\) 倍。雖然低頻維度能感知超長距離，但所有高頻維度（波長極短）的旋轉相位也被迫壓縮，導致相鄰 Token 之間的精確相對距離被模糊化，局部文法與注意力敏銳度崩塌。
2. **直接外推 (Extrapolation)**：不做任何縮放。高頻特徵保持精確，但低頻特徵在超過訓練長度時會旋轉到從未見過的高角度區域，造成注意力分數完全失真。
3. **YaRN 解決方案**：依據特徵維度的頻率波長實施三段式自適應縮放：
   - **高頻維度（短波長）**：不進行內插（縮放比 \(s = 1\)），保留原始相鄰相對位置敏銳度。
   - **低頻維度（長波長）**：進行完全線性內插（縮放比 \(s = factor\)），確保旋轉角度不超出預訓練範圍。
   - **中頻過渡維度**：由 `beta_fast` 與 `beta_slow` 門檻值構建平滑過渡斜坡函數，避免頻率邊界突變。
   - **注意力溫度修正 (Attention Temperature Scaling)**：YaRN 額外計算一個溫度縮放係數，抵消上下文擴展時 Softmax 分佈熵增高的失真現象，大幅穩定長序列推論表現。

### 利用 initialize_rope() 實作 YaRN 長上下文擴展

在 `scaling_config` 中指定 `rope_type` 為 `"yarn"`（或相容於 Hugging Face 的配置）：

- `rope_type` 或 `type`：字串，設為 `"yarn"`。
- `factor`：擴展倍率浮點數（目標長度 / 原生預訓練長度，例如 \(16384 / 4096 = 4.0\)）。
- `original_max_position_embeddings`：原生預訓練長度整數（例如 `4096`）。
- `beta_fast`：高頻截至門檻（預設為 `32.0`）。
- `beta_slow`：低頻截至門檻（預設為 `1.0`）。

```python
import mlx.core as mx
import mlx.nn as nn
from mlx_lm.models.rope_utils import initialize_rope

# 1. 定義單個注意力頭特徵維度與頻率基數
head_dim = 64
rope_base = 10000.0

# 2. 定義 YaRN 長上下文擴展組態字典 (將 4K 原生長度擴充 4 倍至 16K)
yarn_scaling_config = {
    "rope_type": "yarn",
    "factor": 4.0,
    "original_max_position_embeddings": 4096,
    "beta_fast": 32.0,
    "beta_slow": 1.0,
}

# 3. 透過工廠函式建立 RoPE 模組 (自動解析 yarn 配置並實例化 YarnRoPE)
rope_layer = initialize_rope(
    dims=head_dim,
    base=rope_base,
    traditional=False,
    scaling_config=yarn_scaling_config,
    max_position_embeddings=16384,
)

# 輸出生成的模組類別名稱
print("模組型別:", type(rope_layer).__name__)  # 假設印出：YarnRoPE

# 4. Prompt Prefill 階段：對長輸入序列進行位置旋轉 (8192 長度)
batch_size, num_heads, seq_len = 1, 8, 8192
q_prefill = mx.random.normal((batch_size, num_heads, seq_len, head_dim))
k_prefill = mx.random.normal((batch_size, num_heads, seq_len, head_dim))

q_prefill_rot = rope_layer(q_prefill, offset=0)
k_prefill_rot = rope_layer(k_prefill, offset=0)
mx.eval(q_prefill_rot, k_prefill_rot)
print("Prefill 旋轉後維度:", q_prefill_rot.shape)  # 假設印出：(1, 8, 8192, 64)

# 5. 自回歸 Decode 增量解碼階段 (KV Cache)：為第 8193 個 Token 編碼 (傳入 offset=8192)
q_step = mx.random.normal((batch_size, num_heads, 1, head_dim))
q_step_rot = rope_layer(q_step, offset=8192)
mx.eval(q_step_rot)
print("Decode 增量 Token 維度:", q_step_rot.shape)  # 假設印出：(1, 8, 1, 64)

# 6. 回退機制驗證：未給予 scaling_config 時自動降級為標準 mlx.nn.RoPE
standard_rope = initialize_rope(
    dims=head_dim,
    base=rope_base,
    scaling_config=None,
)
print("回退模組型別:", type(standard_rope).__name__)  # 假設印出：RoPE
print("回退驗證為 nn.RoPE 實例:", isinstance(standard_rope, nn.RoPE))  # 假設印出：True
```

> **[小提醒]**：`initialize_rope()` 是專屬於 `mlx-lm` 生態的工程輔助工廠函式，主要負責載入第三方預訓練長文本模型（如 Qwen2 32K 或 Llama 3 128K）。若是在純 MLX 專案中自行從零構建固定短長度的神經網絡，直接使用標準 `mlx.nn.RoPE` 能維持最少依賴與最純粹的執行邏輯。

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
- **正向優勢應用**：正因張量具備不可變特性，在神經網路撰寫**殘差連接 (Residual Connection)** 時，可以直接以 `residual = x` 引用保留輸入，完全不必擔心主分支的前向運算會就地修改或覆蓋原始資料，兼具零記憶體拷貝開銷與百分之百的副作用安全。
- **記憶口訣**：張量本質不可變，賦值重組建新圖，殘差連接隨手存，嚴禁就地切片改。

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

---

## 5. 於 GPU 上使用雙精度浮點數 float64 觸發降速或崩潰

- **錯誤症狀**：執行帶有 `float64` 型別的張量運算時，效能雪崩式暴跌，或拋出 Metal 不支援該型別的執行期例外。
- **背後原理**：Apple Silicon 的 GPU 架構專為單精度 (FP32) 與半精度 (FP16 / BF16) 深度學習打造，硬體上並未配備原生 64 位元雙精度 (FP64) 浮點數硬體計算單元。
- **記憶口訣**：Apple GPU 無雙精，預設一律 float32，訓練推論走 bfloat16。

```python
# [錯誤寫法]：強制使用 float64 在 GPU 上運行
a = mx.array([1.0, 2.0], dtype=mx.float64)
b = mx.array([3.0, 4.0], dtype=mx.float64)
c = a @ b  # 降速或引發 Metal 算子執行期異常！

# [正確寫法]：採用硬體原生支援之 float32 或 bfloat16
a = mx.array([1.0, 2.0], dtype=mx.float32)
b = mx.array([3.0, 4.0], dtype=mx.bfloat16)
```

---

## 6. 在無狀態隨機取樣中重複使用未分割的 PRNG Key

- **錯誤症狀**：在資料增強、多步迭代或並行分支計算中，不同步驟產生的隨機數序列完全相同，導致多樣性消失或模型過擬合。
- **背後原理**：MLX 純函數式 PRNG Key 遵循確定性數學映射。只要傳入相同的 Key 張量，隨機函式保證產生完全相同的二進位數值序列。
- **記憶口訣**：金鑰用過即失效，分支平行先 split，一鍵一算保隨機。

```python
# [錯誤寫法]：迴圈中重複傳遞同一個 key
key = mx.random.key(42)
for epoch in range(5):
    noise = mx.random.normal((3,), key=key)  # 每一輪產生的 noise 完全相同！

# [正確寫法]：每輪迭代透過 split 衍生新金鑰
key = mx.random.key(42)
for epoch in range(5):
    key, subkey = mx.random.split(key)
    noise = mx.random.normal((3,), key=subkey)  # 每一輪獲得完全獨立的隨機序列
```

---

## 7. 誤認 np.random.seed 或 torch.manual_seed 能控制 MLX 隨機性

- **錯誤症狀**：在腳本開頭設定了 `np.random.seed(42)` 或 `torch.manual_seed(42)`，但 MLX 模型的權重初始化與隨機採樣結果依舊不可預期且每次變動。
- **背後原理**：MLX 擁有完全獨立於 NumPy 與 PyTorch 的原生 Threefry 計數隨機數生成引擎。外部框架的全域狀態完全無法滲透至 MLX 內部狀態。
- **記憶口訣**：各家隨機各家管，MLX 必設 mx.random.seed，純函數務必傳 Key。

```python
# [錯誤寫法]：僅依賴 NumPy 設定隨機種子
import numpy as np
import mlx.core as mx

np.random.seed(42)
w = mx.random.normal((4, 4))  # 依然不可重現！每次執行結果皆不同

# [正確寫法]：明確呼叫 MLX 專屬 seed 函式
mx.random.seed(42)
w = mx.random.normal((4, 4))  # 完全保證跨次執行精確一致
```

---

## 8. 於 mx.compile 編譯函數中包含隱式副作用或未宣告的狀態變更

- **錯誤症狀**：在被 `@mx.compile` 修飾的函數內部修改全域列表或 Python 變數，二次呼叫時該副作用未被重複執行，或產出過期快取結果。
- **背後原理**：JIT 編譯器在快取命中時會略過 Python 直譯執行，直接分派 Metal 核心計算。若狀態更新未透過純函數引數或顯式宣告傳遞，編譯器無法感知變動。
- **記憶口訣**：編譯函數須純粹，禁止外部隱式改，狀態傳遞走引數。

```python
# [錯誤寫法]：在編譯函數內修改 Python 外部列表
history = []

@mx.compile
def step_bad(x):
    history.append(x.item())  # 二次呼叫命中 JIT 快取時，此行被跳過！
    return x * 2

# [正確寫法]：將函數保持為純函數，或在編譯器外側維護指標狀態
@mx.compile
def step_good(x):
    return x * 2

val = step_good(mx.array(3.0))
history.append(val.item())
```

---

## 9. 評估與推論期忘記切換 model.eval() 導致 Dropout 破壞預測確定性

- **錯誤症狀**：對同一張圖片或同一個 Prompt 輸入進行兩次推論，模型輸出分數忽高忽低且不一致。
- **背後原理**：包含 `nn.Dropout` 等隨機層的模型預設處於訓練模式 (`training=True`)。未呼叫 `model.eval()` 時，部分特徵會持續被隨機置零。
- **記憶口訣**：訓練完畢切評估，model.eval 關隨機，保證推論確定性。

```python
# [錯誤寫法]：訓練完畢直接進行評估
model = nn.Sequential(nn.Linear(10, 20), nn.Dropout(0.5), nn.Linear(20, 2))
# ... 訓練完成 ...
pred1 = model(x_test)
pred2 = model(x_test)  # pred1 與 pred2 不相等！因 Dropout 依然在隨機丟棄特徵

# [正確寫法]：推論前切換為評估模式
model.eval()
pred1 = model(x_test)
pred2 = model(x_test)  # 完全一致！
```
