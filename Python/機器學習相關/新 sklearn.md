# 概念與原理

## 什麼是 Scikit-learn (sklearn)？

Scikit-learn 是 Python 生態系中最權威、最成熟且最廣泛使用的傳統機器學習與資料探勘標準函式庫。

其底層基於 NumPy、SciPy 與 Matplotlib 等科學計算工具構建，專為結構化二維表格型資料（Tabular Data）的監督式學習（回歸、分類）與非監督式學習（分群、降維）而設計。

在人工智慧與資料科學領域中，並非所有問題都需要動用龐大且耗費算力的深度學習神經網路。Scikit-learn 適合處理以下四大核心情境：

1. **結構化、二維表格化資料**：如客戶流失名單、房價特徵表、財務報表與醫療檢驗指標。
2. **中小規模資料集（單機 CPU 運算）**：運算效率極高，幾秒內即可在一般筆電完成數十萬筆資料的訓練與交叉驗證，無需昂貴的 GPU 叢集。
3. **高可解釋性要求（Explainability & Interpretability）**：在金融風控、醫療診斷與法規合規等領域，決策樹與線性模型能清楚展示特徵權重（Weights）與決策邊界，明確說明「為何做出此預測」。
4. **建立基準模型（Baseline Modeling）**：面對任何新的機器學習任務，標準工程實踐是先用 Scikit-learn 幾行程式碼訓練一個基準模型（如 Logistic 回歸或隨機森林），確認資料特徵是否具備足夠預測能力，再決定是否需要投入時間開發複雜模型。

> **[生動比喻]**：  
> - **NumPy / Pandas**：像是「精密的標準零件盒與整理櫃」，負責把原料（數值、字串、表格）收納齊全且分門別類。  
> - **Scikit-learn**：像是「現代化智慧無人工廠的自動生產線」！從進料檢驗（資料清洗與填補）、規格加工（特徵縮放與編碼）、組裝成型（模型訓練）到品管驗收（交叉驗證與指標評估），整條流水線具備完全標準化的模組插槽（Estimator API），任何模組都能自由抽換與串聯。  
> - **PyTorch / TensorFlow**：像是「客製化重工業特種航太工廠」，專為非結構化資料（高解析度影像、即時音訊、超大型語言模型 LLM）打造，需要大量算力與客製化神經層設計。

---

## 核心架構：統一的 Estimator API 設計哲學

Scikit-learn 最偉大的設計在於其優雅且一致的物件導向介面規範（Unified API Design）。所有演算法物件皆遵循三大核心契約：

```text
  ┌──────────────────────────────────────────────────────────────┐
  │                 Scikit-learn Estimator API                   │
  ├──────────────────────────────┬───────────────────────────────┤
  │ 1. Estimator (估計器/學習器) │ model.fit(X, y)               │
  │    從資料中學習規律與模型參數│ 學習完成之參數帶有底線後綴    │
  │                              │ (例如: coef_, intercept_)     │
  ├──────────────────────────────┼───────────────────────────────┤
  │ 2. Transformer (特徵轉換器)  │ transformer.transform(X)      │
  │    轉換資料特徵形狀或分佈    │ transformer.fit_transform(X)  │
  ├──────────────────────────────┼───────────────────────────────┤
  │ 3. Predictor (預測器)        │ y_pred = model.predict(X)     │
  │    對未知特徵進行推論與打分  │ probs = model.predict_proba(X)│
  │                              │ score = model.score(X, y)     │
  └──────────────────────────────┴───────────────────────────────┘
```

1. **一致性（Consistency）**：所有物件共享相同的函式名稱與參數命名習慣，學會一套等於學會全庫。
2. **檢驗性（Inspection）**：由 `fit()` 學習得到的內部參數，其屬性名稱一律帶有**尾端底線（Trailing Underscore）**，如 `model.coef_`、`model.classes_`、`scaler.mean_`，與使用者傳入的超參數（如 `max_depth`）有著涇渭分明的區別。
3. **無激增物件（Non-proliferation of classes）**：資料容器完全遵循標準 NumPy ndarray 與 Pandas DataFrame，不另外發明私有專屬陣列型態。
4. **組合性（Composition）**：前處理轉換器與最終估計器可透過 `Pipeline` 與 `ColumnTransformer` 如樂高積木般自由串聯組裝。

---

## 維度鐵律：特徵矩陣 2D vs 目標標籤 1D 的核心規則

Scikit-learn 對傳入資料的「陣列維度階數（Shape）」有著極為嚴格且一致的鐵律，也是新手最常踩到 `ValueError` 報錯的痛點：

```text
 特徵矩陣 X (所有轉換器與模型：強制 2D)         目標標籤 y (監督式模型答案：通常 1D)
 ┌──────────────┬──────────────┬──────────────┐       ┌──────────────┐
 │  特徵欄位 0  │  特徵欄位 1  │  特徵欄位 2  │       │   標籤答案   │
 ├──────────────┼──────────────┼──────────────┤       ├──────────────┤
 │  樣本 row 0  │  樣本 row 0  │  樣本 row 0  │       │  樣本 row 0  │
 ├──────────────┼──────────────┼──────────────┤       ├──────────────┤
 │  樣本 row 1  │  樣本 row 1  │  樣本 row 1  │       │  樣本 row 1  │
 └──────────────┴──────────────┴──────────────┘       └──────────────┘
  形狀 Shape: (n_samples, n_features)                  形狀 Shape: (n_samples,)
```

1. **特徵矩陣 $X$（所有 Transformer 與 Estimator 的特徵輸入）：強制要求 2D 二維結構**
   - **為什麼？**：Scikit-learn 的轉換器與模型在設計上預設支援「多個特徵欄位」。即使你的資料「只有 1 個特徵欄位」，它的形狀也必須是 `(樣本數, 1)`（二維表格），絕對不能是一維向量 `(樣本數,)`！
   - **常見陷阱與 Pandas 雙中括號解法**：
     - `df["年齡"]`（單括號）$\to$ 回傳 **1D Series**（形狀為 `(n,)`），傳入 `scaler` 或 `model` 會**立即報錯崩潰**：  
       `ValueError: Expected 2D array, got 1D array instead: array=[...]. Reshape your data either using array.reshape(-1, 1)...`
     - `df[["年齡"]]`（**雙中括號，推薦！**）$\to$ 回傳 **2D DataFrame**（形狀為 `(n, 1)`），直接完美相容！
     - 若使用 NumPy，必須手動呼叫 `x.reshape(-1, 1)` 強制升維為 2D。
   - **單筆樣本線上推論（Inference）**：
     - 若要預測單一筆資料，不能傳入 `[15, 200]`（1D），必須傳入 `[[15, 200]]`（2D 矩陣，形狀為 `(1, 2)`）或 `pd.DataFrame({"點擊次數": [15], "停留秒數": [200]})`。

2. **目標標籤 $y$（監督式學習的預測答案）：通常要求 1D 一維結構**
   - 模型的目標答案 $y$ 是一維數列，形狀應為 `(n_samples,)`（例如 `y = df["標籤"]`）。
   - 若誤傳 2D 欄向量 `(n_samples, 1)`（如 `y = df[["標籤"]]`），模型訓練時會跳出警告：  
     `DataConversionWarning: A column-vector y was passed when a 1d array was expected. Please change the shape of y to (n_samples, ), for example using ravel().`
   - **特殊例外（當 $y$ 自身需要做特徵縮放時）**：
     - 因為 `MinMaxScaler` / `StandardScaler` 只收 2D，若想對 $y$ 做標準化，必須先將其升維為 2D（如 `y.values.reshape(-1, 1)`），縮放完畢後若要傳入模型訓練，再用 `.ravel()` 或 `.reshape(-1)` 壓回 1D。

---

## 現代語法升級與版本演進對照表 (Scikit-learn 1.0 ~ 1.4+)

Scikit-learn 近年歷經多次現代化重構，請務必遵循當前推薦標準寫法：

| 功能操作 | 舊式語法 (已棄用 / 易踩坑) | 現代標準語法 (推薦) | 核心差異與說明 |
| :--- | :--- | :--- | :--- |
| **轉換後欄位名稱查詢** | `enc.get_feature_names()` | `enc.get_feature_names_out()` | 舊方法已正式移除，新方法統一於所有 Transformer 支援輸出特徵名 |
| **前處理輸出維持 DataFrame** | 輸出無欄名純 NumPy 陣列 | `scaler.set_output(transform="pandas")` | 1.2+ 重大革新！轉換後可直接保留 Pandas 欄位名稱與索引，告別盲猜欄位 |
| **均方根誤差計算** | `mean_squared_error(..., squared=False)` | `root_mean_squared_error(...)` | 1.4+ 正式獨立為專屬評估函式，`squared=False` 參數已廢棄 |
| **獨熱編碼稀疏矩陣控制** | `OneHotEncoder(sparse=False)` | `OneHotEncoder(sparse_output=False)` | `sparse` 參數名稱在 1.2+ 已正式改為更具語意的 `sparse_output` |
| **梯度提升樹處理缺值** | 手工補值後傳入 `GradientBoosting` | `HistGradientBoostingClassifier` | 現代官方原生推薦！原生支援缺值、類別特徵且運算速度媲美 LightGBM |
| **損失函數指定方式** | 舊式自訂損失別名 | 現代統一標準字串命名 | 全面棄用廢棄別名（如 `log` 統一改為 `log_loss`） |

---

# 核心 API 類別、函式與評估指標字典對照表

