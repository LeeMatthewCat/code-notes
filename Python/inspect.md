# 概念與原理

## 什麼是 inspect 模組？

Python 官方標準庫中專為**「程式碼自省 (Introspection / Reflection) 與動態物件解構」**設計的核心工具（無需 `pip install`，直接 `import inspect`）。

在 Python 中，「自省」是指程式在**運行時 (Runtime)** 能夠檢查、探測自身或其他物件的型別、屬性、原始碼、函式簽名與執行堆疊的能力。

`inspect` 模組是現代 Web 框架（如 FastAPI、Flask）、依賴注入系統（如 Pytest）、以及 **AI Agent 工具自動註冊（Tool Calling / Function Calling Schema 生成）** 背後的靈魂底層模組！

> **🍿 生動白話比喻**：  
> - **一般物件**：像一個封裝嚴密的**「黑盒子」**，外部只能呼叫它，看不到它裡面長什麼樣子。  
> - **`inspect` 模組**：像一台**「X 光掃描機與內視鏡 🔬」**。往黑盒子上一照，函式有幾個參數、型別標註是什麼、預設值多少、原始碼寫在硬碟哪一行，全被照得一清二楚！

---

## 為什麼需要 inspect？核心應用場景對照表

| 應用場景 | 傳統做法的痛點 | inspect 的神級威力 |
| :--- | :--- | :--- |
| **AI Agent 工具自動註冊** | 手動為每個 Python 函式寫死繁瑣的 JSON Schema 字典 | 用 `inspect.signature()` 自動提取參數名、Type Hints 與 Docstring，**0 行重複代碼自動生成 Tool Schema** |
| **FastAPI 式自動依賴注入** | 呼叫函式時無法動態匹配未知名稱的參數 | 用 `sig.parameters` 分析目標函式需要什麼，自動從請求中抽取出對應參數傳入 |
| **非同步/生成器自動適配** | 無法預先得知傳入的函式是否需要 `await` | 用 `inspect.iscoroutinefunction()` 一秒判斷，決定走同步或非同步邏輯 |
| **動態日誌與除錯追蹤** | 只能靠報錯時的 Traceback | 用 `inspect.stack()` 即時得知「是誰在第幾行呼叫了當前函式」 |

---

# 核心 API 函數字典對照表

