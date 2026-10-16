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
| **[[#mx.array() 建立 MLX 多維陣列張量\|mx.array()]]** | **建立支援統一記憶體的多維陣列張量** | 資料前處理、張量建立、數值運算起手式 |
| **[[#mx.eval() 強制觸發惰性運算求值\|mx.eval()]]** | **顯式強制計算惰性求值圖譜並釋放中繼節點** | 批次訓練迴圈、釋放圖暫存、取得具體數值 |
| **[[#mx.default_device() 查詢或設定預設運算裝置\|mx.default_device()]]** | **查詢或切換預設執行硬體 (GPU / CPU)** | 效能調優、強制特定算子於 CPU 上執行 |
| **[[#mx.savez() 與 mx.load() 序列化與載入張量權重\|mx.savez() / mx.load()]]** | **將多個張量以 safetensors 或 npz 格式儲存與讀取** | 模型檢查點保存、權重匯出、跨平台參數交換 |
| **[[#mx.concatenate() 沿指定軸拼接張量序列\|mx.concatenate()]]** | **將多個相同維度張量沿指定軸向進行無縫拼接** | 特徵拼接、多模態表示融合、批次資料整合 |
| **[[#mx.split() 沿指定軸切分張量為子陣列清單\|mx.split()]]** | **將單一張量沿指定維度均等或按索引切分為子張量** | 多頭注意力 QKV 投影切分、特徵分塊處理 |
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

##### mx.concatenate() 沿指定軸拼接張量序列

- **使用時機**：在多特徵融合、RNN/LSTM 狀態串聯或沿特徵維度組合多個張量時使用。
- **語法**：`mx.concatenate(arrays, axis=0)`
- **參數說明**：
  - `arrays`：由待拼接張量組成的序列清單。
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
print(combined.shape)  # 假設印出：(2, 6)
```

---

##### mx.split() 沿指定軸切分張量為子陣列清單

- **使用時機**：多頭注意力機制中將大線性投影矩陣切分為 Query、Key、Value，或將資料集切分為多塊時使用。
- **語法**：`mx.split(array, indices_or_sections, axis=0)`
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
print(q.shape)  # 假設印出：(2, 16, 256)
print(k.shape)  # 假設印出：(2, 16, 256)
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

##### nn.Module 神經網路模型抽象基底類別

- **使用時機**：自訂深度學習架構、卷積網路或 Transformer 子模組時的繼承起點。MLX 中所有的神經網路層（如 `nn.Linear`、`nn.Embedding`、`nn.Dropout` 等）均繼承自 `nn.Module`（底層繼承自 Python `dict`）。
- **語法**：`class MyModel(nn.Module): ...`
- **物件核心方法與成員 (Methods & Attributes)**：
  - **前向計算與生命週期**：
    - `__init__()`：宣告內部神經網路層、子模組與初始張量參數。
    - `__call__(*args, **kwargs)`：定義前向傳播 (Forward Pass) 計算邏輯，直接以 `model(x)` 調用。
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
  - `output_dims`：輸出特徵維度大小。
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
- **物件方法與成員 (Methods & Attributes)**：
  - `__call__(x)`：前向傳播。接收任意形狀的整數索引張量，回傳查表特徵向量陣列 `self.weight[x]`。
  - `as_linear(x)`：將詞嵌入矩陣當作線性層調用，執行 `x @ self.weight.T` 矩陣乘法。在自回歸 Transformer 語言模型中，非常適合用於**權重綁定 (Weight Tying / Tied Embeddings)**，以相同的詞嵌入矩陣直接作為 LM Head 的分類預測投影層，大幅節省顯存。
  - `to_quantized(group_size=64, bits=4, mode="affine")`：回傳近似此嵌入層的高效量化版本 `QuantizedEmbedding`，將龐大的詞彙表嵌入矩陣壓縮為 4-bit 或 8-bit。
  - `weight`（屬性）：形狀為 `(num_embeddings, dims)` 的可學習詞嵌入權重矩陣。
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

# 2. 調用 as_linear() 執行權重綁定反向投影 (LM Head)
# 將隱藏層表示投影回詞彙表維度的未歸一化對數機率 (Logits)
hidden_states = mx.random.normal((2, 4, 512))
logits = embedding.as_linear(hidden_states)
print(f"LM Head 輸出 Logits 維度: {logits.shape}")  # 印出：(2, 4, 32000)

# 3. 轉換為 4-bit 量化詞嵌入層
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

- **使用時機**：構建現代大型語言模型（Llama 3、Qwen、DeepSeek）的核心歸一化層。捨棄傳統均值計算與偏置參數，僅依據均方根統計縮放特徵，具備顯著更優的數值穩定度與硬體推論吞吐量。
- **語法**：`nn.RMSNorm(dims, eps=1e-5)`
- **參數說明**：
  - `dims`：歸一化維度大小。
  - `eps`：極小常數，防止除以零。
- **物件方法與成員 (Methods & Attributes)**：
  - `__call__(x)`：前向傳播。直接除以均方根並乘以縮放參數 $y = \frac{x}{\sqrt{\frac{1}{d} \sum x^2 + \epsilon}} \gamma$。底層呼叫 Metal 硬體優化核心。
  - `weight`（屬性）：形狀為 `(dims,)` 的可學習縮放張量 $\gamma$。
  - 關鍵繼承方法：`set_dtype(mx.bfloat16)`、`parameters()`、`update()`。
- **回傳值**：
  - 均方根層歸一化神經模組實例。

```python
import mlx.core as mx
import mlx.nn as nn

# 初始化 RMSNorm 層
norm_layer = nn.RMSNorm(dims=512, eps=1e-6)

# 1. 前向計算均方根歸一化
x = mx.random.normal((4, 32, 512))
out = norm_layer(x)
mx.eval(out)
print(f"RMSNorm 輸出維度: {out.shape}")  # 印出：(4, 32, 512)

# 2. 檢視權重向量
print(f"權重向量形狀: {norm_layer.weight.shape}")  # 印出：(512,)
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
