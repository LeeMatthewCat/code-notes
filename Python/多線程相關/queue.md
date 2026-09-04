# 概念與原理

## 什麼是 queue 模組？

Python 官方標準庫中專為**多執行緒安全通訊 (Thread-Safe Communication)** 設計的核心資料結構模組（無需 `pip install`）。

在多執行緒程式設計中，多個工人（Thread）之間經常需要互相傳遞資料與分派任務。

`queue` 模組提供了自帶防撞鎖機制的先進先出 (FIFO)、後進先出 (LIFO) 與優先權 (Priority) 佇列，是建構**「生產者-消費者模式 (Producer-Consumer Pattern)」**的標準核心武器！

> **🍿 生動白話比喻**：  
> - **普通列表 `list`**：像一張**「毫無管制的公共桌子」**。多個工人同時伸手搶著放東西或拿東西，容易撞成一團、打翻盤子（引發資料錯亂或程式崩潰）。  
> - **`queue.Queue`**：像一條**「自帶紅綠燈與安全感應器的防撞輸送帶 🛤️」**。當輸送帶滿了，放貨的人自動停下排隊；當輸送帶空了，拿貨的人自動閉眼等待貨物抵達。完全不需要手動寫鎖！

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

## 為什麼不能直接用 Python 原生 list 代替？

| 比較維度 | Python 原生 `list` | `queue.Queue` (執行緒安全佇列) |
| :--- | :--- | :--- |
| **執行緒安全性** | ❌ **非執行緒安全 (Thread-Unsafe)**<br>多個執行緒同時 `append` 或 `pop` 會引發資料競爭 | 💯 **100% 執行緒安全 (Thread-Safe)**<br>底層內建互斥鎖與條件變數，絕不打架 |
| **資料為空時的行為** | 呼叫 `pop()` 直接拋出 `IndexError` 崩潰 | 自動**掛起並阻塞等待**，直到有新貨物被放入為止 |
| **容量限制與反壓** | 記憶體無限增長，容易塞爆記憶體 (OOM) | 支援 `maxsize` 上限，滿載時自動讓生產者暫停 |
| **任務完成追蹤** | 無追蹤機制，無法得知一批任務何時全做完 | 內建 `task_done()` 與 `join()` 計數信號機制 |

---

# 核心四大佇列結構對照表

| 類別名稱 | 資料結構特性 | 取貨順序規則 | 典型適用情境 |
| :--- | :--- | :--- | :--- |
| **`queue.Queue`** | **FIFO (先進先出)** | **先放進去的資料最先被拿出來**（如同排隊買票） | 絕大多數生產者-消費者流水線、任務排隊 |
| **`queue.LifoQueue`** | **LIFO (後進先出 / 堆疊 Stack)** | **最後放進去的資料最先被拿出來**（如同疊盤子） | 深度優先搜尋 (DFS)、瀏覽器歷史上一頁復原 |
| **`queue.PriorityQueue`** | **優先權佇列 (最小堆積 Heap)** | **優先級數字越小（越重要）的越先被拿出來** | VIP 緊急任務插隊、路徑搜尋演算法 (A*) |
| **`queue.SimpleQueue`** | **簡化無上限 FIFO 佇列** | 先進先出，底層使用 C 語言加速 (Python 3.7+) | 極致效能追求、不需要容量上限與任務追蹤時 |

---

# 核心 API 方法字典對照表

