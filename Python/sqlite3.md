# 概念與原理

## 什麼是 sqlite3？

`sqlite3` 是 Python 內建的輕量級關聯式資料庫引擎標準函式庫（無需額外安裝，直接 `import sqlite3`）。

它包裝了全球廣泛應用的 C 語言 SQLite 嵌入式資料庫引擎，具備以下核心技術特徵：
- **無伺服器架構 (Serverless)**：不需要安裝、設定或啟動獨立的資料庫背景服務進程。
- **單一檔案持久化 (Single-file Storage)**：整個資料庫（包含結構定義、資料表、索引、觸發器與所有資料紀錄）完全封裝於本機磁碟的單一跨平台二進位檔案中。
- **完整 ACID 交易支援**：支援原子性 (Atomicity)、一致性 (Consistency)、隔離性 (Isolation) 與持久性 (Durability)。
- **動態型別親和性 (Type Affinity)**：不同於傳統嚴格靜態型別的資料庫，SQLite 採用型別親和性架構，支援五種核心儲存類別：`NULL`、`INTEGER`、`REAL`、`TEXT`、`BLOB`。
- **純記憶體模式**：支援建立於 RAM 中的記憶體資料庫 (`:memory:`)，提供極致的單元測試與快取速度。

> **[生動比喻]**：  
> - **大型關聯式資料庫 (如 PostgreSQL / MySQL)**：像「體制龐大的中央自來水廠與地下管線系統」，需要專職工程師維護伺服器、監聽通訊埠、管理連線池與帳號權限。  
> - **`sqlite3` 嵌入式資料庫**：像「隨身攜帶的高級保溫水壺」，零管線依賴、開箱即用、拎包就走。打開蓋子就能直接盛裝資料，檔案複製到隨身碟就能整庫帶著走！

---

## 為什麼需要 sqlite3？核心優勢與適用場景

- **桌面應用與命令列工具本機儲存**：儲存應用程式組態設定、使用者工作階段、快取快照與歷史紀錄。
- **自動化測試與單元測試 (Unit Tests)**：透過記憶體資料庫 (`:memory:`)，在幾毫秒內建立、測試並摧毀測試資料庫，完全無需 Mock 也不殘留磁碟垃圾。
- **嵌入式裝置與邊緣運算**：適用於資源受限環境（如物聯網設備、行動裝置本機離線資料快取）。
- **中小型網站與內部管理系統**：在讀取密集且寫入併發可控的場景下，搭配 WAL 模式可提供驚人的回應效能。

---

# 現代語法升級與版本演進對照表

Python 的 `sqlite3` 模組在不同版本中持續演進，特別是在 Python 3.12 引入了符合 PEP 249 的現代連線交易控制：

| 功能操作 | 舊式語法 (已棄用 / 傳統寫法) | 現代標準語法 (推薦最佳實踐) | 核心差異與說明 |
| :--- | :--- | :--- | :--- |
| **交易控制模式** | `isolation_level=""` (隱式交易模式) | `sqlite3.connect(..., autocommit=False)` | (Python 3.12+) 顯式遵循 PEP 249，消除 DDL 自動觸發 Commit 的隱性交易邊界混亂 |
| **自動認可模式** | `isolation_level=None` (自動認可) | `sqlite3.connect(..., autocommit=True)` | (Python 3.12+) 提供官方語意化參數宣告每句 SQL 自動認可 |
| **資料列查詢存取** | 純元組索引 `row[0]`, `row[1]` | `conn.row_factory = sqlite3.Row` 搭配 `row["name"]` | 支援字典式欄位鍵名存取，程式碼具備高可讀性與抗重構能力 |
| **交易安全控制** | 手動 `try...except...conn.commit()` | `with conn:` 交易情境管理器 | 區塊正常結束自動 commit，拋出例外自動 rollback |
| **連線資源管理** | 腳本末端手動 `conn.close()` | `with sqlite3.connect(...) as conn:` 連線情境管理器 | 結束時自動關閉連線，保證磁碟鎖安全釋放 |
| **批次資料寫入** | `for item in data: cur.execute(...)` | `cur.executemany(sql, data)` | 批次傳遞序列或生成器，大幅減少 Python 與 C 邊界的上下文切換開銷 |
| **多句腳本執行** | 多次呼叫 `execute()` | `cur.executescript(ddl_sql)` | 一次性原子化執行多句分號分隔之 DDL/DML 語句 |

---

# 核心 API 方法與屬性字典對照表

