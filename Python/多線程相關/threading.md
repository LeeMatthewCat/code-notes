# 概念與原理

## 什麼是 threading 模組？

Python 官方標準庫中專為**底層多執行緒 (Multi-threading) 程式設計與執行緒同步控制**設計的核心模組（無需 `pip install`）。

執行緒（Thread）是作業系統能夠進行運算排程的**最小單位**。在同一個 Python 行程 (Process) 內部，可以同時衍生出多個子執行緒，**共享相同的記憶體空間與全域變數**。

`threading` 模組提供了手動創建工人（`Thread`）、管理工人生命週期（`start` / `join` / `daemon`）以及維持工人秩序的防撞鎖機制（`Lock` / `Event` / `Semaphore`）！

> **🍿 生動白話比喻**：  
> - **Python 行程 (Process)**：像一間**「辦公室」**，擁有獨立的房間、桌椅與冷氣資源。  
> - **執行緒 (Thread)**：像辦公室裡的**「多個員工（工人 👷‍♂️）」**。大家共享同一間辦公室的黑板（記憶體空間）。  
> - **鎖機制 (Lock)**：像黑板前面唯一的**「粉筆」**。誰搶到粉筆誰才能在黑板上寫字，其他人必須排隊等待，防止大家同時下筆把字寫成一團亂麻！

---

## 概念全景對照：threading vs queue vs subprocess

這三個模組在 Python 系統與並行程式設計中分工明確、各司其職：

| 比較維度 | `threading` (多執行緒) | `queue` (安全佇列) | `subprocess` (子進程/子行程) |
| :--- | :--- | :--- | :--- |
| **本質定位** | **程式內部的多個「工人 👷‍♂️」** | **工人之間的「安全輸送帶 🛤️」** | **呼叫系統外部的「外包公司 / CLI 🏢」** |
| **執行空間** | 同一個 Python 行程**內部**（共享記憶體空間） | 同一個 Python 行程**內部**（執行緒安全通訊） | 作業系統的**完全獨立外部行程**（獨立記憶體與環境） |
| **跨語言/工具** | 僅限執行 **Python 內部函式** | 僅限在 **Python 執行緒間傳遞物件** | 可執行 **任何外部系統指令與二進位程式**（如 `git`, `ffmpeg`, `ls`, Bash 等） |
| **資料通訊方式** | 共享全域變數（需搭配 `Lock` 防撞） | 透過 `put()` / `get()` 安全傳遞 Python 物件 | 透過標準串流二進位/文字（`stdin`, `stdout`, `stderr`） |
| **GIL 影響** | 受 GIL 限制（適合 I/O 密集型） | 本身為資料結構，內部自帶互斥鎖 | **完全不受 Python GIL 限制**（獨立 OS 行程） |
| **典型應用** | 同時發送多個 HTTP 請求、背景監聽 | 生產者-消費者任務緩衝、解耦任務流水線 | 呼叫 ffmpeg 影片轉檔、執行 git 指令、呼叫第三方 CLI 工具 |

> **💡 一秒記憶選型口訣**：  
> - **想開多個 Python 工人** ➔ 用 `threading`  
> - **工人之間要安全交接貨物** ➔ 用 `queue`  
> - **想叫外部終端指令或第三方軟體** ➔ 用 `subprocess`

---

## Python GIL 實相解密：多執行緒何時有效？何時無效？

Python（CPython 實作）內部存在一個著名的機制：**全域直譯器鎖 (GIL, Global Interpreter Lock)**。

GIL 規定：**在任何一個微小的時間點上，Python 直譯器只允許「單一執行緒」在 CPU 上執行 Bytecode**。

```text
【I/O 密集型】：Thread 1 (發送請求) ➔ 進入等待 (主動釋放 GIL) ➔ Thread 2 立即接手執行 (高度平行加速！)
【CPU 密集型】：Thread 1 (算數學) ➔ 搶住 GIL 不放 ➔ Thread 2 乾等 (多執行緒完全無加速效果，甚至變慢！)
```

