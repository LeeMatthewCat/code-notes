簡稱 Matplot，是專注於數據視圖化的函式庫

```Python
import matplotlib.pyplot as plt   
```

```Python
plt.表格樣式(x 軸資料,y  軸資料,
                    c = "顏色", # color
                    ls = "線條樣式", # linestyle
                    marker = "點的形狀",
                    alpha = 數值, # 透明度 0~1
                    label = "名字", # 線的名字 --> 搭配圖例
                    figsize = (寬, 高) # 畫布大小
                    lw = "線條寬", # linewidth，預設 1.5
                    ms = "點的大小") # markersize，預設 10ㄌ
                    
plt.title("Random Data") # 圖標題
plt.xlabel("Number") # x 軸標題
plt.ylabel("Random Number") # y 軸標題
plt.legend() # 圖例 --> loc = "upper right"，放右上角（留空則自動）
plt.grid(True) # 格線

plt.show() # 輸出圖
```

> 顏色部分可用單字或縮寫及色碼
> 
> 'r' \(紅 Red\)
> 
> 'g' \(綠 Green\)
> 
> 'b' \(藍 Blue\)
> 
> 'k' \(黑 Black \- 注意是 k 不是 b\)
> 
> 'w' \(白 White\)
> 
> 'y' \(黃 Yellow\)
> 
> 'c' \(青色 Cyan\)
> 
> 'm' \(洋紅 Magenta\)







---

# plot\(\) 折線圖 

```Python
plt.plot(x 軸資料,y  軸資料)
```

```Python
import random
import matplotlib.pyplot as plt

num = list(range(1,100))

data = [random.randint(1,100) for x in num] # 1~100 隨機取 100 

plt.plot(num,data)

plt.title("Random Data")
plt.xlabel("Number")
plt.ylabel("Random Number")

plt.show()
```

![截圖 2026\-05\-04 晚上7\.58\.15\.png](圖片/截圖%202026-05-04%20晚上7.58.15.png)

---

# scatter\(\) 散佈圖

```Python
plt.scatter(x 軸資料,y  軸資料)
```

```Python
import matplotlib.pyplot as plt

# 準備三組數據（數量必須完全一致，這裡有五個國家）
gdp = [1000, 5000, 15000, 30000, 50000]       # X軸：人均 GDP
life_expectancy = [50, 65, 70, 80, 85]        # Y軸：平均壽命
population = [50, 100, 300, 50, 10]           # 點的大小：人口數 (單位：百萬)

# 將人口數放大一點，視覺效果比較好
# (使用列表推導式將每個數字乘以 10)
sizes = [pop * 10 for pop in population] # 設定大小

# 畫出泡泡圖
plt.scatter(gdp, life_expectancy, 
            s=sizes,        # ✨ 關鍵！將點的大小設定為剛剛算出來的 sizes 串列
            alpha=0.5,      # 設定透明度，避免大泡泡把小泡泡蓋住
            c='orange')     # 統一設定為橘色

plt.title("GDP vs Life Expectancy (Bubble size = Population)")
plt.xlabel("GDP per Capita")
plt.ylabel("Life Expectancy")
plt.grid(True, linestyle=':', alpha=0.6)

plt.show()
```

![截圖 2026\-05\-04 晚上9\.52\.40\.png](圖片/截圖%202026-05-04%20晚上9.52.40.png)

---

# 多圖

多張圖放在一起

```Python
import matplotlib.pyplot as plt

# 1. 準備資料
fruits = ['蘋果', '香蕉', '橘子', '葡萄']
store_a_sales = [120, 80, 150, 90]
store_b_sales = [90, 110, 130, 100]

# 2. 買一張稍微高一點的畫布 (寬8, 高6)
fig = plt.figure(figsize=(8, 6))

# ====== 開始畫第一格 (A 店) ======
# 切成 2層、1排，放在第 1 格 (上半部)
ax1 = fig.add_subplot(2, 1, 1)

# 使用 ax1.bar 畫長條圖，設定為天空藍
ax1.bar(fruits, store_a_sales, color='skyblue')
ax1.set_title("A 店水果銷量")
ax1.set_ylabel("銷量 (顆)")

# ====== 開始畫第二格 (B 店) ======
# 切成 2層、1排，放在第 2 格 (下半部)
ax2 = fig.add_subplot(2, 1, 2)

# 設定為淺綠色，方便區分
ax2.bar(fruits, store_b_sales, color='lightgreen')
ax2.set_title("B 店水果銷量")
ax2.set_ylabel("銷量 (顆)")

# ====== 🌟 終極排版秘訣 ======
# 因為上下兩張圖的標題和座標軸可能會擠在一起
# 加上這行，Matplotlib 會自動幫你把間距拉好！
plt.tight_layout() 

# 顯示圖表
plt.show()
```

---

# 立體圖

```Python
X...
Y...
Z...

fig = plt.figure
ax = fig.add_subplot(行, 列, 位置, projection="3d") #前三個數字可以寫一起步逗號

# 標題設定
ax.set_title(title)
ax.set_xlabel('X') 
ax.set_ylabel('Y') 
ax.set_zlabel('Z')

plt.show()
```

---

## plot\_surface\(\) 立體曲面

```Python
X...
Y...
Z...

fig = plt.figure
ax = fig.add_subplot(行, 列, 位置, projection="3d") 

surf = ax.plot_surface(X, Y, Z, cmap="viridis", alpha=0.9) # cmap 為色彩樣式（漸層）

fig.colorbar(surf, shrink=0.5, aspect=5) # 傳入參照物（上面建的曲面）, 縮放比例, 長寬比（越大越細長）

plt.show()
```

---

