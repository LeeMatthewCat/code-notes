機器學習模組庫
處理**傳統數據分析與預測**的利器，適合以下情境

1. 結構化、表格化資料

2. 資料量小，不需靠 GPU 運算

3. 需要解釋性（決策樹模型可以清楚告訴你，是哪個具體的特徵（如血壓過高）導致了最終的預測結果）

4. 建立基準模型（面對一個新的預測問題，通常會先用 Scikit\-learn 寫幾行程式碼跑一個基礎模型，確認資料是否有預測價值，再來決定是否需要投入更多時間開發更複雜的模型）

---

# train\_test\_split\(\) 資料分組

將資料分為訓練資料及測試資料

```Python
x = ...
y = ...

from sklearn.model_selection import train_test_split # 匯入

# 第一份 x, 第二份 x, 第一份 y, 第二份 y
X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size = 0.2,  # 設定 20% 為測試集，80% 為訓練集
    random_state = 42,  # 設定隨機種子，確保每次切割結果一致 
    shuffle = True, # 隨機打亂資料 (預設即為 True) ，若沒有，隨機碼無用
    stratify = y # 依照 y 資料分層
    )              
```

- random\_state 隨機碼：如果不設，預設會讀取電腦現在的時間，自動得到隨機碼，同樣的隨機碼，會得到同樣的分配結過

- stratify 分層取樣：依照指定資料進行分層，避免過度集中類似的資料（大量資料時可省）

---

# fit\(\) 學習

## LinearRegression\(\) 線性回歸

```Python
from sklearn.linear_model import LinearRegression

model = LinearRegression() # 建立模型物件

model.fit(x_train, y_train) # 開始學習
```

訓練結果 $y=w_1x_1+w_2x_2...+w_nx_n+b$

```Python
print(model.coef_) # 權重 w

print(model.intercept_) # 截距 b
```

---

## MLPRegressor 多層感知回歸

全名 **Multi\-Layer Perceptron Regressor**，利用不同的激活函數來改善線性函數預測的偏差

```Python
from sklearn.neural_network import MLPRegressor

model = MLPRegressor(
    hidden_layer_sizes=(64, 64), # 層數和神經元數
    activation='relu', # 激活函數使用 ReLU（預設）
    max_iter=500, # 梯度下降次數（預設 200，還沒找到會報錯）--> 超參數
    solver = 'adam', # 搜尋策略為自動（預設）
    random_state=42, # 固定隨機種子以求穩定
    learning_rate_init = 0.0001 # 學習率
)
    
model.fit(x_train, y_train) # 開始訓練
```

當在**特徵數值差距過大**時，會是 Loss 暴增，必須**進行特徵縮放**

---

## 分類問題





---

# StandardScaler\(\) 特徵縮放

對於**數值差距過大**時可轉換，以達成**標準化**

$X_{scaled}=\frac{X-\mu}{\sigma}$

```Python
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler() # 建立至轉換物件
x_train_scaled = scaler.fit_transform(x_train) 
x_test_scaled = scaler.transform(x_test) # 不能再重新 fit 算一次基準點
```

> 第二個資料集轉換時，**不能再 fit 一次**，因為兩個資料集的**平均和標準差算起來會不同**，需**沿用上一個資料集的結果**

---

# joblib 儲存及導入

將模型儲存為 **\.pkl** 檔，下次用模型時可直接載入，不需重新訓練

```Python
import joblib
joblib.dump(模型, "檔案名.pkl")
```

```Python
import joblib
model = joblib.load("檔案名.pkl")
```

> joblib 是通用的 Python 物件序列化工具
> 
> 只要是 Python 程式碼裡存在的東西（變數、容器、函數、甚至整個工作流），幾乎都可以交給它存成 \.pkl 檔

---

# GridSearchCV 網格搜索

自動測試檢驗哪種層數和超參數會得出最好的結果

```Python
model = MLPRegressor(max_iter=1000, random_state=42) # 梯度次數、隨機碼

from sklearn.model_selection import GridSearchCV
test_params = {
    "hidden_layer_sizes": [(32,), (64, 64), (100, 50)], # 層數和神經元數
    "activation": ['relu', 'tanh'], # 激活函數
    "alpha": [0.0001, 0.01] # L2 正規化強度（權重稅）
} # 字典放參數
Grid_search = GridSearchCV(model, test_params, cv=5, n_jobs=-1, verbose=2)
                         # 模型, 字典, 單一輪次數, CPU 分配數量（-1 表示有多少用多少）, 
Grid_search.fit(x_train_scaled, y_train)
```

```Python
print(Grid_search.best_params_) # 最佳參數
print(Grid_search.best_score_) # 最佳分數（R^2 Score）
```































---

# 預測

```Python
y_pred = model.predict(x_test) # 利用測試集的特徵預測出 y
```

## MSE 

也就是 **Loss 值**的計算

```Python
from sklearn.metrics import mean_squared_error

print(mean_squared_error(y_test, y_pred)) # 答案, 預測值
```









---

