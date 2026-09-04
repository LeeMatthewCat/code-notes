是一基於 NumPY 的套件，對數學計算提供了更深入的**數學函數**

建議使用 from **對目標功能做匯入**

# constants 科學常數

提供科學計算所需的**固有常數**（如萬有引力、光速等），且提供**單位換算**的工具

```Python
import numpy as np
from scipy import constants

print(constants.c)
```

[Constants \(scipy\.constants\) — SciPy v1\.17\.0 Manual](https://docs.scipy.org/doc/scipy/reference/constants.html)

---

# io 讀寫

```Python
from scipy import io as spio
```

**MATLAB** 是一種將**任何資料以矩陣儲存**的軟體與程式語言，而將其存為副檔名 **\.mat**

這種格式像一個**字典**，而鍵就是資料的標籤（變數名），值則是**以矩陣表示的資料**

## savemat\(\) 存為 \.mat

將任何物件儲存為 \.mat 格式，而存放的資料必須為**字典格式**

```Python
spio.savemat("檔案名", {"鍵名": 資料, ...})
```

```Python
import numpy as np
from scipy import io as spio

a = np.ones((3, 3))

spio.savemat("a.mat", {"a": a}) 
```

---

## loadmat\(\) 讀取 \.mat

```Python
spio.loadmat("檔案名")

spio.loadmat("huge_file.mat", variable_names= 讀取的鍵) # 已知鍵可讀取指定鍵就好（節省記憶體）
spio.loadmat("huge_file.mat", variable_names= [..., ...]) # 用串列包起多個鍵
```

```Python
data = spio.loadmat('file.mat')
data['a'] # 像字典一樣存取
```

若**檔案過大**（超 2 GB）但也**只想讀取某已知鍵**，就只能使用 **h5py 套件**

---

# whosmat\(\) 預覽 \.mat

核心目的是為了**節省記憶體**，避免像 loadmat\(\) 沒設參數時一樣將整個檔案內部存在記憶體內

只讀取出內部**字典的鍵、矩陣形狀及型別**，不讀取內部資料

```Python
import numpy as np
from scipy import io as spio

a = np.ones((3, 3))

spio.savemat("a.mat", {"a": a, 
                        "b": "box",
                        "c": "cat"}) 
print(spio.whosmat('a.mat'))
# [('a', (3, 3), 'double'), ('b', (1,), 'char'), ('c', (1,), 'char')]
```

---

# sepecial 特殊功能

```Python
from scipy import special
```















---

