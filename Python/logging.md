# 概念與原理

## 什麼是 logging 模組？

**`logging`** 是 Python 內建標準庫中專為應用程式提供**彈性、可自訂、分級別事件記錄系統 (Event Logging System)** 的核心模組。

在軟體開發與伺服器維運中，記錄程式的執行狀態、除錯資訊、警示與崩潰異常，是確保系統穩定運行的第一道防線。

> **🍿 生動白話比喻**：  
> - **`print()`** 像**「在路邊隨口大喊一句話」**：喊完就隨風消散，沒有留下時間紀錄、沒有標註嚴重程度，且所有訊息混雜在一起。  
> - **`logging`** 像**「專業客機的飛行紀錄黑盒子 ✈️ 或醫院病歷系統」**：每一筆記錄都會精確蓋上**發生時間戳記**、**嚴重等級印章 (INFO/ERROR)**、**由哪位醫生 (模組名稱 `__name__`) 記錄**，並自動依規則歸檔到檔案中或發送警報！

---

## 為什麼要用 logging 取代 print()？

| 比較維度 | `print()` (傳統方式 ❌) | `logging` (標準工程化方式 ✅) |
| :--- | :--- | :--- |
| **重要度分級** | 無分級（所有輸出權重相同） | **5 大等級**（DEBUG / INFO / WARNING / ERROR / CRITICAL） |
| **輸出目的地** | 只能輸出到終端機標準輸出 (`stdout`) | **多目標分流**（終端機、本地檔案、遠端日誌伺服器、Email） |
| **生產環境控制** | 無法關閉，必須手動刪除代碼 | **全域一鍵控制**（只需調整等級臨界點，即可關閉所有除錯訊息） |
| **上下文元數據** | 需手動字串拼接時間、行號與函式名 | **自動注入** 時間戳記、檔案名稱、函式名、行號與執行緒 ID |
| **崩潰追蹤** | 只能印出字串，丟失完整 Traceback | **`logger.exception()`** 自動抓取完整錯誤堆疊與行號 |
| **硬碟防爆保護** | 若自行寫入檔案容易塞爆硬碟 | **日誌自動輪轉 (Rotation)**（限制檔案大小與保留份數） |

---

## logging 的 4 大核心支柱架構 (Four Core Components)

Python `logging` 採用模組化的管線架構設計：

```text
┌──────────────┐      發送 LogRecord       ┌───────────────┐
│    Logger    │ ────────────────────────► │    Handler    │
│ (日誌記錄入口) │                           │  (目的地分發器) │
└──────────────┘                           └───────┬───────┘
                                                   │
                     ┌─────────────────────────────┴─────────────────────────────┐
                     ▼                                                           ▼
         ┌───────────────────────┐                                   ┌───────────────────────┐
         │     StreamHandler     │                                   │      FileHandler      │
         │  (輸出到終端控制台)    │                                   │   (寫入磁碟日誌檔案)   │
         └───────────┬───────────┘                                   └───────────┬───────────┘
                     │                                                           │
                     ▼                                                           ▼
         ┌───────────────────────┐                                   ┌───────────────────────┐
         │       Formatter       │                                   │       Formatter       │
         │ (彩色文字格式化版面)    │                                   │ (詳細時間戳記與檔案行號) │
         └───────────────────────┘                                   └───────────────────────┘
```

1. **`Logger` (記錄器)**：提供應用程式直接呼叫的介面（如 `logger.info()`）。每個 Logger 擁有階層式命名空間。
2. **`Handler` (處理器)**：決定 Log 該送往哪裡（如 `StreamHandler` 往螢幕印、`FileHandler` 往檔案寫）。一個 Logger 可綁定多個 Handler。
3. **`Formatter` (格式化器)**：決定每一行日誌的版面外觀與結構（如 `[時間] [等級] [檔案:行號] 訊息`）。
4. **`Filter` (過濾器)**：提供比等級更細緻的條件過濾規則（可選）。