| 任務類型 | 特徵與實例 | 多執行緒 (threading) 效果 | 最佳替代解法 |
| :--- | :--- | :--- | :--- |
| **I/O 密集型 (I/O-Bound)** | 網路爬蟲、下載圖片、呼叫 API、讀寫硬碟 | 💯 **大幅加速**（等待 I/O 時會主動釋放 GIL 給其他執行緒） | `threading` / `concurrent.futures.ThreadPoolExecutor` |
| **CPU 密集型 (CPU-Bound)** | 複雜數學運算、大量資料排序、影像轉檔 | ❌ **無效**（受限於 GIL，只能用單核算） | `multiprocessing` / `ProcessPoolExecutor` |

---

## 核心 API 函數與類別字典對照表

| 類別 / 函式名稱 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- |
| **[[#threading.Thread() 建立子執行緒實例\|threading.Thread()]]** | **建立子執行緒實例** | 手動建立並指派特定函式給工人執行 |
| **[[#.start() 與 .join() 啟動與阻塞等待\|t.start()]]** | **正式啟動執行緒** | 讓工人進入作業系統排程並開始跑任務 |
| **[[#.start() 與 .join() 啟動與阻塞等待\|t.join()]]** | **阻塞等待該執行緒結束** | 主程式需要等待工人做完才能繼續往下走 |
| **[[#threading.Lock() 互斥鎖 (Mutex)\|threading.Lock()]]** | **互斥鎖 (Mutex)** | 防止多個執行緒同時讀寫共用變數造成資料競爭 (Race Condition) |
| **[[#threading.RLock() 可重入互斥鎖 (Recursive Lock)\|threading.RLock()]]** | **可重入互斥鎖 (Recursive Lock)** | 允許同一個執行緒重複取得同一個鎖，防止自己鎖死自己 |
| **[[#threading.Event() 事件信號機制 (旗標廣播)\|threading.Event()]]** | **事件通知信號 (旗標廣播)** | 一個執行緒發出信號，一次性喚醒多個等待中的執行緒 |
| **[[#threading.Semaphore() 計數信號標 (限流閥門)\|threading.Semaphore()]]** | **計數信號標 (限流閥門)** | 限制同時最多允許 $N$ 個執行緒訪問某項昂貴資源 |
| **`threading.current_thread()`** | **取得當前執行緒物件** | 查詢當前運行的工人名稱或除錯 |
| **`threading.active_count()`** | **取得當前存活的執行緒總數** | 監控系統負載與執行緒數量 |

---

# 核心功能與語法大解密

## 1. 執行緒建立與生命週期管理

##### threading.Thread() 建立子執行緒實例

- **使用時機**：當你需要將一個耗時的函式指派給獨立的後台工人並行執行時使用。
- **語法**：`threading.Thread(target=None, name=None, args=(), kwargs={}, daemon=None)`
- **參數說明**：
  - `target`：要在子執行緒中執行的目標函式（傳入函式名稱，**千萬不要加括號 `()`**）。
  - `name`：該執行緒的自訂名稱（方便日誌除錯，預設為 `Thread-1`, `Thread-2`）。
  - `args`：傳給目標函式的位置引數**元組 (Tuple)**。
  - `kwargs`：傳給目標函式的關鍵字引數**字典 (Dict)**。
  - `daemon`：布林值。設定為 `True` 代表此執行緒為**守護執行緒**。
- **回傳值**：
  - `Thread`：執行緒物件實例。

```python
import threading
import time

def worker(name: str, delay: int):
    print(f"👷‍♂️ 工人 {name} 開始上工...")
    time.sleep(delay)
    print(f"✅ 工人 {name} 工作完成！")

# 1. 建立執行緒物件 (注意 args 必須為 tuple，若只有一個元素需加逗號)
t1 = threading.Thread(target=worker, args=("Alice", 1), name="Worker-A")
t2 = threading.Thread(target=worker, args=("Bob", 2), name="Worker-B")
```

---

##### .start() 與 .join() 啟動與阻塞等待

- **使用時機**：
  - `.start()`：告訴作業系統「該工人已就緒，可以開始執行了」。
  - `.join()`：讓主執行緒（MainThread）停下來，**卡在原地等待該子執行緒執行結束**才繼續往下走。
- **語法**：`.start()` / `.join(timeout=None)`
- **參數說明**：
  - `timeout`：（僅限 `join`）最大等待秒數。若超時子執行緒未結束，主執行緒不再等待直接繼續往下走。
- **回傳值**：
  - `None`。

```python
import threading
import time

def task(n):
    time.sleep(1)
    print(f"任務 {n} 完成")

# 1. 建立並啟動多個執行緒
threads = []
for i in range(3):
    t = threading.Thread(target=task, args=(i,))
    threads.append(t)
    t.start()  # 啟動工人！

print("🚀 所有工人已出發，主程式正在等待他們全部收工...")

# 2. 迴圈等待所有工人結束 (join)
for t in threads:
    t.join()

print("🎉 所有工人全部收工，主程式安全退出！")
```

---

##### .daemon 守護執行緒屬性

- **使用時機**：當你需要在背景運行一個輔助任務（如心跳檢測、背景監聽、自動存檔），且希望**當主程式結束時，這些背景工人自動被強迫結束（不要卡住程式退出）**時使用。
- **語法**：`.daemon = True` 或在建立時指定 `Thread(..., daemon=True)`
- **核心特性**：
  - **非守護執行緒 (`daemon=False`，預設)**：主程式跑完了，只要還有任何一個非守護子執行緒在跑，**整個 Python 進程就會死等它做完才肯退出**！
  - **守護執行緒 (`daemon=True`)**：只要所有非守護執行緒（包括 MainThread）結束，**所有守護執行緒會被作業系統瞬間抹殺，程式立即退出**！

```python
import threading
import time

def background_monitor():
    while True:
        print("💓 背景監控中...")
        time.sleep(0.5)

# 設定為守護執行緒
t = threading.Thread(target=background_monitor, daemon=True)
t.start()

# 主程式只睡 1.5 秒
time.sleep(1.5)
print("🚪 主程式結束！守護執行緒會被自動瞬間關閉！")
# 程式直接結束，背景迴圈不再繼續印出
```

---

## 2. 執行緒同步與防撞鎖機制 (Locks & Synchronization)

### 資料競爭 (Race Condition)

> **💡 核心觀念**：當多個執行緒同時對同一個全域變數進行 `count += 1` 時，因為 `+=` 在底層其實包含了「讀取 ➔ 加一 ➔ 寫回」三個微小步驟，多個執行緒交錯執行會把彼此的結果覆蓋掉，導致最終計算結果完全錯誤！

```python
# ❌ 無鎖情況下的資料競爭災難：
count = 0
def add_one():
    global count
    for _ in range(100000):
        count += 1

threads = [threading.Thread(target=add_one) for _ in range(10)]
for t in threads: t.start()
for t in threads: t.join()
print(count)  # 預期 1,000,000，但實際印出可能只有 421589！
```

---

##### threading.Lock() 互斥鎖 (Mutex)

- **使用時機**：保護關鍵共享資源（臨界區 Critical Section），確保同一瞬間**只有一個執行緒**能夠讀寫該資源。
- **語法**：`lock = threading.Lock()` 搭配 `with lock:` 語法使用。
- **核心方法**：
  - `.acquire(blocking=True, timeout=-1)`：嘗試上鎖（若已被別人鎖住會卡住等待）。
  - `.release()`：解鎖釋放。
- **回傳值**：
  - `Lock` 物件。

```python
import threading

count = 0
lock = threading.Lock()  # 1. 建立互斥鎖

def safe_add():
    global count
    for _ in range(100000):
        # 2. 透過 with lock 自動獲取鎖並保證釋放 (最佳實踐 ⭐)
        with lock:
            count += 1

threads = [threading.Thread(target=safe_add) for _ in range(10)]
for t in threads: t.start()
for t in threads: t.join()

print(f"✅ 安全加總結果: {count}")  # 輸出: 1000000 (100% 精準！)
```

> **💡 小提醒**：永遠優先使用 `with lock:` 語法，千萬不要手動 `lock.acquire()` / `lock.release()`，否則一旦中間程式碼拋出例外崩潰，鎖永遠無法釋放，引發全域死鎖！

---

##### threading.RLock() 可重入互斥鎖 (Recursive Lock)

- **使用時機**：在同一個執行緒中，同一個工人可能會**多次進入需要上鎖的函式**（例如遞迴函式，或方法 A 呼叫了同樣帶有鎖的方法 B）。
- **為什麼普通 Lock 會死鎖？**：普通 `Lock` 不認得是誰上鎖的。如果工人甲自己鎖了門，接著又走進裡面的房間嘗試再上同一把鎖，工人甲就會**自己被自己鎖死在門外**！
- **RLock 機制**：`RLock` 會記錄「當前持鎖的執行緒 ID」與「上鎖次數計數器」。同一個工人可以重複上鎖 N 次，只要最後解鎖 N 次即可。

```python
import threading

rlock = threading.RLock()

def recursive_job(n):
    with rlock:
        print(f"進入第 {n} 層")
        if n > 0:
            recursive_job(n - 1)  # 同一個執行緒再次進入 with rlock，完全不會卡死！

t = threading.Thread(target=recursive_job, args=(3,))
t.start()
t.join()
```

---

##### threading.Event() 事件信號機制 (旗標廣播)

- **使用時機**：當你需要**「一個執行緒發布指令，所有等待中的執行緒同時開工/停止」**時使用（例如：紅綠燈交通管制、一鍵暫停/恢復背景服務）。
- **語法**：`event = threading.Event()`
- **核心方法**：
  - `.wait(timeout=None)`：將當前執行緒**卡住暫停**，直到該 Event 變成綠燈 (`set`)。
  - `.set()`：亮起綠燈（將旗標設為 `True`），同時喚醒所有正在 `wait()` 的執行緒。
  - `.clear()`：轉為紅燈（將旗標設為 `False`），後續來的執行緒繼續卡住。
  - `.is_set()`：查詢目前是綠燈還是紅燈（回傳布林值）。

```python
import threading
import time

traffic_event = threading.Event()  # 預設為紅燈 (False)

def car(name: str):
    print(f"🚗 車輛 {name} 抵達路口，等待綠燈...")
    traffic_event.wait()  # 卡住等待信號！
    print(f"💨 綠燈亮起！車輛 {name} 極速通過！")

for i in range(3):
    threading.Thread(target=car, args=(f"#{i+1}",)).start()

time.sleep(2)
print("🚦 交警按下了綠燈開關！")
traffic_event.set()  # 一鍵喚醒所有車輛！
```

---

##### threading.Semaphore() 計數信號標 (限流閥門)

- **使用時機**：限制同時訪問某項昂貴資源的**最大執行緒數量**（例如：資料庫連線池最多只允許 5 個連線、外部 API 限制最大 3 個並發）。
- **語法**：`sem = threading.Semaphore(value=3)`
- **機制**：內部維護一個計數器。每次進入 `with sem` 計數器減 1；當計數器歸零時，後續的執行緒必須排隊等待其他執行緒離開。

```python
import threading
import time

# 限制最多允許 2 個執行緒同時進入
sem = threading.Semaphore(2)

def worker(item_id: int):
    with sem:
        print(f"⚡ 工人 #{item_id} 正在使用昂貴資源...")
        time.sleep(1)
        print(f"🚪 工人 #{item_id} 離開資源")

for i in range(5):
    threading.Thread(target=worker, args=(i,)).start()
```

---

# 實戰除錯與核心天條

## 1. 參數 args 傳遞單一元素的逗號陷阱

> **⚠️ args 格式錯誤陷阱**：  
> `Thread(target=func, args=(10))` 傳入的不是元組，而是整數 `10`，會直接拋出 `TypeError: args must be an iterable`！  
> 若只有一個引數，**務必加上逗號**：`args=(10,)`。

---

## 2. 避免雙重鎖定引發死鎖 (Deadlock)

> **⚠️ 哲學家就餐死鎖警告**：  
> 當執行緒 A 拿了 `Lock_1` 正在等 `Lock_2`，同時執行緒 B 拿了 `Lock_2` 正在等 `Lock_1`，雙方會永久互相卡死！  
> **防禦準則**：所有執行緒在獲取多個鎖時，**必須保持嚴格相同的上鎖順序**！