| 類別 / 函式 / 屬性名稱 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- |
| **[[#sqlite3.connect() 建立或開啟資料庫連線\|sqlite3.connect()]]** | **建立與本機磁碟或記憶體資料庫的通訊連線實例** | 應用程式初始化、建立測試記憶體庫、配置連線逾時 |
| **[[#with conn 交易情境管理器 (自動認可與回滾)\|with conn]]** | **將程式碼區塊包裹在單一交易中，實現自動 commit 與 rollback** | 銀行轉帳等多步驟金融運算、批次資料安全寫入 |
| **[[#with sqlite3.connect() as conn 連線生命週期管理器\|with sqlite3.connect() as conn]]** | **管理連線生命週期，離開區塊時自動關閉連線釋放資源** | 短生命週期批次任務、自動化測試腳本資源釋放 |
| **[[#conn.cursor() 建立游標物件\|conn.cursor()]]** | **建立用於下達 SQL 指令與遍歷查詢結果集的游標實例** | 多游標獨立查詢、管理結果集提取進度狀態 |
| **[[#conn.commit() 提交當前作用中交易\|conn.commit()]]** | **顯式確認並將交易內的所有資料變更永久寫入磁碟檔案** | 完成 INSERT/UPDATE/DELETE 資料持久化儲存 |
| **[[#conn.rollback() 回滾撤銷當前交易\|conn.rollback()]]** | **放棄當前交易內的所有未確認變更，還原至交易開始前狀態** | 捕捉到例外錯誤時還原資料狀態、確保資料一致性 |
| **[[#conn.close() 關閉連線並釋放檔案鎖\|conn.close()]]** | **關閉資料庫連線並釋放作業系統層級之磁碟檔案讀寫鎖** | 應用程式關閉前釋放資源、解除多進程檔案占用 |
| **[[#conn.execute() 與 conn.executemany() 連線層級便捷執行器\|conn.execute() / executemany()]]** | **略過手動建立 cursor()，直接透過連線物件執行 SQL 語句** | 簡化單次查詢或寫入代碼、簡潔鏈式語法 |
| **[[#conn.row_factory 設定資料列工廠\|conn.row_factory]]** | **自訂查詢結果列的回傳格式工廠 (如 sqlite3.Row 或自訂模型)** | 將查詢結果轉換為字典式物件、自訂 dataclass 轉換 |
| **[[#sqlite3.Row 字典式欄位存取資料列物件\|sqlite3.Row]]** | **提供同時支援「欄位名稱鍵名」與「數值索引」的高效資料列封裝** | 取代純元組查詢回傳、提高 ORM 與資料存取可讀性 |
| **[[#cursor.execute() 執行單句 SQL 指令\|cursor.execute()]]** | **執行單句具備參數化佔位符的 DDL 或 DML 語句** | 建立表格、單筆資料新增、條件資料查詢 |
| **[[#cursor.executemany() 批次參數化執行 SQL 指令\|cursor.executemany()]]** | **批次多次套用參數序列並高效執行同一句 SQL 範本** | 匯入大量 CSV 資料、批次批次建立關聯紀錄 |
| **[[#cursor.executescript() 執行多句腳本語句\|cursor.executescript()]]** | **一次性解析並連續執行包含分號分隔的多句 SQL 腳本字串** | 資料庫初始化腳本、批次結構遷移、執行 SQL Dump |
| **[[#cursor.fetchone() 提取結果集單筆紀錄\|cursor.fetchone()]]** | **從查詢結果集提取下一筆單一行紀錄** | 根據唯一主鍵查詢單一使用者、檢查紀錄是否存在 |
| **[[#cursor.fetchmany() 提取結果集指定筆數清單\|cursor.fetchmany()]]** | **依據指定數量批次提取結果集中的前 N 筆紀錄清單** | 串流分頁處理龐大資料集、避免一次載入過大記憶體 |
| **[[#cursor.fetchall() 提取結果集所有紀錄清單\|cursor.fetchall()]]** | **一次性將查詢結果集內剩餘的所有紀錄全數提取為列表** | 讀取全表資料、小型資料集報表聚合 |
| **[[#cursor.lastrowid 獲取最後插入列之主鍵 ID\|cursor.lastrowid]]** | **獲取當前連線最後一次 INSERT 產生的自動遞增整數主鍵值** | 建立主表紀錄後取得 ID 用於寫入子關聯表 |
| **[[#cursor.rowcount 獲取最後執行受影響之行數\|cursor.rowcount]]** | **獲取前一次 DML 語句所修改、刪除或新增的總資料行數** | 驗證 UPDATE/DELETE 是否確實命中目標紀錄 |
| **[[#cursor.description 獲取結果集欄位元數據\|cursor.description]]** | **獲取當前查詢結果集中各個欄位的結構元數據元組清單** | 動態取得資料表欄位名稱清單、自訂格式化輸出 |
| **[[#conn.create_function() 註冊自訂 SQL 函式\|conn.create_function()]]** | **向 SQLite 核心註冊可在 SQL 語法中直接調用的 Python 純量函式** | 在 SQL 內部進行正則表達式比對、特殊字串清洗或密碼雜湊 |
| **[[#conn.backup() 資料庫線上即時備份\|conn.backup()]]** | **在資料庫保持運作狀態下，執行非阻塞的高效線上二進位備份** | 將記憶體資料庫儲存至磁碟、線上熱備份至副本庫 |
| **[[#PRAGMA 指令與 WAL 高效寫入模式\|PRAGMA]]** | **執行 SQLite 專屬的底層運作設定、效能調校與結構檢查指令** | 啟用外鍵約束檢查、開啟 WAL 寫入預寫日誌模式提高併發 |
| **[[#sqlite3.Error 模組例外階層大腦\|sqlite3.Error]]** | **sqlite3 模組所有例外類別的基底類別 (包含 IntegrityError 等)** | 捕捉主鍵衝突、語法錯誤或鎖表例外並進行安全復原 |

---

# 1. 連線管理與交易架構 (Connection & Transactions)

##### sqlite3.connect() 建立或開啟資料庫連線

- **使用時機**：在程式啟動或工作單元開始時，開啟磁碟上的 SQLite 檔案或建立暫時的記憶體資料庫。
- **語法**：
  ```python
  sqlite3.connect(
      database,
      timeout=5.0,
      detect_types=0,
      isolation_level="",
      check_same_thread=True,
      factory=Connection,
      cached_statements=128,
      uri=False,
      autocommit=sqlite3.LEGACY_TRANSACTION_CONTROL
  )
  ```
- **核心參數說明**：

| 參數名稱 | 期待型別 | 預設值 | 說明與作用 |
| :--- | :--- | :--- | :--- |
| `database` | `str` / `Path` | *(必填)* | 資料庫檔案路徑。若檔案不存在會自動建立。<br>傳入 `":memory:"` 則建立記憶體暫時資料庫。 |
| `timeout` | `float` | `5.0` | 當資料庫被其他連線寫入鎖定時，系統在拋出 `OperationalError: database is locked` 前等待解鎖的最大逾時秒數。 |
| `check_same_thread` | `bool` | `True` | 是否限制只能在建立連線的同一個執行緒中使用此連線。<br>若設為 `False` 允許多執行緒共享，但呼叫端必須自行確保寫入序列化防護。 |
| `autocommit` | `bool` | `LEGACY...` | **(Python 3.12+)** 現代交易模式開關。<br>設為 `False` 啟用符合 PEP 249 的標準交易控制；設為 `True` 啟用完全自動認可。 |
| `uri` | `bool` | `False` | 是否允許使用 URI 格式連線字串（例如 `file:memdb1?mode=memory&cache=shared` 共享記憶體庫）。 |

- **回傳值**：
  - `Connection`：資料庫連線實例。

```python
import sqlite3
from pathlib import Path

# 1. 連線至磁碟資料庫檔案（支援 Path 物件）
db_path = Path("app_data.db")
conn = sqlite3.connect(db_path, timeout=10.0)

# 2. 建立純記憶體資料庫（執行期超高速，程式結束後自動消失）
mem_conn = sqlite3.connect(":memory:")
```

---

##### with conn 交易情境管理器 (自動認可與回滾)

- **使用時機**：
  - 當你需要將多句資料庫修改操作包裝在單一不可分割的交易 (Transaction) 區塊中時使用。
  - 保證若區塊內部任何一行發生未預期的例外，所有已執行的資料變更**全數自動回滾 (Rollback)**；若順利完成則**全數自動確認 (Commit)**。
- **語法**：
  ```python
  with conn:
      # 在此區塊中執行的操作受單一交易保護
      conn.execute(...)
  ```
- **重要概念澄清**：
  - `with conn:` 管理的是**交易 (Transaction)**，**而不是關閉連線**！離開 `with conn:` 區塊後，連線依然保持開啟狀態，可以繼續執行後續查詢。
- **回傳值**：
  - 連線物件本身。

```python
import sqlite3

conn = sqlite3.connect(":memory:")
conn.execute("CREATE TABLE accounts (id INT PRIMARY KEY, balance INT)")
conn.execute("INSERT INTO accounts VALUES (1, 1000), (2, 500)")

# 模擬銀行轉帳交易：確保扣款與入帳具備原子性
try:
    with conn:
        # 1. 帳戶 1 扣款 200
        conn.execute("UPDATE accounts SET balance = balance - 200 WHERE id = 1")
        
        # 2. 模擬中間發生重大錯誤（例如網路中斷或斷言失敗）
        # raise RuntimeError("轉帳系統中斷！")
        
        # 3. 帳戶 2 入帳 200
        conn.execute("UPDATE accounts SET balance = balance + 200 WHERE id = 2")
        
    print("轉帳成功完成！已自動 commit")
except RuntimeError as err:
    print(f"轉帳失敗: {err}，系統已自動執行 rollback 還原餘額！")
```

---

##### with sqlite3.connect() as conn 連線生命週期管理器

- **使用時機**：在批次腳本或獨立任務中，確保使用完畢後自動呼叫 `conn.close()` 釋放檔案鎖。
- **語法**：
  ```python
  with sqlite3.connect("data.db") as conn:
      ...
  ```

> **[核心天條]：區分 `with sqlite3.connect()` 與 `with conn:` 的職責差異！**  
> - `with sqlite3.connect(...) as conn:`：管理**連線的關閉生命週期**。但注意：在舊版 Python 中它同時也會包裹交易；在現代 Python 中建議將連線管理與內部交易顯式分層：
>   ```python
>   with sqlite3.connect("data.db") as conn:
>       with conn:  # 內層專職管理交易原子性
>           conn.execute("INSERT INTO logs VALUES (?)", ("系統啟動",))
>   # 離開外層後，連線自動 close()
>   ```

---

##### conn.commit() 提交當前作用中交易

- **使用時機**：在手動交易模式下，顯式確認當前作用中交易所產生的所有資料寫入與變更，並將其永久刷入磁碟檔案。
- **語法**：`conn.commit()`
- **回傳值**：`None`。

```python
conn = sqlite3.connect("app.db")
cur = conn.cursor()
cur.execute("INSERT INTO users (name) VALUES (?)", ("Matthew",))

# 顯式持久化寫入
conn.commit()
```

---

##### conn.rollback() 回滾撤銷當前交易

- **使用時機**：在手動交易模式中，當捕捉到業務邏輯錯誤或資料衝突時，主動撤銷當前交易內所有的未確認變更。
- **語法**：`conn.rollback()`
- **回傳值**：`None`。

```python
try:
    conn.execute("DELETE FROM orders WHERE id = 99")
    # 發現關聯檢查不符
    # conn.rollback()
except Exception:
    conn.rollback()
    raise
```

---

##### conn.close() 關閉連線並釋放檔案鎖

- **使用時機**：當應用程式結束、執行緒工作完成，或不再需要存取資料庫時，釋放作業系統開啟的檔案描述符與磁碟讀寫鎖。
- **語法**：`conn.close()`
- **回傳值**：`None`。

---

# 2. 游標與 SQL 語句執行 (Cursor & Execution)

##### conn.cursor() 建立游標物件

- **使用時機**：建立用於維護查詢結果集狀態與指標的游標實例。
- **語法**：`conn.cursor(factory=Cursor)`
- **參數說明**：
  - `factory`：自訂游標類別，預設為 `sqlite3.Cursor`。
- **回傳值**：
  - `Cursor`：游標物件。

---

##### conn.execute() 與 conn.executemany() 連線層級便捷執行器

- **使用時機**：在無需手動保留游標變數的簡潔腳本中，直接透過連線物件執行 SQL。
- **底層原理**：連線物件上的 `.execute()` 內部會自動建立一個暫時游標並呼叫其 `execute()` 方法，最後將該游標回傳。
- **語法**：`conn.execute(sql, parameters=())`

```python
conn = sqlite3.connect(":memory:")

# 直接透過 conn.execute 執行 DDL 與查詢，代碼簡潔洗鍊
conn.execute("CREATE TABLE config (k TEXT PRIMARY KEY, v TEXT)")
conn.execute("INSERT INTO config VALUES (?, ?)", ("theme", "dark"))

val = conn.execute("SELECT v FROM config WHERE k = ?", ("theme",)).fetchone()[0]
print(f"當前主題設定: {val}")  # 輸出: 當前主題設定: dark
```

---

##### cursor.execute() 執行單句 SQL 指令

- **使用時機**：執行單句 DDL（如 `CREATE TABLE`）或帶有參數的 DML（如 `INSERT`, `UPDATE`, `DELETE`, `SELECT`）。
- **語法**：`cursor.execute(sql, parameters=())`
- **參數說明**：
  - `sql`：SQL 語句字串。
  - `parameters`：參數化資料。支援兩種形式：
    - **位置參數 (Positional)**：使用 `?` 作為佔位符，傳入 `tuple` 或 `list`（例如 `("Alice", 20)`）。
    - **具名參數 (Named)**：使用 `:name` 作為佔位符，傳入 `dict`（例如 `{"name": "Alice", "age": 20}`）。
- **回傳值**：
  - `Cursor`：游標物件本身（支援鏈式操作）。

```python
conn = sqlite3.connect(":memory:")
cur = conn.cursor()
cur.execute("CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT, age INT)")

# 1. 方式 A：位置參數佔位符 (?) 傳入元組
cur.execute("INSERT INTO users (name, age) VALUES (?, ?)", ("Alice", 25))

# 2. 方式 B：具名參數佔位符 (:key) 傳入字典 (語意更清晰，欄位繁多時強烈推薦)
cur.execute(
    "INSERT INTO users (name, age) VALUES (:name, :age)",
    {"name": "Bob", "age": 30}
)
```

---

##### cursor.executemany() 批次參數化執行 SQL 指令

- **使用時機**：需要將同一個 SQL 範本（如 `INSERT INTO ...`）套用至多筆資料時使用。內部以原生 C 迴圈快速迭代，效能遠高於手寫 Python `for` 迴圈。
- **語法**：`cursor.executemany(sql, seq_of_parameters)`
- **參數說明**：
  - `sql`：帶有佔位符的單句 SQL 語句。
  - `seq_of_parameters`：包含參數元組或字典的可迭代序列（支援 `list`, `tuple` 或生成器）。
- **回傳值**：
  - `Cursor`：游標物件本身。

```python
conn = sqlite3.connect(":memory:")
conn.execute("CREATE TABLE logs (id INTEGER PRIMARY KEY, message TEXT, level TEXT)")

# 準備大量資料清單
log_entries = [
    ("服務初始化成功", "INFO"),
    ("連線逾時，正在重試...", "WARNING"),
    ("資料庫連線中斷！", "ERROR"),
]

# 批次原子化插入
conn.cursor().executemany(
    "INSERT INTO logs (message, level) VALUES (?, ?)",
    log_entries
)
```

---

##### cursor.executescript() 執行多句腳本語句

- **使用時機**：需要一次性執行一段包含多句 SQL、以分號 `;` 分隔的完整腳本字串時（如執行資料庫架構初始化 `.sql` 檔案）。
- **語法**：`cursor.executescript(sql_script)`
- **參數說明**：
  - `sql_script`：包含多句 SQL 的字串。
- **重要特性**：
  - 呼叫 `executescript()` 前，它會**先自動發起 COMMIT** 提交先前未結算的事務，隨後依序執行腳本中的所有語句。**不支援 `?` 佔位符參數**。

```python
init_sql = (
    "CREATE TABLE departments (id INTEGER PRIMARY KEY, name TEXT NOT NULL);"
    "CREATE TABLE employees (id INTEGER PRIMARY KEY, name TEXT NOT NULL, dept_id INTEGER);"
    "INSERT INTO departments (name) VALUES ('研發部'), ('維運部');"
)

conn = sqlite3.connect(":memory:")
conn.cursor().executescript(init_sql)
```

---

##### cursor.lastrowid 獲取最後插入列之主鍵 ID

- **使用時機**：執行 `INSERT` 語句後，若該資料表定義了自增主鍵（如 `INTEGER PRIMARY KEY`），需要取得系統剛產生的主鍵 ID 值時。
- **屬性型別**：`int | None`。

```python
conn = sqlite3.connect(":memory:")
conn.execute("CREATE TABLE orders (id INTEGER PRIMARY KEY AUTOINCREMENT, item TEXT)")

cur = conn.cursor()
cur.execute("INSERT INTO orders (item) VALUES (?)", ("筆記型電腦",))
order_id = cur.lastrowid
print(f"新生產的訂單編號 ID: {order_id}")  # 輸出: 新生產的訂單編號 ID: 1
```

---

##### cursor.rowcount 獲取最後執行受影響之行數

- **使用時機**：執行 `UPDATE` 或 `DELETE` 語句後，確認實際被修改或刪除的資料行數。
- **屬性型別**：`int`。
- **小提醒**：在執行 `SELECT` 查詢後，SQLite 的 `rowcount` 固定回傳 `-1`（因為 SQLite 在游標未遍歷完畢前無法預知總結果行數）。

```python
cur.execute("UPDATE orders SET item = '平板電腦' WHERE id = 1")
print(f"受影響的訂單數量: {cur.rowcount}")  # 輸出: 受影響的訂單數量: 1
```

---

##### cursor.description 獲取結果集欄位元數據

- **使用時機**：在執行 `SELECT` 查詢後，需要動態取得所有欄位的名稱清單時（例如動態生成報表標頭或轉為 Pandas DataFrame）。
- **屬性型別**：包含 7 元組的清單，每個元組的第 0 項即為欄位名稱：`(name, type_code, display_size, internal_size, precision, scale, null_ok)`。

```python
cur.execute("SELECT id, item FROM orders")
column_names = [col[0] for col in cur.description]
print(f"查詢結果欄位名稱: {column_names}")  # 輸出: 查詢結果欄位名稱: ['id', 'item']
```

---

# 3. 查詢結果提取與 Row 物件 (Fetching & Row Factory)

##### cursor.fetchone() 提取結果集單筆紀錄

- **使用時機**：查詢單一特定資料（如根據 ID 查詢）、或只關心第一筆結果時。
- **語法**：`cursor.fetchone()`
- **回傳值**：
  - 成功回傳包含欄位數值的單一資料列物件（預設為 `tuple`，或受 `row_factory` 影響）。
  - 若已無剩餘資料，回傳 `None`。

---

##### cursor.fetchmany() 提取結果集指定筆數清單

- **使用時機**：結果集數據量過大，無法一次全載入記憶體，需要分批（如每次抓 1,000 筆）進行串流處理時。
- **語法**：`cursor.fetchmany(size=cursor.arraysize)`
- **參數說明**：
  - `size`：期望提取的紀錄數量整數（預設取 `cursor.arraysize`，預設值為 1）。
- **回傳值**：
  - `list`：包含指定數量資料列的清單（若不足則回傳剩餘所有；若無資料回傳空清單 `[]`）。

---

##### cursor.fetchall() 提取結果集所有紀錄清單

- **使用時機**：結果集規模較小，希望一次性提取所有資料行以利後續操作。
- **語法**：`cursor.fetchall()`
- **回傳值**：
  - `list`：包含剩餘所有資料列物件的清單。

```python
conn = sqlite3.connect(":memory:")
conn.execute("CREATE TABLE nums (n INT)")
conn.cursor().executemany("INSERT INTO nums VALUES (?)", [(i,) for i in range(10)])

cur = conn.cursor()
cur.execute("SELECT n FROM nums")

# 1. 取出一筆
print(cur.fetchone())    # 輸出: (0,)

# 2. 批次取出三筆
print(cur.fetchmany(3))  # 輸出: [(1,), (2,), (3,)]

# 3. 取得剩餘所有
remaining = cur.fetchall()
print(len(remaining))    # 輸出: 6
```

---

##### conn.row_factory 設定資料列工廠

- **使用時機**：預設的查詢結果均為匿名元組（如 `('Matthew', 25)`），程式碼中充斥 `row[0]`, `row[1]` 極度難以維護且容易因欄位增刪出錯。透過設定 `row_factory`，可將資料列自動封裝為具備鍵名查找特性的高階物件。
- **語法**：`conn.row_factory = factory_callable`
- **常用指派值**：
  - `sqlite3.Row`（官方推薦標準，高效且具備字典與元組雙重特性）。
  - 自訂函式（如將資料列自動轉為自訂 `dict` 或 Pydantic 模型）。

---

##### sqlite3.Row 字典式欄位存取資料列物件

- **使用時機**：最受歡迎的資料列轉換器。兼具極低記憶體開銷與極高程式可讀性。
- **核心特徵**：
  - 支援以**欄位名稱字串**存取（大小寫不敏感）：`row["name"]` 或 `row["NAME"]`。
  - 支援以**數值索引**存取：`row[0]`。
  - 支援獲取所有欄位鍵名清單：`row.keys()`。
  - 支援透過 `dict(row)` 瞬間轉為標準 Python 字典。

```python
import sqlite3

conn = sqlite3.connect(":memory:")
# 啟用 sqlite3.Row 工廠
conn.row_factory = sqlite3.Row

conn.execute("CREATE TABLE staff (id INT, name TEXT, role TEXT)")
conn.execute("INSERT INTO staff VALUES (1, 'Alice', 'Engineer')")

row = conn.execute("SELECT * FROM staff WHERE id = 1").fetchone()

# 1. 透過欄位名稱鍵名存取 (極度推薦)
print(row["name"])  # 輸出: Alice
print(row["role"])  # 輸出: Engineer

# 2. 依然保留整數索引存取相容性
print(row[1])       # 輸出: Alice

# 3. 提取所有欄位鍵名
print(row.keys())   # 輸出: ['id', 'name', 'role']

# 4. 輕鬆轉換為標準 dict (可直接進行 JSON 序列化)
user_dict = dict(row)
print(user_dict)    # 輸出: {'id': 1, 'name': 'Alice', 'role': 'Engineer'}
```

---

# 4. 進階功能與延伸機制 (Advanced Features)

##### conn.create_function() 註冊自訂 SQL 函式

- **使用時機**：
  - 需要在 SQL 查詢中直接呼叫自訂的 Python 邏輯（例如進行正則表達式篩選、複雜數學運算或文字雜湊解密）。
  - SQLite 預設未提供完整的字串或數學庫，透過此機制能無縫將 Python 的強大能力注入至 SQL 內部！
- **語法**：`conn.create_function(name, narg, func, *, deterministic=False)`
- **參數說明**：
  - `name`：在 SQL 語句中使用的函式名稱字串。
  - `narg`：函式接收的引數數量整數（若設為 `-1` 代表接受任意數量引數）。
  - `func`：Python 可呼叫物件。
  - `deterministic`：(Python 3.8+) 若此函式在給定相同輸入時保證回傳相同結果（純函式），設為 `True` 可讓 SQLite 查詢優化器進行快取。

```python
import sqlite3
import re
import hashlib

conn = sqlite3.connect(":memory:")

# 1. 註冊自訂正則表達式匹配函式 (讓 SQL 支援 REGEXP 語法)
def regex_match(pattern: str, text: str) -> bool:
    if text is None:
        return False
    return bool(re.search(pattern, text))

conn.create_function("REGEXP", 2, regex_match, deterministic=True)

# 2. 註冊 MD5 雜湊計算函式
conn.create_function(
    "md5", 1,
    lambda s: hashlib.md5(s.encode()).hexdigest() if s else None,
    deterministic=True
)

conn.execute("CREATE TABLE files (name TEXT)")
conn.execute("INSERT INTO files VALUES ('report_2026.pdf'), ('notes.txt'), ('img_01.png')")

# 在 SQL 中直接使用 REGEXP 篩選符合模式的檔案
res = conn.execute("SELECT name FROM files WHERE name REGEXP ?", (r"\d+",)).fetchall()
print([r[0] for r in res])  # 輸出: ['report_2026.pdf', 'img_01.png']

# 在 SQL 中直接計算 MD5
hash_val = conn.execute("SELECT md5('hello')").fetchone()[0]
print(f"MD5 運算結果: {hash_val}")
```

---

##### conn.backup() 資料庫線上即時備份

- **使用時機**：
  - 需要在資料庫持續服務運作、不中斷讀寫連線的情況下，進行線上熱備份。
  - **核心場景**：將超高速的記憶體資料庫 (`:memory:`) 的所有內容一次性完整傾印保存至磁碟檔案中；或將磁碟資料庫在啟動時預載至記憶體中加速查詢！
- **語法**：`conn.backup(target, *, pages=-1, progress=None, name="main", sleep=0.25)`
- **參數說明**：
  - `target`：目標 `Connection` 連線物件。
  - `pages`：每次拷貝的頁數（預設 `-1` 代表一次全量拷貝完成）。
  - `progress`：可選回呼函式，接收 `(status, remaining, total)` 用於回報備份進度。

```python
import sqlite3

# 1. 在記憶體中建立高頻操作資料庫並寫入資料
mem_conn = sqlite3.connect(":memory:")
mem_conn.execute("CREATE TABLE cache (key TEXT, val TEXT)")
mem_conn.execute("INSERT INTO cache VALUES ('app_state', 'active')")

# 2. 建立目標磁碟資料庫連線
disk_conn = sqlite3.connect("backup.db")

# 3. 執行線上即時備份：將記憶體資料庫直接傾印至磁碟檔案中
mem_conn.backup(disk_conn)

# 驗證磁碟庫已成功同步
val = disk_conn.execute("SELECT val FROM cache WHERE key = 'app_state'").fetchone()[0]
print(f"磁碟備份內容檢驗: {val}")  # 輸出: 磁碟備份內容檢驗: active

disk_conn.close()
mem_conn.close()
```

---

##### PRAGMA 指令與 WAL 高效寫入模式

`PRAGMA` 是 SQLite 專屬的特殊 SQL 指令，用於檢查、配置與調校資料庫底層的運作行為。

#### 1. 啟用外鍵約束檢查 (Foreign Key Constraints)
> **[核心天條]：SQLite 基於歷史向下相容，預設關閉外鍵完整性約束檢查！**  
> 若要讓 `FOREIGN KEY` 真正發揮攔截無效參照的功能，**每次建立新連線時必須主動執行一次**：
> ```python
> conn.execute("PRAGMA foreign_keys = ON")
> ```

#### 2. 開啟預寫日誌模式 (WAL Mode, Write-Ahead Logging)
預設情況下，SQLite 採用傳統的回滾日誌 (Rollback Journal)。在此模式下，**寫入操作會完全鎖定整個資料庫**，導致所有並行的讀取查詢全部被卡住 (Block)，引發頻繁的 `database is locked`。

開啟 **WAL 模式**是將 SQLite 效能推升至生產級的關鍵調校：
- **讀寫完全互不阻塞**：讀取者進行查詢時，寫入者可以同時寫入日誌；寫入者修改資料時，讀取者依然能暢通無阻讀取舊版本快照！
- **磁碟寫入次數大幅降低**：連續寫入全集中在預寫日誌檔 (`.db-wal`)，由背景檢查點機制定期合併回主庫。

```python
import sqlite3

conn = sqlite3.connect("production.db")

# 1. 永久將資料庫日誌模式切換為 WAL (此設定會持久化於資料庫檔案標頭)
cur = conn.execute("PRAGMA journal_mode = WAL")
print(f"目前日誌模式: {cur.fetchone()[0]}")  # 輸出: wal

# 2. 啟用外鍵約束檢查 (連線層級設定，每次連線皆需設定)
conn.execute("PRAGMA foreign_keys = ON")

# 3. 調整同步寫入模式 (NORMAL 在 WAL 模式下兼顧極速與防崩潰安全性)
conn.execute("PRAGMA synchronous = NORMAL")
```

---

# 5. 例外處理與錯誤階層 (Error Handling)

`sqlite3` 提供了清晰的階層化例外體系，所有自訂例外皆繼承自 `sqlite3.Error`：

```text
Exception
 └── sqlite3.Error
      ├── sqlite3.DatabaseError
      │    ├── sqlite3.DataError
      │    ├── sqlite3.OperationalError (最常見：語法錯誤、找不到表、database is locked)
      │    ├── sqlite3.IntegrityError   (最常見：外鍵約束失敗、UNIQUE 主鍵重複衝突)
      │    ├── sqlite3.InternalError
      │    ├── sqlite3.ProgrammingError (連線關閉後操作、多執行緒共享連線錯誤)
      │    └── sqlite3.NotSupportedError
      └── sqlite3.Warning
```

##### sqlite3.Error 模組例外階層大腦

- **使用時機**：精準捕獲特定資料庫錯誤並給予適當補救措施（例如捕獲 `IntegrityError` 提示名稱已存在，或捕獲 `OperationalError` 進行重試）。

```python
import sqlite3

conn = sqlite3.connect(":memory:")
conn.execute("CREATE TABLE users (email TEXT UNIQUE)")
conn.execute("INSERT INTO users VALUES ('test@example.com')")

try:
    # 再次插入相同 email 觸發唯一約束衝突
    conn.execute("INSERT INTO users VALUES ('test@example.com')")
except sqlite3.IntegrityError as err:
    print(f"資料完整性約束衝突攔截: {err}")
    # 輸出: 資料完整性約束衝突攔截: UNIQUE constraint failed: users.email
except sqlite3.OperationalError as err:
    print(f"資料庫操作錯誤 (如語法或鎖表): {err}")
except sqlite3.Error as err:
    print(f"通用 SQLite 基礎例外: {err}")
```

---

# 實戰避坑與核心天條

## 1. 字串拼接引爆 SQL Injection 注入漏洞

> **[核心天條]：嚴禁使用 f-string 或 + 字串拼接組合 SQL 變數，一律使用 ? 或 :name 參數化佔位符！**

- **錯誤症狀**：
  若傳入惡意格式字串（如 `' OR 1=1 --`），原本設定的條件篩選全部失效，整張資料表機密外洩或整張表被 `DROP TABLE` 惡意清除。
- **背後原理**：
  字串拼接會使使用者輸入的字元直接成為 SQL 語法解析樹的一部分。而參數化查詢（Prepared Statement）會將 SQL 語句與變數數值嚴格隔離，直譯器只將傳入值視為純字面純量，徹底阻斷惡意程式碼執行。

```python
import sqlite3

conn = sqlite3.connect(":memory:")
conn.execute("CREATE TABLE credentials (user TEXT, pass TEXT)")
conn.execute("INSERT INTO credentials VALUES ('admin', 'super_secret')")

malicious_user = "' OR 1=1 --"

# [錯誤寫法]：字串拼接造成 SQL Injection 漏洞
# query = f"SELECT * FROM credentials WHERE user = '{malicious_user}'"
# res = conn.execute(query).fetchall() # 成功登入並盜取所有密碼！

# [正確寫法]：強制使用佔位符參數傳遞
safe_query = "SELECT * FROM credentials WHERE user = ?"
res = conn.execute(safe_query, (malicious_user,)).fetchall()
print(f"安全查詢結果: {res}")  # 輸出: 安全查詢結果: [] (安全阻絕攻擊)
```

---

## 2. DML 操作後忘記 commit 導致資料未持久化

> **[核心天條]：執行 INSERT、UPDATE、DELETE 後若未 commit，關閉連線時所有修改將全數灰飛煙滅！**

- **錯誤症狀**：
  在腳本中執行了新增或修改，終端也沒有任何報錯。但打開資料庫檔案檢視時，剛才寫入的資料完全不存在。
- **背後原理**：
  SQLite 的預設交易模式下，所有的資料異動都只暫存在當前連線的作用中交易與記憶體緩衝區中。若沒有顯式呼叫 `conn.commit()` 或使用 `with conn:` 包裹，在連線關閉時，SQLite 會將所有未結算的交易視為廢棄並自動執行 rollback。

```python
import sqlite3

# [錯誤寫法]：手動 execute 後忘記 commit，程式結束後資料遺失
# conn = sqlite3.connect("data.db")
# conn.execute("INSERT INTO users VALUES ('Alice')")
# conn.close() # 資料未儲存！

# [正確寫法 A]：顯式呼叫 commit()
conn = sqlite3.connect("data.db")
conn.execute("INSERT INTO users VALUES ('Alice')")
conn.commit()
conn.close()

# [正確寫法 B (推薦)]：使用 with conn 交易情境管理器自動保證 commit
with sqlite3.connect("data.db") as conn:
    with conn:
        conn.execute("INSERT INTO users VALUES ('Alice')")
```

---

## 3. 多執行緒共享同一連線物件觸發 ProgrammingError

> **[核心天條]：SQLite 連線物件預設不具執行緒安全性，不可在不同執行緒中並行操作同一連線！**

- **錯誤症狀**：
  在多執行緒環境下（如 ThreadPoolExecutor），程式拋出 `sqlite3.ProgrammingError: SQLite objects created in a thread can only be used in that same thread`。
- **背後原理**：
  SQLite 連線的底層 C 結構包含單一狀態機與錯誤緩衝區。為了防止競態條件造成記憶體崩潰，Python 預設在連線中啟用 `check_same_thread=True` 進行安全防禦。
- **最佳實踐**：
  不要跨執行緒共用連線，**在各執行緒內部獨立呼叫 `sqlite3.connect()` 建立專屬連線**。在 WAL 模式下，多個執行緒各自持有獨立連線進行並行讀寫是完全支援且極度高效的。

```python
import sqlite3
import threading

def worker_task(thread_id: int):
    # [正確寫法]：在各執行緒內部建立獨立連線
    with sqlite3.connect("app.db") as conn:
        conn.execute("INSERT INTO thread_logs VALUES (?)", (f"執行緒 {thread_id}",))

threads = [threading.Thread(target=worker_task, args=(i,)) for i in range(5)]
for t in threads:
    t.start()
for t in threads:
    t.join()
```

---

## 4. 忘記開啟 WAL 模式導致高頻讀寫出現 Database is locked

> **[核心天條]：生產環境或高頻讀寫場景下，連線建立後必須主動將 journal_mode 切換為 WAL！**

- **錯誤症狀**：
  並行發起稍微頻繁的查詢與更新時，程式頻繁拋出 `sqlite3.OperationalError: database is locked`。
- **背後原理**：
  預設的 `DELETE` 回滾日誌模式會在寫入時鎖定整座資料庫檔案排他存取，任何讀取操作哪怕只有幾毫秒也會引發鎖衝突。開啟 WAL 模式後，讀取者與寫入者完全解耦互不阻塞。

```python
import sqlite3

conn = sqlite3.connect("app.db")
# 建立連線後第一步：切換為 WAL 模式
conn.execute("PRAGMA journal_mode = WAL")
```

---

## 5. executemany 未打包為序列或生成器導致執行異常

> **[核心天條]：傳入 executemany 的參數結構必須是「外層為序列/生成器、內層為元組/字典」的二維結構！**

- **錯誤症狀**：
  傳入普通字串清單（如 `["Alice", "Bob"]`），程式拋出 `ProgrammingError: Incorrect number of bindings supplied`；或是整批資料被拆解成單一字元逐字塞入。
- **背後原理**：
  `executemany` 預期外層迭代器的每個元素都代表「一整行紀錄的所有參數集合」。若傳入 `["Alice", "Bob"]`，外層產出字串 `"Alice"`，SQLite 會將其視為長度為 5 的字元序列 `('A', 'l', 'i', 'c', 'e')`，與單一佔位符 `?` 的數量產生衝突。

```python
import sqlite3

conn = sqlite3.connect(":memory:")
conn.execute("CREATE TABLE names (val TEXT)")

# [錯誤寫法]：內層元素未打包為元組
# raw_list = ["Alice", "Bob"]
# conn.cursor().executemany("INSERT INTO names VALUES (?)", raw_list) # 報錯！

# [正確寫法]：內層元素必須是單元素元組 (注意逗號) 或字典
safe_list = [("Alice",), ("Bob",)]
conn.cursor().executemany("INSERT INTO names VALUES (?)", safe_list)
```
