# 概念與原理

## 什麼是 concurrent.futures 模組？

Python 官方標準庫中專為**高階非同步並行 (Concurrency) 與平行運算 (Parallelism)** 設計的核心模組（無需 `pip install`）。

在 Python 開發中，傳統手動管理多執行緒 (`threading.Thread`) 或多行程 (`multiprocessing.Process`) 語法繁瑣，且容易引發死鎖、資源洩漏或程式卡死。

`concurrent.futures` 引入了現代化的**「任務池 (Executor)」**與**「期約物件 (Future)」**概念，將複雜的執行緒與行程排程抽象化為極度簡潔的高階介面！

> **🍿 生動白話比喻**：  
> - **傳統手動多執行緒 (`threading`)**：像你自己當工頭，**手動去招聘 10 個工人、手動分配工作、還要一個個盯著他們做完**，很容易手忙腳亂。  
> - **`concurrent.futures` (任務池 Executor)**：像你找了一家**「專業外包工程隊」**。你只要把整份工作清單（例如 100 個網址）扔給外包隊長，隊長會自動調派固定數量的工人平行去做，做完自動把成果（Future 領取憑證）交還給你！

---

## 核心雙雄對決：ThreadPoolExecutor vs ProcessPoolExecutor

Python 存在全域直譯器鎖 (GIL, Global Interpreter Lock)。選擇執行緒池還是行程池，取決於你的任務類型：

| 比較維度 | ThreadPoolExecutor (多執行緒池) | ProcessPoolExecutor (多行程/多進程池) |
| :--- | :--- | :--- |
| **底層架構** | 共享同一個 Python 行程的記憶體空間 | 建立多個獨立的 Python 行程 (Process) |
| **GIL 影響** | 受到 GIL 限制（同一時刻只有一個 CPU 核心執行 Python Bytecode） | **徹底繞過 GIL 限制**（各進程各持獨立 GIL，真正吃滿多核 CPU） |
| **記憶體開銷** | 極小（輕量級，快速建立與切換） | 較大（每個行程都要複製獨立的 Python 直譯器環境） |
| **資料共享** | 容易（共享全域變數，但需注意執行緒安全） | 需透過 IPC 序列化傳遞（所有參數與回傳值必須支援 `pickle`） |
| **最佳適用場景** | **I/O 密集型 (I/O-Bound)**<br>- 網路爬蟲 / 批量下載<br>- 調用第三方 API<br>- 檔案與資料庫大量讀寫 | **CPU 密集型 (CPU-Bound)**<br>- 高解析度圖片/影音壓縮轉檔<br>- 複雜數學矩陣運算<br>- 大規模資料加密與雜湊運算 |

> **💡 一句話黃金選型口訣**：  
> **「等網路、等檔案用 Thread；算數學、吃 CPU 用 Process！」**

---

# 核心 API 函數與類別字典對照表