| 類別 / 函式名稱 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- |
| **[[#inspect.signature() 提取函式簽名\|inspect.signature()]]** | **提取函式/類別的簽名物件 (Signature)** | 解析參數清單、Type Hints 型別標註、預設值與參數驗證 |
| **[[#inspect.getdoc() 智慧獲取文件\|inspect.getdoc()]]** | **智慧獲取並清理說明文件 (Docstring)** | 自動去除多行 Docstring 的邊界縮排，取得乾淨說明 |
| **[[#inspect.getsource() 與 inspect.getfile() 原始碼與路徑獲取\|inspect.getsource()]]** | **獲取物件的原始碼字串** | 動態程式碼分析、即時代碼審查與日誌展示 |
| **[[#inspect.getsource() 與 inspect.getfile() 原始碼與路徑獲取\|inspect.getfile()]]** | **取得物件被定義的檔案絕對路徑** | 定位套件與模組在系統中的實際安裝位置 |
| **[[#inspect.iscoroutinefunction() 與常用型別判斷\|inspect.iscoroutinefunction()]]** | **判斷目標是否為 async def 非同步函式** | 建立同時支援同步與非同步函式的通用裝飾器或框架 |
| **[[#inspect.iscoroutinefunction() 與常用型別判斷\|inspect.isclass() / isfunction()]]** | **判斷物件的具體 Python 型別** | 動態載入外掛模組時掃描所有合法類別或函式 |
| **[[#inspect.getmembers() 批次獲取物件成員\|inspect.getmembers()]]** | **批次獲取物件的所有屬性與方法** | 自動化測試、外掛系統掃描與反射探索 |
| **[[#inspect.stack() 追蹤呼叫來源\|inspect.stack()]]** | **取得當前呼叫堆疊鏈 (Call Stack)** | 高級日誌記錄器、追蹤呼叫來源函數名稱與行號 |
| **[[#取得 Annotated 的型別與元數據\|typing.get_origin()]]** | **判斷型別的原型 (如是否為 Annotated)** | 搭配 `inspect` 解構複雜型別標註與泛型容器 |
| **[[#取得 Annotated 的型別與元數據\|typing.get_args()]]** | **提取型別的所有參數與元數據** | 拆解 `Annotated[T, metadata]` 取得基礎型別與元數據 |

---

# 核心功能與語法大解密

## 1. 函式簽名與參數解構 (AI Agent & 框架開發核心)

##### inspect.signature() 提取函式簽名

- **使用時機**：需要分析一個 Callable（函式、類別建構子、方法）的參數結構、型別標註或預設值時使用。
- **語法**：`sig = inspect.signature(callable, follow_wrapped=True)`
- **參數說明**：
  - `callable`：要檢查的目標函式或類別。
  - `follow_wrapped`：布林值，預設為 `True`。若函式被裝飾器包裹（且使用了 `@functools.wraps`），會自動穿透裝飾器取得最底層原始函式的簽名。
- **回傳值**：
  - `Signature` 物件，包含 `.parameters` （本質上就是**字典**）與 `.return_annotation`。

```python
import inspect

def calculate_tax(price: float, rate: float = 0.05) -> float:
    """計算含稅金額"""
    return price * (1 + rate)

# 1. 獲取簽名物件
sig = inspect.signature(calculate_tax)

print(f"完整簽名: {sig}")                 # 輸出: (price: float, rate: float = 0.05) -> float
print(f"回傳型別標註: {sig.return_annotation}")  # 輸出: <class 'float'>
```

---

##### sig.parameters 與 Parameter 物件解剖

- **使用時機**：遍歷函式的每一個參數，提取參數名稱、型別標註與預設值。
- **資料結構**：`sig.parameters` 本質上是一個**有序字典 (`mappingproxy`)**。每次呼叫 `for name, param in sig.parameters.items():` 時，成對產出兩大內容：
  - **1. 鍵 (Key, `name`)**：**參數名稱字串**（`str`，如 `"price"`、`"rate"`）。
  - **2. 值 (Value, `param`)**：**`inspect.Parameter` 物件**（專門封裝該參數所有詳細資訊的物件實例）。

而其中的 **`Parameter` 物件（字典的值）**，又可進一步提取出以下 4 大核心屬性：

| 屬性名稱 | 型別 | 說明與範例 |
| :--- | :--- | :--- |
| **`param.name`** | `str` | 參數名稱字串（如 `"price"`、`"rate"`）。 |
| **`param.annotation`** | `Any` | 參數的型別標註。若未標註則為 `inspect.Parameter.empty`。 |
| **`param.default`** | `Any` | 參數的預設值。若無預設值則為 `inspect.Parameter.empty`。 |
| **`param.kind`** | `_ParameterKind` | 參數傳遞型態（位置參數、關鍵字參數、`*args`、`**kwargs` 等）。 |

```python
import inspect

def send_email(to: str, content: str, cc: list[str] = [], *args, retry: int = 3, **kwargs):
    pass

sig = inspect.signature(send_email)

for name, param in sig.parameters.items():
    # 判斷是否有預設值
    default_val = param.default if param.default is not inspect.Parameter.empty else "無預設值"
    # 判斷是否有型別標註
    type_hint = param.annotation if param.annotation is not inspect.Parameter.empty else "未標註"
    
    print(f"🔹 參數: {name:7} | 型別: {str(type_hint):15} | 預設值: {str(default_val):10} | 種類: {param.kind.name}")
```

> **⚠️ 判斷「無預設值」的致命陷阱 (`empty` vs `None`)**：  
> 絕對**不能**用 `if param.default is None:` 判斷參數有沒有預設值！因為使用者的預設值可能剛好就是 `None`（如 `def foo(x=None):`）。  
> **唯一標準判斷法**：必須與單例物件 `inspect.Parameter.empty` 進行比對：  
> `if param.default is inspect.Parameter.empty:` ➔ 代表這是一個必填參數！

---

##### 取得 Annotated 的型別與元數據

搭配 `typing.get_args()` 直接拆解 `param.annotation`（回傳為標準元組）：

- **`[0]` (索引取值)**：取出第 0 個位置的**真實型別**
- **`[1:]` (切片操作)**：從第 1 個位置切到最後，打包取出**所有元數據**

```python
import inspect
from typing import Annotated, get_args

def task(name: Annotated[str, "用戶名稱", {"max_len": 20}]):
    pass

param = inspect.signature(task).parameters["name"]

# 呼叫 get_args() 取得元組: (<class 'str'>, '用戶名稱', {'max_len': 20})
args = get_args(param.annotation)

type_hint = args[0]   # 索引 0 ➔ 輸出: <class 'str'>
metadata  = args[1:]  # 切片 1: ➔ 輸出: ('用戶名稱', {'max_len': 20})
```

---

##### sig.bind() 動態引數綁定與校驗

- **使用時機**：在不真正執行函式的情況下，**預先檢查使用者傳入的引數是否符合函式簽名要求**（例如：是否有漏傳必填參數、是否傳入了未知的關鍵字引數）。
- **語法**：`bound_args = sig.bind(*args, **kwargs)`
- **核心方法**：
  - `sig.bind(*args, **kwargs)`：嚴格綁定，若缺少必填參數直接拋出 `TypeError`。
  - `sig.bind_partial(*args, **kwargs)`：部分綁定，允許缺少必填參數（適合柯里化或裝飾器）。
  - `bound_args.apply_defaults()`：自動為未傳入的參數填上其預設值。
  - `bound_args.arguments`：解析完成後的最終引數字典。

```python
import inspect

def create_user(username: str, role: str = "guest"):
    pass

sig = inspect.signature(create_user)

# 1. 模擬傳入正確參數並套用預設值
bound = sig.bind("Alice")
bound.apply_defaults()
print(f"綁定後的參數字典: {bound.arguments}")  # 輸出: {'username': 'Alice', 'role': 'guest'}

# 2. 模擬非法呼叫 (多傳了未知參數)
try:
    sig.bind("Bob", age=25)
except TypeError as e:
    print(f"❌ 參數驗證失敗: {e}")  # 輸出: got an unexpected keyword argument 'age'
```

---

## 2. 物件型別與能力檢查 (Type Predicates)

##### inspect.iscoroutinefunction() 與常用型別判斷

- **使用時機**：在裝飾器、中間件或非同步框架中，需要在運行時動態判斷一個函式到底是普通同步函式、還是帶有 `async def` 的協程函式。

| 檢查函式 | 判定條件 | 回傳值 |
| :--- | :--- | :--- |
| **`inspect.isfunction(obj)`** | 目標是否為 Python 函式（包含 lambda） | `bool` |
| **`inspect.ismethod(obj)`** | 目標是否為已綁定實例的物件方法 (Bound Method) | `bool` |
| **`inspect.isclass(obj)`** | 目標是否為 Python 類別 (Class) | `bool` |
| **`inspect.iscoroutinefunction(obj)`** | 目標是否為 `async def` 宣告的非同步函式 | `bool` |
| **`inspect.isgeneratorfunction(obj)`** | 目標是否為包含 `yield` 的生成器函式 | `bool` |

```python
import inspect
import asyncio

def normal_sync_func(): pass
async def async_coro_func(): pass

print(inspect.isfunction(normal_sync_func))          # 輸出: True
print(inspect.iscoroutinefunction(async_coro_func))  # 輸出: True
print(inspect.iscoroutinefunction(normal_sync_func)) # 輸出: False
```

---

##### inspect.getmembers() 批次獲取物件成員

- **使用時機**：需要**批次掃描並列出一個類別或模組中的所有方法與屬性**（常搭配過濾條件 `predicate` 用於外掛外掛系統或自動化測試掃描）。
- **語法**：`inspect.getmembers(object, predicate=None)`
- **回傳值**：
  - `list[tuple[str, Any]]`：包含 `(成員名稱, 成員物件)` 的元組列表。

```python
import inspect

class Calculator:
    version = "1.0"
    def add(self, a, b): return a + b
    def subtract(self, a, b): return a - b

# 批次掃描 Calculator 類別中的所有「函式/方法」，自動忽略普通變數與私有屬性
methods = inspect.getmembers(Calculator, predicate=inspect.isfunction)

for name, func in methods:
    print(f"🔍 發現方法: {name}()")
```

---

## 3. 原始碼與文件自省 (Source & Docstring)

##### inspect.getdoc() 智慧獲取文件

- **使用時機**：讀取類別或函式的 Docstring，並**自動去除多行字串因縮排產生的多餘前置空格**（比原生物件 `obj.__doc__` 乾淨得多）。
- **語法**：`inspect.getdoc(obj)`
- **回傳值**：
  - 清理乾淨的說明文字字串（若無文件則回傳 `None`）。

```python
import inspect

class Database:
    """
    這是一個資料庫連線類別。
    
    支援自動重連與交易管理。
    """
    pass

# 1. 原生 __doc__ 包含大量原始縮排與換行雜訊
# 2. inspect.getdoc() 自動修剪乾淨
clean_doc = inspect.getdoc(Database)
print(clean_doc)
```

---

##### inspect.getsource() 與 inspect.getfile() 原始碼與路徑獲取

- **使用時機**：需要動態查看某個函式的原始碼、或找出某個第三方套件安裝在哪個硬碟檔案中時使用。
- **語法**：`inspect.getsource(obj)` / `inspect.getfile(obj)`

```python
import inspect
import json

def greet(name: str):
    return f"Hello, {name}!"

# 1. 取得函式定義的原始碼字串
print(inspect.getsource(greet))

# 2. 取得物件定義所在的檔案路徑
print(inspect.getfile(json))  # 輸出: .../lib/python3.x/json/__init__.py
```

---

## 4. 執行堆疊自省 (Stack & Call Frames)

##### inspect.stack() 追蹤呼叫來源

- **使用時機**：在日誌記錄、權限校驗或進階除錯中，動態探測**「是哪一個函式在第幾行呼叫了我」**。
- **語法**：`stack_list = inspect.stack()`
- **回傳值**：
  - `list[inspect.FrameInfo]`：由內而外（由近到遠）排列的 `FrameInfo` 具名元組物件清單。
    - **`stack[0]`**：**當前位置**（呼叫 `inspect.stack()` 的這行代碼）。
    - **`stack[1]`**：**上一層呼叫者 (Caller)**（直接呼叫當前函式的人）。
    - **`stack[-1]`**：**最外層程式入口**（頂層模組 `<module>`）。

###### FrameInfo 核心屬性字典

`FrameInfo` 底層為具名元組 (NamedTuple)，同時支援索引下標 (`[0]`) 與點記號 (`.屬性`) 存取：

| 元組索引 | 屬性名稱 (推薦點存取 ⭐) | 資料型別 | 代表意義與內容 | 範例回傳值 |
| :--- | :--- | :--- | :--- | :--- |
| **`[0]`** | **`frame.frame`** | `FrameType` | **底層執行影格物件**（可存取局部變數 `f_locals` 與全域變數 `f_globals`） | `<frame at 0x...>` |
| **`[1]`** | **`frame.filename`** | `str` | **程式碼所在的檔案絕對路徑** | `'/app/service.py'` |
| **`[2]`** | **`frame.lineno`** | `int` | **正在執行的程式碼行號** | `42` |
| **`[3]`** | **`frame.function`** | `str` | **當前函式名稱**（頂層模組為 `'<module>'`） | `'send_email'` |
| **`[4]`** | **`frame.code_context`** | `list[str]` | **當前執行的原始程式碼文字清單** | `['    log_audit()\n']` |
| **`[5]`** | **`frame.index`** | `int` | 當前行在 `code_context` 列表中的索引 | `0` |

```python
import inspect

def log_audit():
    # 1. 取得上一層呼叫者 (Caller) 的 FrameInfo 物件
    caller = inspect.stack()[1]
    
    print(f"🔍 稽核偵測：函式 [{caller.function}] 在檔案 [{caller.filename}] 的第 {caller.lineno} 行呼叫了我！")
    print(f"   當時執行的代碼: {caller.code_context[0].strip()}")

def service_action():
    log_audit()

service_action()
# 輸出:
# 🔍 稽核偵測：函式 [service_action] 在檔案 [...] 的第 10 行呼叫了我！
#    當時執行的代碼: log_audit()
```

> **💡 效能小提醒**：  
> `inspect.stack()` 會收集整個呼叫棧的所有影格與原始碼片段，在極高頻率的迴圈中調用會有額外的效能開銷。如果只需要簡單除錯或日誌，建議在關鍵路徑或錯誤捕捉處使用。

---

# 實戰：AI Agent 工具自動轉 JSON Schema (支援 Annotated 參數說明)

現代 AI 框架（如 OpenAI、Gemini）呼叫工具時都需要結構化的 Tool JSON Schema。結合 `inspect` 與 `Annotated`，可以做到**直接從函式型別註記中提取參數說明，全自動生成 Tool Definition**：

```python
import inspect
from typing import Annotated, get_origin, get_args

# 定義帶有 Annotated 說明的工具函式
def search_products(
    keyword: Annotated[str, "搜尋商品關鍵字（如：手機、筆電）"],
    max_results: Annotated[int, "最大回傳商品數量上限"] = 5
) -> list[str]:
    """根據關鍵字搜尋商品庫存並回傳清單"""
    return [f"商品_{keyword}_{i}" for i in range(max_results)]

def function_to_tool_schema(func) -> dict:
    sig = inspect.signature(func)
    doc = inspect.getdoc(func) or ""
    
    type_map = {str: "string", int: "integer", float: "number", bool: "boolean"}
    properties = {}
    required = []
    
    for name, param in sig.parameters.items():
        ann = param.annotation
        desc = f"參數 {name}"
        
        # 1. 若使用了 Annotated，自動提取真實型別與第一條元數據說明
        if get_origin(ann) is Annotated:
            args = get_args(ann)
            target_type = args[0]
            if len(args) > 1 and isinstance(args[1], str):
                desc = args[1]
        else:
            target_type = ann
            
        param_type = type_map.get(target_type, "string")
        
        properties[name] = {
            "type": param_type,
            "description": desc
        }
        
        # 2. 無預設值者加入 required 必填清單
        if param.default is inspect.Parameter.empty:
            required.append(name)
            
    return {
        "name": func.__name__,
        "description": doc.split("\n")[0],
        "parameters": {
            "type": "object",
            "properties": properties,
            "required": required
        }
    }

# 執行自動轉換：
schema = function_to_tool_schema(search_products)
print(schema)
# 自動生成具備精準參數說明的 OpenAI / Gemini Tool JSON Schema！
```

> **💡 進階替代方案：搭配 Pydantic 自動化**：  
> 若專案中已有安裝 `pydantic`，可以直接定義 `BaseModel` 或使用 `TypeAdapter(型別).json_schema()`，Pydantic 內部會自動將 `dict` 轉為 `"object"`、`list` 轉為 `"array"` 等，無需自己維護 `type_map` 字典！

---

# 實戰除錯與核心天條

## 1. C 擴展函式無法讀取原始碼與簽名

> **⚠️ 內建 C 語言擴展 (Built-in C Functions) 自省限制**：  
> `inspect.getsource()` 與部分簽名解析**僅支援純 Python 寫成的物件**！  
> 若對 C 語言編譯的內建函式（如 `inspect.getsource(len)` 或 `inspect.getsource(math.sin)`）呼叫，會拋出 `TypeError: is not a Python module, class, method, function, traceback, frame, or code object`！

---

## 2. 裝飾器覆蓋簽名陷阱 (@functools.wraps)

> **⚠️ 裝飾器洗掉簽名陷阱**：  
> 自寫裝飾器時，如果忘記在 Wrapper 函式上方加上 `@functools.wraps(func)`，原函式的 `__name__`、Docstring 以及參數簽名會被 Wrapper 的 `(*args, **kwargs)` 徹底覆蓋掉，導致 `inspect.signature()` 無法解析真實參數！
