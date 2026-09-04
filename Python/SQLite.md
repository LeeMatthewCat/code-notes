這是 Python 內建的輕量級關聯式資料庫引擎模組 (`sqlite3`)。

它不需要安裝任何伺服器軟體，整個資料庫就是一個簡單的本地端檔案，極度適合用來做為單機應用程式的本地儲存或小型專案開發。

> **🍿 比喻單元**：
> 如果 MySQL 是一座**需要數十人維護、體制龐大的國家圖書館**；
> 那麼 SQLite 就是你隨身攜帶的**精緻個人手帳本**，雖然小巧，但該有的目錄與檢索功能一應俱全！

---

## 連線與初始化

在進行任何資料操作前，必須先建立與資料庫檔案的連線。

##### sqlite3.connect() 建立資料庫連線

- **使用時機**：在程式一開始，需要建立或開啟一個 SQLite 資料庫檔案以準備進行讀寫操作時。
- **語法**：`sqlite3.connect(database: str, timeout: float = 5.0, ...)`
- **參數說明**：
  - `database`：資料庫檔案路徑。若檔案不存在會自動建立；若傳入 `":memory:"` 則會建立存活於記憶體中的暫時資料庫。
  - `timeout`：當資料庫被鎖定時，系統等待解鎖的最大秒數。
- **回傳值**：
  - `Connection`：代表與資料庫連線的連線物件。

```python
...
import sqlite3

# 專注展示建立本地資料庫連線
conn = sqlite3.connect("my_app.db")
...
```

---

##### connection.cursor() 建立游標物件

- **使用時機**：連線建立後，你需要一位「替你執行 SQL 指令的使者」時。
- **語法**：`connection.cursor()`
- **參數說明**：無須傳入參數。
- **回傳值**：
  - `Cursor`：游標物件。幾乎所有的 SQL 操作都要透過它來執行。

```python
...
import sqlite3

conn = sqlite3.connect(...)

# 專注展示建立游標物件
cursor = conn.cursor()
...
```

---

## 執行與獲取資料

擁有游標後，就能下達我們在 `SQL.md` 中學過的指令。

##### cursor.execute() 執行 SQL 語句

- **使用時機**：當你需要對資料庫下達單句 SQL 指令（如 `CREATE`, `INSERT`, `SELECT`）時。
- **語法**：`cursor.execute(sql: str, parameters: tuple = ())`
- **參數說明**：
  - `sql`：包含 SQL 語句的字串。可以使用 `?` 作為安全變數的佔位符。
  - `parameters`：要代入佔位符的資料 Tuple，能有效防止 SQL Injection 攻擊。
- **回傳值**：
  - `Cursor`：回傳游標物件本身（方便進行鏈式呼叫）。

```python
...
cursor = ...

# 專注展示使用佔位符安全插入資料
sql = "INSERT INTO users (name, age) VALUES (?, ?)"
cursor.execute(sql, ("Matthew", 25))
...
```

> **💡 小提醒**：永遠、絕對不要用 Python 的 f-string 或是字串拼接來組合 SQL 指令！這會讓你的應用程式面臨被駭客用 SQL 注入 (SQL Injection) 攻擊的巨大風險。請一律使用 `?` 佔位符！

---

##### cursor.fetchone() 獲取單筆資料

- **使用時機**：在執行 `SELECT` 查詢後，你只想要拿取結果中的「第一筆」資料時。
- **語法**：`cursor.fetchone()`
- **參數說明**：無。
- **回傳值**：
  - `tuple`：一組包含欄位資料的 Tuple。若沒有找到結果則回傳 `None`。

```python
...
cursor.execute("SELECT name, age FROM users WHERE id = ?", (1,))

# 專注展示拿取單筆資料
user = cursor.fetchone()
print(user)  # 假設印出：('Matthew', 25)
...
```

---

##### cursor.fetchall() 獲取所有資料

- **使用時機**：在執行 `SELECT` 查詢後，你想把所有符合條件的資料一次性全部拿出來時。
- **語法**：`cursor.fetchall()`
- **參數說明**：無。
- **回傳值**：
  - `List[tuple]`：一個包含多個 Tuple 的串列。若沒有找到資料則回傳空串列 `[]`。

```python
...
cursor.execute("SELECT name FROM users")

# 專注展示一次拿取所有符合的資料
users = cursor.fetchall()
print(users)  # 假設印出：[('Matthew',), ('Alex',), ('Bob',)]
...
```

---

## 交易控制與資源釋放

這是在操作資料庫時最容易被遺忘，卻又最關鍵的收尾步驟。

##### connection.commit() 確認並儲存交易

- **使用時機**：當你執行了任何會修改資料庫內容的指令（`INSERT`, `UPDATE`, `DELETE`）後，必須告訴系統「確認儲存」時。
- **語法**：`connection.commit()`
- **參數說明**：無。
- **回傳值**：`None`。

```python
...
conn = ...
cursor = conn.cursor()

cursor.execute("UPDATE users SET status = 'active' WHERE id = 1")

# 專注展示確認交易並寫入檔案
conn.commit()
...
```

> **💡 小提醒**：如果你執行了新增或修改操作卻發現資料庫檔案沒有變化，通常就是因為你忘記呼叫 `commit()`。若是純粹的 `SELECT` 查詢，則不需呼叫此方法。

---

##### connection.close() 關閉資料庫連線

- **使用時機**：當程式即將結束，或不再需要與資料庫互動時，用來釋放檔案鎖與系統資源。
- **語法**：`connection.close()`
- **參數說明**：無。
- **回傳值**：`None`。

```python
...
# 專注展示釋放資源
conn.close()
...
```

---
