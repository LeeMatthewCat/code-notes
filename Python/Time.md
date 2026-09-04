# 概念與原理

## 什麼是 time 模組？

Python 內建標準庫中專門用於處理時間、日期轉換、程式碼暫停延遲與精準效能計時的**核心系統時間模組 (System Time Module)**。

在 Python 開發中，`time` 模組直接與底層 C 語言操作系統 API 接軌。不論是需要讓程式暫停等待（如 `time.sleep()`）、獲取電腦系統的 Unix 時間戳記（Timestamp），或是為程式碼測量執行速度與效能耗時（`time.perf_counter()`），`time` 模組都是最基礎且不可或缺的核心工具。

> **🍿 生動白話比喻**：  
> - `time.time()`：像是**「從 1970 紀元起算的萬年滴答秒數器」**，不管你在全世界哪裡，它都在記錄從 1970 年 1 月 1 日至今跑了多少秒！  
> - `time.sleep()`：像是**「裁判吹哨暫停比賽」**，讓程式碼原地凍結、面壁思過幾秒鐘，時間到了才准繼續往下走。  
> - `time.perf_counter()`：像是**「賽跑裁判手中的極速電子碼錶」**，專門用來精確測量兩段程式碼之間的短暫執行時間（精準度達微秒/奈秒級）！

---

# 核心 API 函數與參數字典對照表

| 函數名稱 | 回傳型別 | 預設/引數 | 函數詳細作用與說明 |
| :--- | :--- | :--- | :--- |
| **[[#time.time() 取得系統 Unix 時間戳記\|time.time()]]** | `float` | 無 | **取得當前 Unix 時間戳記**（1970 年 1 月 1 日 UTC 至今的累積秒數）。 |
| **[[#time.sleep() 程式暫停與執行延遲\|time.sleep()]]** | `None` | `secs` (`float`/`int`) | **暫停執行**。讓當前執行緒原地凍結指定秒數（支援小數點如 `0.5` 秒）。 |
| **[[#time.perf_counter() 精準測量程式碼執行耗時\|time.perf_counter()]]** | `float` | 無 | **高精確度效能量測碼錶**。專門用於測量程式碼區塊的執行耗時 (Benchmark)。 |
| **[[#time.strftime() 格式化時間字串\|time.strftime()]]** | `str` | `format`, `t` | **將時間格式化為可讀字串**（如 `"%Y-%m-%d %H:%M:%S"`）。 |
| **`time.localtime()`** | `struct_time` | `secs` (可選) | 將 Unix 時間戳記轉換為本地時區的結構化時間物件。 |

---

# 核心功能與語法大解密

##### time.sleep() 程式暫停與執行延遲

- **使用時機**：讓程式執行暫停指定的秒數（常用於輪詢爬蟲、控制 API 呼叫頻率或動畫間隔）。
- **語法**：`time.sleep(secs)`

```python
import time

print("🚀 任務開始...")
# 讓程式碼原地暫停 1.5 秒
time.sleep(1.5)
print("✅ 1.5 秒後任務完成！")
```

> **⚠️ `time.sleep()` 阻塞主執行緒警告**：  
> `time.sleep()` 會徹底凍結**當前執行緒 (Thread)**。若在 GUI 介面（如 PyQt/Tkinter）或 Rich 畫面動態刷新 (`with Live(...)`) 區塊內過度呼叫，會導致 UI 無回應或畫面卡死！

---

##### time.time() 取得系統 Unix 時間戳記

- **使用時機**：獲取從 Unix 紀元（1970 年 1 月 1 日 00:00:00 UTC）開始計算的累積浮點數秒數（常用於資料庫時間戳儲存、Token 過期判定）。
- **語法**：`timestamp = time.time()`

```python
import time

# 取得當前時間戳記
timestamp = time.time()
print(f"當前 Unix 時間戳記: {timestamp}")  # 輸出: 1774430761.123456
```

---

##### time.perf_counter() 精準測量程式碼執行耗時

- **使用時機**：專門用於計算程式碼區塊的執行時間（不受系統校時/修改電腦時間干擾）。
- **語法**：`start = time.perf_counter()` / `end = time.perf_counter()`

```python
import time

# 1. 按下碼錶起點
start_time = time.perf_counter()

# 執行一段耗時的演算法
total = sum(i ** 2 for i in range(1000000))

# 2. 按下碼錶終點
end_time = time.perf_counter()

# 計算執行耗時
elapsed = end_time - start_time
print(f"⏱️ 運算耗時: {elapsed:.6f} 秒")  # 輸出: ⏱️ 運算耗時: 0.045123 秒
```

> **💡 效能量測首選 `perf_counter()`**：  
> 測量程式碼效能時，**嚴禁使用 `time.time()` 相減**！因為 `time.time()` 會受電腦調整時間（如網路校時 NTP）影響；而 `time.perf_counter()` 是獨立單調的極高精度碼錶，結果最準確！

---

##### time.strftime() 格式化時間字串

- **使用時機**：將結構化時間轉換為人類可讀的格式化字串。
- **語法**：`time_str = time.strftime(format, t)`

```python
import time

# 取得本地結構化時間，並轉成 "%Y-%m-%d %H:%M:%S" 格式
current_str = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())
print(f"📅 當前本地時間: {current_str}")  # 輸出: 📅 當前本地時間: 2026-07-25 17:26:01
```

> **💡 格式化代碼速查指南**：  
> - `%Y`：四位數年份 (2026)  
> - `%m`：二位數月份 (01-12)  
> - `%d`：二位數日期 (01-31)  
> - `%H`：24 小時制小時 (00-23)  
> - `%M`：二位數分鐘 (00-59)  
> - `%S`：二位數秒數 (00-59)