| 方法名稱 | 阻塞特性 | 主要用途與功能說明 |
| :--- | :--- | :--- |
| **[[#q.put() 放入資料至佇列\|q.put(item)]]** | 預設阻塞 | **將資料放入佇列**。若佇列已滿 (`maxsize`)，會卡住等待直到有空位。 |
| **[[#q.get() 從佇列取出資料\|q.get()]]** | 預設阻塞 | **從佇列取出並移除一筆資料**。若佇列為空，會卡住等待直到有新資料。 |
| **[[#q.put() 放入資料至佇列\|q.put_nowait(item)]]** | 非阻塞 | 嘗試放入資料。若已滿**立即拋出 `queue.Full` 例外**。 |
| **[[#q.get() 從佇列取出資料\|q.get_nowait()]]** | 非阻塞 | 嘗試取出資料。若為空**立即拋出 `queue.Empty` 例外**。 |
| **[[#q.task_done() 與 q.join() 工作流收尾\|q.task_done()]]** | 無 | 消費者處理完一筆資料後呼叫，將內部未完成任務計數器減 1。 |
| **[[#q.task_done() 與 q.join() 工作流收尾\|q.join()]]** | 阻塞 | **卡住等待直到佇列中所有任務全部被 `task_done()` 標記完成**。 |
| **`q.qsize()`** | 無 | 回傳當前佇列中的大致元素數量（⚠️ 多執行緒下僅供參考）。 |
| **`q.empty()`** | 無 | 檢查佇列當下是否為空（⚠️ 不可靠，隨時可能被其他執行緒改變）。 |
| **`q.full()`** | 無 | 檢查佇列當下是否已滿（⚠️ 不可靠）。 |

---

# 核心功能與語法大解密

## 1. 佇列基本讀寫與阻塞控制

##### q.put() 放入資料至佇列

- **使用時機**：生產者執行緒將產出的資料放入輸送帶時使用。
- **語法**：`q.put(item, block=True, timeout=None)`
- **參數說明**：
  - `item`：要存入的任意 Python 物件。
  - `block`：布林值，預設為 `True`。當佇列已滿時是否阻塞等待。
  - `timeout`：最大等待秒數。若設為正數且超時仍無空位，拋出 `queue.Full` 例外。
- **回傳值**：
  - `None`。

```python
import queue

# 建立一個容量上限為 2 的佇列
q = queue.Queue(maxsize=2)

q.put("工作 A")
q.put("工作 B")
print("✅ 成功放入 2 個工作")

# 第 3 個工作因為已滿，若無人取走會卡住等待 1 秒後報錯
try:
    q.put("工作 C", block=True, timeout=1)
except queue.Full:
    print("⚠️ 佇列已滿，無法再放入新工作！")
```

---

##### q.get() 從佇列取出資料

- **使用時機**：消費者執行緒從輸送帶拿取資料進行處理時使用。
- **語法**：`q.get(block=True, timeout=None)`
- **參數說明**：
  - `block`：布林值，預設為 `True`。當佇列為空時是否掛起等待。
  - `timeout`：最大等待秒數。若設為正數且超時仍無資料，拋出 `queue.Empty` 例外。
- **回傳值**：
  - 取出的 Python 物件。

```python
import queue

q = queue.Queue()
q.put("包裹 1")

item = q.get()
print(f"📦 取出包裹: {item}")  # 輸出: 📦 取出包裹: 包裹 1

# 若嘗試從空佇列拿取，設定超時避免永久卡死
try:
    q.get(block=True, timeout=1.5)
except queue.Empty:
    print("⚠️ 輸送帶目前空無一物！")
```

---

## 2. 任務追蹤與工作量計數 (Task Tracking)

##### q.task_done() 與 q.join() 工作流收尾

- **使用時機**：當主執行緒丟了一大批工作進佇列後，需要**卡住等待直到所有工人把每項工作全部消化完畢**時使用。
- **運作核心機制**：
  1. 每次呼叫 `q.put()`，內部計數器 **`unfinished_tasks` 自動加 1**。
  2. 工人每處理完一筆資料並呼叫 `q.task_done()`，計數器 **自動減 1**。
  3. `q.join()` 會一直卡住等待，**直到計數器歸零（所有工作全部完成）才放行**！

```python
import queue
import threading
import time

q = queue.Queue()

def worker():
    while True:
        item = q.get()
        print(f"⚙️ 正在處理: {item}...")
        time.sleep(0.5)
        
        # ⚠️ 關鍵：處理完畢必須回報 task_done()！
        q.task_done()

# 啟動 2 個守護工人
for _ in range(2):
    threading.Thread(target=worker, daemon=True).start()

# 主程式塞入 4 筆任務
for i in range(4):
    q.put(f"任務 #{i+1}")

print("⏳ 主程式等待所有任務被消化完畢...")
q.join()  # 阻塞卡住，直到 4 個任務全部被 task_done() 標記完成！
print("🎉 所有佇列任務全部完成！安全收工！")
```

---

## 3. 特殊佇列結構實戰

##### queue.PriorityQueue() 優先權佇列

- **使用時機**：任務具備輕重緩急之分（如 VIP 請求、緊急警報），需要**「插隊」優先處理**時使用。
- **語法與規則**：
  - 存入格式建議使用元組：`q.put((優先級數字, 資料內容))`。
  - **優先級數字越小，越先被取出**（例如 `1` 比 `10` 更優先）。

```python
import queue

pq = queue.PriorityQueue()

# 存入 (優先級數字, 資料)
pq.put((3, "一般郵件"))
pq.put((1, "🔥 伺服器火警警報！"))
pq.put((2, "VIP 客戶請求"))

# 依照優先順序取出 (數字小 ➔ 數字大)
while not pq.empty():
    priority, msg = pq.get()
    print(f"[{priority}] 優先處理: {msg}")

# 輸出順序:
# [1] 優先處理: 🔥 伺服器火警警報！
# [2] 優先處理: VIP 客戶請求
# [3] 優先處理: 一般郵件
```

---

# 經典架構：生產者-消費者模式實戰 (Producer-Consumer)

在標準工程實踐中，如何讓多個生產者、多個消費者與佇列完美協同，並**優雅安全停機 (Graceful Shutdown)**？

最經典的解法是使用**毒藥丸信號 (Poison Pill，即傳入 `None` 作為停機密碼)**：

```python
import queue
import threading
import time

# 1. 建立容量有限的輸送帶 (防止記憶體塞爆)
task_queue = queue.Queue(maxsize=5)

# 2. 定義生產者 (Producer)：抓取資料並塞入佇列
def producer(prod_id: int):
    for i in range(3):
        item = f"資料 P{prod_id}-{i}"
        task_queue.put(item)
        print(f"🏭 生產者 #{prod_id} 產出: {item}")
        time.sleep(0.3)

# 3. 定義消費者 (Consumer)：從佇列拿取資料並處理
def consumer(cons_id: int):
    while True:
        item = task_queue.get()
        # 收到「毒藥丸 None」密碼，代表任務全部結束，工人主動下班退出！
        if item is None:
            task_queue.task_done()
            print(f"🚪 消費者 #{cons_id} 收到停機信號，安全下班！")
            break
        
        print(f"👷‍♂️ 消費者 #{cons_id} 處理完: {item}")
        time.sleep(0.5)
        task_queue.task_done()

# 4. 啟動 2 個生產者與 2 個消費者
producers = [threading.Thread(target=producer, args=(i,)) for i in range(2)]
consumers = [threading.Thread(target=consumer, args=(i,)) for i in range(2)]

for t in consumers: t.start()
for t in producers: t.start()

# 等待所有生產者生產完畢
for t in producers: t.join()

# 等待佇列中現有貨物全部被消化完畢
task_queue.join()

# 5. 向佇列投入等同於消費者數量的「毒藥丸 None」，通知所有消費者下班
for _ in range(len(consumers)):
    task_queue.put(None)

# 等待所有消費者安全關閉
for t in consumers: t.join()
print("🎉 整條工廠流水線安全優雅關閉完畢！")
```

---

# 實戰除錯與核心天條

## 1. 誤信 q.empty() 與 q.full() 造成的競態條件

> **⚠️ empty() 檢查後被搶先的幽靈陷阱**：  
> 在多執行緒環境下，`if not q.empty(): data = q.get()` 是**非常危險的程式碼**！  
> 因為在你剛判斷完 `not q.empty()` 的百萬分之一秒內，另一個執行緒可能剛好把最後一個元素拿走了，導致下一行 `q.get()` 直接卡死！  
> **正確寫法**：直接使用 `try...except` 搭配 `get_nowait()`：
> ```python
> try:
>     data = q.get_nowait()
> except queue.Empty:
>     print("目前沒貨")
> ```

---

## 2. task_done() 呼叫次數必須與 put() 嚴格 1:1

> **⚠️ 計數器崩潰警告**：  
> - 如果少呼叫了 `task_done()`：`q.join()` 會**永久卡死**，因為計數器永遠無法歸零！  
> - 如果多呼叫了 `task_done()`：Python 會直接拋出 `ValueError: task_done() called too many times` 崩潰！