| 類別 / 函式 / 屬性名稱 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- |
| **[[#train_test_split() 資料集隨機與分層切割\|train_test_split()]]** | **將原始特徵矩陣與目標標籤依照指定比例切割為訓練集與測試集** | 避免過擬合評估、分層抽樣（Stratified）、機器學習流程起手式 |
| **[[#fit_transform() 特徵轉換核心機制與訓練測試集邊界\|fit_transform()]]** | **同時完成參數統計 (fit) 與資料轉換 (transform)，嚴格僅限訓練集使用** | 特徵工程起手式、訓練集尺度建立、杜絕資料外洩 (Data Leakage) |
| **[[#StandardScaler() 與 MinMaxScaler() 特徵縮放標準化\|StandardScaler() / MinMaxScaler()]]** | **將數值特徵縮放為 Z-Score 常態分佈（均值 0，方差 1）或 [0, 1] 區間** | 梯度下降演算法加速、距離敏感模型（SVM、KNN、MLP、PCA）必備 |
| **[[#OneHotEncoder() 與 OrdinalEncoder() 類別特徵編碼\|OneHotEncoder() / OrdinalEncoder()]]** | **將文字類別特徵轉換為二元虛擬變數（One-Hot）或整數順序編碼** | 類別特徵數值化、處理未知類別（handle_unknown='ignore'）、非數值特徵工程 |
| **[[#SimpleImputer() 缺失值填補\|SimpleImputer()]]** | **使用特定統計量（平均數、中位數、眾數或固定常數）填補缺失值** | 資料清洗、特徵工程前處理管道、防止 NaN 導致演算法報錯 |
| **[[#Pipeline() 與 make_pipeline() 序列管線封裝\|Pipeline() / make_pipeline()]]** | **將多個前處理步驟與最終模型串聯為單一物件，自動按序調用** | 防禦資料外洩（Data Leakage）、簡化交叉驗證與超參數搜尋程式碼 |
| **[[#ColumnTransformer() 與 make_column_transformer() 異質欄位組合轉換器\|ColumnTransformer()]]** | **針對資料表的不同欄位分別套用不同轉換器（數值縮放、類別編碼）** | 真實表格資料處理、混合型態欄位前處理、模組化特徵工程 |
| **[[#set_output(transform="pandas") 轉換器原生回傳 DataFrame\|set_output()]]** | **配置轉換器在呼叫 transform 時直接回傳具備欄位名稱的 DataFrame** | 特徵工程後保留欄位名、管道輸出檢查、銜接下游特徵重要性分析 |
| **[[#LinearRegression() 標準線性回歸\|LinearRegression()]]** | **利用普通最小平方法（OLS）擬合線性回歸模型，求解最佳權重係數** | 連續數值預測基準、計量經濟分析、特徵線性權重解釋 |
| **[[#Ridge() 與 Lasso() 正則化回歸\|Ridge() / Lasso()]]** | **引入 L2（嶺回歸）或 L1（套索回歸）懲罰項，抑制權重過大或稀疏化特徵** | 共線性資料處理、防止模型過擬合、高維資料自動特徵選取（Lasso） |
| **[[#RandomForestRegressor() 隨機森林回歸\|RandomForestRegressor()]]** | **基於 Bagging 集成多棵決策樹，投票平均預測連續數值** | 非線性回歸、抗過擬合能力強、自帶特徵重要性分析、無需特徵縮放 |
| **[[#MLPRegressor() 多層感知神經網路回歸\|MLPRegressor()]]** | **基於反向傳播算法訓練的多層前饋神經網絡回歸模型** | 複雜非線性函數擬合、傳統演算法效果不彰時的神經網絡過渡方案 |
| **[[#LogisticRegression() 邏輯回歸\|LogisticRegression()]]** | **利用 Sigmoid 函數將線性組合映射為機率值，執行二元或多元分類** | 信用違約評分、二元分類基準模型、點擊率（CTR）預估 |
| **[[#DecisionTreeClassifier() 決策樹分類\|DecisionTreeClassifier()]]** | **根據資訊增益或吉尼係數遞迴劃分資料空間，生成白箱樹狀決策規則** | 高業務解釋性需求、快速探索規則邊界、無需特徵標準化 |
| **[[#RandomForestClassifier() 隨機森林分類\|RandomForestClassifier()]]** | **結合自助取樣（Bootstrap）與隨機特徵選取的強大整合分類器** | 表格資料分類首選模型之一、高準確度、抗雜訊強、不平衡資料權重平衡 |
| **[[#SVC() 支持向量機分類\|SVC()]]** | **在高維空間中尋找能最大化兩類間隔的超平面，支援核技巧（RBF）** | 中小樣本高維分類、複雜非線性邊界劃分、文本分類 |
| **[[#HistGradientBoostingClassifier() 高效直方圖梯度提升樹\|HistGradientBoostingClassifier()]]** | **利用特徵直方圖分箱大幅加速的梯度提升決策樹（類似 LightGBM）** | 大規模表格資料分類首選、原生支援 NaN 缺失值與類別特徵、極致運算效能 |
| **[[#KMeans() K-Means 分群演算法\|KMeans()]]** | **非監督式分群，反覆迭代質心將樣本劃分為 K 個緊湊的群集** | 客戶分群畫像、市場區隔（Segmentation）、異常行為探查 |
| **[[#PCA() 主成分分析維度縮減\|PCA()]]** | **利用正交變換將高維相關特徵投影至最大方差的主成分方向** | 資料視覺化降維至 2D/3D、消除多重共線性、高維影像特徵壓縮 |
| **[[#回歸模型評估指標：MSE、RMSE、MAE 與 R² Score\|回歸評估指標]]** | **量化連續預測值與真實數值之間的誤差程度與解釋變異量** | 評估房價/銷量預測品質、MAE 衡量平均誤差、RMSE 放大懲罰離群值 |
| **[[#分類模型評估指標：Accuracy、Precision、Recall 與 F1-Score\|分類評估指標]]** | **從準確率、精確率（查準率）、召回率（查全率）與 F1 全方位評估分類器** | 醫療診斷（強調 Recall）、垃圾郵件（強調 Precision）、不平衡資料評估 |
| **[[#confusion_matrix() 與 classification_report() 混淆矩陣與綜合評估報告\|confusion_matrix() / classification_report()]]** | **以矩陣形式呈現 TP, FP, TN, FN，並產出一鍵式分類評估指標摘要文字報表** | 深入排查各類別混淆誤判情況、多類別分類效能快速診斷 |
| **[[#roc_auc_score() ROC 曲線下面積評估\|roc_auc_score()]]** | **衡量分類模型在所有可能分類閾值下的整體排序辨別能力（0.5 ~ 1.0）** | 評估機率輸出模型、不平衡資料分類器能力比較、不受單一門檻限制 |
| **[[#cross_val_score() 與 KFold / StratifiedKFold 交叉驗證\|cross_val_score() / KFold]]** | **將資料切割為 K 折進行輪流驗證，評估模型泛化能力的穩定性與變異數** | 防禦單次切割偶然性、小樣本資料集嚴謹驗證、模型選型客觀比對 |
| **[[#GridSearchCV() 窮舉網格搜尋\|GridSearchCV()]]** | **在指定的超參數字典網格中窮舉所有組合，搭配交叉驗證找出全局最佳組合** | 尋找模型最優超參數、小範圍超參數微調、全自動最佳模型選拔 |
| **[[#RandomizedSearchCV() 隨機分佈搜尋\|RandomizedSearchCV()]]** | **從連續或離散的參數分佈中隨機抽樣固定次數，以極高效率逼近最佳超參數** | 搜尋空間龐大時的高效調參、兼顧計算資源與探索廣度的首選 |
| **[[#joblib.dump() 與 joblib.load() 模型序列化與載入\|joblib.dump() / joblib.load()]]** | **將訓練完成之模型物件高效序列化存為 .pkl 磁碟檔案，並於生產環境快速復原** | 模型產線部署、API 服務推論對接、耗時訓練結果持久化存檔 |

---

# 資料集劃分與前處理 (Data Preprocessing & Splitting)

資料清洗、缺失補齊、尺度縮放與樣本切分的核心基礎元件。

---

##### train_test_split() 資料集隨機與分層切割

- **使用時機**：在建立模型前，將資料劃分為訓練集（用以學習規律）與獨立測試集（用以評估泛化能力），徹底預防模型死記硬背訓練資料（過擬合）。
- **語法**：`train_test_split(*arrays, test_size=None, train_size=None, random_state=None, shuffle=True, stratify=None)`
- **參數說明**：
  - `*arrays`：特徵矩陣 $X$ 與目標標籤 $y$（可傳入 NumPy 陣列、Pandas DataFrame/Series）。
  - `test_size`：測試集比例（浮點數如 `0.2` 代表 20%）或具體整數筆數。
  - `random_state`：隨機種子整數（固定此值能保證每次執行切分結果完全相同，實驗可重現）。
  - `shuffle`：切割前是否隨機打亂資料（預設 `True`；時間序列資料務必設為 `False`）。
  - `stratify`：分層取樣基準（**分類任務極關鍵**！通常設為 `stratify=y`，確保訓練集與測試集具備完全相同的正負樣本比例）。
- **回傳值**：
  - 切割後的陣列清單：`X_train, X_test, y_train, y_test`。

```python
import pandas as pd
from sklearn.model_selection import train_test_split

df = pd.DataFrame({
    "年齡": [25, 30, 45, 35, 22, 48, 52, 28, 40, 31],
    "年資": [1, 5, 15, 8, 1, 20, 25, 3, 12, 6],
    "薪資": [45000, 60000, 110000, 75000, 40000, 130000, 150000, 52000, 95000, 68000],
    "離職": [0, 0, 1, 0, 1, 1, 1, 0, 0, 0]
})

X = df[["年齡", "年資", "薪資"]]
y = df["離職"]

# 劃分 80% 訓練集與 20% 測試集，並依據離職標籤分層抽樣
X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,
    random_state=42,
    stratify=y
)
print(f"訓練集形狀: {X_train.shape}, 測試集形狀: {X_test.shape}")
# 輸出: 訓練集形狀: (8, 3), 測試集形狀: (2, 3)
```

**`df` 表格結構**：

| Index | 年齡 | 年資 | 薪資 | 離職 |
| :---: | :---: | :---: | :---: | :---: |
| 0 | 25 | 1 | 45000 | 0 |
| 1 | 30 | 5 | 60000 | 0 |
| 2 | 45 | 15 | 110000 | 1 |
| 3 | 35 | 8 | 75000 | 0 |
| 4 | 22 | 1 | 40000 | 1 |
| 5 | 48 | 20 | 130000 | 1 |
| 6 | 52 | 25 | 150000 | 1 |
| 7 | 28 | 3 | 52000 | 0 |
| 8 | 40 | 12 | 95000 | 0 |
| 9 | 31 | 6 | 68000 | 0 |

---

##### fit_transform() 特徵轉換核心機制與訓練測試集邊界

- **使用時機**：在對**訓練集（$X_{train}$）**進行特徵縮放（Scaling）、類別編碼（Encoding）、缺失值填補（Imputing）或降維（PCA）時，一鍵同時完成「統計學習」與「矩陣轉換」。
- **輸入維度限制**：**強制要求 2D 二維結構 (DataFrame / 2D Array)**，形狀為 `(n_samples, n_features)`。若僅有單一特徵欄位，請務必使用雙中括號 `df[['col']]` 或 `.values.reshape(-1, 1)`，否則將觸發 `ValueError`。
- **三者本質定義與職責分工**：
  1. **`fit(X)`（只學不轉）**：
     - 從輸入資料中統計計算並記錄內部轉換參數（例如：`StandardScaler` 計算平均值 $\mu$ 與標準差 $\sigma$；`OneHotEncoder` 記憶類別字典；`SimpleImputer` 計算中位數）。
     - **不改變資料本體**，回傳轉換器物件本身（`self`）。
  2. **`transform(X)`（只轉不學）**：
     - **嚴格沿用先前 `fit()` 所學得的固定參數**，對輸入資料進行數學轉換。
     - 絕不重新計算任何統計量。**測試集（$X_{test}$）與線上推論新資料必須且只能使用此方法**！
  3. **`fit_transform(X)`（先學後轉）**：
     - 等價於先執行 `fit(X)` 再執行 `transform(X)` 的合成操作。
     - **效能優勢**：在特定演算法（如 PCA 或特定 Imputer）中，底層會使用單一優化算法同步完成求解與投影，避免兩度重複遍歷資料，速度更快且更省記憶體。

> **[核心概念剖析：這裡說的「統計學習」到底是什麼意思？]**：  
> 在前處理器（Transformer）的語境下，「學習」並不是指深度學習中複雜的梯度下降或反向傳播，而是指：**「在真正動手修改資料前，轉換器必須先觀察整批資料，計算並記住『轉換公式所需要的基準常數（標竿統計量）』」**。  
> - **`StandardScaler` 的學習**：計算並記住各欄位的 **平均值 $\mu$（`scaler.mean_`）** 與 **標準差 $\sigma$（`scaler.scale_`）**。有了這兩個數字，公式 $z = \frac{x - \mu}{\sigma}$ 才能成立。  
> - **`MinMaxScaler` 的學習**：掃描並記住各欄位的 **最小值 $x_{min}$（`scaler.data_min_`）** 與 **最大值 $x_{max}$（`scaler.data_max_`）**。  
> - **`SimpleImputer` 的學習**：統計並記住要拿來填補空缺的 **中位數或平均數（`imputer.statistics_`）**。  
> - **`OneHotEncoder` 的學習**：遍歷並記住資料中到底有哪幾種 **相異類別選項與排列順序（`encoder.categories_`）**。  
> **生動比喻（全班考試調分）**：  
> 老師要幫全班「依班平均調分」（每個人分數減去班平均）：  
> - **`fit()`**：老師拿出計算機，把全班考卷加總除以人數，算出「全班平均 65 分」，並把 `65` 寫在黑板上記住。這就是「統計學習」，此時尚未更動任何同學的考卷分數。  
> - **`transform()`**：老師拿著記下的 `65`，逐一在每位同學考卷上進行減法（小明 80 分 $\to 80 - 65 = +15$ 分）。  
> - **`fit_transform()`**：老師一邊算平均是 65 分，一邊當場把全班考卷分數扣除 65 分，兩步合一。  
> - **若對測試集（外校轉學生）也做 `fit` 的後果**：等於轉學生自己幾個人單獨算自己的平均來扣分，兩邊的「零分基準點」完全不同，分數根本無法放在同一個模型公平比較！

- **黃金使用邊界原則**：

| 資料集角色 | 該調用哪一個方法？ | 原因與後果 |
| :--- | :---: | :--- |
| **訓練集（$X_{train}$）** | **`fit_transform(X_train)`** | 必須從訓練集建立基準尺度，並轉為模型可訓練的矩陣型態 |
| **測試集（$X_{test}$）** | **`transform(X_test)`** | **嚴格禁止 fit！** 必須以訓練集的尺度為唯一定位坐標，否則造成資料外洩與尺度錯位 |
| **線上真實推論（Production）** | **`transform(X_new)`** | 單筆或批次新資料必須沿用訓練時的參數，不可單獨 fit |

```python
import pandas as pd
from sklearn.preprocessing import StandardScaler

# 模擬切割後的訓練集與測試集
df_train = pd.DataFrame({"年齡": [20, 30, 40], "薪資": [30000, 50000, 70000]})
df_test = pd.DataFrame({"年齡": [25, 35], "薪資": [40000, 60000]})

scaler = StandardScaler()

# 1. [正確操作]：訓練集使用 fit_transform 建立基準並轉換
X_train_scaled = scaler.fit_transform(df_train)
print("訓練集學得之均值 (mean_):", scaler.mean_)
# 輸出: 訓練集學得之均值 (mean_): [   30. 50000.]

# 2. [正確操作]：測試集只能使用 transform 沿用訓練集均值轉換
X_test_scaled = scaler.transform(df_test)
print("測試集正確縮放結果:\n", X_test_scaled.round(2))
# 輸出:
# [[-0.61 -0.61]   (25歲相對於訓練集平均30歲，為 -0.61 個標準差，語意完全正確！)
#  [ 0.61  0.61]]

# 3. [嚴重錯誤示範]：若對測試集也執行 fit_transform：
bad_scaler = StandardScaler()
bad_test_scaled = bad_scaler.fit_transform(df_test)
print("錯誤示範 (尺度崩壞):\n", bad_test_scaled.round(2))
# 輸出:
# [[-1. -1.]       (25歲竟被視為 -1.0！因為它只跟測試集內部的35歲比較，基準完全失真！)
#  [ 1.  1.]]
```

**`df_train` 表格結構**：

| Index | 年齡 | 薪資 |
| :---: | :---: | :---: |
| 0 | 20 | 30000 |
| 1 | 30 | 50000 |
| 2 | 40 | 70000 |

**`df_test` 表格結構**：

| Index | 年齡 | 薪資 |
| :---: | :---: | :---: |
| 0 | 25 | 40000 |
| 1 | 35 | 60000 |

**`X_train_scaled` 訓練集轉換後表格結構**：

| Index | 年齡 | 薪資 |
| :---: | :---: | :---: |
| 0 | -1.22 | -1.22 |
| 1 | 0.0 | 0.0 |
| 2 | 1.22 | 1.22 |

**`X_test_scaled` 測試集沿用訓練集均值轉換後表格結構**：

| Index | 年齡 | 薪資 |
| :---: | :---: | :---: |
| 0 | -0.61 | -0.61 |
| 1 | 0.61 | 0.61 |

---

##### StandardScaler() 與 MinMaxScaler() 特徵縮放標準化

- **使用時機**：當不同特徵欄位的數值尺度相差懸殊時（如年齡 20~60 歲 vs 年薪 3 萬~150 萬元），大數值特徵會主導距離或梯度計算。必須將所有特徵壓縮至相同尺度。
- **輸入維度限制**：**強制要求 2D 二維矩陣 / DataFrame**，形狀為 `(n_samples, n_features)`。若誤傳 1D Series（如 `df['身高']`），會立即拋出 `ValueError: Expected 2D array, got 1D array instead`。請務必使用雙中括號 `df[['身高']]` 或 `.values.reshape(-1, 1)`！
- **數學原理**：
  - **StandardScaler (Z-Score 標準化)**：$z = \frac{x - \mu}{\sigma}$，轉換後各特徵均值為 0、標準差為 1，保留離群值分佈，適合多數演算法。
  - **MinMaxScaler (最大最小縮放)**：$x_{scaled} = \frac{x - x_{min}}{x_{max} - x_{min}}$，將數值嚴格壓縮至 $[0, 1]$ 區間。
- **語法**：
  - `scaler = StandardScaler(with_mean=True, with_std=True)`
  - `scaler = MinMaxScaler(feature_range=(0, 1))`
- **回傳值**：
  - 縮放後的數值矩陣（NumPy ndarray 或 DataFrame）。

```python
import pandas as pd
from sklearn.preprocessing import StandardScaler

df = pd.DataFrame({
    "身高": [160, 175, 180, 165],
    "體重": [55, 70, 85, 60],
    "月薪": [35000, 60000, 120000, 48000]
})

scaler = StandardScaler()
# 1. 訓練集學習均值與標準差並轉換
X_scaled = scaler.fit_transform(df)
print(f"各欄均值: {scaler.mean_.round(1)}")
# 輸出: 各欄均值: [ 170.     67.5  65750. ]

print("標準化後前兩列:\n", X_scaled[:2].round(2))
# 輸出:
# [[-1.27 -0.89 -0.96]
#  [ 0.63  0.18 -0.18]]
```

**`df` 表格結構**：

| Index | 身高 | 體重 | 月薪 |
| :---: | :---: | :---: | :---: |
| 0 | 160 | 55 | 35000 |
| 1 | 175 | 70 | 60000 |
| 2 | 180 | 85 | 120000 |
| 3 | 165 | 60 | 48000 |

**`X_scaled` 標準化後表格結構**：

| Index | 身高 | 體重 | 月薪 |
| :---: | :---: | :---: | :---: |
| 0 | -1.26 | -1.09 | -0.94 |
| 1 | 0.63 | 0.22 | -0.18 |
| 2 | 1.26 | 1.52 | 1.69 |
| 3 | -0.63 | -0.65 | -0.56 |

> **[核心天條：測試集嚴禁呼叫 `fit()`]**：  
> `fit()` 代表從資料中統計學習均值 $\mu$ 與標準差 $\sigma$。在處理測試集或推論真實資料時，必須**嚴格使用 `transform()` 沿用訓練集的統計量**，切勿呼叫 `fit()` 或 `fit_transform()`，否則將造成嚴重的資料外洩（Data Leakage）！

---

##### OneHotEncoder() 與 OrdinalEncoder() 類別特徵編碼

- **使用時機**：機器學習模型底層皆為矩陣數學運算，無法直接解析文字字串。
  - **OneHotEncoder (獨熱編碼)**：適用於**無先後順序的名目類別**（如部門：IT、HR、業務；城市：台北、台中）。每個類別展開為獨立二元虛擬變數（0 或 1）。
  - **OrdinalEncoder (順序編碼)**：適用於**具備高低等級順序的序數類別**（如學歷：高中 0、學士 1、碩士 2、博士 3）。
- **輸入維度限制**：**強制要求 2D 二維矩陣 / DataFrame**，形狀為 `(n_samples, n_features)`。即使只編碼單一文字欄位，也必須使用雙中括號 `df[['部門']]` 而非單括號 `df['部門']`。
- **語法**：`OneHotEncoder(categories='auto', drop=None, sparse_output=False, handle_unknown='ignore')`
- **關鍵參數說明**：
  - `sparse_output`：布林值（現代預設為 `True`，建議設為 `False` 直接輸出易讀的密集陣列）。
  - `handle_unknown`：設為 `'ignore'`（**生產環境必備避坑參數**！當測試集或線上環境遇到訓練集未曾見過的新類別時，自動編碼為全 0，防止程式報錯崩潰）。
  - `drop`：可設為 `'first'` 刪除第一類以避免線性回歸的多重共線性（虛擬變數陷阱 Dummy Variable Trap）。
- **回傳值**：
  - 二維數值矩陣。

```python
import pandas as pd
from sklearn.preprocessing import OneHotEncoder

df = pd.DataFrame({
    "部門": ["研發", "業務", "客服", "研發"],
    "性別": ["男", "女", "女", "男"]
})

ohe = OneHotEncoder(sparse_output=False, handle_unknown="ignore")
encoded = ohe.fit_transform(df[["部門"]])

# 查詢生成之新欄位名稱清單
feature_names = ohe.get_feature_names_out(["部門"])
print(feature_names.tolist())
# 輸出: ['部門_客服', '部門_業務', '部門_研發']
print(encoded)
# 輸出:
# [[0. 0. 1.]
#  [0. 1. 0.]
#  [1. 0. 0.]
#  [0. 0. 1.]]
```

**`df` 表格結構**：

| Index | 部門 | 性別 |
| :---: | :--- | :--- |
| 0 | 研發 | 男 |
| 1 | 業務 | 女 |
| 2 | 客服 | 女 |
| 3 | 研發 | 男 |

**`encoded` 獨熱編碼後表格結構（部門特徵）**：

| Index | 部門_客服 | 部門_業務 | 部門_研發 |
| :---: | :---: | :---: | :---: |
| 0 | 0.0 | 0.0 | 1.0 |
| 1 | 0.0 | 1.0 | 0.0 |
| 2 | 1.0 | 0.0 | 0.0 |
| 3 | 0.0 | 0.0 | 1.0 |

---

##### SimpleImputer() 缺失值填補

- **使用時機**：真實資料常包含空值（NaN、None）。多數經典機器學習模型（線性回歸、SVM、神經網路）遇到缺失值會直接報錯，需透過統計量填補。
- **輸入維度限制**：**強制要求 2D 二維結構** `(n_samples, n_features)`。單欄填補請務必使用雙中括號 `df[['年齡']]`。
- **語法**：`SimpleImputer(missing_values=np.nan, strategy='mean', fill_value=None)`
- **參數說明**：
  - `strategy`：填補策略字串：
    - `'mean'`：平均值（僅限數值欄位，預設）。
    - `'median'`：中位數（抗極端離群值干擾推薦）。
    - `'most_frequent'`：眾數（數值與文字類別欄位皆適用）。
    - `'constant'`：自訂常數（配合 `fill_value` 指定填補字串或數值，如 `'Unknown'` 或 `-1`）。
- **回傳值**：
  - 填補後的數值矩陣。

```python
import numpy as np
import pandas as pd
from sklearn.impute import SimpleImputer

df = pd.DataFrame({
    "年齡": [25.0, np.nan, 35.0, 40.0],
    "評分": [80.0, 90.0, np.nan, 85.0]
})

imputer = SimpleImputer(strategy="mean")
df_imputed = imputer.fit_transform(df)
print(df_imputed.round(1))
# 輸出:
# [[25.  80. ]
#  [33.3 90. ]
#  [35.  85. ]
#  [40.  85. ]]
```

**`df` 表格結構**：

| Index | 年齡 | 評分 |
| :---: | :---: | :---: |
| 0 | 25.0 | 80.0 |
| 1 | `NaN` | 90.0 |
| 2 | 35.0 | `NaN` |
| 3 | 40.0 | 85.0 |

**`df_imputed` 缺失值平均數填補後表格結構**：

| Index | 年齡 | 評分 |
| :---: | :---: | :---: |
| 0 | 25.0 | 80.0 |
| 1 | 33.3 | 90.0 |
| 2 | 35.0 | 85.0 |
| 3 | 40.0 | 85.0 |

---

# 管線工作流與複合轉換器 (Pipelines & ColumnTransformer)

將前處理、特徵轉換與最終模型封裝為不可分割的標準生產線。

---

##### Pipeline() 與 make_pipeline() 序列管線封裝

- **使用時機**：標準化與模型散裝處理極易導致測試集資料洩漏（Data Leakage）。透過 `Pipeline` 將所有步驟捆綁，在呼叫 `fit()`、`predict()` 或交叉驗證時一鍵自動按序執行。
- **語法**：
  - 具名管線：`Pipeline(steps=[('名稱1', 轉換器1), ('名稱2', 模型)])`
  - 便捷管線：`make_pipeline(轉換器1, 模型)`（自動以類別小寫作為步驟名稱）
- **核心規則**：
  - 前面 $N-1$ 個步驟必須為 **Transformer**（實作 `fit()` 與 `transform()`）。
  - 最後一個步驟必須為 **Estimator**（模型或最終轉換器）。

```python
import pandas as pd
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression

df = pd.DataFrame({
    "特徵A": [10, 20, 30, 40, 50],
    "特徵B": [1000, 2500, 1800, 4000, 3200],
    "標籤": [0, 0, 1, 1, 1]
})

X = df[["特徵A", "特徵B"]]
y = df["標籤"]

# 封裝標準化與邏輯回歸模型為單一 Pipeline 物件
pipe = Pipeline([
    ("scaler", StandardScaler()),
    ("classifier", LogisticRegression(random_state=42))
])

# 訓練時自動：scaler.fit_transform(X) -> classifier.fit(...)
pipe.fit(X, y)

# 預測時自動：scaler.transform(X) -> classifier.predict(...)
preds = pipe.predict(X[:2])
print("Pipeline 預測結果:", preds)
# 輸出: Pipeline 預測結果: [0 0]
```

**`df` 表格結構**：

| Index | 特徵A | 特徵B | 標籤 |
| :---: | :---: | :---: | :---: |
| 0 | 10 | 1000 | 0 |
| 1 | 20 | 2500 | 0 |
| 2 | 30 | 1800 | 1 |
| 3 | 40 | 4000 | 1 |
| 4 | 50 | 3200 | 1 |

---

##### ColumnTransformer() 與 make_column_transformer() 異質欄位組合轉換器

- **使用時機**：真實資料表通常同時包含「數值欄位」與「文字類別欄位」。數值欄需要做 `StandardScaler`，類別欄需要做 `OneHotEncoder`。`ColumnTransformer` 能依據欄位名稱分別指定專屬處理工序，最後水平拼接為完整矩陣。
- **語法**：`ColumnTransformer(transformers=[('名稱', 轉換器, 欄位清單)], remainder='drop')`
- **參數說明**：
  - `transformers`：三元元組清單 `('name', transformer, columns)`。
  - `remainder`：其餘未列出的欄位處理方式：
    - `'drop'`：直接丟棄未列出的欄位（預設）。
    - `'passthrough'`：原封不動保留未列出的欄位。
- **回傳值**：
  - 拼接轉換後的特徵矩陣。

```python
import pandas as pd
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder

df = pd.DataFrame({
    "年齡": [25, 40, 35],
    "薪資": [50000, 90000, 75000],
    "城市": ["台北", "台中", "高雄"],
    "部門": ["IT", "HR", "IT"]
})

num_features = ["年齡", "薪資"]
cat_features = ["城市", "部門"]

# 組合異質轉換器
preprocessor = ColumnTransformer(transformers=[
    ("num", StandardScaler(), num_features),
    ("cat", OneHotEncoder(sparse_output=False), cat_features)
])

X_transformed = preprocessor.fit_transform(df)
print("轉換後總特徵欄數:", X_transformed.shape[1])
# 輸出: 轉換後總特徵欄數: 7 (2 個數值縮放欄 + 3 城市獨熱欄 + 2 部門獨熱欄)
```

**`df` 表格結構**：

| Index | 年齡 | 薪資 | 城市 | 部門 |
| :---: | :---: | :---: | :--- | :--- |
| 0 | 25 | 50000 | 台北 | IT |
| 1 | 40 | 90000 | 台中 | HR |
| 2 | 35 | 75000 | 高雄 | IT |

**`X_transformed` 異質組合轉換後表格結構**：

| Index | num__年齡 | num__薪資 | cat__城市_台中 | cat__城市_台北 | cat__城市_高雄 | cat__部門_HR | cat__部門_IT |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| 0 | -1.34 | -1.31 | 0.0 | 1.0 | 0.0 | 0.0 | 1.0 |
| 1 | 1.07 | 1.11 | 1.0 | 0.0 | 0.0 | 1.0 | 0.0 |
| 2 | 0.27 | 0.20 | 0.0 | 0.0 | 1.0 | 0.0 | 1.0 |

---

##### set_output(transform="pandas") 轉換器原生回傳 DataFrame

- **使用時機**：Scikit-learn 1.2+ 的里程碑重大功能！過去 Transformer 轉換後一律輸出冷冰冰的 NumPy 矩陣，所有欄位名與索引全數丟失。配置此設定後，所有轉換器會直接輸出乾淨帶標籤的 Pandas DataFrame！
- **語法**：`transformer.set_output(transform="pandas")`

```python
import pandas as pd
from sklearn.preprocessing import StandardScaler

df = pd.DataFrame({
    "身高": [160.0, 175.0, 180.0],
    "體重": [55.0, 70.0, 85.0]
})

scaler = StandardScaler()
# 開啟原生 Pandas 輸出！
scaler.set_output(transform="pandas")

df_scaled = scaler.fit_transform(df)
print(type(df_scaled))
# 輸出: <class 'pandas.core.frame.DataFrame'>
print(df_scaled.columns.tolist())
# 輸出: ['身高', '體重']
```

**`df` 表格結構**：

| Index | 身高 | 體重 |
| :---: | :---: | :---: |
| 0 | 160.0 | 55.0 |
| 1 | 175.0 | 70.0 |
| 2 | 180.0 | 85.0 |

**`df_scaled` 縮放後原生 DataFrame 表格結構**：

| Index | 身高 | 體重 |
| :---: | :---: | :---: |
| 0 | -1.37 | -1.22 |
| 1 | 0.39 | 0.0 |
| 2 | 0.98 | 1.22 |

---

# 經典監督式學習模型：回歸 (Supervised: Regression)

預測連續數值型目標變數（房價、銷售額、溫度、耗電量）。

> **[維度通則]**：所有回歸模型均嚴格要求**特徵矩陣 $X$ 為 2D** `(n_samples, n_features)`；而**目標標籤 $y$ 通常為 1D** `(n_samples,)`。推論單筆新資料時，也必須包裹為 2D 矩陣 `(1, n_features)`（如 `[[25, 10]]` 或 `pd.DataFrame(...)`）。

---

##### LinearRegression() 標準線性回歸

- **使用時機**：最經典的基準回歸模型。透過最小化殘差平方和（OLS，Ordinary Least Squares）擬合線性超平面。
- **數學公式**：$y = w_1 x_1 + w_2 x_2 + \dots + w_n x_n + b$
- **模型屬性**：
  - `model.coef_`：各特徵之權重係數 $w$（代表該特徵每增加 1 單位，目標值變化的幅度）。
  - `model.intercept_`：截距項 $b$。
- **語法**：`LinearRegression(fit_intercept=True, copy_X=True, n_jobs=None, positive=False)`

```python
import pandas as pd
from sklearn.linear_model import LinearRegression

df = pd.DataFrame({
    "坪數": [20, 25, 30, 35, 40],
    "屋齡": [15, 10, 5, 8, 2],
    "總價萬": [1200, 1600, 2100, 2300, 2900]
})

X = df[["坪數", "屋齡"]]
y = df["總價萬"]

model = LinearRegression()
model.fit(X, y)

print("特徵係數 coef_:", model.coef_.round(2))
# 輸出: 特徵係數 coef_: [ 80.88 -13.24] (坪數每多1坪漲80.88萬，屋齡每多1年跌13.24萬)
print("截距 intercept_:", round(model.intercept_, 2))
# 輸出: 截距 intercept_: -211.76
```

**`df` 表格結構**：

| Index | 坪數 | 屋齡 | 總價萬 |
| :---: | :---: | :---: | :---: |
| 0 | 20 | 15 | 1200 |
| 1 | 25 | 10 | 1600 |
| 2 | 30 | 5 | 2100 |
| 3 | 35 | 8 | 2300 |
| 4 | 40 | 2 | 2900 |

---

##### Ridge() 與 Lasso() 正則化回歸

- **使用時機**：
  - **Ridge (嶺回歸 / L2 正則化)**：當特徵之間存在高度相關性（多重共線性 Multicollinearity）時，普通線性回歸係數會暴增失真。Ridge 在損失函數中加入權重平方懲罰 $\alpha \sum w_i^2$，有效將權重壓小、平滑化模型。
  - **Lasso (套索回歸 / L1 正則化)**：在損失函數中加入權重絕對值懲罰 $\alpha \sum |w_i|$。其幾何特性具備**稀疏性**，會主動將無關特徵的權重直接壓縮為精確的 $0$，天然具備**特徵選取（Feature Selection）**能力。
- **語法**：
  - `Ridge(alpha=1.0, random_state=None)`
  - `Lasso(alpha=1.0, max_iter=1000, random_state=None)`
- **參數說明**：
  - `alpha`：正則化懲罰強度（超參數）。$\alpha$ 越大懲罰越重，模型越平滑/欠擬合；$\alpha=0$ 則退化為普通線性回歸。

```python
import pandas as pd
from sklearn.linear_model import Ridge, Lasso

df = pd.DataFrame({
    "X1": [1.0, 2.0, 3.0, 4.0, 5.0],
    "X2": [2.1, 3.9, 6.2, 7.8, 10.1],  # 與 X1 高度線性相關
    "Y": [5.2, 9.8, 15.1, 19.9, 25.0]
})

X = df[["X1", "X2"]]
y = df["Y"]

# L2 嶺回歸：平衡分配權重，抑制共線性震盪
ridge = Ridge(alpha=1.0)
ridge.fit(X, y)
print("Ridge 係數:", ridge.coef_.round(2))
# 輸出: Ridge 係數: [2.36 2.39]

# L1 套索回歸：直接將重複特徵 X1 係數歸零，實現自動特徵選取
lasso = Lasso(alpha=0.1)
lasso.fit(X, y)
print("Lasso 係數:", lasso.coef_.round(2))
# 輸出: Lasso 係數: [0.   4.88]
```

**`df` 表格結構**：

| Index | X1 | X2 | Y |
| :---: | :---: | :---: | :---: |
| 0 | 1.0 | 2.1 | 5.2 |
| 1 | 2.0 | 3.9 | 9.8 |
| 2 | 3.0 | 6.2 | 15.1 |
| 3 | 4.0 | 7.8 | 19.9 |
| 4 | 5.0 | 10.1 | 25.0 |

---

##### RandomForestRegressor() 隨機森林回歸

- **使用時機**：非線性關係複雜、資料分佈未知時的強大開箱即用回歸模型。基於裝袋法（Bagging）結合數十至數百棵決策樹的平均預測，極抗過擬合。
- **核心優勢**：
  - **無需特徵標準化**：決策樹的分裂只看大小順序，對特徵數值尺度完全免疫。
  - **自帶特徵重要性**：訓練完成後可直接查詢 `model.feature_importances_` 找出關鍵驅動特徵。
- **語法**：`RandomForestRegressor(n_estimators=100, max_depth=None, random_state=None, n_jobs=-1)`
- **參數說明**：
  - `n_estimators`：森林中決策樹的數量（預設 100 棵，通常越多越穩定，但邊際效益遞減且耗時）。
  - `max_depth`：單棵樹的最大深度（控制模型複雜度防擬合的核心超參數）。
  - `n_jobs`：平行運算 CPU 核心數（設為 `-1` 代表使用全電腦所有核心全力運算）。

```python
import pandas as pd
from sklearn.ensemble import RandomForestRegressor

df = pd.DataFrame({
    "引擎排氣量": [1500, 2000, 2500, 3000, 3500],
    "車重": [1200, 1450, 1600, 1850, 2100],
    "油耗": [18.5, 15.2, 12.8, 10.5, 8.9]
})

rf = RandomForestRegressor(n_estimators=50, random_state=42)
rf.fit(df[["引擎排氣量", "車重"]], df["油耗"])

print("特徵重要性比例:", rf.feature_importances_.round(2))
# 輸出: 特徵重要性比例: [0.49 0.51] (兩者對油耗影響力相當)
```

**`df` 表格結構**：

| Index | 引擎排氣量 | 車重 | 油耗 |
| :---: | :---: | :---: | :---: |
| 0 | 1500 | 1200 | 18.5 |
| 1 | 2000 | 1450 | 15.2 |
| 2 | 2500 | 1600 | 12.8 |
| 3 | 3000 | 1850 | 10.5 |
| 4 | 3500 | 2100 | 8.9 |

---

##### MLPRegressor() 多層感知神經網路回歸

- **使用時機**：全名 Multi-Layer Perceptron Regressor。傳統線性與樹模型擬合效果遭遇瓶頸時，利用多層隱藏神經元與非線性激活函數（如 ReLU）捕捉極度曲折的非線性關聯。
- **語法**：`MLPRegressor(hidden_layer_sizes=(100,), activation='relu', solver='adam', alpha=0.0001, learning_rate_init=0.001, max_iter=200, random_state=None)`
- **參數說明**：
  - `hidden_layer_sizes`：隱藏層架構元組（如 `(64, 32)` 代表兩層隱藏層，分別有 64 與 32 個神經元）。
  - `activation`：非線性激活函數（`'relu'` 預設推薦；`'tanh'` 雙曲正切；`'identity'` 線性）。
  - `solver`：權重優化演算法（`'adam'` 大資料集首選；`'lbfgs'` 超小資料集收斂極快）。
  - `alpha`：L2 權重衰減係數（Weight Decay，懲罰過大權重防過擬合）。
  - `max_iter`：最大梯度下降迭代次數（若訓練提早結束或未收斂可調大）。
- **避坑死穴**：神經網路以梯度下降更新權重，**若未做特徵縮放（Feature Scaling），Loss 會直接爆炸或永遠無法收斂**！

```python
import pandas as pd
from sklearn.neural_network import MLPRegressor
from sklearn.preprocessing import StandardScaler

df = pd.DataFrame({
    "溫度": [15, 20, 25, 30, 35],
    "濕度": [40, 55, 65, 80, 90],
    "用電量": [120, 150, 210, 320, 450]
})

# 神經網絡前置必備：標準化縮放特徵
scaler = StandardScaler()
X_scaled = scaler.fit_transform(df[["溫度", "濕度"]])

mlp = MLPRegressor(
    hidden_layer_sizes=(32, 16),
    activation="relu",
    max_iter=1000,
    random_state=42
)
mlp.fit(X_scaled, df["用電量"])
print("MLP 訓練迭代完成，最終損失:", round(mlp.loss_, 2))
```

**`df` 表格結構**：

| Index | 溫度 | 濕度 | 用電量 |
| :---: | :---: | :---: | :---: |
| 0 | 15 | 40 | 120 |
| 1 | 20 | 55 | 150 |
| 2 | 25 | 65 | 210 |
| 3 | 30 | 80 | 320 |
| 4 | 35 | 90 | 450 |

---

# 經典監督式學習模型：分類 (Supervised: Classification)

預測離散類別標籤（是否違約、是否罹病、垃圾郵件判別、圖像類別）。

> **[維度通則]**：所有分類模型均嚴格要求**特徵矩陣 $X$ 為 2D** `(n_samples, n_features)`；而**目標標籤 $y$ 通常為 1D** `(n_samples,)`。若將 $y$ 傳成 2D 欄向量 `(n_samples, 1)`，會觸發 `DataConversionWarning`。

---

##### LogisticRegression() 邏輯回歸

- **使用時機**：分類任務的第一基準模型。將線性回歸的輸出透過 Sigmoid 函數 $\sigma(z) = \frac{1}{1 + e^{-z}}$ 壓縮至 $[0, 1]$ 區間，解釋為發生該事件的「機率值」。
- **語法**：`LogisticRegression(C=1.0, penalty='l2', solver='lbfgs', max_iter=100, class_weight=None, random_state=None)`
- **參數說明**：
  - `C`：正則化強度的倒數（浮點數，預設 1.0）。$C$ 越小正則化越強（模型越保守）；$C$ 越大越容易擬合。
  - `class_weight`：類別權重。面對不平衡資料集（如信用卡盜刷率 0.1%）時，傳入 `'balanced'` 會自動依類別頻率反比賦予不同權重。
- **預測方法對比**：
  - `predict(X)`：直接輸出離散類別（0 或 1）。
  - `predict_proba(X)`：輸出屬於各類別的機率分佈二維矩陣（業務上常手動調整門檻閾值，如大於 0.3 即判定高風險）。

```python
import pandas as pd
from sklearn.linear_model import LogisticRegression

df = pd.DataFrame({
    "信用分數": [600, 750, 520, 800, 680],
    "年收入萬": [40, 90, 35, 120, 65],
    "核卡": [0, 1, 0, 1, 1]
})

clf = LogisticRegression(random_state=42)
clf.fit(df[["信用分數", "年收入萬"]], df["核卡"])

# 檢視前兩筆樣本屬於 [拒卡(0), 核卡(1)] 的真實預測機率
probs = clf.predict_proba(df[["信用分數", "年收入萬"]][:2])
print("機率輸出:\n", probs.round(2))
# 輸出:
# [[0.94 0.06]   (第0筆有94%機率拒卡)
#  [0.01 0.99]]  (第1筆有99%機率核卡)
```

**`df` 表格結構**：

| Index | 信用分數 | 年收入萬 | 核卡 |
| :---: | :---: | :---: | :---: |
| 0 | 600 | 40 | 0 |
| 1 | 750 | 90 | 1 |
| 2 | 520 | 35 | 0 |
| 3 | 800 | 120 | 1 |
| 4 | 680 | 65 | 1 |

---

##### DecisionTreeClassifier() 決策樹分類

- **使用時機**：需要極高業務透明度（White-box Model）的白箱模型。利用資訊增益（Entropy）或吉尼不純度（Gini Impurity）遞迴切分特徵空間，可直接導出 If-Else 決策規則。
- **語法**：`DecisionTreeClassifier(criterion='gini', max_depth=None, min_samples_split=2, min_samples_leaf=1, random_state=None)`
- **參數說明**：
  - `criterion`：切分節點標準：`'gini'`（計算快速）或 `'entropy'`（資訊量變化）。
  - `max_depth`：樹最大深度（**最核心防過擬合參數**！未設深度決策樹會一直生長直到所有葉節點純度 100%，導致極度過擬合）。
  - `min_samples_leaf`：葉節點所需的最小樣本數（限制葉子不能太小，抗雜訊噪點）。

```python
import pandas as pd
from sklearn.tree import DecisionTreeClassifier

df = pd.DataFrame({
    "風速": [10, 25, 5, 30, 15],
    "濕度": [50, 85, 40, 90, 60],
    "是否打球": [1, 0, 1, 0, 1]
})

dt = DecisionTreeClassifier(max_depth=2, random_state=42)
dt.fit(df[["風速", "濕度"]], df["是否打球"])
print("決策樹生長深度:", dt.get_depth())
# 輸出: 決策樹生長深度: 2
```

**`df` 表格結構**：

| Index | 風速 | 濕度 | 是否打球 |
| :---: | :---: | :---: | :---: |
| 0 | 10 | 50 | 1 |
| 1 | 25 | 85 | 0 |
| 2 | 5 | 40 | 1 |
| 3 | 30 | 90 | 0 |
| 4 | 15 | 60 | 1 |

---

##### RandomForestClassifier() 隨機森林分類

- **使用時機**：結構化表格資料的綜合表現王者之一。集合多棵具備隨機特徵採樣的決策樹，採取多數決投票（Majority Voting），抗過擬合與抗雜訊能力遠勝單一決策樹。
- **語法**：`RandomForestClassifier(n_estimators=100, max_depth=None, class_weight=None, n_jobs=-1, random_state=None)`

```python
import pandas as pd
from sklearn.ensemble import RandomForestClassifier

df = pd.DataFrame({
    "點擊次數": [5, 20, 1, 18, 3, 25],
    "停留秒數": [30, 240, 10, 310, 45, 500],
    "是否購買": [0, 1, 0, 1, 0, 1]
})

rf = RandomForestClassifier(n_estimators=100, random_state=42)
rf.fit(df[["點擊次數", "停留秒數"]], df["是否購買"])

# 預測點擊15次、停留200秒的新訪客
new_sample = pd.DataFrame({"點擊次數": [15], "停留秒數": [200]})
pred = rf.predict(new_sample)
print("預測是否購買:", pred[0])
# 輸出: 預測是否購買: 1
```

**`df` 表格結構**：

| Index | 點擊次數 | 停留秒數 | 是否購買 |
| :---: | :---: | :---: | :---: |
| 0 | 5 | 30 | 0 |
| 1 | 20 | 240 | 1 |
| 2 | 1 | 10 | 0 |
| 3 | 18 | 310 | 1 |
| 4 | 3 | 45 | 0 |
| 5 | 25 | 500 | 1 |
**`new_sample` 表格結構**：

| Index | 點擊次數 | 停留秒數 |
| :---: | :---: | :---: |
| 0 | 15 | 200 |

---

##### SVC() 支持向量機分類

- **使用時機**：全名 Support Vector Classification。在中小型資料集（千筆至萬筆級別）但特徵維度極高時表現卓越。透過「核技巧（Kernel Trick，如 RBF 高斯核）」將低維非線性資料映射至高維空間，尋找間隔最大化（Max Margin）的分離超平面。
- **語法**：`SVC(C=1.0, kernel='rbf', gamma='scale', probability=False, random_state=None)`
- **參數說明**：
  - `C`：懲罰參數（軟間隔容錯率）。$C$ 越大對錯誤分類懲罰越重，越容易過擬合。
  - `kernel`：核函數類型：`'rbf'`（徑向基/高斯核，預設，非線性能力強）、`'linear'`（線性核，適合高維文本）、`'poly'`（多項式核）。
  - `probability`：是否啟用機率估計（預設 `False`，若需使用 `predict_proba` 必須設為 `True`，訓練速度會稍微變慢）。
- **避坑死穴**：SVC 是純幾何距離模型，**訓練前必須嚴格做特徵縮放**！

```python
import pandas as pd
from sklearn.svm import SVC
from sklearn.preprocessing import StandardScaler

df = pd.DataFrame({
    "血壓": [120, 145, 115, 160],
    "膽固醇": [180, 240, 170, 280],
    "心血管風險": [0, 1, 0, 1]
})

scaler = StandardScaler()
X_scaled = scaler.fit_transform(df[["血壓", "膽固醇"]])

svc = SVC(kernel="rbf", probability=True, random_state=42)
svc.fit(X_scaled, df["心血管風險"])

print("支撐向量個數 (Support Vectors):", svc.n_support_)
# 輸出: 支撐向量個數 (Support Vectors): [2 2]
```

**`df` 表格結構**：

| Index | 血壓 | 膽固醇 | 心血管風險 |
| :---: | :---: | :---: | :---: |
| 0 | 120 | 180 | 0 |
| 1 | 145 | 240 | 1 |
| 2 | 115 | 170 | 0 |
| 3 | 160 | 280 | 1 |

---

##### HistGradientBoostingClassifier() 高效直方圖梯度提升樹

- **使用時機**：**現代 Scikit-learn 處理大型表格資料的最強秘密武器！** 借鑑 LightGBM 的演算法理念，將連續數值預先分箱為 256 個整數直方圖 Bin，將時間複雜度大幅降低一個數量級，速度比傳統 GradientBoosting 快數十倍。
- **原生兩大超能力**：
  1. **原生支援缺失值（NaN）**：遇到缺值完全不需手工填補，演算法在分箱時會自動為缺失值尋找最佳分裂方向！
  2. **原生支援類別特徵（Categorical Features）**：透過參數 `categorical_features` 直接指定類別欄，無需先做龐大的 OneHotEncoding。
- **語法**：`HistGradientBoostingClassifier(loss='log_loss', learning_rate=0.1, max_iter=100, max_leaf_nodes=31, random_state=None)`

```python
import numpy as np
import pandas as pd
from sklearn.ensemble import HistGradientBoostingClassifier

df = pd.DataFrame({
    "特徵1": [1.5, np.nan, 3.2, 4.8, 2.1], # 包含空值亦能直接訓練！
    "特徵2": [10, 25, 18, 40, 22],
    "目標": [0, 1, 0, 1, 0]
})

hgb = HistGradientBoostingClassifier(random_state=42)
hgb.fit(df[["特徵1", "特徵2"]], df["目標"])

print("訓練集評分 (Accuracy):", hgb.score(df[["特徵1", "特徵2"]], df["目標"]))
# 輸出: 訓練集評分 (Accuracy): 1.0
```

**`df` 表格結構**：

| Index | 特徵1 | 特徵2 | 目標 |
| :---: | :---: | :---: | :---: |
| 0 | 1.5 | 10 | 0 |
| 1 | `NaN` | 25 | 1 |
| 2 | 3.2 | 18 | 0 |
| 3 | 4.8 | 40 | 1 |
| 4 | 2.1 | 22 | 0 |

---

# 非監督式學習 (Unsupervised Learning)

無標籤資料的自動分群、聚類結構探查與維度壓縮。

---

##### KMeans() K-Means 分群演算法

- **使用時機**：沒有預先標籤（No Ground Truth）時，希望依照特徵相似度將樣本自動歸納為 $K$ 個群集（Clusters，如客戶分層、行為分群）。
- **輸入維度限制**：**特徵矩陣 $X$ 強制要求 2D 二維結構** `(n_samples, n_features)`。
- **數學原理**：隨機指定 $K$ 個初始質心（Centroids），反覆計算所有樣本到質心的歐氏距離，將樣本分派給最近的質心，再重新計算質心座標，直到質心不再移動。
- **語法**：`KMeans(n_clusters=8, init='k-means++', n_init='auto', max_iter=300, random_state=None)`
- **核心屬性**：
  - `model.labels_`：每個樣本被分配到的分群群號（0 到 $K-1$）。
  - `model.cluster_centers_`：各群集的幾何中心點座標。
  - `model.inertia_`：各點到所屬質心的距離平方和（用以繪製手肘法圖表 Elbow Method 決定最佳 $K$ 值）。

```python
import pandas as pd
from sklearn.cluster import KMeans

df = pd.DataFrame({
    "消費金額": [100, 150, 120, 800, 950, 880],
    "拜訪次數": [2, 3, 2, 15, 18, 14]
})

kmeans = KMeans(n_clusters=2, random_state=42, n_init="auto")
df["群集標籤"] = kmeans.fit_predict(df[["消費金額", "拜訪次數"]])

print("分群質心座標 (Centroids):\n", kmeans.cluster_centers_.round(1))
# 輸出:
# [[123.3   2.3]  (小資低頻群)
#  [876.7  15.7]] (VIP 高頻群)
```

**`df` 原始表格結構**：

| Index | 消費金額 | 拜訪次數 |
| :---: | :---: | :---: |
| 0 | 100 | 2 |
| 1 | 150 | 3 |
| 2 | 120 | 2 |
| 3 | 800 | 15 |
| 4 | 950 | 18 |
| 5 | 880 | 14 |

**變動後 `df` 表格結構（新增分群標籤）**：

| Index | 消費金額 | 拜訪次數 | 群集標籤 |
| :---: | :---: | :---: | :---: |
| 0 | 100 | 2 | 0 |
| 1 | 150 | 3 | 0 |
| 2 | 120 | 2 | 0 |
| 3 | 800 | 15 | 1 |
| 4 | 950 | 18 | 1 |
| 5 | 880 | 14 | 1 |

---

##### PCA() 主成分分析維度縮減

- **使用時機**：全名 Principal Component Analysis。當特徵欄位高達幾十甚至數百維時，不僅面臨「維度災難（Curse of Dimensionality）」，也無法直接以視覺化圖表呈現。PCA 尋找資料變異量最大的正交方向（主成分），將高維特徵投影至低維空間（如 2D 或 3D）。
- **輸入維度限制**：**特徵矩陣 $X$ 強制要求 2D 二維結構** `(n_samples, n_features)`。
- **語法**：`PCA(n_components=None, copy=True, whiten=False, random_state=None)`
- **參數與屬性**：
  - `n_components`：欲保留的主成分數量（整數如 `2` 代表降至 2 維；或小數如 `0.95` 代表保留 95% 累積解釋變異量）。
  - `pca.explained_variance_ratio_`：各主成分所解釋的原始變異量百分比比例（如 `[0.85, 0.10]` 代表前兩主成分捕捉了 95% 的原始資訊）。

```python
import pandas as pd
from sklearn.decomposition import PCA

df = pd.DataFrame({
    "數學": [90, 80, 50, 60],
    "物理": [95, 85, 45, 65],
    "化學": [88, 82, 55, 60],
    "歷史": [60, 65, 85, 90]
})

pca = PCA(n_components=2)
X_reduced = pca.fit_transform(df)

print("降維後維度形狀:", X_reduced.shape)
# 輸出: 降維後維度形狀: (4, 2)
print("各主成分解釋變異量比例:", pca.explained_variance_ratio_.round(3))
# 輸出: 各主成分解釋變異量比例: [0.939 0.057] (第1主成分已涵蓋93.9%資訊！)
```

**`df` 表格結構**：

| Index | 數學 | 物理 | 化學 | 歷史 |
| :---: | :---: | :---: | :---: | :---: |
| 0 | 90 | 95 | 88 | 60 |
| 1 | 80 | 85 | 82 | 65 |
| 2 | 50 | 45 | 55 | 85 |
| 3 | 60 | 65 | 60 | 90 |

**`X_reduced` PCA 降維後表格結構**：

| Index | 主成分 1 (PC1) | 主成分 2 (PC2) |
| :---: | :---: | :---: |
| 0 | 37.56 | -0.52 |
| 1 | 21.60 | -1.97 |
| 2 | -38.46 | -6.41 |
| 3 | -20.70 | 8.90 |

---

# 模型評估指標與交叉驗證 (Metrics & Validation)

客觀量化模型預測品質，預防資料外洩與評估偏差。

---

##### 回歸模型評估指標：MSE、RMSE、MAE 與 R² Score

- **四大核心指標意義**：
  1. **MAE (Mean Absolute Error，平均絕對誤差)**：$\frac{1}{n} \sum |y_i - \hat{y}_i|$。直觀易懂，代表預測值平均偏離真實值多少單位。
  2. **MSE (Mean Squared Error，均方誤差)**：$\frac{1}{n} \sum (y_i - \hat{y}_i)^2$。透過平方放大了較大的預測失誤，對離群極端誤差極為敏感。
  3. **RMSE (Root Mean Squared Error，均方根誤差)**：$\sqrt{MSE}$。開根號後將單位還原為與目標變數相同，兼具「重罰極端值」與「單位一致」的優點。（*Scikit-learn 1.4+ 提供專屬函式 `root_mean_squared_error`*）。
  4. **$R^2$ Score (決定係數 / 擬合優度)**：衡量模型解釋目標變數變異量的比例。$R^2=1.0$ 為完美預測；$R^2=0$ 代表模型等同於盲猜平均值；$R^2<0$ 代表模型比直接猜平均值還糟糕。

```python
import numpy as np
from sklearn.metrics import mean_squared_error, root_mean_squared_error, mean_absolute_error, r2_score

y_true = np.array([100.0, 150.0, 200.0, 250.0])
y_pred = np.array([110.0, 140.0, 210.0, 240.0])

print(f"MAE  (平均絕對誤差): {mean_absolute_error(y_true, y_pred):.2f}")
print(f"MSE  (均方誤差):     {mean_squared_error(y_true, y_pred):.2f}")
print(f"RMSE (均方根誤差):   {root_mean_squared_error(y_true, y_pred):.2f}")
print(f"R²   (決定係數):     {r2_score(y_true, y_pred):.4f}")
# 輸出:
# MAE  (平均絕對誤差): 10.00
# MSE  (均方誤差):     100.00
# RMSE (均方根誤差):   10.00
# R²   (決定係數):     0.9600
```

---

##### 分類模型評估指標：Accuracy、Precision、Recall 與 F1-Score

- **四大指標定義與適用情境**：
  - **Accuracy (準確率)**：$\frac{TP + TN}{TP + TN + FP + FN}$。在**資料分佈完全均衡**時方有參考價值。若 99% 的樣本為正常交易，模型全猜正常亦能得到 99% 準確率（致命陷阱！）。
  - **Precision (查準率 / 精確率)**：$\frac{TP}{TP + FP}$。所有被模型預測為「正類」的樣本中，究竟有幾成是真正的正類？（**代價高於放過時使用**，如：垃圾郵件過濾，絕不能將正常重要商務信件誤判為垃圾郵件）。
  - **Recall (查全率 / 召回率)**：$\frac{TP}{TP + FN}$。所有真實為「正類」的樣本中，到底被抓出來幾成？（**漏抓代價極其高昂時使用**，如：癌症腫瘤篩檢、信用卡盜刷偵測，寧可抓錯也不可放過一個！）。
  - **F1-Score (調和平均數)**：$2 \times \frac{Precision \times Recall}{Precision + Recall}$。綜合兼顧 Precision 與 Recall，是不平衡資料集最重要的單一評估標準。

---

##### confusion_matrix() 與 classification_report() 混淆矩陣與綜合評估報告

- **混淆矩陣結構**：
  ```text
                  預測為 0 (負類)    預測為 1 (正類)
  真實為 0 (負類)      TN (真負)         FP (偽正 / 誤報)
  真實為 1 (正類)      FN (偽負 / 漏報)   TP (真正)
  ```
- **語法**：
  - `confusion_matrix(y_true, y_pred)`
  - `classification_report(y_true, y_pred, target_names=None)`

```python
import numpy as np
from sklearn.metrics import confusion_matrix, classification_report

y_true = np.array([0, 0, 0, 1, 1, 1])
y_pred = np.array([0, 0, 1, 0, 1, 1])

# 1. 輸出混淆矩陣
print("混淆矩陣:\n", confusion_matrix(y_true, y_pred))
# 輸出:
# [[2 1]  (2個TN, 1個FP)
#  [1 2]] (1個FN, 2個TP)

# 2. 一鍵生成包含 Precision, Recall, F1 與 Support 的綜合評估報表
print("\n分類綜合報告:\n", classification_report(y_true, y_pred, target_names=["正常", "違約"]))
# 輸出:
#               precision    recall  f1-score   support
#           正常       0.67      0.67      0.67         3
#           違約       0.67      0.67      0.67         3
#     accuracy                           0.67         6
#    macro avg       0.67      0.67      0.67         6
# weighted avg       0.67      0.67      0.67         6
```

---

##### roc_auc_score() ROC 曲線下面積評估

- **使用時機**：全名 Receiver Operating Characteristic - Area Under Curve。評估分類器在**所有可能判定門檻（0.0 ~ 1.0）**下的排序分辨能力。
- **評估標準**：
  - $0.5$：等同於盲猜拋硬幣。
  - $0.7 \sim 0.8$：具備良好分辨能力。
  - $> 0.85$：模型性能優異。
  - $1.0$：完美分類器。
- **語法**：`roc_auc_score(y_true, y_score)`（**注意：第二個參數傳入的是機率預測值 `predict_proba()[:, 1]`，而非硬分類標籤！**）

```python
import numpy as np
from sklearn.metrics import roc_auc_score

y_true = np.array([0, 0, 1, 1])
y_score = np.array([0.1, 0.4, 0.35, 0.8])  # 屬於類別 1 的預測機率

auc = roc_auc_score(y_true, y_score)
print(f"ROC-AUC 分數: {auc:.2f}")
# 輸出: ROC-AUC 分數: 0.75
```

---

##### cross_val_score() 與 KFold / StratifiedKFold 交叉驗證

- **使用時機**：單次 `train_test_split` 切分容易受到運氣與隨機種子影響。K 折交叉驗證將資料均分為 $K$ 等份，輪流以其中 1 份當測試集、其餘 $K-1$ 份當訓練集，訓練 $K$ 次並平均分數，得出最客觀穩定的泛化能力評估。
- **語法**：`cross_val_score(estimator, X, y, cv=5, scoring=None, n_jobs=-1)`
- **參數說明**：
  - `cv`：折數（整數如 `5`；或傳入 `StratifiedKFold(n_splits=5)` 確保每一折的正負樣本比例完全均衡）。
  - `scoring`：評分標準字串（如 `'accuracy'`、`'f1'`、`'roc_auc'`、`'r2'`、`'neg_mean_squared_error'`）。

```python
import pandas as pd
from sklearn.model_selection import cross_val_score, StratifiedKFold
from sklearn.ensemble import RandomForestClassifier

df = pd.DataFrame({
    "特徵1": range(10),
    "特徵2": [x * 2 for x in range(10)],
    "目標": [0, 0, 0, 0, 0, 1, 1, 1, 1, 1]
})

model = RandomForestClassifier(random_state=42)
cv = StratifiedKFold(n_splits=3, shuffle=True, random_state=42)

# 進行 3 折交叉驗證
scores = cross_val_score(model, df[["特徵1", "特徵2"]], df["目標"], cv=cv, scoring="accuracy")
print(f"各折準確率: {scores.round(2)}, 平均分數: {scores.mean():.2f}")
```

**`df` 表格結構**：

| Index | 特徵1 | 特徵2 | 目標 |
| :---: | :---: | :---: | :---: |
| 0 | 0 | 0 | 0 |
| 1 | 1 | 2 | 0 |
| 2 | 2 | 4 | 0 |
| 3 | 3 | 6 | 0 |
| 4 | 4 | 8 | 0 |
| 5 | 5 | 10 | 1 |
| 6 | 6 | 12 | 1 |
| 7 | 7 | 14 | 1 |
| 8 | 8 | 16 | 1 |
| 9 | 9 | 18 | 1 |

---

# 超參數調優與搜尋 (Hyperparameter Tuning)

自動化系統性探索最優模型參數組合，釋放模型極限性能。

---

##### GridSearchCV() 窮舉網格搜尋

- **使用時機**：當有一組備選超參數清單時，窮舉計算所有的笛卡爾積組合，並自動進行交叉驗證，選出在驗證集上表現最佳的冠軍模型。
- **語法**：`GridSearchCV(estimator, param_grid, cv=5, scoring=None, n_jobs=-1, verbose=0)`
- **核心屬性**：
  - `grid.best_params_`：取得最佳超參數組合字典。
  - `grid.best_score_`：取得最佳平均交叉驗證分數。
  - `grid.best_estimator_`：取得已經用全量訓練集重新訓練完成的最終模型實例（可直接用於 `.predict()`！）。

```python
import pandas as pd
from sklearn.model_selection import GridSearchCV
from sklearn.ensemble import RandomForestClassifier

df = pd.DataFrame({
    "F1": [1, 2, 3, 4, 5, 6, 7, 8],
    "F2": [10, 20, 15, 30, 25, 40, 35, 50],
    "Label": [0, 0, 0, 0, 1, 1, 1, 1]
})

model = RandomForestClassifier(random_state=42)
# 定義參數網格 (共 2 x 2 = 4 種組合)
param_grid = {
    "n_estimators": [10, 50],
    "max_depth": [2, 4]
}

grid = GridSearchCV(model, param_grid, cv=2, scoring="accuracy")
grid.fit(df[["F1", "F2"]], df["Label"])

print("最佳超參數組合:", grid.best_params_)
print(f"最佳驗證分數: {grid.best_score_:.2f}")
# 直接使用最佳模型進行預測
best_model = grid.best_estimator_
```

**`df` 表格結構**：

| Index | F1 | F2 | Label |
| :---: | :---: | :---: | :---: |
| 0 | 1 | 10 | 0 |
| 1 | 2 | 20 | 0 |
| 2 | 3 | 15 | 0 |
| 3 | 4 | 30 | 0 |
| 4 | 5 | 25 | 1 |
| 5 | 6 | 40 | 1 |
| 6 | 7 | 35 | 1 |
| 7 | 8 | 50 | 1 |

---

##### RandomizedSearchCV() 隨機分佈搜尋

- **使用時機**：當超參數維度過多（如 5 個參數，每個各有 5 種選擇，組合高達 $5^5=3125$ 種）時，`GridSearchCV` 會計算到天荒地老。`RandomizedSearchCV` 在指定的分佈空間中**隨機抽樣 $N$ 次**，統計學證明抽樣 60 次即有 95% 的機率找到前 5% 的最優解，運算效率提升數十倍。
- **語法**：`RandomizedSearchCV(estimator, param_distributions, n_iter=10, cv=5, scoring=None, n_jobs=-1, random_state=None)`
- **參數說明**：
  - `n_iter`：隨機抽樣的組合總次數（例如 `50` 次）。

```python
import pandas as pd
from sklearn.model_selection import RandomizedSearchCV
from sklearn.ensemble import RandomForestClassifier

df = pd.DataFrame({
    "F1": [1, 2, 3, 4, 5, 6, 7, 8],
    "F2": [10, 20, 15, 30, 25, 40, 35, 50],
    "Label": [0, 0, 0, 0, 1, 1, 1, 1]
})

model = RandomForestClassifier(random_state=42)
param_dist = {
    "n_estimators": range(10, 100, 10),
    "max_depth": [2, 3, 4, 5, None]
}

# 僅隨機抽樣 5 組進行測試
rand_search = RandomizedSearchCV(model, param_dist, n_iter=5, cv=2, random_state=42)
rand_search.fit(df[["F1", "F2"]], df["Label"])
print("隨機搜尋最佳參數:", rand_search.best_params_)
```

**`df` 表格結構**：

| Index | F1 | F2 | Label |
| :---: | :---: | :---: | :---: |
| 0 | 1 | 10 | 0 |
| 1 | 2 | 20 | 0 |
| 2 | 3 | 15 | 0 |
| 3 | 4 | 30 | 0 |
| 4 | 5 | 25 | 1 |
| 5 | 6 | 40 | 1 |
| 6 | 7 | 35 | 1 |
| 7 | 8 | 50 | 1 |

---

# 模型持久化保存 (Model Persistence)

將耗時訓練完成之模型物件序列化存檔，對接生產環境 API 推論。

---

##### joblib.dump() 與 joblib.load() 模型序列化與載入

- **使用時機**：訓練一個複雜模型可能耗時數小時甚至數天。訓練完成後必須儲存為磁碟檔案（通常為 `.pkl` 或 `.joblib`），以便日後在 FastAPI、Flask 或後端微服務中秒級載入進行推論。
- **為什麼用 joblib 而非原生 pickle？**：
  - `joblib` 專為含有大型 NumPy 密集陣列與權重矩陣的科學計算物件進行深度優化，存檔速度快且大幅壓縮磁碟佔用。
- **語法**：
  - 儲存：`joblib.dump(value, filename, compress=3)`
  - 載入：`joblib.load(filename)`

```python
import os
import joblib
import pandas as pd
from sklearn.linear_model import LinearRegression

df = pd.DataFrame({"X": [1, 2, 3], "Y": [2, 4, 6]})
model = LinearRegression().fit(df[["X"]], df["Y"])

# 1. 序列化儲存模型至檔案
model_path = "/tmp/trained_model.pkl"
joblib.dump(model, model_path)

# 2. 產線服務啟動時快速載入模型
loaded_model = joblib.load(model_path)
new_sample = pd.DataFrame({"X": [5]})
print("載入模型推論 X=5 的結果:", loaded_model.predict(new_sample)[0])
# 輸出: 載入模型推論 X=5 的結果: 10.0

if os.path.exists(model_path):
    os.remove(model_path)
```

**`df` 表格結構**：

| Index | X | Y |
| :---: | :---: | :---: |
| 0 | 1 | 2 |
| 1 | 2 | 4 |
| 2 | 3 | 6 |
**`new_sample` 表格結構**：

| Index | X |
| :---: | :---: |
| 0 | 5 |

---

# 實戰避坑與核心天條 (Best Practices & Critical Rules)

工業級機器學習專案中絕不可觸犯的 5 大工程天條。

---

## 1. 嚴禁資料外洩（Data Leakage）！測試集絕對不可呼叫 fit()

> **[核心天條]：嚴禁在劃分資料集前做前處理！測試集嚴格只能呼叫 `transform()`！**  
> 若你在 `train_test_split` 之前就對全量資料做了 `StandardScaler()` 或 `SimpleImputer()`，測試集的均值與分佈資訊便已經「偷偷滲透」進了前處理轉換器中。這會導致模型在本地測試集上跑出超高分數（虛假繁榮），但一上線推論立即原形畢露（泛化能力低落）。

```python
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

# [嚴重錯誤寫法]：全表一起 fit_transform，資料外洩！
# X_scaled = scaler.fit_transform(X)
# X_train, X_test, y_train, y_test = train_test_split(X_scaled, y)

# [唯一正確寫法]：先切割，再以訓練集為基準
# X_train, X_test, y_train, y_test = train_test_split(X, y)
# scaler = StandardScaler()
# X_train_scaled = scaler.fit_transform(X_train)  # 訓練集學習均值並轉換
# X_test_scaled  = scaler.transform(X_test)       # 測試集嚴格僅沿用轉換，絕不 fit！
```

---

## 2. 距離與梯度敏感模型必做特徵縮放

> **[核心天條]：凡依賴歐氏距離或梯度下降之演算法，特徵未標準化前嚴禁直接送入訓練！**  
> - **敏感模型清單**：KNN、SVM、Logistic 回歸（帶正則化）、Ridge/Lasso、MLP（神經網絡）、K-Means、PCA。  
> - **免疫模型清單**：決策樹（Decision Tree）、隨機森林（Random Forest）、梯度提升樹（GBDT/XGBoost/LightGBM）。樹模型只在乎特徵內部數值的排序相對大小，乘上 100 萬倍都不影響樹節點的分裂門檻。

---

## 3. 類別特徵編碼推論避坑：一律開啟 handle_unknown='ignore'

> **[核心天條]：OneHotEncoder 宣告時務必顯式指定 `handle_unknown='ignore'`！**  
> 在真實世界中，生產環境上線後隨時會湧入包含全新、未在訓練集見過之類別值（如新使用者來自全新的城市）。預設設定會直接拋出 `ValueError: Found unknown categories` 讓整個系統當機！開啟 `'ignore'` 會優雅地將該未知欄位編碼為全 0，保障系統服務穩定運行。

---

## 4. 模組化工程天條：一律使用 Pipeline 整合前處理與模型

> **[核心天條]：嚴禁散裝維護前處理與模型物件！一律包裝為單一 `Pipeline` 產出！**  
> 散裝前處理需要維護一大堆物件變數（`scaler`、`encoder`、`imputer`、`model`），在交叉驗證與超參數搜尋時極易出現洩漏漏做。使用 `Pipeline` 封裝後，前處理與模型合而為一，儲存與載入時只需保存單一 `.pkl` 檔，部署極為簡潔。

---

## 5. 分類資料切割與不平衡資料指標天條

> **[核心天條]：分類資料切割必加 `stratify=y`；不平衡資料評估嚴禁單看 `Accuracy`！**  
> 1. 切割資料時若未加上 `stratify=y`，罕見類別（如僅佔 1% 的違約用戶）可能整批掉進測試集或訓練集，導致兩邊資料分佈完全不一致。  
> 2. 在類別不平衡資料中，準確率（Accuracy）是最具欺騙性的指標。請一律以 **`F1-Score`**、**`PR-AUC`** 或 **`ROC-AUC`** 作為模型選型之黃金標準！

---

## 6. 維度錯配陷阱：單欄特徵務必使用雙中括號 df[['col']] 保留 2D

> **[核心天條]：傳入 Transformer 或 Model 的特徵矩陣 $X$，無論有幾個欄位，一律必須是 2D！**  
> 新手最常寫出 `scaler.fit_transform(df['年齡'])`，因為單中括號會將 Pandas 物件降維為 1D Series，直接觸發 `ValueError: Expected 2D array, got 1D array instead`。  
> **永遠記住雙中括號口訣**：  
> - **特徵只有一欄**：`df[['年齡']]`（雙中括號，維持 2D DataFrame）或 `x.values.reshape(-1, 1)`。  
> - **目標標籤**：`df['標籤']`（單中括號，維持 1D Series）。
