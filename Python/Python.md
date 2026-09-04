# 資料結構

## 可變 Mutable／不可變 Immutable

所有的東西（包括資料）都是一個**物件（object）**，而這些物件被分為：

> a 是標籤，而 3 才是真正的物件

- **可變**：

    可隨時改變內部資料，而它在記憶體中始終為相同的，且需注意**別名**現象

    想像為**白板**，可隨時擦掉或添加新東西

	對於可便物件， **`... = 型別(被複製物)`** 可達到複製效果

    > **串列 \(list\)、字典 \(dict\)、集合 \(set\)、自訂物件**

- **不可變**：

    資料被記憶體建立後，**永遠無法被更改**，想更改只能重做一個

    想像為**刻字在石頭上**，要變更時只能拿一塊新的石頭（**在記憶體中開一個新的空間，**且原本的空間被** Garbage Collection（GC）回收**）

    > **數字 \(int, float, complex\)、字串 \(str\)、元組 \(tuple\)、布林值 \(bool\)**

    ```Python
    name = "Matthew"
    print(id(name))  # 假設印出：4388921136 (這是舊石頭的位址)
    
    name = "MATTHEW"
    print(id(name))  # 假設印出：4388921584 (位置變了！證明這是一塊全新的石頭)
    ```

---

### 別名 Aliasing

Python 的變數本質上為一張標籤，當把多個標籤貼在記憶體裡的同一物件時，這些變數互為別名

也就是**多個名稱可指向同一個值**

而在不可變物件上不會造成問題，

但在可變物件時常會出問題

```Python
a = [1, 2, 3]

b=a
# a：[1, 2, 3]、b：[1, 2, 3]

b[0] = 100
# a：[100, 2, 3]、b：[100, 2, 3]，只改 a，但 a b 都一起被變更
```

---

## 命名規則

必須是一般文字（英文、中文等字元都可）或底線（\_）開頭，

數字不能在開頭

需注意不要和**保留字**以及**函數**撞名

而命名時通常會有以下慣例：

|     | 慣例        | 舉例                 |
| --- | --------- | ------------------ |
| 常數  | 全大寫的蛇形    | PI                 |
| 變數  | 蛇形        | student_age        |
| 函數  | 蛇形        | get_student_age    |
| 類別  | 帕斯卡（大駝峰形） | StudentInformation |


---

## 變數型別

```Python
type(查詢物件)
```

### 型別注記

透過註記型別，來註明此變數應為何種型別

```Python
age: int = 18

something: Tuple[int, int] # 序列型別內的元素也可註明
```

```Python
#      a 的型別  b 的型別  回傳的型別
def add(a: int, b: int) -> int:
    return a + b
```

但它**不是強制性的**（型別標注錯誤不報錯，但可安裝 mypy 來對其檢查），

一般會利用自動化測試來確保型別正確，**不特別註明出型別**