| 類別 / 函式名稱 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- |
| **[[#ThreadPoolExecutor() 多執行緒池\|ThreadPoolExecutor()]]** | 建立**多執行緒任務池** | 處理大量網路請求、API 呼叫與檔案 I/O |
| **[[#ProcessPoolExecutor() 多行程池\|ProcessPoolExecutor()]]** | 建立**多行程任務池** | 處理多核心密集運算、影像轉檔與繁重計算 |
| **[[#executor.submit() 提交單一任務\|executor.submit()]]** | **非同步提交單一任務**，立即回傳 `Future` 物件 | 任務複雜度不一、需要個別獲取回傳值或監控狀態時 |
| **[[#executor.map() 批量映射任務\|executor.map()]]** | **批量映射任務**，按原順序回傳結果產生器 | 將同一個函式平行套用到一個大清單的所有元素上 |
| **[[#executor.shutdown() 關閉任務池與資源清理\|executor.shutdown()]]** | **手動關閉任務池並釋放系統資源** | 未使用 `with` 語句、常駐背景服務結束時手動收尾 |
| **[[#as_completed() 即時處理先完成的任務\|as_completed()]]** | **「誰先算完就先拿誰」**的產生器 | 即時接收多個任務的回傳結果，無須傻等最慢的那個 |
| **[[#wait() 集合等待與檢查點\|wait()]]** | **阻塞等待一組 Future 物件**滿足特定條件（如全部完成或第一個完成） | 需要設置檢查點 (Checkpoint) 統一收攏進度時 |
| **[[#Future 期約物件\|Future]]** | 代表非同步運算的**期約物件 (未來的回傳結果憑證)** | 檢查任務是否完成 (`.done()`)、取消 (`.cancel()`) 或取值 (`.result()`) |

---

# 核心功能與語法大解密

## 1. 任務池管理與任務提交

##### ThreadPoolExecutor() 多執行緒池

- **使用時機**：執行大量 I/O 密集型操作（如並發呼叫 API、批量抓取網頁）時，搭配 `with` 語句自動管理執行緒的生命週期與安全關閉。
- **語法**：`ThreadPoolExecutor(max_workers=None, thread_name_prefix='', ...)`
- **參數說明**：
  - `max_workers`：最大工作執行緒數量。預設為 `min(32, (os.cpu_count() or 1) + 4)`。
  - `thread_name_prefix`：執行緒名稱前綴，方便日誌除錯。
- **回傳值**：
  - `ThreadPoolExecutor`：任務池執行器實例。

```python
from concurrent.futures import ThreadPoolExecutor
import time

def download_site(url: str) -> str:
    time.sleep(1)  # 模擬網路請求耗時 1 秒
    return f"{url} 下載完成！"

# 1. 透過 with 語句安全建立執行緒池 (自動呼叫 shutdown 等待所有任務收尾)
with ThreadPoolExecutor(max_workers=3) as executor:
    # 2. 提交任務
    future = executor.submit(download_site, "https://example.com")
    print("已提交任務，繼續執行其他工作...")
    
    # 3. 獲取結果 (阻塞等待直到該任務完成)
    print(future.result())
```

---

##### ProcessPoolExecutor() 多行程池

- **使用時機**：執行 CPU 密集型操作（如大量數字運算、影像轉檔），希望吃滿多核心 CPU 算力時使用。
- **語法**：`ProcessPoolExecutor(max_workers=None, ...)`
- **參數說明**：
  - `max_workers`：最大工作行程數量。預設為電腦的實體 CPU 核心數 (`os.cpu_count()`)。
- **回傳值**：
  - `ProcessPoolExecutor`：多行程任務池實例。

```python
from concurrent.futures import ProcessPoolExecutor
import time

def cpu_heavy_task(n: int) -> int:
    # 模擬複雜計算
    return sum(i * i for i in range(n))

# ⚠️ Windows / macOS 多行程程式碼必須放在 if __name__ == '__main__': 保護區內！
if __name__ == "__main__":
    with ProcessPoolExecutor(max_workers=4) as executor:
        future = executor.submit(cpu_heavy_task, 10_000_000)
        print(f"計算結果: {future.result()}")
```

> **⚠️ 多行程 Windows / macOS 崩潰陷阱 (`__main__` 保護)**：  
> 在 Windows 與 macOS (spawn 模式) 下使用 `ProcessPoolExecutor`，**絕對必須**將程式碼放在 `if __name__ == '__main__':` 區塊內！否則子行程在複製啟動時會無限遞迴重新載入主程式，導致記憶體瞬間塞滿並當機！

---

##### executor.submit() 提交單一任務

- **使用時機**：當你需要非同步派發單一任務，並想要取得一張「提貨券 (`Future`)」以便稍後檢查狀態、取得回傳值或處理個別例外時使用。
- **語法**：`executor.submit(fn, *args, **kwargs)`
- **參數說明**：
  - `fn`：要執行的目標函式。
  - `*args` / `**kwargs`：傳給目標函式的引數與關鍵字引數。
- **回傳值**：
  - `Future`：代表該次非同步運算的期約物件。

```python
from concurrent.futures import ThreadPoolExecutor

def calculate(a: int, b: int, operation: str = "add") -> int:
    return a + b if operation == "add" else a * b

with ThreadPoolExecutor() as executor:
    # 提交任務並傳入位置引數與關鍵字引數
    future = executor.submit(calculate, 10, 20, operation="multiply")
    
    # 透過 .result() 取回成果
    print(f"結果: {future.result()}")  # 輸出: 結果: 200
```

---

##### executor.map() 批量映射任務

- **使用時機**：當你有一個固定的目標函式，以及一個龐大的資料串列，想要用最簡單的一行語法進行平行運算，並**嚴格依照輸入順序**接收回傳值時使用。
- **語法**：`executor.map(fn, *iterables, timeout=None, chunksize=1)`
- **參數說明**：
  - `fn`：要平行套用的函式。
  - `*iterables`：可迭代資料序列（如串列）。若傳入多個，會如同內建 `map()` 一樣並行傳給 `fn`。
  - `timeout`：等待各元素回傳的最大秒數。
- **回傳值**：
  - 產生器 (Generator)：按**輸入順序**依序產出各任務的回傳值。

```python
from concurrent.futures import ThreadPoolExecutor
import time

def process_item(item_id: int) -> str:
    time.sleep(0.5)
    return f"項目 #{item_id} 處理完成"

items = [1, 2, 3, 4, 5]

with ThreadPoolExecutor(max_workers=5) as executor:
    # executor.map 直接平行處理整個串列，語法最精簡！
    results = executor.map(process_item, items)

# 依序走訪印出 (保證順序與 items 原順序一致)
for res in results:
    print(res)
```

> **💡 `submit()` vs `map()` 怎麼選？**  
> - **`executor.map()`**：適合**「大量同質性資料」**，語法極簡，保證結果按原本順序輸出。  
> - **`executor.submit()`**：適合**「異質性任務」**，可以個別設定不同函式與參數，且能搭配 `as_completed()` 做到「誰先做完就先處理誰」！

---

##### executor.shutdown() 關閉任務池與資源清理

- **使用時機**：當你**沒有使用 `with` 語句**（例如常駐背景服務或類別實例中），在所有任務派發完成後，需要通知任務池停止接受新任務並釋放系統執行緒/行程資源時手動呼叫。
- **語法**：`executor.shutdown(wait=True, cancel_futures=False)`
- **參數說明**：
  - `wait`：布林值，預設為 `True`。設為 `True` 時會**阻塞等待**所有正在運行與排隊中的任務全部執行完畢才返回；設為 `False` 會立即返回，任務會在背景繼續執行完畢後自行釋放。
  - `cancel_futures`：布林值，預設為 `False` (Python 3.9+ 引入)。設為 `True` 時會自動取消所有**尚未開始執行（排隊中）**的任務。
- **回傳值**：
  - `None`。

```python
from concurrent.futures import ThreadPoolExecutor
import time

def background_job(i: int):
    time.sleep(1)
    print(f"工作 #{i} 完成")

# 1. 手動建立執行緒池 (不使用 with)
executor = ThreadPoolExecutor(max_workers=2)

# 2. 提交任務
executor.submit(background_job, 1)
executor.submit(background_job, 2)

# 3. 手動關閉：不再接受新任務，並等待所有現有任務安全做完
executor.shutdown(wait=True)
print("所有背景工作安全收尾完畢！")
```

> **💡 小提醒**：一旦呼叫了 `shutdown()`，該 `executor` 就無法再透過 `.submit()` 提交新任務，否則會拋出 `RuntimeError: cannot schedule new futures after shutdown`！

---

## 2. 期約物件與任務狀態控制

##### Future 期約物件

- **使用時機**：由 `executor.submit()` 回傳，用來追蹤背景任務的執行進度、取消任務、捕獲例外以及取得最終結果。
- **核心方法與屬性**：

| 方法名稱 | 回傳型別 | 功能說明 |
| :--- | :--- | :--- |
| **`future.result(timeout=None)`** | `Any` | 取得任務的最終回傳值（若尚未完成會**阻塞等待**；若任務內部出錯會直接拋出該例外）。 |
| **`future.done()`** | `bool` | 非阻塞檢查。回傳 `True` 代表任務已執行完畢或已被取消。 |
| **`future.cancelled()`** | `bool` | 檢查任務是否成功被取消。 |
| **`future.cancel()`** | `bool` | 嘗試取消任務（**只有排隊中尚未開始執行的任務才能取消**，已在運行的任務無法取消）。 |
| **`future.exception(timeout=None)`** | `Exception` / `None` | 若任務執行出錯，回傳其拋出的例外物件；若正常完成則回傳 `None`。 |
| **`future.add_done_callback(fn)`** | `None` | 綁定回呼函式，當該任務完成時**自動觸發呼叫** `fn(future)`。 |

---

##### future.result() 獲取任務回傳值

- **使用時機**：當子執行緒/子行程任務執行完畢後，用來取出該任務的最終 return 回傳值。
- **語法**：`future.result(timeout=None)`
- **參數說明**：
  - `timeout`：最大等待秒數（預設為 `None` 無限等待）。若超過此時間任務仍未完成，會拋出 `TimeoutError`。
- **回傳值**：
  - 目標函式 `return` 的實際資料。若任務執行期間拋出未捕獲的例外，呼叫 `.result()` 會**重新在當前執行緒拋出該例外**！

```python
from concurrent.futures import ThreadPoolExecutor

def faulty_task():
    raise ValueError("資料庫連線失敗！")

with ThreadPoolExecutor() as executor:
    future = executor.submit(faulty_task)

# 1. 檢查任務是否出錯 (不拋出例外，安全取得錯誤物件)
err = future.exception()
if err:
    print(f"⚠️ 捕獲到任務內部例外: {err}")

# 2. 若直接呼叫 .result()，Python 會重新把子任務內部的例外拋出來
try:
    future.result()
except ValueError as e:
    print(f"❌ 捕獲到 result() 拋出的錯誤: {e}")
```

> **⚠️ 致命天條：嚴禁在迴圈內「剛 submit 就立刻呼叫 result()」！**  
> 這是初學者使用並行庫時 **90% 最常犯下的致命邏輯錯誤**！  
> 
> - ❌ **錯誤寫法 (並發效果瞬間歸零！退化成慢吞吞的循序單工執行)**：
>   ```python
>   urls = ["url_1", "url_2", "url_3", "url_4", "url_5"]
>   
>   # ❌ 錯誤示範：提交一個、立刻卡住等一個
>   with ThreadPoolExecutor(max_workers=5) as executor:
>       for url in urls:  # 假設 5 個網址各耗時 1 秒
>           future = executor.submit(download, url)
>           data = future.result()  # 🚨 致命卡死！主執行緒在此原地阻塞等待！
>   # 結果：總耗時依然是 5 秒！多執行緒完全沒發揮任何平行加速效果！
>   ```
> - **為什麼會失去並行？**：  
>   因為 `.result()` 是**阻塞操作 (Blocking)**。當你提交第 1 個任務後立刻呼叫 `.result()`，主執行緒就被定在原地等它做完，才會進入迴圈下一輪去提交第 2 個任務。雖然開了 5 個工人，但**同一時間永遠只有 1 個工人在做事**！
> 
> - ✅ **正確寫法 1 (兩階段：先全部投遞收集 Futures，再統一取值)**：
>   ```python
>   with ThreadPoolExecutor(max_workers=5) as executor:
>       # 1. 第一階段：一口氣把所有任務全部 submit 出去 (所有工人同時開工！)
>       futures = [executor.submit(download, url) for url in urls]
>       
>       # 2. 第二階段：等大家都開跑了，再迴圈讀取結果
>       results = [f.result() for f in futures]
>   # 結果：5 個任務同時平行執行，總耗時縮短為「1 秒」！
>   ```
> 
> - ✅ **正確寫法 2 (搭配 as_completed 動態即時取值，或直接用 executor.map)**：
>   ```python
>   # 方式 A：使用 as_completed 誰先好就先取誰
>   for f in as_completed(futures):
>       print(f.result())
>   
>   # 方式 B：使用 executor.map 最省事，內部自動全並行處理
>   results = list(executor.map(download, urls))
>   ```

---

## 3. 高級協調函式 (as_completed & wait)

##### as_completed() 即時處理先完成的任務

- **使用時機**：當你同時發出 100 個網路請求，各個請求耗時長短不一，你希望**「誰先下載完就立刻先處理誰」**，而不是傻傻等待最慢的那一個時使用。
- **語法**：`as_completed(fs, timeout=None)`
- **參數說明**：
  - `fs`：包含多個 `Future` 物件的可迭代容器（如 `list` 或 `dict`）。
  - `timeout`：等待的最大秒數。
- **回傳值**：
  - 迭代產生器：每次產出一個**已經完成 (Done)** 的 `Future` 物件。

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import time, random

def fetch_data(task_id: int) -> tuple[int, float]:
    delay = random.uniform(0.5, 2.0)
    time.sleep(delay)
    return task_id, delay

with ThreadPoolExecutor(max_workers=5) as executor:
    # 1. 建立任務字典 (以 Future 為鍵，對應自訂元數據)
    future_to_id = {executor.submit(fetch_data, i): i for i in range(1, 6)}
    
    # 2. 誰先完成就先產出誰 (動態即時反饋)
    for future in as_completed(future_to_id):
        task_id, delay = future.result()
        print(f"🎉 任務 #{task_id} 率先完成！(耗時 {delay:.2f} 秒)")
```

---

##### wait() 集合等待與檢查點

- **使用時機**：需要控制一組任務的執行節奏（例如等待「第一批任務全部完成」或「只要任何一個任務出錯/完成」就立刻繼續主流程）時使用。
- **語法**：`wait(fs, timeout=None, return_when=ALL_COMPLETED)`
- **參數說明**：
  - `fs`：包含 `Future` 物件的清單或集合。
  - `timeout`：最大等待秒數。
  - `return_when`：觸發解除阻塞的條件（常數）：
    - `ALL_COMPLETED` (預設)：等待**所有任務全部完成**。
    - `FIRST_COMPLETED`：只要**任何一個任務完成**就立刻返回。
    - `FIRST_EXCEPTION`：只要**任何一個任務拋出例外**就立刻返回。
- **回傳值**：
  - `tuple(done, not_done)`：回傳兩個集合，分別為「已完成的 Future 集合」與「未完成的 Future 集合」。

```python
from concurrent.futures import ThreadPoolExecutor, wait, FIRST_COMPLETED
import time

def fast_job():
    time.sleep(0.5)
    return "快速任務"

def slow_job():
    time.sleep(3.0)
    return "慢速任務"

with ThreadPoolExecutor() as executor:
    f1 = executor.submit(fast_job)
    f2 = executor.submit(slow_job)
    
    # 只要第一個任務完成就立刻放行
    done, not_done = wait([f1, f2], return_when=FIRST_COMPLETED)
    
    print(f"已完成數量: {len(done)}, 還在跑的數量: {len(not_done)}")
    for f in done:
        print(f"率先抵達: {f.result()}")
```

---

# 實戰除錯與核心天條

## 1. 忘記捕捉子執行緒內部例外 (Silent Exception)

> **⚠️ 子執行緒例外被吞噬陷阱**：  
> 當子執行緒內部發生崩潰報錯時，Python **不會在主主控台直接印出 Traceback 堆疊**，而是把例外靜悄悄地打包放進 `Future` 物件中！  
> 如果你提交任務後沒有呼叫 `future.result()` 或 `future.exception()`，這個錯誤就會徹底被吞噬，讓你誤以為程式執行完全正常！

---

## 2. 避免在 Worker 內部建立新的 Pool (防止資源耗盡)

> **⚠️ 嵌套任務池 (Nested Pools) 崩潰警告**：  
> 絕對不要在一個已經由 `ThreadPoolExecutor` 排程的函式內部，再去建立另一個 `ThreadPoolExecutor`！這會導致執行緒數量呈幾何級數暴增，迅速耗盡作業系統的 File Descriptors 與記憶體！

---

## 3. 為什麼用 with？以及為什麼包含 yield 的生成器「不適合」用 with？

### with 語句的使用時機與優勢

> **💡 核心觀念**：當你使用 `with ThreadPoolExecutor() as executor:` 時，Python 會在離開 `with` 縮排區塊的瞬間，**自動在底層執行 `executor.shutdown(wait=True)`**！

這帶來了三大核心保障：
1. **自動等待收尾**：確保所有子執行緒 100% 執行完畢，防止主程式提前退出導致背景工作丟失。
2. **防止資源洩漏**：避免忘記手動寫 `shutdown()` 產生常駐卡死的殭屍執行緒。
3. **異常安全防護 (Exception Safety)**：即使 `with` 區塊內部發生報錯崩潰，Python 也保證一定會執行 `shutdown()` 清理資源。

---

### 生成器 (Generator) 與 with 的相容性限制

> **⚠️ 核心天條**：包含 `yield` 的生成器函式「絕對不適合」在內部使用 `with`！

當一個函式內部包含 `yield` 關鍵字時，它就變成了**生成器 (Generator)**。生成器具備「惰性求值、隨產隨停」的串流特性。

若你在生成器內部使用 `with executor`，會引發嚴重的邏輯災難：

- **❌ 錯誤寫法 (生成器內部包 with 導致提前卡死或提早關閉)**：
  ```python
  from concurrent.futures import ThreadPoolExecutor

  # ❌ 錯誤示範：在生成器內部使用 with
  def stream_results(items):
      with ThreadPoolExecutor() as executor:
          futures = [executor.submit(do_work, x) for x in items]
          for f in futures:
              yield f.result()  # 只要產生一次 yield，若外部沒有一口氣消耗完...
      # 離開 with 時會自動觸發 shutdown(wait=True)，直接卡死主執行緒！
  ```

- **為什麼會出事？兩大痛點**：
  1. **阻塞卡死 (打破串流初衷)**：一旦生成器執行到 `with` 結尾，它會強制執行 `shutdown(wait=True)`，導致外部呼叫端被**同步卡死**等待所有任務做完，徹底破壞了串流「隨產隨用」的非同步特性！
  2. **提早關閉 (GeneratorExit 崩潰)**：如果外部呼叫者用 `for` 迴圈只取了前 2 個結果就 `break` 離開，生成器會被銷毀並提早離開 `with`，導致後續尚未運行的任務全部被強制中斷或引發 `RuntimeError`！

- **✅ 正確寫法：把任務池放在生成器「外部」管理**：

  **方式 A：函式等級——由外層呼叫端統一掌控 with 生命週期**
  ```python
  from concurrent.futures import ThreadPoolExecutor

  # 1. 生成器函式只單純接收外部已存在的 executor
  def stream_results(executor, items):
      futures = [executor.submit(do_work, x) for x in items]
      for f in futures:
          yield f.result()

  # 2. 由最外層呼叫端統一掌控 with 生命週期
  with ThreadPoolExecutor() as executor:
      for result in stream_results(executor, [1, 2, 3]):
          print(f"即時收到: {result}")
  ```

  **方式 B：類別等級 (OOP)——在 `__init__` 建立，並由類別管理關閉**
  > **💡 物件導向架構首選**：如果你的串流生成器是一個類別（如 `Agent`、`DataPipeline`），**在 `__init__` 中初始化 `self.executor` 是極為標準的業界設計模式**！

  ```python
  from concurrent.futures import ThreadPoolExecutor
  import time

  class TaskStreamService:
      def __init__(self, max_workers: int = 4):
          # 1. 在 __init__ 內部建立長駐任務池 (放在生成器外部)
          self.executor = ThreadPoolExecutor(max_workers=max_workers)

      def stream_process(self, tasks):
          # 2. 串流生成器方法：直接使用 self.executor 派發並 yield
          futures = [self.executor.submit(self._worker, t) for t in tasks]
          for f in futures:
              yield f.result()

      def _worker(self, item):
          time.sleep(0.5)
          return f"資料 {item} 處理完成"

      def close(self):
          # 3. 提供手動關閉介面
          self.executor.shutdown(wait=True)

      # 4. (可選) 實作上下文管理器，讓整個類別支援 with 語法
      def __enter__(self):
          return self

      def __exit__(self, exc_type, exc_val, exc_tb):
          self.close()

  # 使用示範 (優雅享受串流與自動關閉)：
  with TaskStreamService() as service:
      for item in service.stream_process(["A", "B", "C"]):
          print(f"✅ 串流收到: {item}")
  ```