---

## 5 大日誌等級 (Log Levels)

`logging` 預設將日誌劃分為 5 個等級，數值越大代表事件越嚴重：

| 日誌等級 (Level) | 數值常量 | 核心意義與使用時機 |
| :--- | :---: | :--- |
| **`DEBUG`** | 10 | **詳細除錯資訊**：開發階段用於追蹤變數狀態、SQL 語句、函數進入/離開。生產環境預設關閉。 |
| **`INFO`** | 20 | **常態營運通知**：記錄系統正常運行的關鍵事件（如「伺服器啟動成功」、「使用者 ID:88 完成登入」）。 |
| **`WARNING`** ⭐*(預設)* | 30 | **潛在風險警示**：程式尚可正常運行，但出現非預期狀況（如「硬碟空間低於 15%」、「使用了棄用 API」）。 |
| **`ERROR`** | 40 | **重大功能錯誤**：特定功能已執行失敗（如「資料庫連線超時」、「檔案不存在」），但系統未完全崩潰。 |
| **`CRITICAL`** | 50 | **致命系統災難**：嚴重的系統級崩潰（如「記憶體耗盡」、「核心資料損毀」），程式可能即將終止。 |

> **⚠️ 預設等級臨界點天條**：  
> Python 預設的日誌等級臨界點是 **`WARNING` (30)**！如果你沒有進行任何設定直接呼叫 `logging.debug()` 或 `logging.info()`，**它們會被靜默忽略且完全不會印出**！日誌放行門檻需透過 [[#setLevel() 動態調整日誌等級門檻與雙重水閘機制|setLevel()]] 進行設定。

---

# 核心 API 函數字典對照表

| 函數 / 類別名稱 | 類型 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- | :--- |
| **[[#logging.basicConfig() 快速全域配置\|logging.basicConfig()]]** | 模組函數 | **快速設定根記錄器的等級、格式與輸出目標** | 小型腳本或專案進入點一鍵初始化 |
| **[[#logging.getLogger() 模組化命名空間記錄器與單例特性\|logging.getLogger()]]** | 模組函數 | **取得或建立具備階層命名空間的 Logger 實例 (單例快取)** | 各模組檔案中使用 `logging.getLogger(__name__)` |
| **[[#logger.handlers 屬性與 handlers.clear() 重置清空\|logger.handlers]]** | 實例屬性 | **存放該 Logger 已綁定之 Handler 處理器的列表** | 檢查目前掛載了哪些 Handler 或判斷長度 |
| **[[#logger.handlers 屬性與 handlers.clear() 重置清空\|logger.handlers.clear()]]** | 物件方法 | **一鍵清空所有已掛載的 Handler 處理器** | 單元測試或熱重載時重置 Logger，防止重複日誌 |
| **[[#setLevel() 動態調整日誌等級門檻與雙重水閘機制\|setLevel()]]** | 物件方法 | **設定 Logger 或 Handler 的最低放行等級水閘** | 調整全域或單一處理器的過濾門檻 |
| **[[#logger.debug() / info() / warning() / error() / critical() 常用分級記錄方法\|logger.debug()]]** | 物件方法 | **記錄 DEBUG 等級的除錯訊息** | 追蹤內部變數或函式進出 |
| **[[#logger.debug() / info() / warning() / error() / critical() 常用分級記錄方法\|logger.info()]]** | 物件方法 | **記錄 INFO 等級的常態營運事件** | 記錄使用者操作或任務完成 |
| **[[#logger.debug() / info() / warning() / error() / critical() 常用分級記錄方法\|logger.warning()]]** | 物件方法 | **記錄 WARNING 等級的潛在風險警告** | 外部服務降級或資源偏低 |
| **[[#logger.debug() / info() / warning() / error() / critical() 常用分級記錄方法\|logger.error()]]** | 物件方法 | **記錄 ERROR 等級的功能執行失敗錯誤** | 捕獲業務邏輯例外錯誤 |
| **[[#logger.debug() / info() / warning() / error() / critical() 常用分級記錄方法\|logger.critical()]]** | 物件方法 | **記錄 CRITICAL 等級的致命災難崩潰** | 核心服務無法啟動或系統崩潰 |
| **[[#logger.exception() 異常堆疊全自動追蹤\|logger.exception()]]** | 物件方法 | **記錄 ERROR 訊息並自動附加完整 Traceback 錯誤堆疊** | 在 `except Exception:` 區塊內記錄崩潰詳情 |
| **[[#logging.Formatter 格式化器與佔位符大全\|logging.Formatter]]** | 類別 | **自訂每一行日誌的文字排版格式與時間格式** | 統一日誌顯示結構 |
| **[[#logging.StreamHandler 終端輸出處理器\|logging.StreamHandler]]** | 類別 | **將日誌導向終端機標準輸出 (stdout / stderr)** | 控制台即時除錯與監看 |
| **[[#logging.FileHandler 檔案寫入處理器\|logging.FileHandler]]** | 類別 | **將日誌持久化寫入本地磁碟檔案** | 儲存長期日誌紀錄 |
| **[[#RotatingFileHandler 按檔案大小自動輪轉\|RotatingFileHandler]]** | 類別 | **按檔案大小自動切割日誌，防止硬碟塞爆** | 生產環境防硬碟爆滿標準配置 |
| **[[#TimedRotatingFileHandler 按時間週期自動輪轉\|TimedRotatingFileHandler]]** | 類別 | **按時間週期（每天/每小時）自動滾動建立新日誌檔** | 依日期分類儲存的伺服器營運日誌 |

---

# 核心功能與語法大解密

##### logging.basicConfig() 快速全域配置

- **使用時機**：小型腳本或獨立工具程式，希望用最少代碼快速啟用日誌。
- **語法**：`logging.basicConfig(level=..., format=..., datefmt=..., filename=..., filemode=..., force=...)`
- **參數說明**：
  - `level`：設定最低日誌過濾等級（如 `logging.DEBUG`、`logging.INFO`）。
  - `format`：日誌輸出格式字串（如 `'%(asctime)s [%(levelname)s] %(message)s'`）。
  - `datefmt`：時間戳記格式（如 `'%Y-%m-%d %H:%M:%S'`）。
  - `filename`：若指定此參數，日誌將改為寫入此檔案而非終端機。
  - `filemode`：開啟檔案模式，預設為 `'a'`（追加寫入），設為 `'w'` 則每次啟動覆寫清空。
  - `force`：布林值（Python 3.8+）。若設為 `True`，會強制移除先前的 Handler 並重新套用設定。

```python
import logging

# 快速全域初始化：設定門檻為 DEBUG，並自訂格式
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s | %(levelname)-8s | %(filename)s:%(lineno)d | %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S"
)

logging.debug("這是一條詳細除錯訊息 (不會被忽略了)")
logging.info("系統初始化完成")
logging.warning("記憶體使用量偏高")
```

> **⚠️ basicConfig 一次性鎖定天條**：  
> `basicConfig()` 具有「先到先得」特性。一旦 Root Logger 已經擁有了 Handler（例如被其他第三方套件偷偷初始化過），後續再次呼叫 `basicConfig()` 會被**完全忽略**！若想強制覆寫，必須傳入 `force=True`（Python 3.8+）！

---

##### logging.getLogger() 模組化命名空間記錄器與單例特性

- **使用時機**：**現代標準工程化最佳實踐 ⭐**。在每個 `.py` 檔案中使用 `logging.getLogger(__name__)`，取得對應命名空間的 `logging.Logger` 物件實例。
- **語法**：`logger = logging.getLogger(name=None)`
- **產生物件本質**：回傳 `logging.Logger` 類別的實例物件。
- **核心特性 1：全域單例快取模式 (Singleton Pattern)**：
  - 在同一 Python 直譯器進程中，**只要傳入相同的 `name`，`getLogger()` 永遠回傳記憶體中的「同一個 Logger 實例」**！
  - `logging.getLogger("app") is logging.getLogger("app")` 結果必為 `True`！
- **核心特性 2：點號階層命名與冒泡傳播 (Propagation)**：
  - 命名空間支援點號分隔（如 `"app.database.mysql"`）。子 Logger 會自動將日誌沿著樹狀結構**向上傳播給父 Logger 處理**，直到 Root Logger。

```python
import logging

# 1. 單例驗證：同名 Logger 共享同一個記憶體實例
log_a = logging.getLogger("MyService")
log_b = logging.getLogger("MyService")
print(log_a is log_b)  # 輸出: True (100% 同一物件！)

# 2. 模組最佳實踐：傳入 __name__ 自動獲取當前模組路徑 (如 myproject.utils)
logger = logging.getLogger(__name__)

def query_user(user_id: int):
    logger.debug("準備查詢資料庫，Target ID: %s", user_id)
    logger.info("成功取得使用者資料: ID=%s", user_id)
```

---

##### logger.handlers 屬性與 handlers.clear() 重置清空

- **`logger.handlers` (實例屬性)**：
  - **型態**：`list[logging.Handler]`。
  - **意義**：Logger 物件內部維護的**「已綁定處理器清單」**。每當呼叫 `logger.addHandler(h)` 時，該處理器就會被追加到此清單中。
- **`logger.handlers.clear()` (方法)**：
  - **使用時機**：一鍵清空當前 Logger 身上掛載的所有 Handler 處理器。
  - **解決痛點**：在單元測試 (pytest)、Jupyter Notebook 或多次執行初始化函式時，每次 `addHandler` 都會往列表多塞一個 Handler，**導致同一行日誌被重複印出 2 次、3 次、N 次**！透過 `logger.handlers.clear()` 能瞬間還原為乾淨狀態！

```python
import logging

logger = logging.getLogger("JobWorker")

# 模擬多次呼叫初始化函式
def init_logger():
    # ⚠️ 關鍵重置：先清空舊的 Handler，防止重複累積！
    logger.handlers.clear()
    
    # 建立並掛載全新的 Handler
    handler = logging.StreamHandler()
    formatter = logging.Formatter("[%(levelname)s] %(message)s")
    handler.setFormatter(formatter)
    logger.addHandler(handler)

# 執行多次初始化
init_logger()
init_logger()

# 檢視目前掛載的 Handler 數量
print(f"目前 Handler 數量: {len(logger.handlers)}")  # 輸出: 1 (若未 clear 會變成 2！)
logger.warning("這條日誌只會精準印出 1 次！")
```

---

##### setLevel() 動態調整日誌等級門檻與雙重水閘機制

- **使用時機**：在程式執行期間動態調整過濾門檻（例如 CLI 接收到 `--verbose` 或 `--debug` 參數時一鍵切換），或是為不同的 Handler 設置專屬的過濾標準。
- **語法**：
  - `logger.setLevel(level)`
  - `handler.setLevel(level)`
- **參數說明**：
  - `level`：可傳入 `logging` 常量（如 `logging.DEBUG`、`logging.INFO`）或整數值。

#### 🌊 Logger vs Handler 雙重水閘門過濾機制

在 `logging` 架構中，一筆日誌必須依序通過 **兩道水閘門** 的嚴格審查：

```text
發送日誌: logger.debug(...)
        │
        ▼
┌──────────────────────────────────────┐
│  【第一道水閘】：Logger.setLevel()   │  ➔ 若 Logger 門檻設為 INFO，
│  (決定這筆日誌是否允許進入管線)       │     此 DEBUG 日誌直接在此被丟棄！Handler 連看都看不到！
└──────────────────┬───────────────────┘
                   │ 放行 (等級 >= Logger 門檻)
                   ▼
       ┌───────────────────────────┴───────────────────────────┐
       ▼                                                       ▼
┌──────────────────────────────────────┐    ┌──────────────────────────────────────┐
│ 【第二道水閘 A】：Console Handler     │    │ 【第二道水閘 B】：File Handler        │
│ setLevel(logging.INFO)               │    │ setLevel(logging.DEBUG)              │
│ ➔ 終端機只印出 INFO、WARNING、ERROR    │    │ ➔ 磁碟檔案完整記錄所有 DEBUG 細節    │
└──────────────────────────────────────┘    └──────────────────────────────────────┘
```

> **💡 雙重水閘防呆法則**：  
> 1. **Logger 的 level 是「總水龍頭」**：若總水龍頭 (`logger.setLevel(logging.INFO)`) 把 DEBUG 擋在門外，即使你的 FileHandler 設成 `DEBUG` 也**絕對收不到任何除錯日誌**！  
> 2. **標準黃金設定法**：通常將 **`logger.setLevel(logging.DEBUG)` 設為最寬鬆（全放行）**，再由各個 Handler 各自設定 `console.setLevel(INFO)` 與 `file.setLevel(DEBUG)` 進行精細分流！

```python
import logging

# 1. 建立 Logger 實例
logger = logging.getLogger("AppEngine")

# 2. 預設將 Logger 總水閘門設為 INFO
logger.setLevel(logging.INFO)

# 3. 模擬接收到 CLI 傳來的 --debug 參數，動態調降門檻以放行 DEBUG 日誌
enable_debug_mode = True
if enable_debug_mode:
    logger.setLevel(logging.DEBUG)
    logger.debug("已動態切換至 DEBUG 模式！")
```

---

##### logger.debug() / info() / warning() / error() / critical() 常用分級記錄方法

- **語法**：`logger.info(msg, *args, **kwargs)`

```python
import logging

logger = logging.getLogger(__name__)

# 延遲格式化寫法（效能最高，推薦 ⭐）
logger.info("使用者 %s 購買了 %d 件商品，總額: $%.2f", "Matthew", 3, 299.5)

# ❌ 不推薦：在傳入前就用 f-string 進行字串拼接（即使該等級被關閉，字串運算仍會浪費 CPU 效能）
# logger.debug(f"大量數據運算結果: {heavy_calculation()}")
```

---

##### logger.exception() 異常堆疊全自動追蹤

- **使用時機**：在 `try...except` 區塊中捕獲錯誤時使用。它等同於 `logger.error()`，但會**自動額外抓取並印出完整的 Exception Traceback（包含出錯行號與呼叫鏈）**！
- **語法**：`logger.exception(msg, *args)`

```python
import logging

logger = logging.getLogger(__name__)

def divide_numbers(a: float, b: float):
    try:
        return a / b
    except ZeroDivisionError:
        # 自動捕獲當前上下文中的異常堆疊
        logger.exception("除法運算發生錯誤！除數不可為 0")

divide_numbers(10, 0)
```

---

##### logging.Formatter 格式化器與佔位符大全

`logging.Formatter` 允許我們使用標準的 `LogRecord` 屬性佔位符來定制版面：

| 佔位符 | 說明 | 輸出範例 |
| :--- | :--- | :--- |
| **`%(asctime)s`** | 人類可讀的日誌產生時間 | `2026-08-31 18:30:15` |
| **`%(levelname)s`** | 日誌等級名稱（文字） | `INFO`, `ERROR` |
| **`%(levelname)-8s`** | 日誌等級名稱（靠左對齊並固定寬度為 8 碼） | `INFO    `, `WARNING ` |
| **`%(name)s`** | Logger 記錄器名稱 (`__name__`) | `app.services.auth` |
| **`%(filename)s`** | 產生該筆日誌的 Python 檔案名稱 | `auth.py` |
| **`%(lineno)d`** | 產生該筆日誌的具體程式碼行號 | `42` |
| **`%(funcName)s`** | 產生該筆日誌的函式/方法名稱 | `login_user` |
| **`%(process)d`** | 當前作業系統進程 ID (PID) | `19820` |
| **`%(threadName)s`** | 當前執行緒名稱 | `MainThread`, `ThreadPoolExecutor-0_1` |
| **`%(message)s`** | 開發者傳入的日誌主體訊息 | `伺服器啟動成功` |

```python
import logging

# 建立自訂格式化器
formatter = logging.Formatter(
    fmt="[%(asctime)s] [%(levelname)-8s] [%(name)s:%(funcName)s:%(lineno)d] %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S"
)
```

---

##### logging.StreamHandler 終端輸出處理器

- **使用時機**：將日誌輸出到控制台終端機 (`sys.stdout` 或 `sys.stderr`)。

```python
import logging
import sys

console_handler = logging.StreamHandler(sys.stdout)
console_handler.setLevel(logging.DEBUG)
```

---

##### logging.FileHandler 檔案寫入處理器

- **使用時機**：將日誌持久化寫入硬碟檔案。
- **語法**：`logging.FileHandler(filename, mode='a', encoding='utf-8')`

```python
import logging

file_handler = logging.FileHandler("app.log", mode="a", encoding="utf-8")
file_handler.setLevel(logging.INFO)
```

---

##### RotatingFileHandler 按檔案大小自動輪轉

- **使用時機**：**生產環境磁碟防爆必備 ⭐**。當日誌檔案達到指定大小（如 10MB）時，自動重新命名封存為 `app.log.1`、`app.log.2`，並建立新的 `app.log` 繼續寫入。當達到最大備份數時自動循環覆蓋最舊的檔案。
- **所屬模組**：`from logging.handlers import RotatingFileHandler`
- **參數說明**：
  - `maxBytes`：單一檔案最大位元組數（如 `10 * 1024 * 1024` 代表 10MB）。
  - `backupCount`：最多保留的歷史備份檔案數量。

```python
from logging.handlers import RotatingFileHandler

# 單檔上限 5MB，最多保留 5 份歷史紀錄 (app.log.1 ~ app.log.5)
rotating_handler = RotatingFileHandler(
    filename="app.log",
    maxBytes=5 * 1024 * 1024,
    backupCount=5,
    encoding="utf-8"
)
```

---

##### TimedRotatingFileHandler 按時間週期自動輪轉

- **使用時機**：按固定的時間週期（如每天午夜、每小時）自動滾動切割檔案。
- **所屬模組**：`from logging.handlers import TimedRotatingFileHandler`
- **參數說明**：
  - `when`：滾動週期單位，常見選項：
    - `'midnight'`：每天午夜輪轉。
    - `'D'`：天 (Days)。
    - `'H'`：小時 (Hours)。
    - `'M'`：分鐘 (Minutes)。
  - `interval`：時間間隔倍數（預設 1）。
  - `backupCount`：保留的歷史檔案數量（如 30 代表保留最近 30 天）。

```python
from logging.handlers import TimedRotatingFileHandler

# 每天午夜自動切出新檔案，自動保留最近 30 天的營運日誌
timed_handler = TimedRotatingFileHandler(
    filename="daily_server.log",
    when="midnight",
    interval=1,
    backupCount=30,
    encoding="utf-8"
)
```

---

# 完整工業級實戰：打造現代化日誌工廠 (Log Factory)

以下展示一個可直接複製到商業專案中使用的標準日誌工具模組，支援**終端彩色分流、檔案自動大小輪轉與防重複綁定保護**：

```python
# logger_factory.py
import logging
import sys
from pathlib import Path
from logging.handlers import RotatingFileHandler

def get_logger(name: str = "app", log_dir: str = "logs", level=logging.DEBUG) -> logging.Logger:
    """ 建立並回傳標準配置的 Logger 實例 (支援終端與自動輪轉檔案) """
    logger = logging.getLogger(name)
    logger.setLevel(level)

    # ⚠️ 核心防禦：若該 Logger 已經配置過 Handler，直接回傳，防止重複印出！
    if logger.hasHandlers():
        return logger

    # 1. 確保日誌存放目錄存在
    log_path = Path(log_dir)
    log_path.mkdir(parents=True, exist_ok=True)
    log_file = log_path / f"{name}.log"

    # 2. 定義格式化器
    file_formatter = logging.Formatter(
        fmt="%(asctime)s [%(levelname)-8s] [%(name)s:%(filename)s:%(lineno)d] %(message)s",
        datefmt="%Y-%m-%d %H:%M:%S"
    )
    console_formatter = logging.Formatter(
        fmt="%(asctime)s | %(levelname)-8s | %(name)s | %(message)s",
        datefmt="%H:%M:%S"
    )

    # 3. 建立終端處理器 (Console StreamHandler) - 僅輸出 INFO 以上
    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setLevel(logging.INFO)
    console_handler.setFormatter(console_formatter)

    # 4. 建立檔案輪轉處理器 (RotatingFileHandler) - 記錄 DEBUG 以上完整日誌
    # 單檔上限 5MB，最多保留 3 份備份
    file_handler = RotatingFileHandler(
        filename=str(log_file),
        maxBytes=5 * 1024 * 1024,
        backupCount=3,
        encoding="utf-8"
    )
    file_handler.setLevel(logging.DEBUG)
    file_handler.setFormatter(file_formatter)

    # 5. 掛載處理器到 Logger
    logger.addHandler(console_handler)
    logger.addHandler(file_handler)

    return logger


# 🧪 實測調用
if __name__ == "__main__":
    log = get_logger("my_service")
    
    log.debug("這是一條只有寫入檔案的除錯資訊")
    log.info("服務啟動成功！終端與檔案都會顯示")
    log.warning("資料庫連線池偏高: 85%")
    
    try:
        1 / 0
    except ZeroDivisionError:
        log.exception("捕獲到數學計算異常！")
```

---

# 實戰除錯與核心天條

## 1. 重複呼叫 addHandler 導致日誌重複印出多次

> **⚠️ Handler 重複綁定天條**：  
> 如果在某個函式中每次執行都寫 `logger.addHandler(handler)`，日誌會被輸出 2 次、3 次甚至幾百次！  
> **鐵律**：  
> 1. 在添加 Handler 前，先使用 `if not logger.hasHandlers():` 進行防禦檢查！  
> 2. 若為單元測試 (pytest)、Jupyter Notebook 或熱重載場景，在重新綁定前先呼叫 **`logger.handlers.clear()`** 一鍵清空舊的處理器！

---

## 2. 模組內部直接呼叫根記錄器導致全域污染

> **⚠️ logging.info() 全域污染天條**：  
> 在編寫套件或模組庫時，切勿直接寫 `logging.info(...)`（這會操作全域 Root Logger 並強制初始化）。  
> **鐵律**：在每個模組最上方一律宣告 `logger = logging.getLogger(__name__)`，透過自己的實例記錄日誌！

---

## 3. 字串格式化請使用延遲求值而非 f-string

> **⚠️ 效能損耗天條**：  
> `logger.debug(f"複雜計算: {calc()}")` 會在傳入函式前**立刻計算 f-string**，即使當前日誌等級是 `INFO`（不需要 DEBUG），CPU 依然被無謂浪費！  
> **鐵律**：善用 logging 的延遲傳參格式 `logger.debug("複雜計算: %s", calc())` 或 `if logger.isEnabledFor(logging.DEBUG):`！