| 標註型別 / 類別                                  | 資料型別        | 說明與用法範例                                                                                                       |
| :----------------------------------------- | :---------- | :------------------------------------------------------------------------------------------------------------ |
| **`Optional[型別]`**                         | `Type`      | **指定型別或為空** (`型別 \| None`)。範例：`x: Optional[int] = None`                                                       |
| **`Union[型別1, 型別2]`**                      | `Type`      | **多種可選型別**（Python 3.10+ 可用 `型別1 \| 型別2` 取代）。範例：`x: Union[int, str]`                                           |
| **`List[型別]`**                             | `List`      | **指定元素型別的串列**。範例：`names: List[str] = ["Alice", "Bob"]`                                                        |
| **`Dict[鍵型別, 值型別]`**                       | `Dict`      | **指定鍵 (Key) 與值 (Value) 型別的字典**。範例：`scores: Dict[str, float] = {"math": 95.5}`                                 |
| **`Tuple[型別, ...]`**                       | `Tuple`     | **指定內部元素型別與長度的元組**。範例：`point: Tuple[int, int] = (10, 20)`                                                     |
| **`Set[型別]`**                              | `Set`       | **指定元素型別的集合**。範例：`tags: Set[str] = {"python", "git"}`                                                         |
| **`Iterable[型別]`**                         | `Iterable`  | **可迭代物件**（包含 List, Tuple, Set, Generator 等）。範例：`def count(items: Iterable[int]): pass`                        |
| **`Iterator[型別]`**                         | `Iterator`  | **迭代器物件**（支援 `next()` 調用）。範例：`def get_stream() -> Iterator[str]: yield "data"`                                |
| **`Generator[Yield型別, Send型別, Return型別]`** | `Generator` | **產生器物件**。範例：`def gen() -> Generator[int, None, None]: yield 1`                                               |
| **`Callable[[參數型別], 回傳型別]`**               | `Callable`  | **可呼叫的函數/物件**（指定參數型別串列與回傳值型別）。範例：`func: Callable[[int, int], int]`                                            |
| **`Any`**                                  | `Any`       | **任意型別**（跳過型別檢查）。範例：`data: Any = get_raw_data()`                                                              |
| **`Literal[...]`**                         | `Literal`   | **限定只能為特定的字面值內容**。範例：`mode: Literal["read", "write"] = "read"`                                                |
| **`Annotated[型別, 元數據, ...]`**              | `Annotated` | **附加元數據的型別**（FastAPI / Pydantic 常用於驗證或依賴注入）。範例：`Age: Annotated[int, Field(gt=0)]`                             |
| **`get_origin(型別)`**                       | `Function`  | **取得最外層原始類別**（如 `list[int]` ➔ `list`）。[[#get_origin() 與 get_args() 執行期型別內省神器 (Runtime Introspection)\|詳解]]    |
| **`get_args(型別)`**                         | `Function`  | **取得內部泛型參數元組**（如 `list[int]` ➔ `(int,)`）。[[#get_origin() 與 get_args() 執行期型別內省神器 (Runtime Introspection)\|詳解]] |
> 關於 **`Literal[...]`**：
> ```Python
> status: Literal["thinking", "thought_end", "replying"]
> ```
> 
> 關於 **`Annotated[型別, 元數據, ...]`**：
> 
> 在 **Python 3.9+** 引入 (`from typing import Annotated`)。第一個參數為**實質型別**，後續參數可附加任意**元數據 (Metadata)**。
> - **型別檢查器 (mypy / IDE)**：只將其視為基礎型別，完全不影響程式執行。
> - **現代框架 (Pydantic / FastAPI)**：在運行時提取元數據，用於**資料範圍驗證、欄位描述或依賴注入**。
> - **型別與元數據解構 (`typing.get_args()`)**：
>   - `get_args()` 回傳標準元組 `(型別, 元數據1, 元數據2, ...)`
>   - **`[0]` (索引取值)**：取出第 0 個位置的**底層真實型別**
>   - **`[1:]` (切片操作)**：從第 1 個位置切到最後，打包取出**所有元數據**
> 
> ```Python
> from typing import Annotated, get_args
> 
> # 1. 定義附加多個元數據的型別
> Price = Annotated[float, "單位: 新台幣 (TWD)", {"min": 0}]
> 
> # 2. 透過 typing.get_args() 取得元組後索引與切片
> args = get_args(Price)
> real_type = args[0]   # 索引 0 ➔ 輸出: <class 'float'>
> metadata  = args[1:]  # 切片 1: ➔ 輸出: ('單位: 新台幣 (TWD)', {'min': 0})
> ```
> 
> 關於 **`Generator[Yield型別, Send型別, Return型別]`**：
> 
> 產生器的型別注記包含三個必填參數，分別對應其生命週期中的三個資料流向：
> 
> 1. **`Yield型別`（產出型別）**：每次 `yield` 向外吐出的資料型別（這也是我們最常關注的）。
> 2. **`Send型別`（發送型別）**：外部透過 `gen.send(value)` 傳進產生器內部的資料型別。若不需要接收外部傳值，一律設為 `None`。
> 3. **`Return型別`（回傳型別）**：產生器徹底結束時（也就是觸發 `StopIteration` 時）最後 `return` 回來的資料型別。絕大多數情況為 `None`。
> 
> ```Python
> from typing import Generator
> 
> # 這是一個會產出 int，不接收外部傳值，最後結束時回傳 str 的產生器
> def count_down() -> Generator[int, None, str]:
>     yield 3  # Yield型別：向外吐出 int
>     yield 2
>     yield 1
>     return "發射！"  # Return型別：結束時回傳 str
> ```

---

#### get_origin() 與 get_args() 執行期型別內省神器 (Runtime Introspection)

在 **Python 3.8+** 的 `typing` 模組中，`get_origin()` 與 `get_args()` 是在**執行期 (Runtime) 拆解與剖析泛型型別 (Generic Types)** 的兩大核心法寶。

##### 💡 為什麼需要它們？解決什麼痛點？

當我們寫下 `list[str]`、`Dict[str, int]`、`Union[int, str]` 或 `Literal["A", "B"]` 時，它們在 Python 內部屬於特殊的泛型別物件（`GenericAlias`）。
- **痛點 1**：無法直接用 `isinstance([1, 2], list[int])` 檢查，會直接拋出 `TypeError: Subscripted generics cannot be used with class and instance checks` 崩潰！
- **痛點 2**：直接比對 `tp == list` 會得到 `False`（因為 `list[int] != list`）。

`get_origin()` 與 `get_args()` 能將任何複雜的泛型別**如同洋蔥般逐層剝開**，這也是 **Pydantic、FastAPI、FastMCP** 等現代框架自動進行資料校驗與 Schema 生成的底層核心原理！

> **🍿 生動白話比喻**：  
> 面對一個快遞包裹 `list[str]`（裝滿字串的列表箱）：  
> - **`get_origin(tp)`**：拆開外箱 ➔ **告訴你外層箱子是什麼容器**（`list`）。  
> - **`get_args(tp)`**：拿出內容物 ➔ **以元組形式告訴你箱子裡裝的規格是什麼**（`(str,)`）。

---

##### 📊 get_origin() 與 get_args() 核心對照速查表

| 泛型型別註記 `tp` | `get_origin(tp)` (最外層原始類別) | `get_args(tp)` (內部參數元組) | 白話解構說明 |
| :--- | :--- | :--- | :--- |
| **`list[int]`** | `list` | `(int,)` | 外層是列表，內部元素為 `int` |
| **`dict[str, float]`** | `dict` | `(str, float)` | 外層是字典，Key 是 `str`，Value 是 `float` |
| **`tuple[str, int, bool]`** | `tuple` | `(str, int, bool)` | 外層是元組，內部各位置型別依序對應 |
| **`Union[int, str]`** *(或 `int \| str`)* | `Union` *(或 `types.UnionType`)* | `(int, str)` | 聯合型別，可為 `int` 或 `str` |
| **`Optional[str]`** | `Union` | `(str, <class 'NoneType'>)` | 等同於 `Union[str, None]` |
| **`Literal["read", "write"]`** | `Literal` | `('read', 'write')` | 字面值限定，參數為允許的字串值 |
| **`Annotated[float, "單位:元"]`** | `Annotated` | `(float, '單位:元')` | 第 0 項為真實型別，第 1 項起為元數據 |
| **`int`** *(一般純型別)* | `None` | `()` | 非泛型別，無外層包裝與內部參數 |

---

##### 💻 語法實戰示範

```Python
from typing import get_origin, get_args, Union, Optional, Literal, Annotated

# 1. 列表與字典容器解析
t_list = list[int]
print(get_origin(t_list))  # 輸出: <class 'list'>
print(get_args(t_list))    # 輸出: (<class 'int'>,)

t_dict = dict[str, float]
print(get_origin(t_dict))  # 輸出: <class 'dict'>
print(get_args(t_dict))    # 輸出: (<class 'str'>, <class 'float'>)

# 2. 聯合型別 (Union / Optional) 解析
t_opt = Optional[str]
print(get_origin(t_opt))   # 輸出: typing.Union
print(get_args(t_opt))     # 輸出: (<class 'str'>, <class 'NoneType'>)

# 3. 字面值 (Literal) 與元數據 (Annotated) 解析
t_mode = Literal["r", "w", "a"]
print(get_origin(t_mode))  # 輸出: typing.Literal
print(get_args(t_mode))    # 輸出: ('r', 'w', 'a') (可直接用於檢查使用者輸入是否合法！)

t_meta = Annotated[int, "使用者 ID", {"gt": 0}]
print(get_origin(t_meta))  # 輸出: typing.Annotated
print(get_args(t_meta))    # 輸出: (<class 'int'>, '使用者 ID', {'gt': 0})
```

---

##### 🛠️ 實戰案例：自製執行期泛型型別驗證器

以下展示如何結合 `get_origin()` 與 `get_args()` 實作一個通用的輕量級泛型驗證函式：

```Python
from typing import get_origin, get_args, Literal, Union

def validate_data(val: object, expected_type: type) -> bool:
    """ 在執行期精準驗證資料是否符合複雜的泛型型別註記 """
    origin = get_origin(expected_type)
    args = get_args(expected_type)
    
    # 情況 1: 一般純型別 (例如 int, str)
    if origin is None:
        return isinstance(val, expected_type)
    
    # 情況 2: Literal 字面值型別 (檢查值是否在允許的候選清單中)
    if origin is Literal:
        return val in args
    
    # 情況 3: Union / Optional 聯合型別 (只要符合其中一種型別即為合法)
    if origin is Union:
        return any(validate_data(val, arg) for arg in args)
    
    # 情況 4: list[...] 泛型列表 (檢查外層是 list，且內部每個元素均符合型別)
    if issubclass(origin, list):
        if not isinstance(val, list):
            return False
        elem_type = args[0]
        return all(validate_data(item, elem_type) for item in val)
        
    return isinstance(val, origin)


# 🧪 實測驗證
print(validate_data([1, 2, 3], list[int]))       # ✅ True
print(validate_data([1, "2", 3], list[int]))     # ❌ False (內部包含字串)
print(validate_data("admin", Literal["admin", "guest"])) # ✅ True
print(validate_data("root", Literal["admin", "guest"]))  # ❌ False
print(validate_data(None, Optional[str]))        # ✅ True
```

---

#### Protocol 結構化介面協定 (Duck Typing / 鴨子型別)

在 **Python 3.8+** 引入 (`from typing import Protocol`)，用於實現**鴨子型別 (Duck Typing)** 的靜態型別檢查與介面契約。

- **核心目的**：無需明確繼承父類別，只要物件「擁有相同名稱與簽名的屬性或方法」，就被視為符合該 Protocol 協定。
- **與 ABC (抽象基底類別) 的差異**：
  - **ABC (名義型別 Nominal Typing)**：必須顯式繼承 (`class GeminiProvider(BaseLLMProvider)`)。
  - **Protocol (結構型別 Structural Typing)**：無需繼承，實現解耦與隱式契約。

> **🍿 比喻單元**：  
> ABC 像 **「特長俱樂部驗明正身」**：你必須出示會員證（明確繼承 `BaseLLMProvider`）門衛才讓你進去；  
> Protocol 像 **「特長俱樂部自動報到門」**：門衛不管你皮夾裡有沒有會員證，只要你「走起來像鴨子、叫起來像鴨子（實作了 `.generate_response()`）」，就自動當作你是合格會員放行！

##### 範例代碼：實作 Protocol 模式

```Python
from typing import Protocol, runtime_checkable

# 1. 定義 Protocol 介面協定 (無需繼承 ABC)
@runtime_checkable  # 讓 isinstance() 能在執行期進行檢查
class Renderable(Protocol):
    """定義只要具備 render() 方法的物件即符合 Renderable 協定"""

    def render(self) -> str:
        """【協定方法】子類別/實現類別應實作此方法：回傳渲染後的字串內容"""
        ...


# 2. 具體類別 A (完全沒有繼承 Renderable，但有實作 render())
class MarkdownArticle:
    def render(self) -> str:
        return "# 這是 Markdown 文章內容"


# 3. 具體類別 B (同樣沒有繼承 Renderable)
class HTMLWidget:
    def render(self) -> str:
        return "<div>這是 HTML 元件</div>"


# 4. 函式使用 Protocol 進行型別標註
def display_component(component: Renderable) -> None:
    """傳入的物件不需要繼承特定父類別，只要有 render() 方法即可傳入"""
    print(f"渲染結果: {component.render()}")


# 測試調用
article = MarkdownArticle()
widget = HTMLWidget()

display_component(article)  # ✅ 正確：靜態檢查與執行期均通過
display_component(widget)   # ✅ 正確：靜態檢查與執行期均通過

# 搭配 @runtime_checkable 時可使用 isinstance 檢查
print(isinstance(article, Renderable))  # 輸出: True
```

> **💡 小提醒**：預設情況下，`Protocol` 僅用於 **mypy / IDE 靜態型別檢查**。若要在程式執行期（Runtime）使用 `isinstance(obj, MyProtocol)`，必須在 Protocol 類別上方加上 `@runtime_checkable` 裝飾器！

##### 📊 ABC vs Protocol 對照表

| 比較維度 | 抽象基底類別 (ABC) | 結構化協定 (Protocol) |
| :--- | :--- | :--- |
| **型別系統** | **名義型別 (Nominal Typing)**<br>必須顯式繼承父類別 | **結構型別 (Structural Typing)**<br>只要介面/方法簽名符合即可 |
| **代碼復用** | **✅ 支援**（可在父類別撰寫通用預設實作） | **❌ 不支援**（僅定義介面規格，不寫具體實作） |
| **強迫實作** | **✅ 執行期強迫**（漏寫 `@abstractmethod` 創建物件時報錯 `TypeError`） | **⚠️ 靜態檢查為主**（主要由 IDE / MyPy 提示，執行期預設不限制） |
| **解耦能力** | **較低**（第三方類別必須明確繼承你的 ABC） | **極高**（第三方類別完全無需感知你的 Protocol 存在） |

---

### isinstance\(\) 型別檢查

```Python
isinstance(檢查變數, 型別) # 得到 bool

isinstance(檢查變數, (型別 1, 型別 2, ...)) # 檢查是否為其中一個：放在元組裡
isinstance(檢查變數, 型別 1 | 型別 2 | ...) # 或用 | 隔開
```

不僅對變數型別支援，還支持**類別物件的繼承**

優於 **type\(\) == 型別**

```Python
class Animal:
    pass

class Dog(Animal):  # Dog 繼承自 Animal
    pass

my_dog = Dog()

# 1. 使用 isinstance()：會向上檢查繼承鏈
print(isinstance(my_dog, Dog))     # 輸出: True
print(isinstance(my_dog, Animal))  # 輸出: True (因為狗也是一種動物)

# 2. 使用 type()：嚴格比對，不考慮繼承
print(type(my_dog) == Dog)         # 輸出: True
print(type(my_dog) == Animal)      # 輸出: False
```

---

### 數

#### 整數 int

---

#### 浮點數 float

屬於 **float64** （雙精度浮點數） \-\-\> 64 bit，8 位元

傳統深度學習（如 PyTorch）使用 float32（單精度浮點數）\-\-\> 32 bit，4 位元

而現代大模型使用 bfloat16（半精度浮點數）\-\-\> 16 bit，2 位元

> FP \-\> Float Point
> 
> BF \-\> Brain Float point

> 浮點數在電腦中通常由三部分組成：**正負號（Sign）**、**指數（Exponent，決定數值範圍）**、**尾數（Mantissa/Fraction，決定精準度）**。其通用數學公式為：
> 
> $V = (-1)^s \times (1 + f) \times 2^{e - \text{bias}}$
> 
> 1. **FP64 \(雙精度浮點數，Double Precision\) \-\> 小數點後 16 位**
> 
>     - **位元分配：** 1 位正負號 \+ 11 位指數 \+ 52 位尾數
> 
>     - **Python 原生：** 當你在 Python 寫 x = 3\.14 時，它在記憶體中就是以 FP64 儲存
> 
>     - **特性：** 精準度極高（可達小數點後約 15\-17 位），但佔用記憶體大（8 位元組）
> 
>     - **應用：** 科學計算、金融計算、需要極高精密的幾何運算
> 
> 2. **FP32 \(單精度浮點數，Single Precision\) \-\> 小數點後 7 位**
> 
>     - **位元分配：** 1 位正負號 \+ 8 位指數 \+ 23 位尾數。
> 
>     - **Python 原生：** 不支援。必須使用 NumPy \(np\.float32\) 或 PyTorch \(torch\.float32\)
> 
>     - **特性：** 精準度適中（小數點後約 6\-9 位），記憶體佔用只有 FP64 的一半（4 位元組）
> 
>     - **應用：** 傳統 3D 繪圖、音訊處理、傳圖深度學習（AI）模型的預設常規格式
> 
> 3. **FP16 \(半精度浮點數，Half Precision\) \-\> 小數點後 3 位**
> 
>     - **位元分配：** 1 位正負號 \+ 5 位指數 \+ 10 位尾數
> 
>     - **特性：** 記憶體再減半（2 位元組），計算速度快。但因為**指數位元太少（僅 5 位）**，數值範圍非常小，極易發生「數值溢位（Overflow）」或「底溢（Underflow）」
> 
>     - **應用： AI 模型推理（Inference）、混合精度訓練（Mixed Precision Training）**
> 
> 4. **BF16 \(Brain Floating Point\) \-\> 小數點後 3 位**
> 
>     - **位元分配：** 1 位正負號 \+ **8 位指數** \+ 7 位尾數
> 
>     - **特性：** 由 Google 開發。它刻意保持與 FP32 相同的指數位元（代表**數值範圍與 FP32 完全一樣，溢位不易發生**），但犧牲了尾數的精準度。這在深度學習中非常有用，因為 AI 模型對範圍敏感、對微小精準度不敏感
> 
>     - **應用：** **現代大語言模型（LLM，如 LLaMA、GPT 系列）的訓練與推理**

##### 常數 Constant

**不會更改**的數，

但其實，只是個普通的浮點數（或整數），

在 python 中，**並沒有不能對其修改的限制**

一般用**全大寫的蛇形**來命名

---

##### 特殊浮點數（無限 非數 負零）

- **正／負無限大：**

    ```Python
    float("inf") # 或是 math.inf
    
    float("-inf") # 或是 -math.inf
    ```

- **非數字：**

    在**數據分析**時常用在**空白處的佔位符**

    > 計算時為什麽要用 NaN 而不是 None：
    > 
    > 因為 NaN **本質上還是屬於浮點數**，就算計算後值一率變成 NaN，但至少不會像 None 一樣直接報錯（因為他型別為 NoneType，而不是數字）
    > 
    > 另一個原因為**記憶體的運算速度**，在處理海量資料的套件裡，為了追求極致的運算速度，它們會在記憶體裡開闢一條純數字的專用高速公路，而 NaN 可以走這條高速公路

    ```Python
    float("nan") # 或是 math.nan
    
    x = float("nan")
    print(x + 100)  # 輸出：nan
    print(x * 0)    # 輸出：nan (連乘以 0 都無法消滅它！)
    ```

    > 為什麼 **nan \!= nan（不相等），NaN == NaN 值為 false**
    > 
    > 因為未定義和另一個未定義在數學上**不保證是同一個東西**，
    > 
    > 就像你不知道 A 盒子裡裝什麼（nan），也不知道 B 盒子裡裝什麼（nan），你就不能說 A 等於 B
    > 
    > 所以**不要用 ==（或 is）來處理非數字**

    使用 **math\.isnan\(\) **尋找非數字，函數回傳 True 則為非數字：

    ```Python
    import math
    
    # 假設這是感測器讀取失敗回傳的幽靈數字
    sensor_value = float("nan") 
    
    ~~# ❌ 絕對不能這樣寫，它永遠是 False~~
    ~~if sensor_value == float("nan"):~~
    ~~    print("感測器壞了！") ~~
    
    # ✅ 正確寫法：使用探測器！
    if math.isnan(sensor_value):
        print("警報！感測器回傳無效數字，請重啟設備！")
    ```

- **負零：**

    由於正負號和數字是獨立的，故存在負零（\-0\.0）

    在常規比較 \(==\) 下**等於 0\.0**，僅在極限數學運算時有差異。

---

##### 精準度流失

有時會遇到**精準度流失（Precision Loss）**

> 幾乎所有現代程式語言如 C\#, C\+\+, Java, JavaScript、python 的浮點數都有此問題

```Python
print(0.1 + 0.2) # 輸出 0.30000000000000004 而非 0.3
```

原因在於十進制和二進制間的轉換會出現誤差

1. **人類的無限循環：** 在十進位裡，我們無法完美寫出 1÷3 的小數，只能寫成 0\.33333\.\.\.（無限循環）。

2. **電腦的無限循環：** 同樣的道理，十進位裡看起來很完美的 0\.1，轉換成二進位時，會變成一個**無限循環小數**0\.00011001100110011\.\.\.。

3. **被迫切斷：** 但是電腦的記憶體（通常是 64 位元）空間是有限的！它不可能把無限循環的數字全部存下來，所以它只好在某個位數「強行切斷並四捨五入」。

解法：**使用 decimal**

---

#### 十進位數 Decimal

在軟體層面**模擬人類的十進位數學**，不會有精準度流失的問題，缺點是**運行速度較慢**

要用用字串來指定數字

> 若用數字指定，因精準度流失，程式接收到的數未必會是你指定的數了

```Python
from decimal import Decimal

n1 = Decimal("0.1") # 大寫 D
```

##### getcontext\(\) 精準度設定

Decimal 的設定，為函數

提供：

1. **\.prec：**有效位數，預設 **28**

    ```Python
    from decimal import Decimal, getcontext
    
    # 1. 取得目前的控制台，並把有效位數改成只有 3 位
    getcontext().prec = 3
    
    # 2. 進行除法運算
    # 原本 100 / 7 應該是 14.285714...
    ans = Decimal("100") / Decimal("7")
    
    # 因為我們規定只能記住「3個數字」，所以它會自動在第3位切斷並進位！
    print(ans) 
    # 輸出：14.3 (只保留了 1, 4, 3 三個有效數字)
    ```

2. **\.rounding：**進位及捨去規則，預設**奇進偶捨**

    > 四捨六入五成雙
    > 
    > 1. 尾數 1\~4 捨去，6\~9 進位（往絕對值大）
    > 
    > 2. 為 \.5 時若後方有數字（哪怕是 \.00000001 也是），往**絕對值大的進位**
    > 
    > 3. 為 \.5 且後方沒有數字，看兩邊的數字，往**偶數的進位**（\-3\.8，兩邊有 \-3、\-4，往 \-4 進位）

    ```Python
    from decimal import Decimal, getcontext, ROUND_HALF_UP
    
    # 把規則手冊裡的捨入方式，強制改成 ROUND_HALF_UP (傳統的四捨五入)
    getcontext().rounding = ROUND_HALF_UP
    ```

3. **\.traps：保險絲**，如果遇到除以零、或是數字大到算不出來（溢位），**不要當機 \(False\)，**直接給我一個代表無限大的 Infinity 或是代表錯誤的 NaN 就好

    ```Python
    from decimal import Decimal, getcontext, DivisionByZero
    
    # 告訴控制台：遇到「除以零 (DivisionByZero)」時，不要觸發當機陷阱！
    getcontext().traps[decimal.DivisionByZero] = False # False 意思為關掉此功能
    
    # 故意除以 0 試試看
    ans = Decimal("10") / Decimal("0")
    
    # 程式完全沒有當機！而是優雅地回傳了 Infinity (無限大)
    print(ans) # 輸出：Infinity
    ```

---

#### 分數 Fraction

```Python
from fractions import Fraction

f1 = Fraction(10, 20)
f2 = Fraction(3, 9)

# 會“自動約分至最簡”
print(f1) # 輸出：1/2
print(f2) # 輸出：1/3
```

可放**小數轉為分數**（不能有小數運算，如 0\.1/3），不過 float 依然有精準度流失的問題，故用**字串**：

```Python
f3 = Fraction("0.1") # 輸出 1/10
```

或用**連分數逼近法** \(Continued Fractions\) **\.limit\_denominator\(\)**：

用以尋找在**分母不超過某個極限的情況下**，最接近這個小數的完美分數

預設參數為**一百萬**

```Python
from fractions import Fraction

# 1. 產生浮點數
monster = Fraction(0.1)
print(f"淨化前：{monster}") 
# 輸出：3602879701896397/36028797018963968

# 2. 施展淨化魔法！
pure_fraction = monster.limit_denominator(1000000)

print(f"淨化後：{pure_fraction}") 
# ✅ 輸出：1/10 (完美還原！)
```

---

#### 複數 complex

數學上以 i 來表示，不過在此以** j 來表示**

無法用 \<, \>, \<=, \>= 比較大小，無法計算 // \(整除\), % \(取餘數\)

```Python
n1 = 3 + 4j # 也可為大寫 J

n2 = complex(3, 4) # 用函數來宣告
```

```Python
print(n1.real)  # 輸出：3.0 (實部，Python 會自動轉成浮點數)

print(n1.imag)  # 輸出：4.0 (虛部)

print(n1.conjugate()) # 求共軛複數，輸出：(3-4j)
```

```Python
import cmath

result = cmath.sqrt(-1)
print(result) # 輸出：1j

# cmath 裡面還有處理極座標 (Polar coordinates)、歐拉公式等超強的數學工具
```

> 負數無法用一般的 **math\.sqrt\(\)** 開根號，要用 cmath\.sqrt\(\)

---

### 序列型別 Sequence Types

**內部元素擁有索引值**（Index）

擁有：

1. 索引取物

2. 切片（串列內提到）

3. 拼貼與複製

4. len\(\) 算長度

5. in 檢查是否含有

    ```Python
    numbers = (10, 20, 30) # 這是一個 tuple
    
    print(20 in numbers)   # 輸出：True
    ```

---

#### 字串 str

用單引號或雙引號包起都可以

```Python
name_1 = 'tim'

name_2 = "matthew"
```

- 故其可以不需進行跳脫（Escapee），

    若字串串中有單引號，用雙引號包起，反之相同

    ```Python
    sentence = "It's a beautiful day."
    
    sentence = 'It\'s a beautiful day.'
    
    # 相同意思
    ```

- 多行字串：

    ```Python
    paragraph = """這是第一行。
    這是第二行。
    用三個引號包起來，你只要按下 Enter 換行，
    印出來的結果就會長得一模一樣。"""
    ```

---

##### 跳脫字元 Escape Characters

| 跳脫字元 | 功能說明 | 程式碼範例 | 輸出結果 |
| --- | --- | --- | --- |
| **\n** | 換行 (New Line) | print("第一行\n第二行") | 第一行<br>第二行 |
| **\t** | 水平定位 (Tab)<br>(通常等於 4 個空白) | print("姓名\t分數") | 姓名 分數 |
| **\\\\** | 印出反斜線<br>(避免被當成跳脫字元的開頭) | print("C:\\Windows") | C:\Windows |
| **\'** | 印出單引號<br>(避免提早結束字串) | print('It\'s OK') | It's OK |
| **\"** | 印出雙引號 | print("他說：\"你好\"") | 他說："你好" |
| **\r** | 游標移至行首 (Carriage Return)<br>(常用來做終端機的進度條覆寫) | print("12345\rAB") | AB345 *(AB 蓋掉了 12)* |
| **\b** | 退格 (Backspace)<br>(刪除前一個字) | print("ABC\bD") | ABD *(C 被刪除了)* |


---

##### 字串前綴四大天王 String Prefixes

在 Python 中，字串引號前面可以加上特定的**字母前綴（Prefix）**，用來改變字串的解析方式或資料型別（大小寫皆可）：

| 前綴符號 | 英文全稱 | 核心功能與說明 | 典型實戰適用情境 |
| :--- | :--- | :--- | :--- |
| **`r"..."`** | **Raw String** (原始字串) | **完全關閉反斜線 `\` 的跳脫功能**，讓字元保持最原始狀態 | 正則表達式 (Regex)、Windows 檔案路徑 |
| **`f"..."`** | **Formatted String** (格式化字串) | 支援在字串中以 `{變數}` 或 `{運算式}` 直接插值 (Python 3.6+) | 文字排版、日誌輸出、動態字串拼接 |
| **`b"..."`** | **Bytes Literal** (位元組字串) | 建立二進位 `bytes` 型別資料，每個字元代表 1 個 Byte (0-255) | 網路 Socket 傳輸、檔案加密雜湊、二進位圖片/音訊處理 |
| **`u"..."`** | **Unicode String** (統一碼字串) | 宣告為 Unicode 字串（Python 2 用於區分 ASCII；Python 3 所有字串預設即為 Unicode，可省略） | 跨版本相容性代碼 |

---

###### 1. `r"..."` (Raw String 原始字串)

- **核心作用**：關閉反斜線 `\` 的跳脫轉譯，防止 `\n` 被當作換行、`\t` 被當作 Tab。
- **典型應用**：正規表達式、Windows 磁碟路徑。

```python
# 1. 解決 Windows 路徑跳脫災難
normal_path = "C:\new_folder\test"  # \n 會被誤判為換行，\t 會被誤判為 Tab！
raw_path = r"C:\new_folder\test"    # 安全！保持原字串 C:\new_folder\test

# 2. 解決正規表達式 (Regex) 反斜線地獄
pattern = r"\d{4}-\d{2}-\d{2}"      # 免寫繁瑣的 \\d{4}-\\d{2}-\\d{2}
```

---

###### 2. `f"..."` (Formatted String 格式化字串)

- **核心作用**：在字串中直接嵌入 Python 變數、運算式或格式化指令。

```python
name = "Alice"
score = 95.567
print(f"學生: {name} | 成績: {score:.1f}")  # 輸出: 學生: Alice | 成績: 95.6
```

---

###### 3. `b"..."` (Bytes Literal 二進位位元組字串)

- **核心作用**：產生不可變的二進位位元組序列 (`<class 'bytes'>`)，而非普通字串 (`<class 'str'>`)。
- **型別互轉**：
  - `str ➔ bytes`：`"hello".encode("utf-8")` 或是 `b"hello"`
  - `bytes ➔ str`：`b"hello".decode("utf-8")`

```python
# 1. 普通字串 vs Bytes 字串
text_str = "hello"
bytes_data = b"hello"

print(type(text_str))    # <class 'str'>
print(type(bytes_data))  # <class 'bytes'>

# 2. 網路傳輸或加密計算 (如 hashlib 雜湊必須傳入 bytes)
import hashlib
md5_hash = hashlib.md5(b"my_secret_password").hexdigest()
print(f"MD5 雜湊值: {md5_hash}")
```

---

###### 4. `u"..."` (Unicode String 統一碼字串)

- **核心作用**：在 Python 2 時代用來明確指定字串為 Unicode 編碼（防止中文字亂碼）。
- **現代 Python 3 地位**：Python 3 中**所有普通字串預設就是 Unicode**，因此 `u"你好"` 與 `"你好"` 完全等價，保留主要是為了向下相容 Python 2/3 雙版本代碼。

---

###### 🌟 進階組合技：`rf"..."` / `fr"..."` (Raw + Formatted)

從 Python 3.8+ 起，前綴可以自由組合！**`rf"..."` 既可以忽略反斜線跳脫，又可以進行變數插值**：

```python
folder_name = "reports"
file_name = "2026.csv"

# 既包含動態變數，又包含 Windows 反斜線
full_path = rf"C:\data\{folder_name}\{file_name}"
print(full_path)  # 輸出: C:\data\reports\2026.csv
```

---

##### 字串格式化 String Formatting

將**變數直接塞入字串中**的方式

有三種方式（新版本隊就方式也兼容）：

1. **f\-str（3\.6 版本）：**

    ```Python
    name = "Alice"
    age = 25
    
    # 語法：字串前面加 f，變數直接塞進 {} 裡
    greeting = f"你好，我是 {name}，今年 {age} 歲。"
    print(greeting) # 輸出：你好，我是 Alice，今年 25 歲。
    
    # 甚至可以直接在裡面算數學或呼叫函數！
    print(f"明年我就 {age + 1} 歲了。") # 輸出：明年我就 26 歲了。
    ```

    1. **數值與金錢排版（千位數逗號、限制小數點）：**

    ```Python
    price = 1234567.89123 # :, 代表加上千位數逗號；.2f 代表保留兩位小數 (會自動四捨五入)
    print(f"總金額為：${price:,.2f}") 
    # 輸出：總金額為：$1,234,567.89
    ```

    2. **百分比轉換：**

    ```Python
    success_rate = 0.8765 # .1% 代表轉成百分比，並保留一位小數
    print(f"成功率：{success_rate:.1%}") 
    # 輸出：成功率：87.7%
    ```

    3. **除錯神技（Python 3\.8）：** 如果你想印出變數的名字和它的值，以前要寫 print\(f"user\_id=\{user\_id\}"\)。現在只要在變數後面**加上一個等號 =**，Python 會自動幫你把名字和值一起印出來！

    ```Python
    user_id = 9527 # 自動印出變數名稱與數值
    print(f"{user_id=}") 
    # 輸出：user_id=9527
    ```

2. **\.format\(\)（3\.0 版本）：**

    ```Python
    name = "Bob"
    age = 30
    
    # 按照“順序”填入空位
    text = "你好，我是 {}，今年 {} 歲。".format(name, age)
    
    # 也可以幫空位取名字，“順序就不重要了”
    text2 = "你好，我是 {n}，今年 {a} 歲。".format(a=age, n=name)
    ```

3. **%（2\.0 版本）：**有嚴格的**型別區分**

    - **%s：**字串

    - **%d：**整數

    - **%f：**浮點數

    ```Python
    name = "Alice"
    age = 25
    height = 165.5
    
    # 語法： "字串裡面挖洞" % (依序放入變數的元組 Tuple)
    text = "姓名：%s，年紀：%d 歲，身高：%.1f cm" % (name, age, height)
    
    print(text) 
    # 輸出：姓名：Alice，年紀：25 歲，身高：165.5 cm
    ```

---

##### 常用字串拆分與合併方法 (splitlines, split, join, strip)

###### 函數

###### str.splitlines() 按行分割多行字串 (Multi-line Split)

- **使用時機**：將包含換行符號的大字串，**按行切割**為一個字串清單 (`list[str]`)。
- **核心優勢（為何比 `split('\n')` 更優雅強大？）**：
  1. **跨平台換行符自動相容**：同時支援 Windows (`\r\n`)、macOS/Linux (`\n`) 與舊式 Mac (`\r`)。
  2. **不留空白尾巴**：字串末尾若帶有換行符號，`split('\n')` 會在串列末尾切出一個令人頭痛的空字串 `""`；而 `splitlines()` 會**自動完美忽略末尾換行符**！
- **語法**：`str.splitlines(keepends=False)`
- **參數說明**：
  - `keepends`：布林值，預設為 `False`（去除換行符）；若傳入 `True`，切割後的每行末尾會保留原有的 `\n` 或 `\r\n`。
- **回傳值**：
  - `list[str]`：切分後的單行字串串列。

```python
text = "第一行內容\r\n第二行內容\n第三行內容\n"

# 1. 使用 splitlines() (推薦！自動相容跨平台換行符且無尾巴空字串)
lines = text.splitlines()
print(lines)
# 輸出: ['第一行內容', '第二行內容', '第三行內容']

# 2. 傳統 split('\n') 的陷阱 (末尾會多出 ""，且 Windows 檔案會殘留 \r)
bad_lines = text.split("\n")
print(bad_lines)
# 輸出: ['第一行內容\r', '第二行內容', '第三行內容', '']

# 3. 保留換行符號 (keepends=True)
raw_lines = text.splitlines(keepends=True)
print(raw_lines)
# 輸出: ['第一行內容\r\n', '第二行內容\n', '第三行內容\n']
```

> **🍿 生動白話比喻**：  
> 傳統 `split('\n')` 像**用固定形狀的普通刀子切吐司**：遇到 Windows 特規換行符 (`\r\n`) 鋸不乾淨（殘留 `\r`），切到吐司最底邊還會留下一小塊討厭的吐司邊 (`""`)；  
> 而 `splitlines()` 像**全自動智能切片機**：自動識別各種跨平台換行標籤，切出來的每一片 (`line`) 乾淨漂亮，完全不留垃圾廢料！

---

###### str.split() 按分隔符或空白切割字串

- **使用時機**：當你需要將一個字串按照特定分隔符號（如逗號、冒號）或連續空白切分成字串串列時使用。
- **語法**：`str.split(sep=None, maxsplit=-1)`
- **參數說明**：
  - `sep`：（可選）分隔符字串。預設為 `None`（會將**連續的任意空白字元、Tab、換行符視為一個分隔符**並自動去除首尾空白）。
  - `maxsplit`：（可選）最大分割次數，預設為 `-1`（全部分割）。若設為 `1` 則只切第一刀分成 2 段。
- **回傳值**：
  - `list[str]`：切割後的字串串列。

```python
# 1. 預設分割 (自動合併連續多個空格與換行)
text = "  Python    AI   Agent  \n"
print(text.split())  # 輸出: ['Python', 'AI', 'Agent']

# 2. 指定分隔符 (例如 CSV 逗號)
tags = "python,ai,ollama"
print(tags.split(","))  # 輸出: ['python', 'ai', 'ollama']

# 3. 限制分割次數 (maxsplit=1，只切一刀，常見於解析 key=value)
kv = "timeout=30=seconds"
print(kv.split("=", maxsplit=1))  # 輸出: ['timeout', '30=seconds']
```

---

###### str.join() 字串合併與拼接

- **使用時機**：當你需要將串列、元組或任何可迭代容器中的多個字串元素，以指定的「分隔符號」連接拼接成一個完整字串時使用（例如組合 CSV 逗號資料、網址路徑、反轉字串還原或多行字串重組）。
- **語法**：`"分隔符".join(iterable)`
- **參數說明**：
  - `iterable`：包含字串元素的可迭代容器（如 `list[str]`、`tuple[str]`、`set[str]` 或生成器）。
- **回傳值**：
  - `str`：由分隔符串接所有元素後的全新字串。

- **反常識語法結構（初學者 90% 會寫錯！）**：
  - ❌ **錯誤習慣寫法**：`my_list.join(",")`（在 Python 中 `list` 沒有 `.join()` 方法）
  - ✅ **正確語法結構**：**`"分隔符字串".join(可迭代容器)`**
  - **為什麼語法這樣設計？**：因為「分隔符」本身是一個 `str` 物件，`.join()` 是 `str` 類別內建的方法，負責在傳入的清單元素之間「塞入」這個分隔符！

- **核心效能優勢（為何遠比用 `+` 號相加好得多？）**：
  - Python 中的字串是**不可變物件 (Immutable)**。如果用 `for` 迴圈搭配 `s += item` 拼接，每次 `+` 號都會在記憶體中重新申請空間並複製舊字串，時間複雜度高達 $O(N^2)$！
  - 使用 `"連結符".join(list)` 時，Python 會事先精確計算出總記憶體長度，**一次性分配完畢並完成填入**，時間複雜度僅為 $O(N)$，效能極致優越！

```python
# 1. 基礎情境：用逗號與空格分隔串列 (最常見)
fruits = ["Apple", "Banana", "Cherry"]
result1 = ", ".join(fruits)
print(result1)  # 輸出: Apple, Banana, Cherry

# 2. 無縫直接串接：使用空字串 "" (常搭配 reversed 進行字串反轉)
letters = ["P", "y", "t", "h", "o", "n"]
result2 = "".join(letters)
print(result2)  # 輸出: Python

# 3. 多行文字重組：用換行符 "\n" 拼接
lines = ["第一行", "第二行", "第三行"]
result3 = "\n".join(lines)
print(result3)

# 4. 網址或路徑段落組裝
url_parts = ["https:", "", "api.example.com", "v1", "users"]
result4 = "/".join(url_parts)
print(result4)  # 輸出: https://api.example.com/v1/users
```

> **⚠️ 最常見陷阱：串列內包含數字（非字串）會引發 `TypeError`**：  
> 若清單內包含整數或浮點數（如 `[1, 2, 3]`），直接傳給 `.join()` 會報錯：`TypeError: sequence item 0: expected str instance, int found`。  
> **💡 2 大黃金解決方案**：
> ```python
> numbers = [10, 20, 30, 40]
> 
> # 方案 A：生成器運算式 (推薦！直觀易讀)
> text_A = "-".join(str(x) for x in numbers)
> print(text_A)  # 輸出: 10-20-30-40
> 
> # 方案 B：使用內建 map(str, numbers) 高效轉換
> text_B = "-".join(map(str, numbers))
> print(text_B)  # 輸出: 10-20-30-40
> ```

---

###### str.strip() 去除字串前後首尾空白與換行符 (Strip Whitespace)

- **使用時機**：當你需要移除字串**開頭 (左端)** 與 **結尾 (右端)** 的所有多餘空白字元（包含空格 `' '`、縮排 Tab `'\t'`、換行符 `'\n'` 或 `'\r'`），避免印出多餘空行或排版歪斜時使用。
- **語法**：`str.strip(chars=None)` / `str.lstrip(chars=None)` / `str.rstrip(chars=None)`
- **參數說明**：
  - `chars`：（可選）指定要剔除的字元集合字串。預設為 `None`（剔除所有空白與換行字元）。
- **回傳值**：
  - `str`：剔除指定字元後的全新字串。

- **`strip()` 家族 3 大成員對比**：
  - `.strip()`：同時剔除**左右兩端**的空白（最常用）。
  - `.lstrip()` (Left)：只剔除**左端 (開頭)** 的空白/縮排。
  - `.rstrip()` (Right)：只剔除**右端 (末尾)** 的空白/換行符。

```python
raw_line = "   return True\n"

# 1. 不用 strip() 的情況 (會殘留縮排與換行)
print(f"app.py:10:{raw_line}")
# 輸出: app.py:10:   return True (中間卡了3個空格縮排)

# 2. 加上 strip() 的情況 (乾淨頂格對齊，最美觀)
print(f"app.py:10:{raw_line.strip()}")
# 輸出: app.py:10:return True

# 3. 指定剔除特定字元 (非空白字元也能剔除)
dirty_text = "=== Hello World ==="
clean_text = dirty_text.strip("= ") # 去除等號與空格
print(clean_text)  # 輸出: Hello World
```

---

##### 字串搜尋

###### 函數

###### str.startswith() 檢查字串開頭

- **使用時機**：當你需要確認字串是否以指定的字串（或多個候選前綴）開頭時使用（例如檢查網址協定是否為 `https://` 或檔名開頭）。
- **語法**：`str.startswith(prefix, start=0, end=len(string))`
- **參數說明**：
  - `prefix`：要比對的前綴字串；亦可傳入**元組 (Tuple)**，只要符合其中任一前綴即回傳 `True`。
  - `start`：（可選）搜尋起始索引位置，預設為 `0`。
  - `end`：（可選）搜尋結束索引位置，預設為字串長度。
- **回傳值**：
  - `bool`：若字串以指定前綴開頭回傳 `True`，否則回傳 `False`。

```python
filename = "test_demo.py"

# 1. 基礎前綴檢查
print(filename.startswith("test_"))  # 輸出: True

# 2. 傳入 Tuple 檢查多個前綴
url = "https://example.com"
print(url.startswith(("http://", "https://")))  # 輸出: True
```

---

###### str.endswith() 檢查字串結尾

- **使用時機**：當你需要確認字串是否以指定的字串（或多個候選後綴）結束時使用（例如篩選 `.jpg` 或 `.png` 圖片副檔名）。
- **語法**：`str.endswith(suffix, start=0, end=len(string))`
- **參數說明**：
  - `suffix`：要比對的後綴字串；亦可傳入**元組 (Tuple)**，只要符合其中任一後綴即回傳 `True`。
  - `start`：（可選）搜尋起始索引位置。
  - `end`：（可選）搜尋結束索引位置。
- **回傳值**：
  - `bool`：若字串以指定後綴結尾回傳 `True`，否則回傳 `False`。

```python
file = "avatar.png"

# 1. 檢查副檔名
print(file.endswith(".png"))  # 輸出: True

# 2. 傳入 Tuple 多重副檔名檢查
print(file.endswith((".jpg", ".jpeg", ".png")))  # 輸出: True
```

---

###### str.find() / str.rfind() 搜尋子字串索引位置 (安全型)

- **使用時機**：當你需要尋找子字串在母字串中第一次或最後一次出現的位置，且希望**找不到時不要報錯**時使用。
- **語法**：`str.find(sub, start=0, end=len(string))` / `str.rfind(sub, start=0, end=len(string))`
- **參數說明**：
  - `sub`：要尋找的子字串。
  - `start`：（可選）搜尋起始位置。
  - `end`：（可選）搜尋結束位置。
- **回傳值**：
  - `int`：找到時傳回子字串第一個字元的**索引值**（從 0 開始）；**找不到時傳回 `-1`**。

```python
text = "hello python world python"

# 1. find(): 從左至右尋找第一個出現的位置
print(text.find("python"))  # 輸出: 6

# 2. rfind(): 從右至左尋找最後一個出現的位置
print(text.rfind("python"))  # 輸出: 19

# 3. 找不到時傳回 -1，不會崩潰
print(text.find("java"))  # 輸出: -1
```

---

###### str.index() / str.rindex() 搜尋子字串索引位置 (嚴格型)

- **使用時機**：當你確定目標子字串必定存在，且希望在**找不到時直接拋出例外**（中斷程式防呆）時使用。
- **語法**：`str.index(sub, start=0, end=len(string))` / `str.rindex(sub, start=0, end=len(string))`
- **參數說明**：
  - `sub`：要尋找的子字串。
  - `start`：（可選）搜尋起始位置。
  - `end`：（可選）搜尋結束索引位置。
- **回傳值**：
  - `int`：找到時傳回索引值；**找不到時拋出 `ValueError`**。

```python
text = "hello python"

# 找到時傳回索引值
print(text.index("python"))  # 輸出: 6

# 找不到時會引發 ValueError 錯誤
try:
    print(text.index("java"))
except ValueError as e:
    print("找不到子字串！")  # 輸出: 找不到子字串！
```

---

###### str.count() 計算子字串出現次數

- **使用時機**：當你需要計算某個子字串在整段文字中總共出現了幾次時使用。
- **語法**：`str.count(sub, start=0, end=len(string))`
- **參數說明**：
  - `sub`：要統計出現次數的子字串。
  - `start`：（可選）搜尋起始位置。
  - `end`：（可選）搜尋結束位置。
- **回傳值**：
  - `int`：子字串在指定範圍內非重疊出現的次數。

```python
text = "banana"

# 計算 "a" 出現的總次數
print(text.count("a"))  # 輸出: 3

# 計算 "ana" 非重疊出現次數
print(text.count("ana"))  # 輸出: 1
```

---



#### 串列 list


想像為置物櫃，依序放入資料，且用索引讀取，

類似於 C\# 的**陣列**，

不同的是它可以接受**不同型別的資料**

> ```Python
> mix = [30, 20.5, "tim"] # 型別混合
> ```

```Python
fruits = ["apple", "banana", "orange"]

# 印出資料：
print(fruits[0])

groups =[["tim, matthew"], ["nini", "leo"], "tina"]
#串列內可再有串列
```

- **負數索引：**

    ```Python
    print(fruits[-1])
    
    # 倒數第一個（orange）
    ```

- **新增資料：**

    ```Python
    fruits.append("watermelon")
    # ["apple", "banana", "orange", "watermelon"]
    
    fruits.append(1, "guava")
    # 插隊到索引 1，變：["apple","guava", "banana", "orange", "watermelon"]
    
    fruits.extend([apple, guava]) 
    # 會把裡面的串列給拆開，不直接當一個元素傳入
    ```

- **修改資料：**

    ```Python
    fruits[0] = "big apple"
    ```

- **刪除資料：**

    ```Python
    del fruits[1]
    
    # 刪除**特定索引**資料
    
    fruits.remove("banana")
    # 刪除**特定名稱**或**值**的資料
    
    n3_fruits = fuirts.pop(2) 
    # 刪除**並取出**特定索引的資料（參數留空則對**最後一個資料**進行動作）
    
    fruits.clear()
    # 刪除**整個**串列
    ```

- **切片：**

    串列\[起始索引 : 結束索引 : 步長（選）\]，且**包頭不包尾**（起始 1 結束 4 \-\-\>從 1 拿到 3）

    ```Python
    letters = ["A", "B", "C", "D", "E", "F", "G"]
    # 索引值：   0    1    2    3    4    5    6
    
    # 從索引 1 拿到索引 4 (不包含索引 4 的 "E")
    print(letters[1:4])  
    # 輸出：['B', 'C', 'D']
    ```

    ```Shell
    # 拿取「前三個」元素 (省略起點，從頭開始切到索引 3)
    print(letters[:3])   
    # 輸出：['A', 'B', 'C']
    
    # 拿取「從索引 4 開始到最後」的所有元素 (省略終點)
    print(letters[4:])   
    # 輸出：['E', 'F', 'G']
    
    # 複製整個串列 (頭尾都省略)
    print(letters[:])    
    # 輸出：['A', 'B', 'C', 'D', 'E', 'F', 'G']
    ```

    ```Python
    # 從頭到尾，每「2 步」拿一個 (也就是拿取所有偶數索引的值)
    print(letters[::2])  
    # 輸出：['A', 'C', 'E', 'G']
    ```

    ```Shell
    # 拿取「最後三個」元素
    print(letters[-3:])  
    # 輸出：['E', 'F', 'G']
    
    # ❗️ 把整個串列「反轉」！(步長設為 -1，代表從尾巴往回走)
    print(letters[::-1]) 
    # 輸出：['G', 'F', 'E', 'D', 'C', 'B', 'A']
    ```

---

#### 元組 tuple

理解為**串列**，但為**不可變物件**，

可以查看、讀取，但**無法修改、新增、刪除**

> 還是可以**給予新的元素的**
> 
> ```Python
> t = (1, 2, 3)
> print(id(t)) # 假設是位址 A
> 
> # 直接給一個全新的元組 (重新貼標籤)
> t = (99, 2, 3)
> print(id(t)) # 變成位址 B 了！完全合法！
> ```

```Python
coordinates = (10, 20, 30)

null_tuple = () # 空元組

# 讀取資料：一樣用中括號和索引值
print(coordinates[0])  # 輸出：10

# 切片操作：一樣支援
print(coordinates[:2]) # 輸出：(10, 20)
```

```Python
# 不加括號，直接用逗號隔開
my_tuple = 10, 20, "Apple"
```

```Python
t = (100,)
# 需在後面加上" , "

~~not_tuple = (100)~~
# 屬於 int，非元組
```

- 解包：

    可以將元素打包起來放入元組，當然也可解包它，將元素取出

    ```Python
    coordinates = 120.5, 23.4  # 打包成元組
    
    # 直接把元組裡的值，分別解包塞給 x 和 y
    x, y = coordinates 
    
    print(x) # 輸出：120.5
    print(y) # 輸出：23.4
    ```

---

#### \[:\] 切片

```Python
預切片序列[起點:終點:步長] # 包頭不包尾
```

```Python
預切片序列[起點:終點]

nums = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

print(nums[2:6]) # [2, 3, 4, 5]
```

```Python
預切片序列[:終點] # 從頭包到終點（不包尾）

預切片序列[起點:] # 起點取到最後（包尾）

預切片序列[:] # 整個複製
```

```Python
預切片序列[::2] #[0, 2, 4, 6, 8]
```

- 索引為**負**代表**反過來取（從最後開始，負幾就是倒數第幾個）**，一樣**包頭不包尾**

---

##### slice\(\) 切片

\[:\] 的切片方式其實就是利用此函數來達成，當我們要把切片數據保留下來，就必須使用此函數

```Python
... = slice(起始, 終點, 步長)

... = slice(None, None, 2) # 同等於 [::2]
```

---

### 雜湊 Hashing

分為兩種：

#### 可／不可雜湊 Hashable/Unhashable

屬於**資料結構的雜湊**，

目的是**讓程式跑的更快**

**字典、集合**的搜尋速度可以如此快，原因是在資料**存放進去時**利用**雜湊函數預先計算好雜湊值**，搜尋時直接利用這個值**快速定位**

**邏輯：計算雜湊值 \-\-\> 定位抽屜號碼 \-\-\> 開抽屜取放值**

- **hash\(\) 函數**的作用在於產生一個**整數索引**，此索引決定資料存在字典／集合的**哪裡**

    ```Python
    # hash() 把任何東西變成一個整數
    print(hash("王小明"))    # → 某個整數，例如 12345
    print(hash(42))         # → 42
    print(hash((1, 2, 3)))  # → 某個整數
    ```

    其擁有**一致性**，**同樣的值會有同樣的雜湊值**

    > 為了防止**雜湊碰撞攻擊（Hash DoS Attack）**，重啟後會得到不同雜湊值
    > 
    > **雜湊碰撞攻擊**是指同時輸入大量雜湊值相同的值，全部存在同一抽屜，導致伺服器癱瘓

    ```Python
    # 同一次程式執行中，相同的值一定得到相同的雜湊
    hash("Hello") == hash("Hello")  # 永遠是 True
    ```

    對於**雜湊函數**，**不需關注（密碼學需要）：**

    1. **可逆：**不需由雜湊值推回存放物件

    2. **雪崩效應：**只是為了尋找出資料放置位置，不需增加複雜度

        > 原始資料發生**微小改變**，輸出的雜湊值**徹底且不可預測的改變**

    3. **安全性**（只追求效能）

    ---

- **可否雜湊與可變及不可變物件的關係：**

    - **可雜湊代表為不可變物件**（反之不成立）

    - 可**雜湊必能成為字典的 key 或集合的值**

    > **可變物件無法雜湊的原因：**
    > 
    > 假設某串列計算出的雜湊值是 100，被分配到第 10 格，
    > 
    > 而對串列進行修改後，**雜湊值改變**，就會對應到**不同格**（新雜湊值 200，對應 20 格），程式**尋找第 20 格**，**但資料依然在 第 10 格**，成資料丟失
    > 
    > 也就是：**資料改變 \-\-\> 尋找新雜湊值，但資料依然在舊雜湊值**

---

**兩種型別的 key 利用了可雜湊的特性：**

---

##### 字典 Dictionary

為一個**包含大量變數的物件**

查找內容物時，不需要他的 **key（鍵）**就能得到對應值（**鍵值對 \(Key\-Value Pair\)），**也就是擁有**映射（Mapping）**

- 有以下特性：

    1. 查找速度**極快**

    2. **Key** 是唯一的，且為**不可變物件（為可雜湊物件）**（通常為字串或數字），值可以為任何型別的物件

```Python
# 建立一個空字典
empty_dict = {}
# 或者也可以這樣寫 (使用內建函數)
empty_dict = dict()

# 建立一個帶有初始資料的字典 (注意：字串要加引號，數字與布林值不用)
ai_model = {
    "name": "Gemini",
    "version": 3.1,
    "is_active": True
}
```

```Python
# 方法一：使用中括號 [] (直接精準指定)
print(ai_model["name"])  # 輸出: Gemini
# ⚠️警告：如果 Key 不存在 (例如 ai_model["price"])，程式會直接報錯

# 方法二：使用 .get() 方法 (安全第一，強烈推薦)
print(ai_model.get("version"))       # 輸出: 3.1
print(ai_model.get("price"))         # 輸出: None (找不到不會報錯，預設回傳 None)
print(ai_model.get("price", 999))    # 輸出: 999 (找不到時，回傳你設定的預設值 999)
```

```Python
# 新增一個原本沒有的 Key
ai_model["company"] = "Google"
# 現在字典裡多了一組 "company": "Google"

# 修改一個已經存在的 Key
ai_model["version"] = 3.5
# 原本的 3.1 被覆蓋掉了

# 一次更新或新增多筆資料：使用 .update() 方法
ai_model.update({"parameters": "1.5T", "is_active": False})
```

```Python
# 方法一：使用 del 關鍵字
del ai_model["company"]

# 方法二：使用 .pop() 方法 (刪除的同時，可以把那個值拿出來存進另一個變數)
old_version = ai_model.pop("version")
print(old_version)  # 輸出: 3.5
```

```Python
# 1. 只拿出所有的 Keys (鍵)
for k in ai_model.keys():
    print(k)

# 2. 只拿出所有的 Values (值)
for v in ai_model.values():
    print(v)

# 3. **同時拿出 Key 和 Value** (開發 AI 時處理資料最常用的語法！)
for k, v in ai_model.items():
    print(f"屬性 {k} 的值是 {v}")
```

---

- **`dict.values()` 取得字典所有值的動態檢視物件**

回傳包含字典中所有**值 (Values)** 的動態檢視物件 (`dict_values`)。

> **🍿 生動白話比喻**：  
> 把字典想像成「全校學生成績冊」（Key 是學號，Value 是數學分數）。`dict.values()` 就像**「只抽印出所有分數的成績清單」**。你不在乎是誰考幾分，只想一口氣計算全校總分 `sum()`、最高分 `max()` 或平均值！

- **核心特性與型別說明**：
  - **回傳型別**：回傳 `dict_values` 可疊代物件 (Iterable)，非標準 Python List。
  - **動態視窗 (Dynamic View)**：`.values()` 是字典記憶體的即時鏡像。若原字典內容發生修改，`dict_values` 物件裡的值會**自動即時同步更新**！

- **程式碼範例**：

  ```python
  # 1. 建立示範字典 (學生成績單)
  scores = {"Alice": 95, "Bob": 88, "Charlie": 92}

  # 2. 取得所有分數 (dict_values 物件)
  all_scores = scores.values()
  print(all_scores)  # 輸出: dict_values([95, 88, 92])

  # -------------------------------------------------
  # ❌ 錯誤寫法：直接對 dict_values 使用索引下標 [0]
  # first_score = all_scores[0] # 💥 拋出 TypeError: dict_values object is not subscriptable

  # ✅ 正確寫法 1：使用 list() 強制轉型後即可使用索引取值
  score_list = list(scores.values())
  print(score_list[0])  # 輸出: 95

  # ✅ 正確寫法 2：搭配聚合函數進行快速數學運算 (無需手動轉成 list)
  print(f"總分: {sum(scores.values())}")   # 輸出: 總分: 275
  print(f"最高分: {max(scores.values())}") # 輸出: 最高分: 95
  print(f"最低分: {min(scores.values())}") # 輸出: 最低分: 88

  # ✅ 正確寫法 3：透過 for 迴圈逐一遍歷
  for s in scores.values():
      print(f"得分: {s}")
  ```

> **💡 `.values()` 記憶體優勢與轉型技巧**：  
> 1. **記憶體極致節省**：`.values()` 建立的是動態視窗 (View)，不會在記憶體中複製第二份實體 List，適合處理百萬筆巨量資料！  
> 2. **索引取值需轉 List**：若需要 `values[0]` 按下標取值，請務必先用 `list(scores.values())` 強制轉型。

---

##### 集合 set

類似串列，但**會過濾重複的元素**（只留下一個）

也類似字典，只有 key 沒有值

通常會將**串列轉成集合**，以此過濾重複

- 只能放**不可變物件（為可雜湊物件）**

> 透過**雜湊值**確認是否重複，物件需為 **Hashable**

```Python
num = {1, 1, 2, 3, 3}

print(num) # 輸出 {1, 2, 3}

non = set() # 建立空集合，不能 {} 因為會變空字典
```

它**沒有索引（無順序）**，我們無法指定索引叫出元素

```Python
my_set = {10, 20, 30}

# ❌ 以下這行會直接當機！
# TypeError: 'set' object is not subscriptable
print(my_set[0]) 

# ✅ 你只能問 Python：「某個人有沒有在裡面？」
print(20 in my_set) # 輸出：True
```

可對兩個集合進行以下操作：

- **聯集 \| **

- **交集 \& **

- **差集 \- **

- **對稱差集 ^ （取不重複的）**

```Python
A = {1, 2, 3, 4}
B = {3, 4, 5, 6}

print(A|B) # {1, 2, 3, 4, 5, 6}

print(A&B) # {3, 4}

print(A-B) # {1, 2}

print(a^B) # {1, 2, 5, 6}
```

---

#### 密碼學雜湊 hashlib

一種**數學演算法**，

將資料輸入，經過運算後輸出一個**長度固定的雜湊值**

有以下特性：

1. **不可逆：**無法從雜湊值**反推**原始資料

2. **抗碰撞：**雖然長度固定，數學上必定存在兩份不同的資料產生相同的雜湊值（稱為碰撞），但**現實中發生機率近似 0**

3. **雪崩效應：**原始資料發生**微小改變**，輸出的雜湊值**徹底且不可預測的改變**

4. 運算速度適中：它必須能快速計算出結果，但又**不能太快**。 如果是用來校驗檔案完整性，速度越快越好；但如果是用來儲存密碼，現代系統反而會**刻意把雜湊運算變慢**（例如重複雜湊幾萬次），目的是為了大幅增加駭客使用暴力破解法時的時間成本。

```Python
import hashlib

message = "Hello, World!".encode('utf-8')  # 必須是 bytes

# MD5（較舊，不建議用於安全用途）
print("MD5:", hashlib.md5(message).hexdigest())
# 輸出: 65a8e27d8879283831b664bd8b7f0ad4

# SHA-1（已不安全，但仍常見）
print("SHA-1:", hashlib.sha1(message).hexdigest())
# 輸出: 0a0a9f2a6772942557ab5355d76af442f8f65e01

# SHA-256（目前主流，推薦使用）
print("SHA-256:", hashlib.sha256(message).hexdigest())
# 輸出: dffd6021bb2bd5b0af676290809ec3a53191dd81c7f70a4b28688a362182986f

# SHA-512（更長，安全性更高）
print("SHA-512:", hashlib.sha512(message).hexdigest())
# 輸出: 374d794a95cdcfd8b35993185fef9ba368f160d8daf432d08ba9f1ed1e5abe6cc69291e0fa2fe0006a52570ef18c19def4e617c33ce52ef0a6e5fbe318cb0387
```

應用：

1. **密碼安全性：不要直接儲存密碼，要用雜湊保護**

    ```Python
    import hashlib
    
    def hash_password(password):
        """將密碼轉換成雜湊值"""
        return hashlib.sha256(password.encode('utf-8')).hexdigest() # 進行雜湊
    
    def verify_password(password, stored_hash):
        """驗證密碼是否正確
        
        驗證雜湊值，返回布林值"""
        return hash_password(password) == stored_hash
    
    # 註冊時
    user_password = "MySecretPassword123"
    stored_hash = hash_password(user_password) # 調用函數進行雜湊
    print(f"儲存在資料庫的雜湊: {stored_hash}")
    
    # 登入時
    login_attempt = "MySecretPassword123"
    if verify_password(login_attempt, stored_hash): # 調用函數檢查布林值
        print("登入成功！")
    else:
        print("密碼錯誤！")
    
    # 輸出：
    # 儲存在資料庫的雜湊: b169822e0e15ac5a2cccc3f740d5f263b501003d183928fc434653ecebd744bc
    # 登入成功！
    ```

2. **檔案完整性驗證：確認下載的檔案沒有被篡改**

    > **提供原始檔案的雜湊值，下載後驗證是否相同**，以驗證是否經過竄改

    ```Python
    import hashlib
    
    def calculate_file_hash(filepath):
        """計算檔案的 SHA-256 雜湊值"""
        sha256_hash = hashlib.sha256()
    
        with open(filepath, "rb") as f:
            # 分塊讀取，避免大檔案佔用太多記憶體
            for chunk in iter(lambda: f.read(4096), b""):
                sha256_hash.update(chunk)
    
        return sha256_hash.hexdigest()
    
    
    # 使用範例
    file_hash = calculate_file_hash("my_file.zip")
    print(f"檔案雜湊值: {file_hash}")
    
    # 比對官方提供的雜湊值
    official_hash = "abc123..."  # 從官網取得
    if file_hash == official_hash:
        print("✅ 檔案完整，未被篡改")
    else:
        print("❌ 警告：檔案可能已被修改！")
    ```

3. 資料去重：判斷資料是否重複

    ```Python
    import hashlib
    
    def get_content_hash(content):
        """計算內容的雜湊值"""
        return hashlib.md5(content.encode('utf-8')).hexdigest()
    
    
    # 使用範例：找出重複的文章
    articles = [
        "這是第一篇文章的內容...",
        "這是第二篇文章的內容...",
        "這是第一篇文章的內容...",  # 重複！
    ]
    
    seen_hashes = set()
    unique_articles = []
    
    for article in articles:
        article_hash = get_content_hash(article)
        if article_hash not in seen_hashes:
            seen_hashes.add(article_hash)
            unique_articles.append(article)
        else:
            print(f"發現重複文章（雜湊: {article_hash[:8]}...）")
    
    print(f"原本 {len(articles)} 篇，去重後 {len(unique_articles)} 篇")
    ```











---

### 布林 bool

只有 **True、False** 兩個值，分別又被當作** 0、1**（可計算）



---

### 轉型

1. **隱式轉換 Implicit Conversion：**當兩個不同型別計算時，**自動**把比窄的型別轉為**比較寬的**（容納更多數據）

    - Int \+ float \-\-\> float

    - Bool \+ int/float \-\-\> int/float

2. **顯式轉換 Explicit Conversion：強制**進行轉換（使用內建函數）

    - **int\(x\)：**強制轉成整數（無條件捨去小數）

    - **float\(x\)：**強制轉成浮點數

        - **complex\(x\) / complex\(實部, 虛部\)：**強制轉成複數

    - **str\(x\)：**強制轉成字串（萬物皆可轉字串）

        - **ord\(x\)：**字元轉**編碼**

        - **chr\(x\)：**編碼轉字元

    - **list\(x\)：**強制轉成串列

    - **tuple\(x\)：**強制轉成元組

    - **dict\(x\)：**強制轉成字典

    - **set\(x\)：**強制轉集合

    - **bool\(x\)：**強制轉成布林值（數子除了 0 都為 True，字串內有東西就為 True）

| 轉換函數 (目標) | 輸入：數字 (int / float) | 輸入：字串 (str) | 輸入：容器 (list/set/dict) |
| --- | --- | --- | --- |
| **int(x)** | 捨去小數<br>`int(3.9)` $\rightarrow$ `3` | 必須是純整數字串<br>`int("3")` $\rightarrow$ `3`<br>`int("3.1")` $\rightarrow$ ❌ 報錯 | ❌ 報錯<br>*(容器不能轉整數)* |
| **float(x)** | 正常加小數點<br>`float(3)` $\rightarrow$ `3.0` | 可吃小數或整數字串<br>`float("3.14")` $\rightarrow$ `3.14` | ❌ 報錯 |
| **str(x)** | 完美包裝<br>`str(3.14)` $\rightarrow$ `"3.14"` | 維持原樣 | 長相直接變字串<br>`str([1, 2])` $\rightarrow$ `"[1, 2]"` |
| **bool(x)** | 0 為 False，其餘 True<br>`bool(-1)` $\rightarrow$ `True` | 空字串 "" 為 False，其餘 True<br>`bool("False")` $\rightarrow$ `True` | 空容器為 False，裡面有東西為 True |

| 轉換函數 (目標) | 輸入：字串 (str) | 輸入：串列/元組 (list/tuple) | 輸入：字典 (dict) |
| --- | --- | --- | --- |
| **list(x)**<br>**tuple(x)** | ✂️ 碎紙機效應<br>`list("ABC")` $\rightarrow$ `['A', 'B', 'C']` | 無痛互轉<br>`tuple([1, 2])` $\rightarrow$ `(1, 2)` | 🔑 只會抓出 Key!<br>`list({'a':1, 'b':2})` $\rightarrow$ `['a', 'b']` |
| **set(x)** | ✂️ 碎紙機 + 去重複<br>`set("AAB")` $\rightarrow$ `{'A', 'B'}` | 過濾重複，順序打亂<br>`set([1, 2, 2])` $\rightarrow$ `{1, 2}` | 🔑 只抓 Key + 過濾重複<br>`set({'a':1, 'a':2})` $\rightarrow$ `{'a'}` |
| **dict(x)** | ❌ 報錯<br>*(字串無法成雙成對)* | 👯‍♀️ 必須是「二維雙人組」<br>`dict([("a", 1), ("b", 2)])` $\rightarrow$ `{'a': 1, 'b': 2}` | 複製出一個新字典 |

| 轉換函數 | 成功的輸入條件與結果 | 失敗的輸入 (必當機) |
| --- | --- | --- |
| **complex(x)** | ✅ 輸入數字 $\rightarrow$ `complex(5)` 變成 `(5+0j)` <br> ✅ 輸入格式正確字串 $\rightarrow$ `complex("1+2j")` | ❌ 字串內有空白 `complex("1 + 2j")`<br>❌ 丟入容器 |
| **ord(x)** | ✅ 只能吃「長度剛好為 1」的字串<br>`ord("A")` $\rightarrow$ `65 (ASCII 電腦編碼)` | ❌ 字串太長 `ord("AB")`<br>❌ 丟入數字 `ord(65)` |
| **chr(x)** | ✅ 只能吃「有效的整數編碼」<br>`chr(65)` $\rightarrow$ `"A"` | ❌ 丟入字串 `chr("65")`<br>❌ 丟入小數 `chr(65.0)` |


---

## None 空值

代表**什麼都沒有**（不是 0、空字串\.\.\.\.\.\.\.），為不存在的狀態，也屬於一個**物件**

- 判斷時用 **is/not is **做

用在：

1. 佔位符：還不知道要放什麼，先給一個 None

2. 代表找不到、失敗

3. 函數的預設回傳值

```Python
# 假設我們去資料庫抓玩家資料，但找不到這個人
player_data = None 

# ✅ 專業、安全的標準寫法
if player_data is None:
    print("找不到該名玩家！")

# ✅ 檢查「不是空值」的寫法
if player_data is not None:
    print("成功載入玩家資料！")
```

---

## 記憶體管理

主要依賴**引用計數（Reference Counting）和循環垃圾回收（Cycle Garbage Collection）（也就是 GC）**來管理記憶體

| 比較項目 | 系統一：引用計數 (Reference Counting) | 系統二：循環垃圾回收 (GC / Cyclic GC) |
| --- | --- | --- |
| **定位** | 第一線守衛 (處理 99% 的日常回收) | 終極清道夫 (專門處理 1% 的極端死角) |
| **運作機制** | 關注引用數 | 從程式主幹出發，順著線往下摸。摸不到的，全部判定為垃圾。 |
| **觸發時機** | **【即時觸發】**<br>當計數變成 0 的那一瞬間。 | **【定期/條件觸發】**<br>當記憶體新增的物件數量達到特定門檻，或手動呼叫 `gc.collect()` 時才會啟動。 |
| **處理速度** | 極快 (幾乎不耗費運算資源) | 極慢 (需要暫停程式，掃描大量物件) |
| **能否處理循環？** | ❌ 不行 (互相牽手會導致計數永遠不為 0) | ✅ 可以 (只要脫離主幹，互相牽手照樣清除) |


> ```Python
> class Node:
>     def __init__(self):
>         self.friend = None
> 
> a = Node()
> b = Node()
> 
> # 建立循環：互相牽手
> a.friend = b  
> b.friend = a
> ```
> 
> a 引用 b，b 也引用 a，執行 del a 和 del b 時，**已經沒有這兩個物件了**，**但記憶體裡仍有殘留**
> 
> 必須裡用 **GC **才能清除記憶體佔用

---

### del 刪除引用

用於刪除**變數**（含數字、字典、串列等）、**屬性**（屬性裝飾器需 @屬性名\.\.deleter）的引用

利用**引用計數**系統，**每使用一次 del 物件的引用數 \-1**，而直到為 0，物件立即被銷毀

```Python
del 刪除物

del a
del player_1.weapon
```

但資料不會立即消失（**只是先把連結刪除了**），也就是說**刪除的是對資料的引用**（**變數**），不是資料本身

> ```Python
> a = 100
> b = a
> 
> del a
> 
> print(b) # 100
> ```
> 
> 資料（100）還是存在，**只是 a 這個引用被刪除了**
> 
> 對於屬性也同樣道理

而**資料本身**的刪除則是由 **GC 負責**

---

### GC 回收機制

又稱**環垃圾回收（Cycle GC）**

順著**程式的運行順序追蹤**，若**找不到此物件則判定為垃圾**（即使有引用數，例如出現循環引用）

- 為了維持順暢度，採用**分代（Generational）**機制，

    將物件分為 3 種：

    1. **0 代：剛創**的物件，如迴**圈計數的暫存變數**，此處的掃描機率最高

    2. **1 代：**0 代掃描後存活下來就會來到此，掃描次數下降

    3. **2 代：**1 代掃描後存活下來就會來到此，掃描機率更低，通常會是**全域變數**，**核心設定檔**等，

        除非記憶體不夠才會掃描到此，掃描到此會稱為全域大掃除（Full GC）

        ---

- 觸發時機為：物件的**新增與消除差額過大**（**沒被引用計數刪除的太多**時）

    ```Python
    import gc
    print(gc.get_threshold()) 
    # 預設輸出通常是：(700, 10, 10)
    ```

    **預設情況下：700/10/10**

    1. **0 代：**物件**物件累積**超過 700 時開始掃描

    2. **1 代：0 代掃描 10 次**， 1 代也會掃描 1 次

    3. **2 代：1 代 10 次**，掃描 1 次

---

# 運算式 expression

運算式由**運算元（operand）**和**運算子（operator）**組成，

前者就是**物件**（包括數字、字串、串列、字元、變數名稱等），又稱**字面值**（literal），後者則是要對物件進行的操作

---

## 算術運算子 Arithmetic Operators

| 運算符號 | 名稱 | 說明 | 範例 | 運算結果 |
| --- | --- | --- | --- | --- |
| **+** | 加法 | 兩數相加 | a + b | |
| **-** | 減法 | 兩數相減 | a - b | |
| **\*** | 乘法 | 兩數相乘 | a * b | |
| **\*\*** | 次方 (指數) | 計算 a 的 b 次方。 | 2 ** 3 | 8 |
| **/** | 除法 (浮點數) | 兩數相除。**注意：在 Python 中，/ 永遠會回傳小數 (float)**，即使能夠整除。 | 10 / 2 | 5.0 |
| **//** | 整除 (地板除法) | 相除後**無條件捨去小數**，只保留整數部分。 | 10 // 3 | 3 |
| **%** | 取餘數 | 取得相除後剩下的餘數。常用來判斷奇偶數或倍數。 | 10 % 3 | 1 |
| **+值** | 求正值 | 開絕對值 | +x | |
| **-值** | 求負值 | 開絕對值加負號 | -x | |


- 對**字串、串列、元組**也可使用部分

    ```Python
    "My name is" + "Matthew"
    # My name is Matthew
    
    [0, 1, 2] + [3, 4, 5]
    # [0, 1, 2, 3, 4, 5]
    
    (0, 1, 2) + (3, 4, 5)
    # (0, 1, 2, 3, 4, 5)
    ```

    ```Python
    "cat" * 2
    # catcat
    
    [0, 1, 2] * 2
    # [0, 1, 2, 0, 1, 2]
    
    (0, 1, 2]) * 2
    # (0, 1, 2, 0, 1, 2)
    ```

- **複合指派：**

    ```Python
    x = 2
    
    x += 2 # x=x+2
    ```

---

## 比較運算子 Comparison Operators

- 做**大小比較**：

| 運算符號 | 名稱 | 說明 | 範例 (x = 10, y = 5) | 判斷結果 |
| --- | --- | --- | --- | --- |
| **==** | 等於 | 判斷兩邊的「值」是否一模一樣 | x == 10 | True |
| **!=** | 不等於 | 判斷兩邊的「值」是否不一樣 | x != y | True |
| **<>** | 不等於 | 和 != 同，python 3 無法使用 | x <> y | True |
| **>** | 大於 | 左邊是否大於右邊 | x > 15 | false |
| **<** | 小於 | 左邊是否小於右邊 | y < x | True |
| **>=** | 大於等於 | 左邊是否大於或等於右邊 | x >= 10 | True |
| **<=** | 小於等於 | 左邊是否小於或等於右邊 | y <= 5 | True |


- **成員運算子 Membership Operators：**

    檢查**某元素是否在容器物件**（字串、串列、元組、字典）裡

    ```Python
    title = "Python 系統性學習指南"
    
    print("Python" in title)   # 輸出：True (有包含這個詞)
    print("Java" not in title) # 輸出：True (確實不包含 Java，所以回傳 True)
    print("python" in title)   # 輸出：False (⚠️ 注意：Python 嚴格區分大小寫！)
    ```

    ```Python
    fruits = ["蘋果", "香蕉", "橘子"]
    
    print("香蕉" in fruits)  # 輸出：True
    print("葡萄" in fruits)  # 輸出：False
    
    # ⚠️ 新手常犯的錯：它比對的是「完整的元素」，不是元素裡的一部分
    print("蘋" in fruits)    # 輸出：False (清單裡有「蘋果」，但沒有單獨一個「蘋」字)
    ```

    ```Python
    student = {
        "name": "Matthew",
        "score": 95
    }
    
    # 檢查 Key (鍵)
    print("name" in student)   # 輸出：True (字典裡確實有 "name" 這個標籤)
    print("age" in student)    # 輸出：False (字典裡沒有 "age" 這個標籤)
    
    # ❌ 試圖檢查 Value (值)
    print("Matthew" in student) # 輸出：False！(因為 in 不會去看 Value)
    
    print("Matthew" in student.values()) # 輸出：True (這樣正確)
    ```

- **同等關係運算子（身分運算子 Identity Operator）：**

    判斷兩個名稱是否指向同一物件

    ```Python
    a = [3, 2]
    b = [1, 2]
    
    a is b # false 為不同物件
    
    a is not b # True
    ```

    ⚠️ 判斷值相不不相等時，永遠用 **==，不要用在數字及字串中**，

    只有在判斷 **None**，或是刻意要檢查**記憶體位址**時，才動用 **is**

    （因為成快取及駐留機制）

---

## 邏輯運算子 Logical Operators

- **and 且：**兩者**同時**成立為 True，否則 False

    > **短路 Short\-Circuit：**
    > 為追求效能，and 檢查第一個為 false 時或 or 檢查第一個為 True 時，就會**直接確定值，不進行下一個判斷**
    > 
    > 裡用短路可達成檢查條件符合才執行（**短路求值**）：
    > 
    > ```Python
    > 條件 and 操作
    > # 條件為 True 才會執行操作
    > ```

- **or 或：**兩者**其一**成立就會為 True

- **not 反轉：**將布林值**反轉**

---

## 運算子優先順序

| 優先順序 | 運算子 | 名稱 / 說明 |
| --- | --- | --- |
| **最高 1** | `()` <br> `[]` <br> `{}` | 括號與資料結構<br>小括號（群組/函數呼叫）、中括號（索引/切片）、大括號（字典/集合） |
| **2** | `**` | 指數（次方） |
| **3** | `+x`, `-x`, `~x` | 正號、負號、位元反轉<br>*(例如把 x 變成負數 -x)* |
| **4** | `*`, `/`, `//`, `%` | 乘法、除法、整除、取餘數<br>*(這就是「先乘除」的部分)* |
| **5** | `+`, `-` | 加法、減法<br>*(這就是「後加減」的部分)* |
| **6** | `<<`, `>>` | 位元移位運算 (Bitwise shifts) |
| **7** | `&` | bitwise AND |
| **8** | `^` | bitwise XOR |
| **9** | `\|` | bitwise OR |
| **10** | `==`, `!=`, `>`, `>=`, `<`, `<=`, `is`, `is not`, `in`, `not in` | 比較運算子、身分運算子、成員運算子<br>*(注意：它們的優先權全部一樣大，會連鎖判斷)* |
| **11** | `not` | 邏輯反轉 (NOT) |
| **12** | `and` | 邏輯交集 (AND) |
| **13** | `or` | 邏輯聯集 (OR) |
| **最低 14** | `=`, `+=`, `-=`, `*=`, `/=`, 等 | 指派運算子 (Assignment)<br>*(永遠是最後才把算完的結果「貼上標籤」)* |


---

# 述句基本 Statement

又稱**陳述句**，

述句是用來對電腦**下達一個完整動作指令**的

| 述句分類 | 關鍵字 / 名稱 | 語法範例 | 核心功能說明 |
| --- | --- | --- | --- |
| **基礎動作** | 賦值 (Assignment) | x = 10<br>a += 1 | **貼標籤 / 存資料**：算出等號右邊的值，並將變數標籤貼上去。 |
| | 運算式 (Expression) | print("Hi")<br>list.append(1) | **單純執行**：呼叫函數或計算數值，算完就把結果丟掉，不存起來。 |
| | 空述句 (Pass) | pass | **佔位符**：什麼都不做，用來維持語法結構的完整性，避免報錯。 |
| **條件判斷** | If / Elif / Else | if score > 60:<br>elif score == 60:<br>else: | **十字路口**：根據條件是 True 還是 False，決定程式要走哪條路。 |
| **迴圈控制** | While | while count > 0: | **條件重複**：只要條件成立，就永無止境地一直重複執行區塊。 |
| | For | for item in data: | **輸送帶**：把容器（如串列、字串）裡的東西一個一個拿出來處理。 |
| | Break | break | **緊急煞車**：立刻、瞬間打破並逃離整個迴圈。 |
| | Continue | continue | **跳過本回合**：放棄這次迴圈剩下的程式碼，直接進入下一次迭代。 |
| **結構與模組** | 定義函數 (Def) | def say_hello(): | **發明機器**：將一段程式碼打包成一個可以重複使用的專用工具。 |
| | 回傳值 (Return) | return result | **吐出結果**：(專用於函數內) 讓函數結束執行，並把算好的值交出來。 |
| | 匯入 (Import) | import random | **擴充工具箱**：把 Python 內建的或其他工程師寫好的外部功能拿來用。 |
| **其他** | 定義類別 (Class) | class Robot: | **打造設計圖**：物件導向程式設計 (OOP) 的核心，用來創造型體。 |
| | 例外處理 (Try) | try: ... except: | **安全氣囊**：嘗試執行程式，如果發生當機錯誤，就攔截並處理它。 |
| | 刪除 (Del) | del my_list[0] | **銷毀**：從記憶體中強制撕毀變數標籤，或刪除容器裡的特定資料。 |


---

## 運算式述句 Expression Statement

單純的運算式就叫運算式述句

```Python
1 + 2

1 + 2; 3 + 4; 5 + 6;
# 一次回傳多個值
```

---

## 指派述句 Assignment Statement

```Python
x = 3 + 2
# 將運算式指派給 x

a = b = c = 10
# 連鎖指派

d, e = 100, 80
# 多重指派

f += x
# f+x 重新指派給 f

# 串列或元組也可重新指派：
x = [100, 80]

x += [0, 20]
# x 為 [100,80, 0, 20]
```

---

### **序列指派 Sequence Assignment**

變數的解包

迅速把物件內的元素抽**取出來**（**拆解物件**）（**解包**）

```Python
# 拆解串列 (List)
x, y, z = [10, 20, 30]
# x=10 y=20 x=30

# 拆解元組 (Tuple)，省略括號
first_name, last_name = ~~(~~"John", "Doe"~~)~~

# 拆解字串 (String) - 每個字母會被獨立拆開！
a, b, c = "Cat"
# a=C b=a c=t
```

**用星號**來**序列指派：**星號的部分打包起來變串列或元組（看原本是什麼），其餘看變數設的位子

若有 \[1, 2, 3, 4, 5, 6\]，打包 2, 3, 4，則：

```Python
num = [1, 2, 3, 4, 5, 6]

n1, *other, n5, n6 = num
# n1=1、other=[2, 3, 4]、n5=5、n6=6
```

而也可在串列裡包串列或元組，反之也可

```Python
# 外面是元組，裡面包著串列，串列裡又包著字串...
complex_data = (1, ["A", "B"], "Cat")

# 左邊精準對應結構
num, [letter1, letter2], (c1, c2, c3) = complex_data

print(letter1) # 輸出：A
print(c2)      # 輸出：a (把 "Cat" 的第二個字母拆出來了)
```

---

## If 條件判斷

判斷 True、False，進行流程控制（flow contorl）

```Python
if 條件（運算式）:
    ....
elif 條件:
    .... (選填，可多個）
else:
    .... （選填，單個）
```

- **elif**：其他**成立**的情況

- **else**：所有條件**不成立時**執行

- 會被當成**假**的（if、elif 不成立）有** False、None、0（整數及浮點數）、""（空字串）、\[\]（空串列）、空元組、空字典、空集合**

---

### 條件運算式

**簡化 if **的方式，若**條件簡單**時可使用縮減行數，

若邏輯複雜，還是使用 if 較優

```Python
變數 = 條件成立時給的值 if 判斷條件 else 條件不成立時給的值
```

```Python
if battery > 20:
    status = "電量正常"
else:
    status = "電量過低"
```

```Python
status = "電量正常" if bettery>20 "電量過低"
```

---

## pass 述句

**縮排處一定要有東西**，若無時，需用到 pass 述句來表示此處什麼都不做**（留空）**

可用在** if、while、for、函數**等

```Python
if 條件:
    pass
elif 條件:
    .... 
```

---

## 迴圈述句

### while 迴圈

若條件**為真**，就不斷重複內容，直到為假

在**不知到要執行幾次時使用**

```Python
while 條件:
    ....
```

---

### for 遍歷迴圈

非像 C\# 的計數器，而是直接**遍歷（迭代）物件裡的每一個元素**

> 把一個箱子（串列、字串、字典）放在輸送帶上，for 迴圈就會自動把裡面的東西**一個一個拿出來**，交給你處理，**直到箱子空了為止**

```Python
for 變數名稱 in 可疊代物件:
    ....
```

- 從物件內取出的元素會放在變數內（下一輪迴圈會被覆蓋，為**暫存**）

    ---

#### 搭配 range\(\) 計數

range 生成的是一個包含**起點、終點、步長的 range 物件**，但可**直接放入 for 中當作計數器使用**（**串列生成式**也可）
> 步長可以理解為是：**每輪迴圈結束後，要對起點 + 步長**，
> 所以如果我們把起點終點對調（起點大終點小），**步長為負數**，可以達到**從後向前遍歷的效果**

```Python
fruits = ["蘋果", "香蕉", "橘子", "葡萄", "西瓜", "芒果"]

# 我們知道串列長度是 6。我們從索引 0 開始，到 6 結束，每次跳 2 步
# i 會依序變成：0, 2, 4
for i in range(0, 6, 2): 
    # (起點, 終點, 間隔)
    # 拿這個產生的數字 i，去串列裡面抓東西
    print(fruits[i])
```

---

#### enumerate 計數器

回傳**索引值以及迭代元素**（兩個被打包成**元組**回傳）

```Python
fruit = ["apple", "banana", ...]

List(enumerate(fruit)) # [(0, "apple"), (1, "banana"), ...]
```

```Python
for idx, 迭代元素 in enumerate(迭代元素, start=計數器從序號幾開始（預設 0）)
# 可得到 索引，以及對應的元素（從迭代元素內取出的）
```

---

#### 串列生成式 List Comprehension

**縮減** for 的方式，還可加上 if

```Python
# 基礎公式：
... = [對取出變數的操作 for 變數 in 迭代物件]

# 加上篩選條件：
... = [對取出變數的操作 for 變數 in 迭代物件 if 篩選條件（符合的才取出進行操作）]
```

```Python
radii = [1, 2, 3, 4, 5]

# 邏輯：我要 [r 的平方] (放進去的東西)，從 [radii] 裡面依序拿出 [r]
areas = [r ** 2 for r in radii]

print(areas) # 輸出：[1, 4, 9, 16, 25]
```

> ```Python
> radii = [1, 2, 3, 4, 5]
> areas = []
> 
> for r in radii:
>     areas.append(r ** 2)
>     
> print(areas) # 輸出：[1, 4, 9, 16, 25]
> ```

---

#### 產生器運算式 Generator Expression

在要另外對運算式產生結果做大量運算時，若用串列生成式，會一次性把所有權重的數量都先遍歷出來

> 例如遍歷出模型內所有參數，並要對其數量做加總時

產生器運算式會在最外層加總函數需要第一個數值時，先給出第一個，一個一個給，將低記憶體佔用

```Python
... = (對取出變數的操作 for 變數 in 迭代物件) # 得到產生器物件
```

##### 產生器物件

產生器物件是一個型態為 generator 的**特殊迭代器**，它利用底層的函數結構來記憶程式執行進度，進而實現暫停執行、交回主控權、需要時再計算的惰性求值機制

---

### break 終止迴圈

若已經**不需要繼續進行**迴圈可用以停止迭代，

通常搭配 **if** 使用

```Python
fruits = ["蘋果", "香蕉", "毒蘋果", "葡萄", "西瓜"]

for fruit in fruits:
    if fruit == "毒蘋果":
        print("🚨 警告！發現毒蘋果！產線立刻全面停工！")
        break  # 啟動緊急煞車，整個 for 迴圈直接死亡
    
    print(f"正常包裝：{fruit}")

print("產線檢查結束。")

# 正常包裝：蘋果
# 正常包裝：香蕉
# 🚨 警告！發現毒蘋果！產線立刻全面停工！
# 產線檢查結束。
```

---

### continue 停止此輪

跳過此輪，繼續下一輪迭代，

通常搭配 **if** 使用

```Python
fruits = ["蘋果", "香蕉", "爛掉的橘子", "葡萄", "西瓜"]

for fruit in fruits:
    if fruit == "爛掉的橘子":
        print("🗑️ 把爛橘子丟掉，換下一個！")
        continue  # 跳過這次迴圈的下半部，直接去拿「葡萄」
    
    # 如果是爛橘子，絕對不會執行到這一行
    print(f"正常包裝：{fruit}")
    
# 正常包裝：蘋果
# 正常包裝：香蕉
# 🗑️ 把爛橘子丟掉，換下一個！
# 正常包裝：葡萄
# 正常包裝：西瓜
```

---

### else 無中斷執行

若迴圈**沒有被 break 停止**，執行完後可進入到 else 區域，

常用於檢查

```Bash
passwords = ["1234", "abcd", "admin"]

for pwd in passwords:
    if pwd == "hacker":
        print("發現駭客密碼！立刻封鎖！")
        break
else:
    # 只有當上面的迴圈「完整檢查完畢」且「沒有踩到 break」時，才會來到這裡
    print("✅ 檢查完畢，沒有發現任何駭客密碼，系統安全！")
```

---

# 函數 function

把一些程式打包起來，寫成一個函數，需要使用時調用此函數，

避免多次重複寫一樣的功能

---

## 函數概念

```Python
def 函數名稱 (參數, ....): -> 型別注記
    """文件字串（函數說明）"""
    
    ....
    
    return ....
```

有五個部分：

1. **輸入（參數）**

2. **型別注記（可選）**

    ```Python
    def 函數名稱 (參數, ....): -> 型別 1|型別 2
    ```

3. **Docstring（文件字串，可選）**

4. **處理區（運作邏輯）**

5. **回傳（沒有則回傳 None）**

> 沒有回傳值：
> 
> ```Python
> # 這是一台只負責「執行動作」的機器，不需要 return
> def open_gripper():
>     # 這裡面可能包含了控制硬體的複雜邏輯
>     print("啟動馬達...")
>     print("夾爪已完全張開！")
> 
> open_gripper()
> ```
> 
> 其實**不是真的沒有回傳值**，而是**編譯器自動加入了 return None**

用以下方式**呼叫函數**：

```Python
函數名稱 (參數, ....)
```

---

### 和方法的關係

函數是獨立的一個程式區塊，**不屬於任何物件或資料型態**

而方法則是屬於**某物件或類別**內部的函數，**依附於物件或資料型態**

方法又分為**實體方法、類別方法**

| 名稱 | 定義位置 | 操作對象 | 呼叫方式 | 第一個參數 |
| --- | --- | --- | --- | --- |
| **函數 (Function)** | 類別 class 的外面 | 獨立運作，處理傳入的參數 | func_name() | 無強制規定 |
| **實體方法 (Method)** | 類別 class 的裡面 | 特定的實體物件 (例如 p1, p2) | p1.method() | 必須是 **self** |
| **類別方法 (Class Method)** | 類別裡面 + @classmethod | 整個類別本身 (共用設定) | Player.method() | 必須是 **cls** |


```Python
func()

m.method() # m 為物件，方法透過物件呼叫

c.c_method()
```

什麼時候用誰？

- **通用時：**當動作是通用型工具（可以套用在很多不同種類的東西上），用**函數**

- **專屬時：**當動作依賴於或專屬於某種特殊資料時，用**方法**

---

### 文件字串 Docstring

**單行或多行**，

存在於**模組、類別、函數、方法的****第一行**（若不放在第一行**不會報錯**，但**會沒有作用**（無法調用函數、讀取屬性））

分成三個詳細的部分：

1. **簡介**

2. **詳細說明**

3. **參數及回傳的定義**

```Python
def calculate_grade(score, is_absent=False):
    """計算學生的最終成績等級。（這是第一行：簡短總結）

    這裡可以寫更詳細的說明。例如：這個函數會根據學生的原始分數，
    轉換成 A, B, C 等級。如果學生缺席，則直接給予 F。
    （這是第二區塊：詳細描述，與第一行之間必須空一行）

    Args:
        score (int 或 float): 學生的原始分數，範圍應在 0 到 100 之間。
        is_absent (bool, optional): 學生是否缺席期末考。預設為 False。

    Returns:
        str: 回傳成績等級，例如 'A', 'B', 'C' 或 'F'。
        
    Raises:
        ValueError: 如果輸入的分數不在 0~100 的範圍內，會拋出此錯誤。
    """
    if is_absent:
        return 'F'
    if not (0 <= score <= 100):
        raise ValueError("分數必須在 0 到 100 之間")
    
    if score >= 90: return 'A'
    elif score >= 80: return 'B'
    else: return 'C'
```

兩個方法**讀取**：

```Python
help(函數名)
```

```Python
print(函數名.__doc__)
```

---

### 參數

分為為：

**形式參數**（形參）：定義函數時寫入的變數名稱，簡稱**參數**
**實際參數**（實參）：呼叫函數時，實際傳入的值，簡稱**引數**

- **位置參數（參數和引數並存）**概念：函數**定義了幾個參數**，就要**傳入多少個**值（引數），且**依照順序**傳入函數

    和它相對的有**關鍵字參數**，指定傳入的實參放在哪個形參，**不需照順序**

    ```Python
    def test(a, b):
        ....
        
        return ....
    
    test(b=100, a=20) # 不需照順序
    ```

以下屬於**參數**：

- **預設參數：**定義函數時，可先設定參數預設值（呼叫時不需傳入這個值，可直接使用預設）

    ⚠️ 預設參數要放在非預設參數後面，否則報錯

    ```Python
    # 我們預設客人都是喝「微糖」和「正常冰」
    def make_coffee(size, sugar="微糖", ice="正常冰"):
        print(f"為您製作一杯：{size}，{sugar}，{ice}的咖啡！")
    
    # 客人 A：只講大小，其他用預設的
    make_coffee("大杯") 
    # 輸出：為您製作一杯：大杯，微糖，正常冰的咖啡！
    
    # 客人 B：想改甜度，冰塊維持預設
    make_coffee("小杯", sugar="全糖") 
    # 輸出：為您製作一杯：小杯，全糖，正常冰的咖啡！
    ```

    若**傳入參數**，**預設將被忽略**

    若預設參數為可變物件（串列、字典等），傳入參數將被留下

    ```Python
    def test(item, box=[]):
         """ box 為空串列 """
         box.append(item)
         return box
    
    print(test("tim"))
    
    print(test("matthew"))
    
    # ['tim']
    # ['tim', 'matthew'] 第一次呼叫的傳入的被留下了
    ```

    ```Python
    def test(item, box=None):
         """ 預設串列為空值，進入函數後在設為空串列，但若有傳入串列則不設為空串列 """
         if box is None:
             box = []
         box.append(item)
         return box
    
    print(test("tim"))
    
    print(test("matthew"))
    
    print(test("nini", ["leo", "cat"]))
    
    # ['tim']
    # ['matthew']
    # ['leo', 'cat', 'nini'] 若有傳入串列時
    ```

    ---

### 函數打包及拆包

#### **不定長度參數打包 \* 及拆包 \*\*** 

把**多出來**的參數都打包成一個**元組**

```Python
def test(*a):
    print(a)

test(1, 2, 3) # 傳入了三個值，被打包成元組

# 直接傳入元組長這樣：
test((1, 2, 3))
```

定義的參數有星號（\*），表示會**吃下所有**的**位置引數**（Positional Argument）

```Python
def test(a, *b, c):
    """ 無埨傳入多少東西，第二個開始的所有物件將被 b 取走，c 沒有值（報錯）"""
    ....    
```

**兩顆星**的，將有 **key** （關鍵字）的值打包成**字典**

```Python
def test(**a):
    print(a)

test(x=1, y=2)
# {'x': 1, 'y': 2}
```

通常這樣為他們命名：

```Python
def func(*args, **kwargs):
    print("一顆星 args 收到:", args)
    print("兩顆星 kwargs 收到:", kwargs)

func(100, 200, "Hello", name="Matthew", target_major="CS")

# 一顆星 args 收到: (100, 200, 'Hello')
# 兩顆星 kwargs 收到: {'name': 'Matthew', 'target_major': 'CS'}
```

> args：Argumeants（引數）
> 
> kwargs：Keyword Arguments（關鍵字引數）

---

#### 引數開箱 Argument Unpacking

和序列指派概念同，為函數的解包，

直接將串列、元組、字典等的資料取出傳入函數內

```Python
basic_settings = ["Transformer-V2", 500]
hyper_params = {"learning_rate": 0.001, "optimizer": "AdamW"}

# 開箱：
train_model(*basic_settings, **hyper_params)
```

---

### return 回傳

函數**必有回傳值**，若沒寫出，則回傳 None

若需回傳多個物件，則需透過串列、元組等來回傳

**而遇到 return 時，函數將會終止**

```Python
def check_age(age):
    """ 若年齡小於 18，在第 4 行時就會回傳且終止函數 """
    if age < 18:
        return "未成年禁止進入"
    return "歡迎進入"

print(check_age(15))
```

---

### 一等公民特性 First\-class citizen

也稱一級物件（First\-class object）

函數可以被當作一個普通的物件進行操作，

可進行：

1. 賦值：

    ```Python
    def test():
      print("嗨")
    
    other = test
    
    other()
    ```

2. 函數名稱當作引數傳給另一函數（**回呼函數 Callback Function**）：

    ```Python
    def shout(text):
        return text.upper()
    
    def whisper(text):
        return text.lower()
    
    # 這個函數的設計，是接收另一個「函數」作為引數
    def process_message(action_func, text):
        # 在內部執行傳進來的函數
        return action_func(text) # action_func 是 shout 或 whisper 直接執行這兩個函數
    
    print(process_message(shout, "Hi"))   # 輸出：HI
    print(process_message(whisper, "Hi")) # 輸出：hi
    ```

3. **作為函數的回傳值（閉包 Closure）：**

    ```Python
    def a(text):
        i = "!!!"
        def b():
            print(test + i)
        return b
    
    msg = a("hello") # 函數賦值
    msg()
    # hello!!!
    ```

    > 1. 呼叫 a 函數，傳入參數 hello
    > 
    > 2. i 被定義
    > 
    > 3. 函數 a 回傳 b 函數，故 msg = b
    > 
    > 4. 呼叫 msg，等同呼叫 b 函數
    > 
    > 5. b 函數進行字串拼噎

4. **置於資料結構中：**

    把函數放入**字典、串列等裡面**

    ```Python
    def add(a, b):
        return a + b
    def sub(a, b):
        return a - b
    
    # 把函數存進字典裡，當作 Value
    calculator = {
        "加法": add,
        "減法": sub
    }
    
    # 透過 Key 拿出函數並當場執行
    print(calculator["加法"](10, 5))  # 輸出：15
    ```

---

### 變數生存範圍 Scope 及 LEGB 規則

分為

- **全域（Global）：**函數外面，所有人都存取的到

- **區域（Local）：函數裡宣告或函數的參數**，函數**外部無法存取 \-\-\> 命名空間 Nampspace 概念（每個函數內的變數獨立）**

遵守 **LEGB 規則**：從上往下尋找變數

1. **Local：**先尋找函數內有沒有被**賦值**（=），注意 **UnboundLocalError 錯誤**

2. **Enclosing：**若為內嵌的函數，向**外層的函數**尋找變數

3. **Global：**找**函數外**

4. **Built\-in：**如果全域也找不到，Python 會去自己的**底層原始碼字典**裡找（包含 print、len、int、list、True 等）

---

對**不可變變數**來說，函數內的改動絕對不會影響到函數外的全域變數

```Python
x = "global"

def func():
    x = "local"
    print(x)

print(x) # 輸出 global

func() # 輸出 local

print(x) # 就算已經調用函數還是輸出 global，函數內改動不會對外部影響
```

> **UnboundLocalError 錯誤**
> 
> 如果我先讀取外部變數，然後在函數裡又修改（賦值），會報錯
> 
> 因為若函數裡有 =，程式會認為你這個變數為區域變數**（不管前後，只要有賦值都會造成）**，但你又在還沒賦值的地方讓它被打印
> 
> ```Python
> score = 10  # 全域變數
> 
> def update_score():
>     # 陷阱在這裡！
>     # 我們想先把外面的 10 印出來，然後再把它加 1
>     print(f"原本的分數是：{score}") 
>     
>     # 接著在函數內重新賦值
>     score = score + 1 
> 
> update_score()
> ```

#### global 修改全域變數

- 用於在函數內部明確聲明要**修改外層的 Global 全域變數**（主要是不可變物件）。

> **💡 最佳實踐提醒**：建議將資料**當作參數傳入函數**，處理完後**使用 return 回傳結果**，避免全域變數被任意修改導致難以除錯！

```Python
# 遊戲最高分紀錄 (全域變數)
high_score = 0

def break_record(new_score):
    # 告訴 Python：「我要改的是外面公佈欄的那個 high_score，不是我自己要創新的！」
    global high_score 
    
    if new_score > high_score:
        high_score = new_score # 這次真的改到外面的值了！
        print("破紀錄啦！")

break_record(500)
print(f"目前最高分：{high_score}") # 輸出：目前最高分：500
```

---

#### nonlocal 修改外層非全域變數

- 用於在**內嵌函數（閉包 Closure）**內部修改**外層 Enclosing 函數**的作用域變數（且該變數不能是 Global 全域變數）。

```Python
def make_counter():
    count = 0  # 屬於外層 Enclosing 作用域的變數
    
    def counter():
        # 告訴 Python：「我要修改的是外層 Enclosing 函數的 count，不是自己建立區域變數！」
        nonlocal count  
        count += 1
        return count
        
    return counter

my_counter = make_counter()
print(my_counter()) # 輸出：1
print(my_counter()) # 輸出：2
```

對於**可變變數**來說：

**如果有用 = \(例如 x = \[\.\.\.\]\)**：Python 就認定你要創造一個全新的**區域變數（和不可變變數同）**

**如果沒有用 =，只是呼叫動作 \(例如 x\.append\(\)\)**：Python 在內部找不到這個變數，就會**抬頭往外找全域變數**，然後順著線找到外面的箱子，直接把手伸進去改東西 \-\-\> **不需寫 global 也可修改外部**

> **UnboundLocalError 錯誤**同樣也會在可變變數出現

---

### Closure 閉包

函數內還有函數，在**外層函數內建立資料**，以讓**內層函數取用**

以這種方式建立的資料，**只保留在這兩層函數中**，以**保護資料**

且內層函數將資料處理完後可以**繼續保存在外層的函數**

閉包時：

- **內層引用外層變數**

- **外層回傳內層函數**

什麼時候適合用？

- **資料封裝：**取代**全域變數**，避免資料被誤改

- **快取／資料保存：**避免重複執行**耗時運算**（將複雜計算存取，**再次使用時直接調用**）

- **重複：**大量**相似**的函數（如某個**固定參數不同**）

```Python
def func_out(...):
    ... # 建立資料
    
    def func_in(...):
        ... # 取用外層的變數進行資料處理
        
        return ... # 回傳處理
    
    return func_in # 回傳內層函數      
```

```Python
func = func_out(...) # 呼叫外層，領取、存入資料

result = func(...) # 呼叫內層進行資料處理
```

```Python
def create_cached_calculator():
    # 這是被閉包保護的「私有快取字典」
    # 它不會在每次運算完被清空，外部程式也絕對無法竄改它
    cache = {} 
    
    def heavy_computation(x):
        # 如果這個數字算過了，直接從閉包的字典裡拿答案
        if x in cache:
            print(f"⚡ 直接從快取讀取: {x}")
            return cache[x]
        
        # 如果沒算過，就真的執行耗時運算
        print(f"⏳ 執行龐大耗時運算: {x}...")
        result = x * x * x  # 這裡假裝是一個極度耗時的運算
        
        # 算完後，把結果存進閉包的字典裡
        cache[x] = result
        return result
        
    return heavy_computation

# ----------------------------------------
# 調用

calculate = create_cached_calculator()

calculate(10) # 第一次遇到 10，必須花時間算 (輸出: ⏳ 執行龐大耗時運算: 10...)
calculate(5)  # 第一次遇到 5，必須花時間算 (輸出: ⏳ 執行龐大耗時運算: 5...)
calculate(10) # 第二次遇到 10，瞬間給出答案！ (輸出: ⚡ 直接從快取讀取: 10)
```

> 25. 呼叫外層函數
> 
> ```Python
> ...create_cached_calculator()
> ```
> 
> 4. 進入函數，建立字典
> 
>     回傳內層函數
> 
> ```Python
> cache = {} 
> ...
> return heavy_computation
> ```
> 
> 25. 將內層函數賦值在變數上
> 
> ```Python
> calculate = ...（外層函數回傳）
> ```
> 
> 27. 呼叫函數並傳入參數

---

```Python
# 這是外層函數（製造機器的工廠）
# 參數 role_multiplier 是準備被鎖進去的「專屬倍率」
def create_damage_calculator(role_multiplier):
    
    # 這是內層函數（被製造出來的專屬計算機）
    def calculate(base_damage):
        # 內層取用了外層的 role_multiplier 進行運算
        final_damage = base_damage * role_multiplier
        return final_damage
        
    # 將製造好的專屬計算機交出去
    return calculate

# ----------------------------------------
# 階段一：開始向工廠「訂製」專屬函數

# 傳入 1.5 倍，工廠吐出一台「戰士專用」的計算機
warrior_calc = create_damage_calculator(1.5)

# 傳入 0.8 倍，工廠吐出一台「牧師專用」的計算機
priest_calc = create_damage_calculator(0.8)

# 傳入 2.0 倍，工廠吐出一台「刺客專用」的計算機
assassin_calc = create_damage_calculator(2.0)

# ----------------------------------------
# 階段二：在遊戲中實際呼叫這些專屬函數

# 假設他們今天都拿了一把基礎攻擊力 100 的劍
print(f"戰士造成的傷害：{warrior_calc(100)}")   # 輸出：150.0
print(f"牧師造成的傷害：{priest_calc(100)}")    # 輸出：80.0
print(f"刺客造成的傷害：{assassin_calc(100)}")  # 輸出：200.0
```

---

#### 函數裝飾器 Decorator

核心目的為：在完全不修改函數原始程式碼的情況下，替其擴充新功能。

> **💡 關鍵觀念：`func` 參數到底是什麼？**  
> `def decorator_name(func):` 中的 **`func` 就是「被加上裝飾器的原始函數本人」**！  
> 當我們在函數頭頂寫上 `@decorator_name` 時，Python 幕後會自動執行這行程式碼：  
> `original_func = decorator_name(original_func)` （將舊函數 `original_func` 作為 `func` 參數傳給裝飾器進行加工！）

> **💡 進階觀念解密：為什麼需要 2 層/3 層？若「僅為函式附加屬性」需要寫 `wrapper` 嗎？**  
> - **答案是：不需要寫 `wrapper`！**
> - **為什麼需要 `wrapper` (包裝紙)？** 因為要**「攔截/延遲執行」**（例如：當外部呼叫 `func()` 時才跑計時、權限驗證或傳遞參數）。
> - **若僅需「附加屬性 / 打標籤 (Attach Attributes)」**：在裝飾的瞬間直接在 `func` 物件上寫入屬性並 `return func` 即可，**不需內層 `wrapper`**！
>   - **無參附加屬性裝飾器**：只需 **1 層**（`func` ➔ 直接賦值 ➔ 回傳 `func`）。
>   - **帶參附加屬性裝飾器**：只需 **2 層**（`maker(屬性值)` ➔ `decorator(func)` 賦值 ➔ 回傳 `func`）。

| 裝飾器類型 | 應用目的 | 嵌套層數 | 接收參數 | 各層關鍵回傳 (`return`) 寫法 | 典型應用場景 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **沒有帶參數** | **改變/攔截執行邏輯** | 2 層 | `func` ➔ `*args, **kwargs` | 外層 `return wrapper` (包裝函式)<br>內層 `return result` (原函式結果) | 效能計時、日誌記錄、存取驗證 |
| **沒有帶參數** | **僅附加屬性 / 打標籤** | 1 層 | `func` | 外層 **`return func`** (直接原封不動回傳原函式) | 標記過期 `is_deprecated`、自動註冊 |
| **帶有參數** | **改變/攔截執行邏輯** | 3 層 | `maker(設定值)` ➔ `func` ➔ `*args, **kwargs` | 最外層 `return decorator`<br>中間層 `return wrapper`<br>最內層 `return result` | 權限控管 `role`、設定重試次數 |
| **帶有參數** | **僅附加屬性 / 打標籤** | 2 層 | `maker(設定值)` ➔ `func` | 最外層 `return decorator`<br>中間層 **`return func`** (直接原封不動回傳原函式) | 動態標籤 `category`、分類配置 |

- **一、沒有帶參數的裝飾器 (Unparameterized Decorator)**
    - **核心架構**：標準 2 層函式嵌套結構（`func` ➔ `wrapper`）。
    - **適用場景**：不需要對裝飾器本身傳入設定參數，僅對原函式進行固定加工、裝飾或自動登錄。

    > **🍿 生動白話比喻**：  
    > 沒有帶參數的裝飾器就像**「標準防偽包裝盒」**。每個進來的商品（舊函式）直接被套上固定規格的防偽盒子（`wrapper`），不需要特別調整機台設定。

    - **架構範本與代碼範例：**

    ```Python
    def decorator_name(func): # func 就是「被裝飾的原始函數本人」
        # 1. 內層包裝紙，使用 *args, **kwargs 才能接收任何形狀的舊函數（因為每個函數接受的參數不同）
        def wrapper(*args, **kwargs):
            
            # --- 【加工區 1：執行前】 ---
            # 例如：檢查權限、記錄開始時間、印出提示
            print("⏱️ [前置處理] 開始執行函式...")
            
            # 2. 執行原函數，並把結果存起來
            result = func(*args, **kwargs)
            
            # --- 【加工區 2：執行後】 ---
            # 例如：記錄結束時間、把結果寫入資料庫
            print("✅ [後置處理] 函式執行完成！")
            
            # 3. 回傳原函數的執行結果
            return result
            
        # 4. 外層交接：把包裝好的新函數丟出去 (不加括號！)
        return wrapper
        
    # -------------------------------------------------
    # 使用：加上 @裝飾器名稱

    @decorator_name
    def original_func():
        print("🚀 原函式業務邏輯執行中")
        return "Success"

    # 執行測試
    res = original_func()
    # 輸出: ⏱️ [前置處理] 開始執行函式...
    # 輸出: 🚀 原函式業務邏輯執行中
    # 輸出: ✅ [後置處理] 函式執行完成！
    ```

    - **極致對比 (❌ 錯誤寫法 vs ✅ 正確寫法)**：

    ```Python
    # ❌ 錯誤寫法：未傳入 *args, **kwargs，導致原函式若有參數時會直接 Crash！
    def bad_decorator(func):
        def wrapper(): # ❌ 漏掉不定參數
            return func()
        return wrapper # 最外層亦不能加括號！

    # ✅ 正確寫法：使用 *args, **kwargs 完美容納各種參數輸入
    def good_decorator(func):
        def wrapper(*args, **kwargs):
            return func(*args, **kwargs)
        return wrapper # ✅ 回傳函式物件本人
    ```

    - **實戰應用：工具註冊器 / 函式自動註冊 (Tool Registry Decorator)**

        在開發 AI Agent (如 LLM Tool Use / Function Calling) 或外掛插件 (Plugin) 系統時，我們經常需要將特定函式自動登錄到全域字典/註冊表中。使用裝飾器可以達到「只要在函式頭頂戴上 `@register_tool` 帽子，就自動完成登記」的極致優雅效果！

        ```Python
        # 1. 建立全域工具註冊字典 (儲存 "函式名稱": 函式物件)
        TOOL_REGISTRY = {}

        # 2. 定義裝飾器函式 (無參數裝飾器)
        def register_tool(func):
            """ 只要戴上 @register_tool 帽子的函式，都會被自動放入 TOOL_REGISTRY 字典中 """
            # Key 鍵: func.__name__ (函式名稱字串, 如 'get_current_weather')
            # Value 值: func (函式物件本人, 拿出來加括號即可直接執行)
            TOOL_REGISTRY[func.__name__] = func
            return func  # 原封不動回傳原函式，不改變其執行行為

        # -------------------------------------------------
        # 3. 使用：只要戴上 @register_tool 帽子就會自動完成註冊登錄

        @register_tool
        def get_current_weather(location: str):
            """ 查詢指定地區的即時天氣狀況 """
            return f"{location} 今天晴空萬里，25°C"

        @register_tool
        def calculate_tax(amount: float):
            """ 計算輸入金額的標準稅額 """
            return amount * 0.05

        # -------------------------------------------------
        # 4. 實戰測試：全域字典已自動收集所有被裝飾的工具函式！
        print("目前已註冊的所有工具:", list(TOOL_REGISTRY.keys()))
        # 輸出: 目前已註冊的所有工具: ['get_current_weather', 'calculate_tax']

        # 5. 動態呼叫工具 (AI Agent Tool Calling 底層原理)
        tool_name = "get_current_weather"
        if tool_name in TOOL_REGISTRY:
            result = TOOL_REGISTRY[tool_name](location="台北")
            print(result) 
            # 輸出: 台北 今天晴空萬里，25°C
        ```

        > **💡 觀念精準解密：`TOOL_REGISTRY[func.__name__] = func` 到底存了什麼？**  
        > - **字典的 Key (鍵)**: `func.__name__` ➔ 純字串 `'get_current_weather'` (用來作名字檢索)。  
        > - **字典的 Value (值)**: `func` ➔ 函式物件本人 (程式碼記憶體位址，拿出來加括號 `()` 就能直接執行！)。

        > **🍿 生動白話比喻**：  
        > `@register_tool` 就像**「特長俱樂部的自動報到感應門」**。任何人只要戴著會員帽子通過這扇門（被 `@register_tool` 裝飾），門口的電子登記簿 (`TOOL_REGISTRY`) 就會自動抄下他的名字與聯繫方式，方便管理員隨時呼叫！

- **二、帶參數的裝飾器 (Parameterized Decorator)**
    - **核心架構**：標準 3 層函式嵌套結構（`maker(設定值)` ➔ `decorator(func)` ➔ `wrapper(*args, **kwargs)`）。
    - **適用場景**：需要根據外部設定條件（如控制權限等級、設定重複次數、調整逾時秒數）動態客製化裝飾器行為。

    > **🍿 生動白話比喻**：  
    > 帶參數的裝飾器就像**「帶有鍵盤的客製化封膜機」**。最外層的按鈕傳入參數（例如：設定封膜溫度為 `200°C`），機台根據你的參數生出一台專屬封膜器（中間層），最後再把飲料（原函式）送進去進行包裝（最內層）。

    - **架構範本與代碼範例：**

    ```Python
    # 1. 【最外層】：專門接收裝飾器自己的參數 (機器設定值)
    def decorator_maker(setting_param):
        
        # 2. 【中間層】：這才是原本標準的裝飾器 (接收舊函數)
        def actual_decorator(func):
            
            # 3. 【最內層】：標準的包裝紙 (接收舊函數的參數)
            def wrapper(*args, **kwargs):
                
                # --- 加工區 ---
                # 這裡可以同時使用「設定參數」與「舊函數」來做事
                print(f"⚙️ [機器設定] 當前設定值為：{setting_param}")
                
                result = func(*args, **kwargs) # 執行原函數
                
                return result
                
            return wrapper # 中間層把包裝紙交出去
            
        return actual_decorator # 最外層把真正的裝飾器交出去

    # -------------------------------------------------
    # 使用：加上 @裝飾器名稱(設定值)

    @decorator_maker(setting_param="VIP模式")
    def original_func():
        print("🚀 執行原函數業務邏輯")

    original_func()
    # 輸出: ⚙️ [機器設定] 當前設定值為：VIP模式
    # 輸出: 🚀 執行原函數業務邏輯
    ```

    - **極致對比 (❌ 錯誤寫法 vs ✅ 正確寫法)**：

    ```Python
    # ❌ 錯誤寫法：試圖在 2 層裝飾器中接收自訂參數，導致傳參混淆報錯
    def wrong_param_decorator(func, role="admin"): # ❌ 混淆兩層與三層架構
        def wrapper(*args, **kwargs):
            return func(*args, **kwargs)
        return wrapper

    # ✅ 正確寫法：嚴格遵守 3 層結構：最外層(參數) ➔ 中間層(func) ➔ 最內層(*args, **kwargs)
    def right_param_decorator(role="admin"):
        def decorator(func):
            def wrapper(*args, **kwargs):
                print(f"🔐 驗證權限需求: {role}")
                return func(*args, **kwargs)
            return wrapper
        return decorator
    ```

    - **實戰應用：權限存取控制 (`@require_permission`)**

    ```Python
    # 模擬系統環境：假設現在是一位「一般會員 (user)」登入中
    current_login_role = "user" 


    # --- 開始定義帶參數裝飾器 (三層架構) ---

    # 1. 最外層：接收我們設定在 @ 後面的參數 (role)
    def require_permission(role):
        
        # 2. 中間層：接收目標函數 (delete_user_data 或 view_profile)
        def decorator(func):
            
            # 3. 最內層：真正執行包裝與判斷的地方
            def wrapper(*args, **kwargs):
                
                # 【核心邏輯區】：比對「要求的權限」與「目前的權限」
                if current_login_role == role:
                    # 權限符合：放行，執行原本的函數
                    print(f"✅ [系統] 權限驗證通過 ({role})。")
                    return func(*args, **kwargs)
                else:
                    # 權限不符：阻擋執行，並給出警告
                    print(f"❌ [警告] 拒絕存取！此動作需要 '{role}' 權限，您目前的權限是 '{current_login_role}'。")
                    # 實務上這裡也可以寫 raise Exception 直接讓程式報錯中斷
                    
            return wrapper
            
        return decorator


    # --- 裝飾器定義完畢，開始套用到業務邏輯上 ---

    @require_permission(role="admin")
    def delete_user_data():
        print("🗑️ 執行動作：資料已徹底刪除")

    @require_permission(role="user")
    def view_profile():
        print("📄 執行動作：顯示個人詳細資料")


    # --- 實戰測試執行 ---

    print("--- 測試 1：一般會員試圖查看資料 ---")
    view_profile()
    # 輸出: --- 測試 1：一般會員試圖查看資料 ---
    # 輸出: ✅ [系統] 權限驗證通過 (user)。
    # 輸出: 📄 執行動作：顯示個人詳細資料

    print("\n--- 測試 2：一般會員試圖刪除資料 ---")
    delete_user_data()
    # 輸出: --- 測試 2：一般會員試圖刪除資料 ---
    # 輸出: ❌ [警告] 拒絕存取！此動作需要 'admin' 權限，您目前的權限是 'user'。
    ```

---


### 產生器 Generator

核心目的在於**延遲計算**

閉包將資料所在記憶體內，而產生器則是不把資料一次全塞進記憶體，

常用於處理大量資料

**使用 next\(\)呼叫**

- yield 和 return 的差異：

    前者**算完一個資料，就會回傳一個資料**，並將整個函數暫停，等下次有人呼叫時再繼續跑下去

    後者則是**把所有資料算完後**，**一次性**全部回傳，且立刻清空記憶體與內部狀態

```Python
def generator_name(...) -> Genrator[Yield型別, Send型別, Return型別] 或 Iterator[型別]:
    ...
    yield ...
    .
    yield ...
    .
    return ...
```
> **型別注記用 `Genrator` 或 `Iterator`** 
> 在 Python 裡，一旦函式內部出現了 `yield`，這個函式就**不再是一般的函式**，它會變成一個「生成器 (Generator)」
> 差異在**`Generator` 是「精確描述它是什麼」，而 `Iterator` 是「描述它可以怎麼被使用」**。

在物件導向的概念裡，`Generator`（生成器）其實是 `Iterator`（迭代器）的一種**子類別 (Subclass)**

```Python
def math_generator():
    number = 10              
    
    print("📍 [進入階段一]")
    number = number + 5          
    yield number                 

    # --- (下次被呼叫時，會從這裡醒來) ---
    print("📍 [進入階段二]")
    number = number * 2          
    yield number                 
    
    # --- (下次被呼叫時，會從這裡醒來) ---
    print("📍 [進入階段三]")
    number = number - 10            
    yield number                 

gen = math_generator()
print(next(gen))

print(next(gen))

print(next(gen))
```

---

## 內建函數

| 分類 | 函數名稱 | 功能 | 簡單範例 |
| --- | --- | --- | --- |
| **輸出輸入與系統** | print() | 印出資料到螢幕上 | print("Hello") |
| | input() | 暫停程式，接收使用者的鍵盤輸入 | ans = input("請輸入：") |
| | type() | 照妖鏡：查詢變數的資料型態 | type(10) $\rightarrow$ `<class 'int'>` |
| | id() | 查戶口：取得變數在記憶體中的真實門牌號碼 | id(x) $\rightarrow$ 14073... |
| | help() | 呼叫內建說明文件 (終端機裡超好用) | help(print) |
| | dir() | 顯示物件所有的屬性與可用動作 (方法) | dir(list) |
| **數學與數值** | abs() | 取絕對值 (去負號) | abs(-5) $\rightarrow$ 5 |
| | round() | 四捨五入 (可指定小數位數) | round(3.1415, 2) $\rightarrow$ 3.14 |
| | max() | 找最大值 (可放入串列或多個數字) | max([1, 5, 3]) $\rightarrow$ 5 |
| | min() | 找最小值 | min(10, 2) $\rightarrow$ 2 |
| | sum() | 加總容器內的所有數字 | sum([1, 2, 3]) $\rightarrow$ 6 |
| | pow() | 次方運算 (同 \*\*) | pow(2, 3) $\rightarrow$ 8 |
| | divmod() | 同時算出「商數」和「餘數」 (回傳元組) | divmod(10, 3) $\rightarrow$ (3, 1) |
| **型別轉換** | int() | 轉成整數 (無條件捨去小數) | int("10") $\rightarrow$ 10 |
| | float() | 轉成浮點數 (小數) | float(5) $\rightarrow$ 5.0 |
| | str() | 轉成字串 | str(100) $\rightarrow$ "100" |
| | bool() | 判斷真假值 (Falsy 測試) | bool([]) $\rightarrow$ False |
| | list() | 轉成串列 (常搭配字串或 range) | list("ABC") $\rightarrow$ ['A', 'B', 'C'] |
| | dict() | 建立字典 | dict(a=1) $\rightarrow$ {'a': 1} |
| | set() | 轉成集合 (超常拿來去重重複元素！) | set([1, 1, 2]) $\rightarrow$ {1, 2} |
| | tuple() | 轉成元組 (不可變清單) | tuple([1, 2]) $\rightarrow$ (1, 2) |
| | len() | 算長度 (裡面有幾個東西) | len("Python") $\rightarrow$ 6 |
| | range() | 產生等差數列 (專為 for 迴圈設計) | range(5) $\rightarrow$ 0,1,2,3,4 |
| | enumerate() | 幫容器加上編號 (遞迴神器) | for i, v in enumerate(A): |
| | zip() | 把兩個串列合併打包成元組，zip(*) 反向解包 | zip([1,2], ['a','b']) $\rightarrow$ [(1, 'a'), (2, 'b')] |
| | sorted() | 排序 (永遠吐出一個全新的排序串列) | sorted([3, 1, 2]) $\rightarrow$ [1, 2, 3] |
| | reversed() | 順序反轉 | reversed([1, 2, 3]) |
| | map() | 把某個函數套用到清單的所有元素上 | map(str, [1, 2]) |
| | filter() | 篩選出符合條件的元素 | filter(func, [1, 2]) |
| | all() | 檢查容器：是不是「全部」都是 True？ | all([True, 1, "A"]) $\rightarrow$ True |
| | any() | 檢查容器：只要「任何一個」是 True 就過關 | any([False, 0, "A"]) $\rightarrow$ True |
| **字元與系統底層** | chr() | 數字轉 ASCII / Unicode 字元 | chr(65) $\rightarrow$ 'A' |
| | ord() | 字元轉 ASCII / Unicode 數字 | ord('A') $\rightarrow$ 65 |
| | bin() | 轉二進位字串 | bin(10) $\rightarrow$ '0b1010' |
| | hex() | 轉十六進位字串 | hex(255) $\rightarrow$ '0xff' |
| | hash() | 算出不可變物件的雜湊值 (底層身分證) | hash("password") |
| | eval() | 把字串當作 Python 程式碼「運算」並回傳結果 | eval("1+1") $\rightarrow$ 2 |
| | exec() | 把字串當作 Python 程式碼「執行」 (很危險少用) | exec("x = 5") |


---

### random\(\) 隨機

```Python
import random

... = random.random() # 0.0~1.0 隨機

... = random.randint(1, 6) # 指定範圍整數

... = random.uniform(10, 20) # 指定範圍浮點數

... = random.choice(['蘋果', '香蕉', '西瓜']) # 從串列中隨機選擇
```

```Python
if random.random() < x:
    ... # x 為小數形式機率
```

```Python
if random.random() > x:
    ... # x 為小數形式機率
```

```Python
random.seed(隨機碼)
```

---

### hasattr\(\) 物件檢查

檢查**物件是否有此屬性或方法**

回覆布林值

```Python
hasattr(物件, "屬性／方法")
```

---

### reversed\(\) 反向迭代器

將序列中的元素**由後往前（逆序）反向走訪**的內建函式

回傳一個**反向迭代器 (reverse iterator)**，具備**惰性求值（Lazy Evaluation）**特性

它**不會在記憶體中建立全新的串列複本**，因此處理百萬筆巨量資料時極度節省記憶體！

- **支援型別**：
  - 必須是**有序序列**（如 `list`、`tuple`、`str`、`range`）
  - 或有實作 `__reversed__()` / `__len__()` 與 `__getitem__()` 的自訂物件

```Python
numbers = [1, 2, 3, 4, 5]

# 1. 直接在 for 迴圈中逆序遍歷 (最推薦寫法，記憶體空間 O(1))
for num in reversed(numbers):
    print(num, end=" ")
# 輸出: 5 4 3 2 1
print()

# 2. 轉回串列 list
rev_list = list(reversed(numbers))
print(rev_list)  # 輸出: [5, 4, 3, 2, 1]

# 3. 反轉字串 (需搭配 join 拼裝回字串)
word = "Python"
rev_word = "".join(reversed(word))
print(rev_word)  # 輸出: nohtyP

# 4. 反轉 range 數列
for i in reversed(range(1, 6)):
    print(i, end=" ")
# 輸出: 5 4 3 2 1
```

> **⚠️ 迭代器只能消耗一次陷阱**：  
> `reversed()` 產生的是一個**迭代器物件**，一旦用 `for` 迴圈或 `list()` 讀取完畢後，裡面的指標就走到盡頭了！再次讀取會是空的。若需要多次重複使用，請轉成 `list(reversed(...))` 儲存！

---

## Python 三大反轉方式大對決 (reversed vs .reverse vs [::-1])

在 Python 中將資料反轉有三種最經典的做法，適用時機與記憶體機制完全不同：

| 反轉方式 | 操作語法 | 是否改動原物件 | 回傳值 | 記憶體開銷 | 適用場景 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`reversed()` 函式** | `reversed(seq)` | ❌ 否（原序列不變） | **反向迭代器** | $O(1)$（極省） | `for` 迴圈遍歷、大數據反向走訪 |
| **`list.reverse()` 方法** | `my_list.reverse()` | ✅ 是（**就地原地修改**） | `None` | $O(1)$（原地） | 單純想將既有串列順序顛倒 |
| **切片語法 `[::-1]`** | `seq[::-1]` | ❌ 否（原序列不變） | **全新的反轉複本** | $O(N)$（複製） | 字串反轉、需要保留原物件並立刻得到新串列 |

```Python
# 1. reversed()：原串列不變，回傳迭代器
nums = [1, 2, 3]
r = reversed(nums)
print(nums)        # 輸出: [1, 2, 3] (原串列完好如初)
print(list(r))     # 輸出: [3, 2, 1]

# 2. list.reverse()：原地修改！回傳 None
nums = [1, 2, 3]
result = nums.reverse()
print(result)      # 輸出: None (新手常犯錯誤：千萬不要拿變數去接它！)
print(nums)        # 輸出: [3, 2, 1] (原串列本身被改掉了)

# 3. [::-1] 切片：原串列不變，產生全新串列
nums = [1, 2, 3]
new_nums = nums[::-1]
print(nums)        # 輸出: [1, 2, 3]
print(new_nums)    # 輸出: [3, 2, 1] (全新的一塊記憶體空間)
```

> **💡 自訂類別如何支援 `reversed()`？**  
> 只要在自訂 Class 中定義 `__reversed__(self)` 方法，外部就能直接對該實例呼叫 `reversed(obj)` 進行反向迭代！

---

## Lambda 表達式 

又稱**匿名函數**

若只需要一個**簡易**且**只用一次的函數**，不需要 def、縮排、return 就可以僅用單行解決

有以下特性：

1. **沒有名字**，通常賦值在變數上或直接當參數使用

2. 不能在內部宣告或賦值、不能型別標記

3. 只能用**一行**

4. **自帶 return**

```Python
lambda 參數列表: 表達式
```

```Python
# 傳統方式：
def add(a, b):
    return a +b
    
add(1,3)
# -----------------
# lambda 處理：
add_lambda = lambda a, b: a + b

add_lambda(1, 3) # 同樣效果
```

較複雜的例子：

```Python
models = [
    {"name": "Model_A", "accuracy": 0.85},
    {"name": "Model_B", "accuracy": 0.92},
    {"name": "Model_C", "accuracy": 0.78} # 字典
] #串列

# 為了排序，特別定義一個小函數來抓取 accuracy
def get_accuracy(model):
    return model["accuracy"]

# 把函數傳進去當排序條件
sorted_models = sorted(models, key=get_accuracy, reverse=True)
```

```Python
models = [
    {"name": "Model_A", "accuracy": 0.85},
    {"name": "Model_B", "accuracy": 0.92},
    {"name": "Model_C", "accuracy": 0.78} # 字典
] #串列

# 直接在 key= 的地方現場寫一個 Lambda 函數
sorted_models = sorted(models, key=lambda x: x["accuracy"], reverse=True)
# 宣告一個參數 x （字典）用於接收傳出的值

# [{'name': 'Model_B', 'accuracy': 0.92}, {'name': 'Model_A', 'accuracy': 0.85}, ...]
```

> **Sorted\(\) **用於對物件排序
> 
> 參數為：排序物件、排序參考\(預設 None）、順序（預設 false，升冪）

> **map\(\)、filter\(\) **也常用 Lambda
> 
> ```Python
> a1 = [1, 2, 3, 4, 5]
> 
> # 把每個數字 x 變成 x 的平方
> squared_nums = list(map(lambda x: x**2, a1))
> 
> print(squared_nums)
> # 輸出：[1, 4, 9, 16, 25]
> ```
> 
> ```Python
> a2 = [1, 2, 3, 4, 5]
> 
> # 判斷條件：除以 2 的餘數是否為 0 (是否為偶數)
> even_nums = list(filter(lambda x: x % 2 == 0, a2))
> 
> print(even_nums)
> # 輸出：[2, 4]
> ```

---

# 模組 套件 Module and Package

開發大型檔案時，會將不同功能的程式碼分門別類，**放在不同的檔案（模組）**，
需要用此功能時**在主程式匯入**使用，稱為**模組化（Modularization）**

而匯入的模組不僅能用 python 也可**使用其他語言**

- 模組是一個檔案，裡面裝著變數、函數、類別

## 關於套件

- 而套件是一個**資料夾**，內部有**多個模組**（多個檔案）

套件透過 **init\.py **被建立（資料夾裡面要有 **init\.py**），

而其會在**套件內任何子模組第一次被匯入時執行內部內容**，當然內部也可以是**空的**

- 為何要用套件將模組放入資料夾？
    1. 避免撞名（同一個資料夾內不能有兩個同名模組）
    2. 方便管理

---

### __init__.py 的核心職責：套件對外門面 (Package Export Door) 與 __all__ 規範

在現代 Python 專案架構中，`__init__.py` 不僅僅是用來標記資料夾為套件的檔案，它更重要的作用是**作為該套件對外的「接待大門 / 門面 (Package Export Door / Facade)」**。

它負責將資料夾內部深層子模組中定義的關鍵類別、函式與常數「匯出 (Export)」，讓外部引用時享有乾淨簡潔的語法！

- **未設定門面匯出 vs 有設定門面匯出**：
  - ❌ **未在 `__init__.py` 設定門面匯出** (外部引用時必須暴露內部細節路徑，極度冗長)：
    ```Python
    # 外部 (agent.py) 必須深挖內部具體檔名 (base.py, gemini_provider.py)
    from miniclaude.providers.base import LLMProvider, ToolCall, LLMResponse
    from miniclaude.providers.gemini_provider import GeminiProvider
    ```

  - ✅ **在 `__init__.py` 設定門面匯出與 `__all__` 清單** (最佳實踐)：
    ```Python
    # src/miniclaude/providers/__init__.py
    from .base import LLMProvider, ToolCall, LLMResponse
    from .gemini_provider import GeminiProvider

    # 明確宣告這個套件對外公開暴露的符號清單 (Public API)
    __all__ = [
        "LLMProvider",
        "ToolCall",
        "LLMResponse",
        "GeminiProvider",
    ]
    ```

  - 🌟 **外部導入極致簡潔優雅**：
    ```Python
    # 外部 (agent.py) 導入時只需指名套件資料夾，語法極致漂亮乾淨！
    from miniclaude.providers import GeminiProvider, LLMProvider, ToolCall
    ```

- **`__all__` 的核心作用**：
  - 是一個字串串列，定義套件的「對外公開介面 (Public API List)」。
  - 能防止內部的私有輔助工具被外部誤導入，同時讓 IDE 補全與文檔工具自動生成精準的暴露清單。

  > **生動白話比喻**：  
  > `src/miniclaude/providers/` 像**一棟商業大樓**，內部的 `base.py` 與 `gemini_provider.py` 是**樓層裡的各間辦公室**。  
  > 沒有 `__init__.py` 門面匯出時，訪客（外部程式碼）必須自己搭電梯爬到 `3 樓 302 室 base.py` 才能拿資料；  
  > 在 `__init__.py` 寫好匯出，就像在 **1 樓大廳設置「接待櫃檯」**：訪客直接在一樓大廳 (`from miniclaude.providers import ...`) 就能一次拿齊所需的所有資料！

---


## 匯入

1. **標準匯入：**

    ```Python
    import 套件名.模組名
    
    # 呼叫時：
    套件名.模組名.函數名/變數
    ```

2. **精準匯入：**

    在**體積龐大、領域獨立（功能混雜）、結構深**的套件時適合使用

    ```Python
    # 可同時匯入多個，用逗號隔開
    from 套件名.模組名 import 目標物件（函數、類別）1, ....
        # 以可在後面目標物件寫
    # 呼叫：
    目標物件1()
    ```

3. **別名匯入：**

    ```Python
    # 法 1
    import 套件名.模組名 as 自訂別名
    
    # 法 2
    from 套件名 import 模組名 as 自訂別名
    
    # 呼叫時：
    自訂別名.目標函數或變數
    ```

4. **萬用字元匯入：**

    ```Python
    # ⚠️ 語法公式 (強烈不建議使用)：
    from 套件名.模組名 import *
    
    # 呼叫方式 (直接寫名字，但你無法輕易追蹤這個名字是從哪來的)：
    任何目標物件()
    ```


| 項目 | 常規匯入 (例如 import math) | 精準匯入 (例如 from math import pi) | 萬用字元匯入 (例如 from math import *) |
| --- | --- | --- | --- |
| **運作原理** | 把整個「工具箱」搬進房間，**並保留外盒**。 | 從工具箱裡**精準拿出一件工具**，放在桌上。 | 把工具箱**暴力拆開，所有工具散落一地**。 |
| **呼叫方式** | 必須指名道姓：`math.pi` | 直接使用：`pi` | 直接使用：`pi` |
| **撞名風險** | **零風險**。因為外面包著 `math.` 的殼。 | **極低**。因為你清楚知道自己拿了什麼，不會拿跟自己同名的。 | **極高 (死亡陷阱)**。可能會在無意間將原本的變數覆蓋掉。 |
| **程式碼可讀性** | **極佳**。一看就知道這個工具來自哪裡。 | **極佳**。開頭就宣告了清單。 | **極差**。看到程式碼不知道是從哪裡來的。 |
| **編輯器支援度** | VS Code 會完美提示這個盒子裡有什麼。 | VS Code 會精準追蹤這個變數。 | VS Code 會完全無法提供自動完成。 |

模組的尋找順序

1. 內建模組（C 語言寫的）

2. 當前資料夾

3. 標準函式庫（Python 寫的）

4. 第三方套件

    ---

- **重複匯入？**

    **只會進行一次**，不會被重複執行

    ```Python
    print("🔥 警告：這行程式碼正在被執行！")
    PI = 3.14159
    ```

    ```Python
    print("準備第一次匯入...")
    import my_math
    
    print("準備第二次匯入...")
    import my_math
    
    print("準備第三次匯入...")
    import my_math
    
    # 輸出 1 次而已：🔥 警告：這行程式碼正在被執行！
    ```

- 在**函數或類別裡**裡匯入？

    匯入只在**區域內有效**，且**性能會有延遲**，盡量在**開頭就先匯入**

    ---

- 匯入同名函數？

    執行**後匯入的**

    > `math_tools.py` 裡面有一個 `calculate()` 函數（用來算數學）
    > 
    > `tax_tools.py` 裡面也有一個 `calculate()` 函數（用來算稅金）

    ```Python
    # 主程式 main.py
    
    from math_tools import calculate
    print("第一次匯入完成")
    
    from tax_tools import calculate 
    print("第二次匯入完成")
    
    # 執行稅金的函數（後匯入的）
    calculate()
    ```

- 匯入模組名字與所在檔案同？

    **將自己匯入**

    ```Python
    import email  # 你原本是想匯入 Python 內建的 email 處理模組
    ```
    
---

## BaseProvider 抽象基底類別 (Abstract Base Class / Provider 模式)

在架構大型模組與套件時，用來**約束子類別必須實作特定介面**的設計模式與契約規範

### 什麼是 BaseProvider 與 ABC？
在 Python 中，我們透過內建的 `abc` 模組 (`from abc import ABC, abstractmethod`) 來定義 **抽象基底類別 (Abstract Base Class, ABC)**。

`BaseProvider` 通常在軟體架構中扮演 **「統一介面契約 (Interface Contract)」** 的角色：
- **無法直接實體化**：你無法直接透過 `provider = BaseProvider()` 來創建物件。
- **強制子類別實作**：凡是被標記為 `@abstractmethod` 的方法，繼承它的所有子類別**必須 100% 實作該方法**，否則 Python 在創建物件時會直接拋出 `TypeError` 阻止程式運行！

> **生動白話比喻**：  
> `BaseProvider` 就像是國家頒布的 **「標準電源插座規範」**：  
> 規範聲明了「只要你是合格的插頭，必須包含腳位尺寸與電壓範圍（抽象方法）」；  
> 吹風機 (`DysonProvider`)、電視機 (`SonyProvider`) 內部電路怎麼設計是它們的事，但它們**必須符合這個規範**，這樣牆上的插座（主程式）才能插上任何電器使用！

---

### 為什麼需要 BaseProvider？（解決的工程痛點）

####  沒有使用 BaseProvider 的混亂：
若團隊裡三位工程師各自開發 AI 模型對接模組：
- 工程師 A 寫的 OpenAI 類別方法叫 `.chat_completion()`
- 工程師 B 寫的 Gemini 類別方法叫 `.generate_text()`
- 工程師 C 寫的 Claude 類別方法叫 `.ask()`

當主程式要更換 AI 供應商時，整個專案的業務邏輯代碼都要跟著大改！

####  使用 BaseProvider 的優雅架構：
定義一個 `BaseLLMProvider` 規範所有 AI 服務提供者必須實作 `.generate_response(prompt)`。主程式只需要依賴這個抽象規範，即可實現**無縫切換模型 (開閉原則 Open-Closed Principle)**！

---

### 範例代碼：實作 BaseProvider 模式

```Python
from abc import ABC, abstractmethod

# 1. 定義 BaseProvider 抽象基底類別 (繼承 ABC)
class BaseLLMProvider(ABC):
    """所有 LLM 模型提供者模組必須遵循的抽象基底類別規格契約"""

    @abstractmethod
    def generate_response(self, prompt: str) -> str:
        """【抽象方法】子類別必須實作此方法：接收 Prompt 並回傳 AI 文字回應"""
        pass

    @abstractmethod
    def get_model_name(self) -> str:
        """【抽象方法】子類別必須實作此方法：回傳模型名稱"""
        pass


# 2. 具體實作 A：Gemini 服務提供者模組
class GeminiProvider(BaseLLMProvider):

    def generate_response(self, prompt: str) -> str:
        # 調用 [[Gemini API#1. 基礎文字生成 (Basic Text Generation)|Gemini API]] 的邏輯
        return f"[Gemini 2.5 Flash 成功回應]: {prompt}"

    def get_model_name(self) -> str:
        return "gemini-2.5-flash"


# 3. 具體實作 B：OpenAI 服務提供者模組
class OpenAIProvider(BaseLLMProvider):

    def generate_response(self, prompt: str) -> str:
        # 調用 OpenAI API 的邏輯
        return f"[GPT-4o 成功回應]: {prompt}"

    def get_model_name(self) -> str:
        return "gpt-4o"


# 4. 主程式業務邏輯：多型 (Polymorphism) 無縫切換
def run_ai_task(provider: BaseLLMProvider, user_prompt: str) -> None:
    """主程式只依賴抽象 BaseLLMProvider，完全不需要關心背後是哪個具體廠商！"""
    print(f"正在調用提供者: {provider.get_model_name()}")
    result = provider.generate_response(user_prompt)
    print(f"最終生成結果: {result}\n")


# 測試：隨意切換 Provider
if __name__ == "__main__":
    # 使用 Gemini 模組
    gemini_service = GeminiProvider()
    run_ai_task(gemini_service, "什麼是 BaseProvider？")

    # 一秒切換為 OpenAI 模組 (主程式 run_ai_task 完全不需要動！)
    openai_service = OpenAIProvider()
    run_ai_task(openai_service, "什麼是 BaseProvider？")
```

---

# 物件導向 OOP


物件為一個**東西**，而物件會有**狀態**及**行為**

狀態就是物件的**屬性（Attribute）**，而行為就是物件的**方法（Method）**，他們被寫在物件的函數裡

---

## class, object, attribute 類別、物件、屬性

### 建立類別和物件

將類別想像為**模具、藍圖**，而透過類別製作出**實體（Instance）**

> 通常會將物件稱為實體

```Python
class 類別名:
    ...
    
物件 = 類別名()
```

---

### 初始化與實體屬性

**\_\_init\_\_\(\)** 是物件建立後被執行的**第一道手續**，用意在為物件**填上屬性**，包含**傳入的和預設的**

```Python
class Player:
    def __init__(self, player_name, job):
        self.name = player_name # 傳進來的參數擺定在屬性上
        self.job = job 
        self.level = 1 # 直接預設屬性值
        self.hp = 100
        
chracter_1 = Player("Matthew", "打野")
chracter_2 = Player("Tim", "法師") 
```

初始化時特別的一點是，在第八行時，只傳入了兩個參數，但第二行卻**有三個引數**

而多出來的第一個引數，由 python **自動填入**，代表**此物件本身**

> self 只是一個**慣用名稱**，可以改別的

```Python
print(chracter_1.name)
print(chracter_1.job)
print(chracter_1.level)
print(chracter_1.hp)

#Matthew
# 打野
# 1
# 100
```

```Python
chracter_2.hp = 90
```

屬性內的變數又稱為**實體變數（Instance Variable）**

---

##### getattr() 動態屬性與方法存取

當屬性或方法名稱是**存放在變數/字串中**（例如資料庫 Key、使用者輸入或 AI Tool Call 的函式名），無法使用 `物件.屬性` 固定存取時，使用 `getattr()` 進行動態反射存取。

- **為什麼不直接用 `物件.屬性` 存取？(`getattr()` 的 3 大不可替代性)**：
  1. **死穴 1：寫 Code 時「屬性名稱還不知道」**  
     傳統寫法 `player.name` 必須在寫程式碼時就把屬性名硬編碼寫死。但如果屬性名存放在變數中（如 `attr = "hp"`），寫 `player.attr` 只會去尋找名為 `attr` 的屬性而出錯！此時必須用 `getattr(player, attr)`！
  2. **死穴 2：屬性不存在時，傳統寫法會直接崩潰**  
     若直接寫 `player.level` 且該屬性不存在，Python 會直接拋出致命的 `AttributeError` 導致程式中斷崩潰；而 `getattr(player, "level", 1)` 可以在找不到時**安全回傳預設值**！
  3. **死穴 3：無法進行動態反射 (Reflection) 與 AI Agent 工具呼叫**  
     當 AI 傳回要呼叫的工具名稱字串時（如 `call.name = "read_file"`），不可能寫上百個 `if/elif` 窮舉。使用 `getattr(tools, call.name)(**args)` 能一行程式碼自動反射執行該函式！

  > **🍿 生動白話比喻**：  
  > - 傳統 `obj.name`：像是**「看著對方的臉直接叫名字」**，寫程式碼時名稱必須死死固定。  
  > - `getattr(obj, var_name)`：像是**「看著手中的名牌卡片去叫人」**，卡片上寫什麼名字（變數內容），程式就在執行期動態去找那個名字！

  - **核心語法與參數**：`getattr(物件名, "屬性或方法名", 預設值)`

  ```python
  class Player:
      def __init__(self):
          self.name = "Matthew"
          self.hp = 100
      
      def attack(self, target: str):
          return f"{self.name} 攻擊了 {target}！"

  player = Player()

  # -------------------------------------------------
  # 場面 1：動態讀取屬性 (當屬性名稱存在變數中)
  attr_name = "name"
  print(getattr(player, attr_name))  # 輸出: Matthew

  # -------------------------------------------------
  # 場面 2：安全防崩潰 (屬性不存在時回傳設定的預設值)
  # 若不用 getattr，存取不存在的屬性會直接爆出 AttributeError
  level = getattr(player, "level", 1)  # 輸出: 1 (預設值)

  # -------------------------------------------------
  # 場面 3：動態呼叫方法 (AI Tool Call 反射極佳搭檔)
  method_name = "attack"
  method = getattr(player, method_name)  # 取出函數物件
  print(method("史萊姆"))  # 輸出: Matthew 攻擊了 史萊姆！
  ```

  > **💡 `getattr()` 與 `setattr()` 搭配技巧**：  
  > 搭配 `setattr(obj, attr_name, value)` 可以輕鬆實現「資料庫字典 ➔ 批量自動賦值給物件屬性」的極速綁定！

---

### 實體方法 

**類別裡的函數，也就是讓物件執行的函數**

用來設定物件的**動作**

只要是同一類別的實體，都可以使用類別內定義的方法

跟初始化時一樣，**第一個引數要傳入物件自己（self）**

```Python
class Player:
    def __init__(self, name):
        self.name = name
        self.hp = 100

    def take_damage(self, damage):
        self.hp -= damage
        print(f"血量為{self.hp}")
    
player_1 = Player("Matthew")

player_1.take_damage(30)
```

---

### 類別屬性

宣告對於**整個類別共有的屬性**，

類別屬性**屬於類別本身**，但類別內的**實體也可存取**

同樣可以使用 **setattr\(\)、getattr\(\) **來使用

```Python
class Player:
    hp = 100 # 類別屬性
    
    def __init__(...):
        ...
```

- 讀取時透過類別本身或實體都可以

    ```Python
    class Player:
        hp = 100
        
        def __init__(self, name):
            self.name = name
        
    player_1 = Player("Matthew")
    player_2 = Player("Tim")
    
    print(Player.hp)
    
    print(player_1.hp)
    print(player_2.hp)
    ```

- 若要修改整個類別的屬性，需**透過類別來修改**

    ```Python
    class Player:
        hp = 100
        
        def __init__(self, name):
            self.name = name
        
    player_1 = Player("Matthew")
    player_2 = Player("Tim")
    
    Player.hp = 50
    
    print(Player.hp)
    
    print(player_1.hp)
    print(player_2.hp)
    ```

    > 若透過實體修改，則只對於此實體修改
    > 
    > ```Python
    > class Player:
    >     hp = 100
    >     
    >     def __init__(self, name):
    >         self.name = name
    >     
    > player_1 = Player("Matthew")
    > player_2 = Player("Tim")
    > 
    > player_1.hp = 50
    > 
    > print(Player.hp)
    > 
    > print(player_1.hp)
    > print(player_2.hp)
    > ```

---

### @property 屬性裝飾器

核心目的：Python 內建的**無參數屬性裝飾器**，讓類別裡的方法可以像屬性一樣直接存取（不需加小括號 `()`），同時能自動攔截屬性的「讀取 (Getter)」、「寫入 (Setter)」與「刪除 (Deleter)」，進行資料驗證與安全保護。

> **💡 關鍵觀念：私有屬性與封裝防禦**  
> 在屬性名稱前加上底線（如 `self._age`）為 Python 習慣的私有屬性保護規範。透過 `@property` 可以避免外部直接存取與竄改內部變數，實現優雅的資料封裝與輸入驗證。

| 裝飾器語法 | 功能作用 | 觸發時機 |
| :--- | :--- | :--- |
| **`@property`** | 建立 Getter (讀取) | 外部讀取 `obj.attr` 時觸發 |
| **`@<attr>.setter`** | 建立 Setter (寫入) | 外部賦值 `obj.attr = val` 時觸發 |
| **`@<attr>.deleter`** | 建立 Deleter (刪除) | 外部執行 `del obj.attr` 時觸發 |

```Python
class Student:
    def __init__(self, age):
        self._age = age # 宣告私有變數 (慣例加上底線)

    # 1. 建立 Getter (讀取屬性，方法名不加底線)
    @property
    def age(self):
        """ 外部讀取 student.age 時自動觸發 """
        return self._age

    # 2. 建立 Setter (安全修改，名稱必須與 @property 方法一致)
    @age.setter
    def age(self, value):
        """ 外部賦值 student.age = 25 時觸發安檢邏輯 """
        if not isinstance(value, int) or value < 0:
            raise ValueError("❌ 年齡必須為非負整數！")
        self._age = value

    # 3. 建立 Deleter (刪除攔截)
    @age.deleter
    def age(self):
        """ 外部執行 del student.age 時觸發 """
        print("⚠️ [警告] 屬性已進入 Deleter 攔截清理流程")
        del self._age

# -------------------------------------------------------
# 實戰測試

s = Student(20)
print(s.age)      # 輸出: 20 (觸發 Getter)

s.age = 25        # 觸發 Setter 安全檢查並賦值
print(s.age)      # 輸出: 25

# s.age = -5      # ❌ 觸發 ValueError: 年齡必須為非負整數！

del s.age         # 輸出: ⚠️ [警告] 屬性已進入 Deleter 攔截清理流程
```

> **💡 小提醒**：Setter 與 Deleter 的裝飾器語法必須寫為 `@<屬性名>.setter` 與 `@<屬性名>.deleter`，其中的 `<屬性名>` 必須嚴格與上方 `@property` 定義的方法名稱 100% 一致！

---

### 描述器 discritor

核心目的和**屬性裝飾器**基本相同

包含了 **get、set、delete** 三個方法

- 和屬性裝飾器的差異：

    - **每個屬性的邏輯是獨立的**，若只是單一的屬性使用上較方便，直接放在**原先類別**

    - 描述器則是可以建立一個**通用的規則套用在多種屬性上**，建議在**獨立的類別中**

        ---

- 非資料描述器和資料描述器的差異：

    - 非資料描述器**只可讀取**（也沒有刪除）

    - 資料描述器包含**讀取及寫入及刪除**，且**只能透過 set 來更改值**

```Python
class NonDataDescriptor:
    
    def __get__(self, instance, owner):
        # 引數分別為 描述器本身（此函數）、呼叫描述器的物件、描述器所在類別
        
        # 防呆機制：如果是透過類別直接呼叫 (例如 Player.屬性)，通常直接回傳描述器本身
        if instance is None:
            return self
            
        # 從實體物件身上，拿出真正隱藏起來的資料
        # (通常會加上底線，例如 instance._age)
        return instance._hidden_value 
```

```Python
class DataDescriptor:
    
    # 【讀取攔截】
    def __get__(self, instance, owner):
        # if instance is None:
            # return self --> 防止用類別呼叫（選用）
            
        # 從實體物件身上，拿出真正隱藏起來的資料
        # (通常會加上底線，例如 instance._age)
        return instance._hidden_value 

    # 【寫入攔截】(資料描述器的靈魂)
    def __set__(self, instance, value):
        # 第三個引數為傳入值
        
        # 邏輯檢查
        if value < 0:
            raise ValueError("數值不能小於零！")
            
        # 檢查通過，將資料存進實體物件的隱藏變數中
        instance._hidden_value = value

    # 【刪除攔截】(選用，實戰中較少用到)
    def __delete__(self, instance):
        # 當使用者執行 del p1.屬性 時觸發
        del instance._hidden_value
```

```Python
class 類別名:
    # ✅ 正確安裝：當作類別變數掛載
    屬性 1 = 描述器名()
    屬性 2 = 描述器名()

    # 不需描述器的屬性
    def __init__(self, name):
        self.name = name
        # ❌ 致命錯誤：上方已經有描述器的不能寫進 __init__
```













---

### \_\_slots\_\_ 限定屬性存取

限制**只能有特定屬性**，無法在外部追加新屬性（會報錯），透過 **\_\_slots\_\_ **來達成

不僅是為了安全性，還可以做到記憶體的壓縮

宣告了 slots 的類別**沒有 \_\_dict\_\_**

> 物件的屬性存放在**隱藏的字典 \_\_dict\_\_**，
> 
> 而他會預先佔用較大的記憶體

```Python
class ...:
    __slots__ = [屬性 1, 屬性 2, ...]
    
    def __init__(self, ...):
    self.屬性 1 = ...
    self.屬性 2 = ...
```

---

## @classmethod 類別方法

負責處理類別裡**不需要具體物件的事**，用來**管理整個類別的共同規劃與生產線**

或需要**對所有物件共用的全域屬性進行處理**時

```Python
class ...:
    ...
    
    @classmethod
    def ...(cls, ...):
        ... # 第一個傳入的必為類別本身，cls 和 self 一樣只是慣用名
    @classmethod
    def ...(cls, ...):
        ... # 可以有很多個類別方法
```

---

## 繼承 Inheritance

### 繼承基本

相當於一種**分類功能**，不需要把同樣的方法重寫一遍

父類別有的所有方法，**子類別都會繼承**

```Python
class 最上層:
    ...
    
class 第二層(最上層):
    ... # 括號表示繼承誰

class 第三層(第二層):
    ... # 繼承了第二層，同樣會繼承到最上層
```

> 沒有括號的並非真正意義上的最上層類別
> 
> **最上層的類別繼承自 object 類別**，裡面包含一些物件的基本方法，如 \_\_init\_\_

```Python
class Player:
    def attack(self, ...):
        ...
class VipPlayer(player):
    ...
    
Matthew = VipPlayer()

Matthew.attack()
```

以底層來看，

會先搜尋物件本身（Matthew），再來尋找身處在的類別（VipPlayer），最後找到繼承自的父類別（Player），最後找到了呼叫的方法，

也就是**由下往上找 \-\-\> MRO 的概念**

---

### 覆寫 Override

若子類別存在和父類別同名的方法，**父類別的會被覆蓋**，只執行子類別的方法，

因為尋找機制是**由下往上**

```Python
class Animal:
    def walk(self):
        print("animal is walking")

class Cat(Animal):
    def walk(self):
        print("cat is walking")

kitty = Cat()
kitty.walk() # cat is walking
```

而包含 \_\_init\_\_ 也會被覆寫，用下面方法解決：

---

#### super\(\) 取用上層

若想取用上層方法，可以直接指名道姓，但不推薦

但如果有**兩個繼承的類別**，而他們都需要傳入參數，就只能用這樣的方式（**寫出名字**）否則**用 super\(\) 會把所有參數傳到第一個類別裡**（依據 MRO 得出的第一順位）

```Python
class Animal:
    def __init__(self, name):
        self.name = name

class Dog(Animal):
    def __init__(self, name, breed):
        Animal.__init__(self, name) #  舊式寫法 (Hardcoding 硬編碼)：直接叫名字
        self.breed = breed
        
a1 = Dog("white", "food")

print(a1.name) # white 擁有原本父類別的屬性
print(a1.breed) # food 子類別加入的新屬性
```

> 因為若想用在中間加一個新類別 Mamal 就會出問題，需要改成 Mamal\.\_\_init\_\_
> 
> ```Python
> class Animal:
>     def __init__(self, name):
>         self.name = name
> 
> class Dog(Animal):
>     def __init__(self, name, breed):
>         # ⚠️ 舊式寫法：硬生生地指名道姓叫老爸
>         Animal.__init__(self, name) 
>         self.breed = breed
> 
> class Cat(Animal):
>     def __init__(self, name, color):
>         Animal.__init__(self, name) 
>         self.color = color
> 
> # ... 假設下面還有 500 種動物，每一種的 __init__ 裡面都寫死了 Animal.__init__
> ```
> 
> 要新增 Mamal 類別
> 
> ```Python
> # 中間新增了這個類別
> class Mammal(Animal):
>     def __init__(self, name):
>         Animal.__init__(self, name)
>         self.warm_blooded = True # 哺乳類特有屬性
> ```
> 
> ```Python
> class Dog(Mammal):  # 🔴 動作 1：把括號裡的 Animal 改成 Mammal
>     def __init__(self, name, breed):
>         # 🔴 動作 2：你必須手動把這行刪掉，改成 Mammal.__init__！
>         # 如果你忘記改這行，Dog 就不會經過 Mammal，牠就不會有 warm_blooded 屬性！
>         Mammal.__init__(self, name) 
>         self.breed = breed
> ```

可以使用 **super\(\)**：

```Python
super().父類別方法名稱(父類別所需參數) # 不需要 self
```

```Python
# 中間新增了這個類別
class Mammal(Animal):
    def __init__(self, name):
        super().__init__(name)
        self.warm_blooded = True

# 😎 優雅工程師的改法
class Dog(Mammal):  # 🟢 唯一動作：只要在這裡把 Animal 換成 Mammal 就好！
    def __init__(self, name, breed):
        # 🟢 這裡的程式碼【一行都不用改】！
        # 因為 super() 會自動偵測到 Dog 現在的上一層變成了 Mammal，
        # 它會自動去呼叫 Mammal 的 __init__！
        super().__init__(name) 
        self.breed = breed
```

其實不是單純取用上層，而是取用** MRO 的下一個**

也就是說多重繼承中使用時，會優先取用**平行層級的類別**

```Python
class Animal:
    def sleep(self):
        print("Zzzzz")

class Bird(Animal):
    pass

class Fish(Animal):
    def sleep(self):
        print("我睡覺不用閉眼睛")

# 多重繼承
class Cat(Bird, Fish):
    def sleep(self):
        super().sleep() # 取用 fish 而不是 Animal

kitty = Cat()
kitty.sleep()  # 我睡覺不用閉眼睛
```

---

### 多重繼承 Multiple Inheritance

物件可以同時繼承自多個類別

```Python
class 類別名(父類別, ...)
# 可同時繼承多個類別
```

若多個父類別中出現同名方法，MRO 規則中取用排在前面的類別

```Python
class A:
    def method(self):
        pass

class B:
    def method(self):
        pass

class C(A, B): # A 在前面，取用 A 的方法
    pass
```

---

#### 混入 Mixins

使用多重繼承時通常會用此的方式，

想像為卡片式的功能卡，一張一張拼湊出所有的技能

```Python
# 基礎類別
class Character:
    def __init__(self, name):
        self.name = name

# --- Mixin ---
class FlyableMixin:
    def fly(self):
        print(f"🕊️ {self.name} 飛上了天空！")

class SwimmableMixin:
    def swim(self):
        print(f"🏊 {self.name} 在水裡潛行。")

class FireBreathMixin:
    def breathe_fire(self):
        print(f"🔥 {self.name} 噴出了熊熊烈火！")

# ==========================================

# 鴨子：會飛、會游
class Duck(Character, FlyableMixin, SwimmableMixin):
    pass

# 噴火龍：會飛、會噴火
class Dragon(Character, FlyableMixin, FireBreathMixin):
    pass

d = Dragon("dragon")
d.fly()           # 來自 FlyableMixin
d.breathe_fire()  # 來自 FireBreathMixin
```

---

## 元類別 Mateclass

類別也是一種物件實體，

而類別以及方法、描述器等非自建實體都是由名叫 type 的元類別創建

- 作用的時間是在**類別被定義的階段，事先攔截進行預處理**

什麼時候該自建元類別？

- **框架開發：** 當你需要為其他開發者**提供一套基類**，並確保他們定義的子類別符合特定的結構（例如 Django 的模型系統）。

- **自動化註冊：** 當你需要讓**新定義的類別自動「註冊」**到某個全局插件清單中。

- **動態修改：** 當你需要**大量修改類別的屬性**，且這些修改**無法透過簡單的繼承或裝飾器**來達成時。

```Python
class 元類別名(type):
    # 引數為：元類別本身、使用元類別的類別名、使用類別的父類別，屬性字典
    def __new__(msc, name, bases, attrs)
        ... # 攔截與處理的邏輯
        
    return super().__new__(msc, name, bases, new_attrs) # 若上方邏輯有對字典進行修改，要設為新的：new_attrs
```

```Python
class 類別名(metaclass=元類別名):
    ...
```

> **屬性字典（Attributes Dictionary），**
> 
> 把類別裡縮排的內容打包成一個字典
> 
> ```Python
> class SpyMeta(type):
>     def __new__(mcs, name, bases, attrs):
>         print(f"\n🕵️‍♂️ 攔截到 {name} 的屬性字典：")
>         for key, value in attrs.items():
>             print(f"   🔑 {key}: {value}")
>         return super().__new__(mcs, name, bases, attrs)
> 
> class Warrior(metaclass=SpyMeta):
>     """這是一個戰士類別"""
>     hp = 100
>     
>     def __init__(self, name):
>         self.name = name
>         
>     def attack(self):
>         return "揮劍！"
> 
> # 🕵️‍♂️ 攔截到 Warrior 的屬性字典：
> #   🔑 __module__: __main__
> #   🔑 __qualname__: Warrior
> #   🔑 __doc__: 這是一個戰士類別
> #   🔑 hp: 100
> #   🔑 __init__: <function Warrior.__init__ at 0x104e...>
> #   🔑 attack: <function Warrior.attack at 0x104e...>
> ```

---

## dataclass 資料類別 (Python 3.7+)

專門用來簡化**「以儲存資料為主要目的之類別」**的進階魔術裝飾器語法糖

- **什麼是 dataclass？（本質就是自建複合型別 Custom Type）**
  在 Python 中，任何 Class 本質上都是在自訂一個全新型別。`@dataclass` 讓你能以極簡語法**自建複合型別 (Custom Composite Type)**：
  - 當內建的 `int`、`str` 無法單獨表達複雜業務物件（如包含姓名、年齡、Email 的「使用者」）時，你可以透過 `@dataclass` 定義一個 `User` 型別！
  - 相比傳統 `dict` 字典：自訂型別能享受 IDE 的**自動補全 (Auto-complete)**、**靜態型別檢查**與**拼錯字即時紅線警報**！

  > **生動白話比喻**：  
  > 內建型別 (`int`, `str`) 像**基礎建材**（水泥、木板）；  
  > 而 `dataclass` 像**將建材組合並登記專利規格的「預製模組套件」**（如「標準雙人床套件」）：你把長、寬、材質組合起來，定義了名為 `DoubleBed` 的新型別，從此函數參數就能直接指定 `def install(bed: DoubleBed)`！

- **傳統 Class vs @dataclass 震撼對比**：

  - ❌ **傳統手動寫法** (冗長、重複性高)：
    ```Python
    class User:
        def __init__(self, name: str, age: int, is_active: bool = True):
            self.name = name
            self.age = age
            self.is_active = is_active

        def __repr__(self):
            return f"User(name={self.name!r}, age={self.age!r}, is_active={self.is_active!r})"

        def __eq__(self, other):
            if not isinstance(other, User):
                return False
            return (self.name, self.age, self.is_active) == (other.name, other.age, other.is_active)
    ```

  - ✅ **使用 @dataclass 極速寫法** (簡潔、優雅、不易出錯)：
    ```Python
    from dataclasses import dataclass

    @dataclass
    class User:
        name: str
        age: int
        is_active: bool = True  # 支援預設值

    # 自動生成 __init__, __repr__, __eq__！
    u1 = User("Matthew", 20)
    u2 = User("Matthew", 20)

    print(u1)          # 印出: User(name='Matthew', age=20, is_active=True)
    print(u1 == u2)    # 印出: True (自動進行屬性值相等比對)
    ```

- **進階實戰技巧與參數**：
  - **`list[dataclass型別]` (容器與自訂型別組合)**：
    代表**「這是一個列表 (List)，且裡面的每一項元素都必須是該 dataclass 物件」**！能提供極致的 IDE 自動補全與型別安全防護：
    ```Python
    from dataclasses import dataclass, field

    @dataclass
    class Product:
        name: str
        price: float

    @dataclass
    class ShoppingCart:
        user_id: int
        # items 的型別註記為 list[Product]：代表列表裡面 100% 都是 Product 物件！
        items: list[Product] = field(default_factory=list)

    # 實戰使用：
    p1 = Product("MacBook", 60000.0)
    cart = ShoppingCart(user_id=1, items=[p1])

    for item in cart.items:
        # IDE 知道 item 是 Product 型別，自動彈出 .name 與 .price 補全！
        print(f"購物車商品: {item.name}")
    ```

  - **可變物件預設值防護 (`field(default_factory=...)`)**：

    在 `dataclass` 中絕對不能直接寫 `items: list = []`（會引發可變預設值陷阱 Error）。必須透過 `field(default_factory=list)` 建立獨立容器：
    ```Python
    from dataclasses import dataclass, field

    @dataclass
    class ShoppingCart:
        user_id: int
        items: list[str] = field(default_factory=list)  # 每次創建新物件時獨立生成 list
    ```

  - **建立唯讀不可變物件 (`frozen=True`)**：
    屬性將**無法被修改 (Immutable)**，並自動支援 Hash 雜湊，可直接當作字典 (`dict`) 的 Key：
    ```Python
    from dataclasses import dataclass

    @dataclass(frozen=True)
    class Point:
        x: float
        y: float

    p = Point(10.0, 20.0)
    # p.x = 30.0  # ❌ 拋出 FrozenInstanceError 阻止修改！
    ```

  - **自動生成大小比較運算 (`order=True`)**：
    自動幫你生成 `>`、`<`、`>=`、`<=` 比較方法（預設依據欄位定義的先後順序進行比較）：
    ```Python
    from dataclasses import dataclass

    @dataclass(order=True)
    class Student:
        score: int          # 比較時優先依據 score
        name: str

    s1 = Student(score=95, name="Matthew")
    s2 = Student(score=88, name="Alex")

    print(s1 > s2)  # 印出: True
    ```

---

## with 上下文管理器


核心目的就是確保資源（如檔案、網路連線）不論在程式正常執行還是發生異常時，都能被**正確地釋放或關閉**

```Python
with 上下文管理器物件:
    動作...

with 上下文管理器物件 as 從 enter() 獲取的物件（可多個）:
    動作...
```

當執行時，由上下文管理器控制，自動呼叫該物件兩個特殊方分法：

- **enter\(\)**：**進入** with 區塊時觸發，它的**回傳值會賦予給 as 後面的變數**

- **exit\(\)**：**離開** with 區塊時觸發（無論是正常結束還是拋出異常），它負責執行清理工作（如關閉檔案）

例如使用檔案：

```Python
with open('example.txt', 'r') as file:
    data = file.read()
    # 處理資料
# 離開 with 區塊後，檔案會自動關閉，不需手動寫 file.close()
```

若要製作無作用的上下文管理器，使用 nullcontext\(\)：

```Python
context = 上下文管理器 if ... 條件 else nullcontext() # 判斷是否需要上下文管理器

with conttext:
    ...
```

---

# 檔案處理與讀寫 File I/O

在 Python 中進行檔案讀寫、路徑操作與資料序列化的完整指南

- **核心基礎：`with open()` 與 `encoding="utf-8"`**
  在讀寫檔案時，最推薦且行業標準的做法是結合 [[#with 上下文管理器|with 上下文管理器]]。
  - **為什麼必須用 `with open()`？**：傳統 `f = open()` 必須手動 `f.close()`，若中途發生 Exception 拋出錯誤，檔案就會鎖死且造成記憶體洩漏；`with open()` 離開區塊時無論成功或出錯都會自動安全關閉檔案！
  - **指定 `encoding="utf-8"` 的重要性**：Windows 系統預設編碼通常為 CP950/ANSI，若不指定 `encoding="utf-8"`，讀取包含中文的 UTF-8 檔案時會引發致命的 `UnicodeDecodeError` 亂碼崩潰！

  > **生動白話比喻**：  
  > 傳統 `f = open()` 像**出門開門後忘記鎖門**，若忘記 `f.close()` 賊（記憶體洩漏/檔案鎖定）就會溜進來；  
  > 而 `with open()` 像**飯店的「自動感應電子門鎖」**：只要你離開門框（離開 with 區塊），門鎖自動咔噠一聲安全鎖上！

- **檔案開啟模式 (Modes) 總覽速查**：

| 模式 | 名稱 | 說明與特點 | 檔案不存在時行為 | 檔案存在時行為 |
| :--- | :--- | :--- | :--- | :--- |
| `'r'` | 唯讀 (Read) | **預設模式**。只能讀取不能寫入。 | 拋出 `FileNotFoundError` | 成功開啟供讀取 |
| `'w'` | 覆寫 (Write) | 清空原有內容並重新寫入。 | **自動建立新檔案** | **直接清空全部舊內容！** |
| `'a'` | 附加 (Append) | 保持原內容，在檔案末尾繼續追加寫入。 | **自動建立新檔案** | 在末尾繼續續寫 |
| `'x'` | 獨佔 (Exclusive) | 安全寫入模式。防止誤覆蓋既有檔案。 | **自動建立新檔案** | **拋出 `FileExistsError` 警報** |
| `'b'` | 二進位 (Binary) | 讀寫二進位檔案（如圖片、PDF、二進位模型）。 | 依前綴 r/w/a 決定 | 依前綴 r/w/a 決定 |
| `'+'` | 讀寫 (Plus) | 讀寫混合模式（如 `'r+'` 或 `'w+'`）。 | 依前綴 r/w 決定 | 依前綴 r/w 決定 |



- **4 大讀取檔案方法 (Read Methods)**：
  - **`f.read()` (一次性全讀)**：將整份檔案內容讀取為單一字串。適合小型檔案（如 `.txt`、配置文件）。
    ```python
    with open("config.txt", "r", encoding="utf-8") as f:
        content = f.read()
    ```
  - **`for line in f:` (逐行迭代器 - 大檔案極致推薦)**：Python 內部採用高效 Iterator 記憶體優化，就算檔案高達 100 GB 也不會爆炸！
    ```python
    with open("large_log.txt", "r", encoding="utf-8") as f:
        for line in f:
            print(line.strip())  # strip() 移除末尾換行符 '\n'
    ```
  - **`f.readline()` (讀取單行)**：每次調用讀取下一行文字，讀到檔尾時回傳空字串 `""`。
  - **`f.readlines()` (讀入字串清單)**：一次讀取所有行並回傳 `list[str]`。

- **現代物件導向路徑處理：`pathlib` (Python 3.4+ 業界標準)**：
  告別傳統 `os.path.join()` 斜線與反斜線在 Windows/macOS 上的相容性噩夢，使用現代 `Path` 物件進行語法糖讀寫：
  ```python
  from pathlib import Path

  # 1. 建立 Path 物件 (跨平台自動相容斜線)
  file_path = Path("docs/data.txt")

  # 2. 一鍵極速讀寫 (自動包含 open/close 邏輯)
  file_path.write_text("Hello Python!", encoding="utf-8")
  text = file_path.read_text(encoding="utf-8")

  # 3. 實用屬性與目錄檢查
  print(file_path.name)        # 檔名全稱 (包含副檔名): 'data.txt'
  print(file_path.stem)        # 檔名主體: 'data'
  print(file_path.suffix)      # 副檔名: '.txt'
  print(file_path.exists())    # 檔案或目錄是否存在: True/False

  # 4. 一鍵建立多層目錄
  Path("logs/2026/07").mkdir(parents=True, exist_ok=True)
  ```

- **JSON 資料結構讀寫 (`json` 模組)**：
  與字典 (dict) / 清單 (list) 相互轉換的最流行結構：
  ```python
  import json
  from pathlib import Path

  data = {"user": "Matthew", "skills": ["Python", "Ollama"], "active": True}

  # 寫入 JSON 檔案 (ensure_ascii=False 確保中文不變 Unicode 轉義碼)
  with open("user.json", "w", encoding="utf-8") as f:
      json.dump(data, f, ensure_ascii=False, indent=4)

  # 讀取 JSON 檔案
  with open("user.json", "r", encoding="utf-8") as f:
      loaded_data = json.load(f)
      print(loaded_data["user"])  # 印出: Matthew
  ```

---

# 錯誤和例外 Error and exception

- **錯誤**是在編譯時期進行掃描而出的，**若出現錯誤，代表程式完全無法運行**

- **例外**則是在執行時發生的，通常是因為**使用的方式超出了寫程式者的預期**

而**錯誤其實就是例外的一種**，只是我們將其區分為**可否挽救**（用 try）

```Python
BaseException
 ├── SystemExit (系統退出)
 ├── KeyboardInterrupt (鍵盤中斷)
 └── Exception (所有的例外與錯誤，都在這裡)
      │
      ├── SyntaxError (語法錯誤)
      │    └── IndentationError (縮排錯誤)
      │
      ├── ValueError (值例外)
      ├── TypeError (型態例外)
      └── ArithmeticError (算術例外)
           └── ZeroDivisionError (除以零例外)

```

以下是常見的錯誤及例外：

| 錯誤名稱 | 發生原因白話文 | 種類 |
| --- | --- | --- |
| **SyntaxError** | 語法寫錯（漏括號、漏冒號、打錯字） | 語法錯誤 |
| **IndentationError** | 縮排錯誤 | 語法錯誤 |
| **NameError** | 呼叫了一個**根本沒有定義過**的變數或函數名稱 | 執行期例外 |
| **TypeError** | **型別錯誤**（例如拿字串去加數字） | 執行期例外 |
| **ValueError** | 型別對了，但裡面的**“值”不合理**（例如想把字母轉成整數） | 執行期例外 |
| **IndexError** | **索引超出**（清單只有 3 個東西，你硬要拿第 100 個） | 執行期例外 |
| **KeyError** | **字典裡沒有這個 Key** | 執行期例外 |
| **AttributeError** | 呼叫了物件**沒有的屬性或方法** | 執行期例外 |
| **ZeroDivisionError** | **分母放 0** | 執行期例外 |
| **FileNotFoundError** | 試圖打開一個根本不存在的檔案 | 執行期例外 |
| **NotImplementedError** | 未實作或實作錯誤 | 執行期例外 |


---

## raise 主動引發例外

若出現編譯器覺得沒問題，但其實有錯，

如年齡輸入負數，編譯器認為是個整數，沒問題，但事實上年齡不可能有負，就需要自行引發例外

```Python
raise 錯誤/例外名
```

這裡的名字需是真實存在的類別，其繼承自 Exception 類別，
可以利用內建的錯誤或例外，也可自行創建錯誤類別

---

## asert 條件引發錯誤

- 如果條件為 True 繼續運行下面程式，否則報錯

```Python
assert 條件,錯誤訊息（選填）
```

```Python
if 錯誤條件():
    raise 錯誤訊息
```

---

## 自建錯誤類別

創建一個類別，並且需要繼承自 Exception 類別

類別的名字慣例上以 Error 結尾

```Python
# 創造專屬錯誤類別 (繼承自 Exception)
class LoginFailedError(Exception):
    pass

# 在系統中使用它
def login(password):
    if password != "1234":
        # 拋出我們自創的錯誤
        raise LoginFailedError("密碼錯誤，登入被拒絕！")
    print("登入成功！")
```

```Python
class APIError(Exception):
    def __init__(self, message, error_code):
        # 記得要繼承屬性
        super().__init__(message) 
        
        # 新增的屬性
        self.error_code = error_code
```

---

## 錯誤處理

### try / except

把可**能會出錯的部分**放在 try 裡，**若出錯了，直接進到 except 區塊內**，不引響下面程式的執行 

```Python
try:
    raise APIError("伺服器爆炸了", 500) # 此處錯誤，直接跳到下面
except APIError as e:
    print(f"錯誤發生了：{e}")
```

完整公式如下，**當然可以只要有 try 跟 except**

```Python
try:
    # ⚔️ 【挑戰區】
    # 把「可能會爆炸、有風險」的程式碼放在這裡。
    # 例如：讀取檔案、網路連線、讓使用者輸入資料、數學運算。

except 特定例外名稱 as e:
    # 🚑 【急診室】(可以有很多個)
    # 如果 try 裡面引發了「特定例外」，就會跳來這裡處理。
    # 程式不會崩潰，而是照著你的備案執行。

else:
    # 🎉 【慶功宴】(選用)
    # 只有當 try 裡面的程式「完全沒有發生任何例外」平安結束時，
    # 才會執行這裡的程式碼。

finally:
    # 🧹 【善後組】(選用)
    # 無論 try 裡面是成功還是失敗，最後「絕對、一定」會執行這裡。
    # 通常用來：關閉檔案、斷開資料庫連線、釋放資源。

```

```Python
def safe_divide(num1, num2):
    try:
        print("🚀 [系統] 開始執行除法運算...")
        result = num1 / num2
        
    except ZeroDivisionError as e:
        print(f"🚨 [攔截] 發生數學錯誤：不能除以零！(詳細：{e})")
        
    except TypeError as e:
        print(f"🚨 [攔截] 發生型態錯誤：你傳入的不是數字吧？(詳細：{e})")
        
    else:
        # 只有在「沒引發例外」時才會來到這裡
        print(f"✅ [成功] 運算完美結束！答案是：{result}")
        
    finally:
        # 不管剛才是成功還是報錯，這行死都會執行
        print("🧹 [清理] 關閉計算機運算引擎。\n")

# --- 測試時間 ---

# 測試 1：完美成功
safe_divide(10, 2)
# 輸出：🚀開始 -> ✅成功 -> 🧹清理

# 測試 2：踩到除以零的地雷
safe_divide(10, 0)
# 輸出：🚀開始 -> 🚨發生數學錯誤 -> 🧹清理

# 測試 3：踩到型態錯誤的地雷
safe_divide(10, "蘋果")
# 輸出：🚀開始 -> 🚨發生型態錯誤 -> 🧹清理
```

---

##### finally: 必定執行的清理區塊

- **使用時機**：不管你的程式碼是順利執行，還是中途報錯崩潰，你都「必須」要執行某個動作時（例如：關閉資料庫連線、釋放檔案資源、關閉硬體接口）。
- **語法**：放在 `try...except` 結構的最尾端。
- **參數說明**：無參數。
- **回傳值/輸出效果**：會無條件執行區塊內的程式碼。

```python
def process_file():
    f = open("data.txt", "r")
    try:
        # 假設這裡發生了讀取錯誤或字串轉換錯誤
        data = f.read()
        print(int(data))
    except ValueError:
        print("🚨 檔案內容不是數字！")
    finally:
        # 即使上面發生 ValueError 被 except 抓走，或者發生了「沒被抓到」的致命錯誤，
        # 這行 close() 都保證會被執行，確保檔案不會被鎖死佔用！
        f.close()
        print("🧹 檔案已安全關閉。")
```

---

## warnings 

```Python
import warnings
```

警告是**不會導致程式無法運行**，但可能會存在風險

> 像是：某個內部函式未來會被刪除、未來的語法變更提示

| 特性 | 警告 (Warning) | 例外 (Exception) | 錯誤 (Error) |
| --- | --- | --- | --- |
| **嚴重程度** | 輕微 ⚠️ | 中等 🚨 | 嚴重 ❌ |
| **程式是否中斷** | 不會中斷，繼續往下執行 | 會立即中斷，除非被捕捉處理 | 無法執行或立即崩潰 |
| **發生的時機** | 執行期 (Runtime) | 執行期 (Runtime) | 解析期 (語法) 或執行期 |
| **處理方式** | 透過 warnings 忽略或記錄 | 透過 try...except 捕捉並修復 | 修正程式碼語法或重啟系統 |
| **常見範例** | DeprecationWarning<br>UserWarning | ValueError<br>KeyError<br>IndexError | SyntaxError (語法錯誤)<br>SystemError |


---

### warnings\.filterwarnings\(\) 排除警告

```Python
warnings.filterwarnings(
    "遇到警告要怎麼做", 
    message="過濾內容", # 用文字過濾警告
    category=Warning, # 要過濾的種類，預設全部
    module="模組名", # 過濾特定模組
)
```

| 行動 (action) | 說明 |
| --- | --- |
| **"ignore"** | 完全忽略、不顯示符合條件的警告 |
| **"always"** | 每次觸發符合條件的警告時，都一定會印出 (即使在同一行代碼重複觸發) |
| **"default"** | 預設行為。相同位置（檔案與行號）發出的警告，只會印出第一次 |
| **"once"** | 整個程式執行期間，符合條件的警告不論在何處觸發，只印出第一次 |
| **"error"** | 將警告轉換為異常 (例如 UserWarning 變成 Exception)，會直接中斷程式 |


---

# Dunder

## 類別相關

### 類別創建

#### \_\_init\_\_



---

#### \_\_new\_\_

在資料交給記憶體之前，蘭街資料進行處理

- 針對尚未建立對不可變物件進行修改

    若將一不可變物件賦值給 a，再將 a 傳入 \_\_new\_\_，也無法對其改變

```Python
class 類別名:
    def __new__(cls, *args, **kwargs): # 接收類別及不定參數打包傳如， cls 只是一個名字，是可改的
        ... # 其他邏輯
        instance = super().__new__(cls, ...) # 其實是在使用 object 類別的 __init__
        return instance
```

---

### 查詢類別資訊

#### \_\_dict\_\_ 實體口袋

物件裡的所有**屬性**都會被寫在一個**字典**裡

```Python
class Player:
    def __init__(self, name, level):
        self.name = name
        self.level = level

p1 = Player("Matthew", 10)

# 偷看他的百寶袋
print(p1.__dict__)
# 輸出：{'name': 'Matthew', 'level': 10}
```

每個類別也有一個字典（除了有 **\_\_slots\_\_ **的）

```Python
class Player:
    game_version = "1.0" # 類別變數

    def walk(self):      # 方法
        print("走路中...")

# 偷看類別藍圖的百寶袋
print(Player.__dict__)

# {
#    '__module__': '__main__', --> 檔案位置
#   'game_version': '1.0',    👈 類別變數在這裡
#    'walk': <function Player.walk at 0x1059709d0>, 👈 方法在這裡
#    '__dict__': <attribute '__dict__' of 'Player' objects>, 
#    '__doc__': None --> 說明書
#}
```

---

#### \_\_bases\_\_ 查找父類別

```Python
# 類別名.__bases__
class Animal:
    pass

class Dog(Animal):
    pass
    
print(Animal.__bases__) # (<class '**object**'>,)
print(Dog.__bases__) # (<class '__main__.**Animal**'>,)，Main 代表處在目前執行的主程式
```

---





## 方法相關

### 資訊標注

#### \_\_str\_\_ 物件轉字串

```Python
class Player:
    def __init__(self, name, level):
        self.name = name
        self.level = level

# 創造一個名為 Matthew 的玩家實體
p1 = Player("Matthew", 10)

# 試圖把他印出來
print(p1) # <__main__.Player object at 0x101526ee0>
```

```Python
class Player:
    def __init__(self, name, level):
        self.name = name
        self.level = level

    # 定義物件的自我介紹
    def __str__(self):
        return f"🎮 玩家 {self.name} (目前等級：{self.level})"

p1 = Player("Matthew", 10)

# 再次試圖把他印出來
print(p1) # 🎮 玩家 Matthew (目前等級：10)
```

---

## 查詢資訊

### \_\_file\_\_ 獲取路徑

獲取當前執行檔案的完整路徑

---

## 身份相關

### \_\_name\_\_ 身份判定

有以下兩種狀態：

1. 為 **"\_\_main\_\_"**：

    為**主程式**

2. 為 "\_\_檔名\_\_"：

    為關聯到其他檔案的配角（**被 import 到別的檔案**）

---





---

# 演算法

## 各個擊破法

將**複雜的問題拆分為簡單的小問題**，分別解決後再進行合併

是一個解決問題的想法

在以下演算法中都有應用：

---



## 線性搜尋 Linear Search

### 動態陣列及連結串列

- **動態陣列（Dynamic Array**）也就是 python 的**串列**，

    它會預先申請比實際需求**稍大的空間**

    當資料塞滿時，它會自動找一塊更大的新記憶體，將舊資料搬移過去，並釋放舊空間

    適用**二進位搜尋**

    他的優勢在於**快速讀取 O\(1\)**

- **連結串列（Linked List）**，由記憶體中不連續節點組成，

    節點中包含資料本身以及下一個節點的位址

    僅適用**線性搜尋**，將資料一個一個搜尋

    優勢在於**快速插值及刪除 O\(1\)**

    ---

線性搜尋也就是從第 0 索引開始搜尋

---

## 二進位搜尋 Binary Search

- 針對**已排序資料**效率極高

- 核心方式為**折半搜尋**，一次**捨棄一半**資料

    > 如果中間值等於目標值，**找到目標**
    > 
    > 如果中間值**大於**目標值，代表目標在左半邊，捨棄右半邊
    > 
    > 如果中間值**小於**目標值，代表目標在右半邊，捨棄左半邊

- **時間複雜度**：O\(logn\)

- **空間複雜度**：O\(1\) \(迭代法\)

    > 這代表演算法在執行過程中，額外消耗的**記憶體空間**

### bisect\(\)

使用二進位搜尋法尋找元素，並**返回索引值**

> 也支持字串（依照英文字母順序排序）

```Python
import bisect

list = [1, 100, 203, 403,987]

print(bisect_left(list, 203))# 2

print(bisect_right(list, 203)) # 3
```

將新元素**照順序插入**

```Python
import bisect

list = [1, 100, 203, 403,987]

bisect.insort(list,200)

print(list) # [1, 100, 200, 203, 403, 987]
```

---

## 選擇排序法 Selection Sort

挑出資料裡的**極端值**，

每一輪**遍歷元素**挑出最極端的與索引 該輪的起始位置交換，以此往復，直到排序完成

次數為 **O\(n^2\)**

---

## 快速排序法 Quick Sort

挑選**基準值**（通常為索引位於第一個、中間或最後一個的），將容器**分割為比基準值小、一樣以及比基準值大的**，再針對小的及大的**新容器進行同樣步驟**（不斷遞回），直到**每個容器只剩一個元素**則排序完成

平均為 **O\(n\*logn\)**，但**最壞情況為 O\(n^2\)**，分別對應基準值在**中間**以及基準在**頭或尾**

```Python
def quick_sort(arr):
    if len(arr) <= 1:
        return arr  # 剩一個或沒有，不用排了
    
    pivot = arr[len(arr) // 2]  # 挑選中間當基準（最優情況）
    # 製作新串列
    left = [x for x in arr if x < pivot]   # 比基準小的
    middle = [x for x in arr if x == pivot] # 等於基準的
    right = [x for x in arr if x > pivot]  # 比基準大的
    
    # 左右兩邊各自再快速排序，然後接起來
    return quick_sort(left) + middle + quick_sort(right)

# 測試
data = [33, 10, 55, 71, 29, 1, 10]
print(quick_sort(data)) # 輸出: [1, 10, 10, 29, 33, 55, 71]
```

> 第 5 行：尋找標準值（索引 3，值為 71）
> 
> 第 7\~9 行（串列生成式）：
> 
> - left: \[33, 10, 55, 29, 1, 10\] \(全都比 71 小\)
> 
> - middle: \[71\]
> 
> - right: \[\] \(沒有比 71 大的\)
> 
> 第 12 行：拼接串列並返回

---

## 合併排序法 Merge Sort





---

## 遞迴法 Recursion Sort

遞迴函數，也就是**函數自身呼叫**，

分為**基本情況**跟**遞迴情況**，前者定義在什麼情況該終止，後者則是定義在條件還沒達成時進行的動作

---

### 堆疊 Stack

一種資料結構，

想像有很多筆資料堆在一起，而取用時，需要先拿出最上面的資料（後進先出）

- **呼叫堆疊（Call Stack）：**當函數（A）呼叫函數（B）時，B 的**執行資訊**（包括參數、區域變數、回傳地址）被打包成一個**堆疊幀（Stack Frame）**，接著被疊在堆疊的**最上方**（**A 被疊在下方**）

    > 所有函數被呼叫都會產生堆疊，無論是否有遞迴

- 而**遞迴函數也使用呼叫堆疊**

    假設現在有很多箱子（A B C D \.\.\.），我們要找到在之中的鑰匙，

    而箱子被互相堆疊，檢查 A 後發現裡面還有 B 跟 C，

    繼續檢查 B，裡面放著 D，而檢查 D 後發現裡面沒東西，

    會到 C 繼續檢查\.\.\.\.\.\.

    採用**深度優先**的方式

    > 與之相對的是**佇列**的方式，為**廣度優先**

    而他們的堆疊結構為：

|    D|    |    D 為空|
|---|---|---|
|    B|    |    B 內找到 D|
|    A|    C|    A 內找到 B C，先檢查 B|

|    D|    |    刪除堆疊|
|---|---|---|
|    B|    ||
|    A|    C|    回頭檢查 C|

    > 若遞迴疊太高，會耗盡電腦記憶體

---































---

# time

```Python
import time
```

## time\.time\(\) 獲取 Unix 時間截記

Unix 時間截記（Unix Timestamp）

回傳的是從 **1970 年 1 月 1 日 00:00:00 UTC**（稱為 Unix Epoch / 紀元時間）開始，到目前這一刻所經過的**總秒數**，資料型態為浮點數

而他是**可以被轉換為幾點幾分的**

可以用來做**時間差的計算**，例如執行此動作耗時多久

但缺點是若剛好遇到設備連網更新時間，會導致

---

## time\.perf\_counter\(\) 開機後經過時間

> **Performance \-\> 效能／表現**
> 
> **Counter \-\> 計數器／碼表**

理論上是從設備開始運行時為起點開始計時，但**實際上參考起點是未被定義的**

兩次呼叫的**差值才有意義**

---

# OS

用以**處理檔案、管理目錄、讀取環境變數，或是執行終端機命令 **

```Python
import os
```

---

## os\.environ 環境變數

像**字典**一樣，利用鍵來取值

- 鍵與值**都是字串**

### os\.environ\.get\(\) 取得環境變數

```Python
... = os.environ.get("鍵")

... = os.environ.get("鍵", "預設") # 如果找不到，使用預設值
```

---

### os\.environ\[\] 更改環境變數

```Python
os.environ["鍵"] = "更改值"
```









---

## 資料

### os\.makedirs\(\) 建立資料夾

- **mkdir\(\) 和 makedirs** 差異，前者無法**自動補上不存在的資料夾**（報錯），後者可以

    > 路徑為 A/B/C 但只有 A 資料夾，後者可以把缺的補上

```Python
os.makedirs(存放路徑, exist_ok=True)
```

- **exist\_ok**：是否運許路徑資料夾**已經建立**

---

### 資料路徑

#### \. / \.\. 目前資料夾／上層資料夾 

檔案本身也算一層資料夾

髒路徑最好要 **os\.path\.abspath 清理**

---

#### os\.path\.abspath 顯示完整路徑

把相**對路徑改為絕對路徑**（給訂一個檔案得到此檔案完整路徑），**優化髒路徑**（把 \.\. 改為完整路徑）

---

#### os\.path\.dirname\(\) 去掉最尾

把路徑中的**最後一個項目去掉**

```Python
# 假設這是當前檔案路徑
file_path = "/Users/username/project/src/main.py"

# 第一層：拿到 src 資料夾
src_dir = os.path.dirname(file_path) 
# /Users/username/project/src

# 第二層：再往上一層，拿到專案根目錄
project_root = os.path.dirname(src_dir) 
# /Users/username/project
```

---

#### os\.path\.basename 保留最尾

保留最尾部路徑

---

#### os\.path\.join 路徑連接

把**一連串的路徑連接在一起**

> **為什麼不用 \+ 進行字串相加？**
> 
> 不同的作業系統，路徑裡使用的**斜線方向是不一樣的**：
> 
> - **Windows** 使用反斜線：data\\config\.json
> 
> - **Mac / Linux** 使用正斜線：data/config\.json
> 
> 如果你用字串相加寫死斜線（例如 folder \+ "/" \+ filename），你的程式在 Windows 上執行可能就會直接崩潰
> 
> 而 os\.path\.join 會**自動偵測執行程式的作業系統**，並補上正確的斜線

```Python
full_path = os.path.join("users", "project", "data", "records.csv")
# users/project/data/records.csv
```

---

#### os\.path\.exists\(\) 路徑是否存在

檢查你指定的**檔案或資料夾**是否存在，回覆**布林值**

```Python
... = os.path.exists("路徑")
```



---

#### 改名

##### os\.rename 改名

在 windows 系統若出現重複檔名檔案，**無法直接付蓋**

```Python
os.rename("原檔路徑", "新檔名") # 檔名可包括路徑
```

---

##### os\.replace 改名並覆蓋

全系統都可直接付蓋

---

# SYS

System\-specific parameters and functions，

主要用來**與 Python 解譯器（Interpreter）以及執行腳本的作業系統環境進行互動**

```Python
import sys
```

## 匯入

### \_\_package\_\_ 所屬套件

指定該腳本（檔案）屬於的套件，可以做**相對匯入**

```Python
__package__ = "套件名"
```

---

### sys\.path 路線清單

一個串列，為搜尋路徑的清單，匯入時會按照索引順序一一尋找對應檔案

如果自行寫了套件**放在非當前資料夾內**，就需要 **sys\.path\.append**（無法直接 import）

用 **\.append\(\) 新增路徑**

> 搭配 **OS** 套件獲取路徑

```Python
sys.path.append(路徑)
```

---













---

# argparse

用來解析命令列參數

## 如何進行

有以下步驟：

1. **建立解析器物件 \-\> argparse\.ArgumentParser\(\)**

    ```Python
    parser = argparse.ArgumentParser(description="這是一個 argparse 的示範教學程式")
    # description 會顯示在 help 說明的最上方
    ```

2. **定義參數規則 \-\> add\_argument\(\)**

    - **位置參數：**不加 \- \-\-，為強制性，不填會報醋

        ```Python
        import argparse
        
        parser = argparse.ArgumentParser(description="模擬檔案複製的位置參數範例")
        
        # 定義位置參數（注意：字首完全沒有 - 或 --）
        parser.add_argument("source", type=str, help="來源檔案的路徑")
        parser.add_argument("destination", type=str, help="目的檔案的路徑")
        
        args = parser.parse_args()
        
        # 在程式中直接使用 args.source 和 args.destination
        print(f"命令正確！正在將檔案從【{args.source}】複製到【{args.destination}】...")
        ```

        ```Bash
        python copy_file.py photo.jpg backup_folder/
        
        photo.jpg 對應到 args.source；backup_folder/ 對應到 args.destination
        ```

    - **選填參數：**字首**必須加上** \- 或 \-\-，**沒填為 None**

        > 有 \- 跟 \-\- 兩種標籤，都指向一樣的參數
        > 
        > ```Bash
        > python script.py -n 5
        > 
        > python script.py --number 5
        > ```

    - **旗標參數：**字首**必須加上** \- 或 \-\-，且要設定  **action="store\_true"**

        > 終端機**有寫這個參數，該變數就為 True**，沒寫就是 False

    ```Python
    # 【位置參數 (Positional)】：預設為「必填」
    parser.add_argument("echo", type=str, help="說明")
    
    # 【選填參數 (Optional)】：字首加上 - 或 --，可限制型態與給予預設值
    parser.add_argument("-n", "--number", type=int, default=1, help="說明") # 沒填就放 default（預設值）
    
    # 【旗標參數 (Flag)】：只要終端機有寫這個參數，該變數就為 True，沒寫就是 False
    parser.add_argument("-u", "--uppercase", action="store_true", help="
    ```

    - **choices **來限制選項，**非定義的選項無法被輸入**

        ```Bash
        parser.add_argument("--env", choices=["dev", "staging", "prod"]) # 限制只能購從串列裡選
        ```

3. **解析參數 \-\> parse\_args\(\)**

    ```Python
    args = parser.parse_args()
    ```

4. 獲取參數

    ```Python
    result = args.echo * args.number
    ```

---

## 終端運作

假設檔名為 script\.py

- 查看說明文件：

    ```Bash
    python script.py -h
    ```






---

[[人工智慧與機器學習]]
