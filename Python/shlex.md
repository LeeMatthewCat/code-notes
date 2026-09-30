# 概念與原理

## 什麼是 shlex 模組？

`shlex` 是 Python 內建的 Shell 語彙分析標準函式庫（無需額外安裝，直接 `import shlex`）。

核心職責在於按照類 Unix Shell（如 Bash、Zsh）的語法規則，對字串進行**剖析拆解 (Tokenization)**、**跳脫防護 (Quoting/Escaping)** 與 **引數重組 (Joining)**。

在調用 `subprocess.run()` 執行外部系統指令時，標準做法是將指令拆分為串列傳入（如 `["git", "commit", "-m", "fix bug"]`），而非使用危險的 `shell=True`。

然而，若外部輸入的是一整段長字串，使用一般的 `str.split(" ")` 會盲目依據空格切割，導致被引號包覆的完整字串（如 `"hello world"`）或路徑被硬生生撕裂。

`shlex` 能精準理解單引號、雙引號、轉義反斜線 `\` 以及註解符號 `#`，是 Python 處理命令列字串時的安全守門員。

> **[生動比喻]**：  
> - **普通 `str.split()`**：就像一把「盲目揮舞的菜刀」，看到空格就一刀切下，硬生生把引號內的情書切成零碎字塊。  
> - **`shlex.split()`**：就像一位「精通 Shell 語法的專業翻譯官」，懂得「引號內部是完整的一句話、反斜線代表轉義」，能完整保護帶有空格的檔名與參數不被破壞！

---

## 為什麼不用 str.split()？核心能力對照

| 比較維度 | 普通字串切割 (`str.split()`) | Shell 語彙解析 (`shlex.split()`) |
| :--- | :--- | :--- |
| **空格切割方式** | 遇到任何空白字元無差別直接切開 | 遇到空白切開，但**嚴格尊重引號邊界** |
| **雙引號 `"..."` 處理** | 保留引號字元，內部空白仍被切斷 | 拔除外層引號，保留內部完整包含空白的字串 |
| **單引號 `'...'` 處理** | 保留引號字元，不具備特殊語意 | 視為不可轉義之字面值，拔除外層引號 |
| **反斜線 `\` 跳脫** | 視為普通反斜線字元 | 依據 POSIX 規範解析跳脫序列（如 `\ ` 代表空格） |
| **`#` 註解處理** | 視為普通文字 | 支援 `comments=True` 自動截斷並丟棄註解 |

---

## 現代語法升級與版本演進對照表

| 功能操作 | 舊式語法 (已棄用 / 易踩坑) | 現代標準語法 (推薦) | 核心差異與說明 |
| :--- | :--- | :--- | :--- |
| **引數串列安全重組** | `" ".join(shlex.quote(x) for x in args)` | `shlex.join(args)` | Python 3.8+ 原生新增，自動對必要參數加上引號，簡潔防呆 |
| **Shell 跳脫字元防護** | `pipes.quote(s)` (Python 3.13 移除) | `shlex.quote(s)` | `pipes` 模組已正式廢棄並移除，統一由 `shlex.quote()` 接管 |
| **保留特殊標點符號** | 手動設定 `lex.punctuation_chars` 屬性 | `shlex.shlex(..., punctuation_chars=True)` | Python 3.6+ 建構子直接支援標點符號保留切詞模式 |
| **命令字串安全解析** | `cmd.split()` 搭配 `shell=True` | `subprocess.run(shlex.split(cmd))` | 徹底消除命令注入 (Command Injection) 風險的最佳實踐 |

---

# 核心 API 函式與類別字典對照表

| 函式 / 類別名稱 | 回傳型別 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- | :--- |
| **[[#shlex.split() 命令列字串語彙拆分\|shlex.split()]]** | `list[str]` | **將 Shell 命令字串依據引號與空格拆解為引數串列** | 配合 `subprocess.run()` 傳遞安全且結構正確的參數列表 |
| **[[#shlex.join() 引數串列重組為 Shell 字串\|shlex.join()]]** | `str` | **將引數串列自動加上跳脫保護並拼接為單一 Shell 指令** | 日誌記錄 (Logging)、終端除錯顯示、生成 Shell 腳本 |
| **[[#shlex.quote() 單一字串 Shell 安全逸出\|shlex.quote()]]** | `str` | **對單一任意字串進行跳脫，使其在 Shell 下安全作為單一引數** | 防禦使用者輸入引發的 Shell 命令注入攻擊 |
| **[[#shlex.shlex 詞法分析器核心類別\|shlex.shlex]]** | `shlex` | **建立串流式 Shell 語彙分析器物件實例** | 撰寫自訂 DSL、剖析複雜設定檔、逐詞分析原始碼 |
| **[[#shlex.shlex.get_token() 讀取下一個語彙單元\|shlex.shlex.get_token()]]** | `str` | **從字串串流中拉取並解析下一個語彙單元 (Token)** | 串流式剖析指令管線、逐詞過濾與語法剖析器實作 |
| **[[#shlex.shlex.push_token() 壓回語彙單元至串流堆疊\|shlex.shlex.push_token()]]** | `None` | **將已讀出的語彙單元回退壓入堆疊供後續重複讀取** | 前瞻剖析 (Lookahead)、回溯處理語法結構 |

---

# 核心功能與語法大解密

## 1. 常用命令處理三大核心函式

##### shlex.split() 命令列字串語彙拆分

- **使用時機**：當你有一串使用者輸入或設定檔中的命令列字串，需要將其拆成清單傳給 `subprocess.run()`，且必須保留引號內包含空格的完整字串時使用。
- **語法**：`shlex.split(s, comments=False, posix=True)`
- **參數說明**：
    - `s`：要拆分的命令列字串（或具備 `read()` 與 `readline()` 的類檔案物件）。
    - `comments`：布林值，預設為 `False`。若設為 `True`，則字串中未被引號包覆的 `#` 及其後方所有文字將被當成註解直接捨棄。
    - `posix`：布林值，預設為 `True`。遵循 POSIX 規範（引號外反斜線作為跳脫字元、引號會被剝離）；若設為 `False` 則切換為非 POSIX 模式。
- **回傳值**：
    - `list[str]`：切分完成的引數單元字串串列。

```python
import shlex
import subprocess

# 1. 傳統 str.split() vs shlex.split() 對比
cmd_text = 'git commit -m "Initial commit message" --author="Alice <alice@example.com>"'

# 普通切割會把引號內句子硬生生切開
print(cmd_text.split(" "))
# 輸出: ['git', 'commit', '-m', '"Initial', 'commit', 'message"', '--author="Alice', '<alice@example.com>"']

# shlex 能精準辨識雙引號邊界並拔除外殼
args = shlex.split(cmd_text)
print(args)
# 輸出: ['git', 'commit', '-m', 'Initial commit message', '--author=Alice <alice@example.com>']

# 2. 安全傳遞給 subprocess.run()
# subprocess.run(args, check=True)
```

---

##### shlex.join() 引數串列重組為 Shell 字串

- **使用時機**：在 Python 3.8+ 引入。當你手上有一個引數清單（例如 `["grep", "-r", "hello world", "/path"]`），需要將其重組為可在終端機貼上執行的安全字串，或者寫入日誌記錄時使用。
- **語法**：`shlex.join(split_command)`
- **參數說明**：
    - `split_command`：由字串組成的引數可迭代物件（如 `list[str]`）。
- **回傳值**：
    - `str`：使用空白連接、並對含有空格或特殊字元的引數自動加上跳脫保護的完整指令字串。

```python
import shlex

# 1. 重組含有空白與特殊符號的引數串列
args = ["curl", "-X", "POST", "https://api.example.com", "-d", '{"name": "Alice"}']

# shlex.join 會自動幫含有空格或引號的項目加上單引號保護
cmd_str = shlex.join(args)
print(cmd_str)
# 輸出: curl -X POST https://api.example.com -d '{"name": "Alice"}'

# 2. 日誌紀錄與終端除錯印出
print(f"即將執行的安全指令：{cmd_str}")
```

---

##### shlex.quote() 單一字串 Shell 安全逸出

- **使用時機**：當你需要將不可信的使用者輸入拼接進 Shell 命令字串時，使用本函式為其包覆單引號並跳脫內部特殊字元，防範命令注入漏洞。
- **語法**：`shlex.quote(s)`
- **參數說明**：
    - `s`：需要進行安全跳脫保護的字串。
- **回傳值**：
    - `str`：經過 POSIX Shell 安全加殼保護後的字串。若原字串完全安全（僅含字母數字底線等）則原樣返回；若含有空格或特殊字元則以單引號包裹並妥善處理內部單引號。

```python
import shlex

# 1. 處理帶有空格的檔名
filename = "my secret report.pdf"
safe_arg = shlex.quote(filename)
print(safe_arg)  # 輸出: 'my secret report.pdf'

# 2. 徹底中和危險的惡意注入指令
malicious_input = "foo; rm -rf / #"
safe_malicious = shlex.quote(malicious_input)
print(safe_malicious)  # 輸出: 'foo; rm -rf / #'

# 即使在 shell 中執行，惡意指令也會被當成純文字參數，無法被直譯執行！
safe_cmd = f"ls -l {safe_malicious}"
print(safe_cmd)  # 輸出: ls -l 'foo; rm -rf / #'
```

---

## 2. 底層分析器類別與進階自訂

##### shlex.shlex 詞法分析器核心類別

- **使用時機**：當你需要更底層、串流式地逐字元剖析字串、撰寫簡單的直譯器/DSL 語法剖析器，或需要自訂註解字元、空格規則時使用。
- **語法**：`shlex.shlex(instream=None, infile=None, posix=False, punctuation_chars=False)`
- **參數說明**：
    - `instream`：輸入字串串流物件（如 `io.StringIO` 或字串）。
    - `infile`：關聯的檔名（主要用於報錯訊息中標註來源）。
    - `posix`：布林值，預設為 `False`（注意：類別預設為 `False`，與 `split()` 的預設值相反）。
    - `punctuation_chars`：布林值或字串（Python 3.6+）。若為 `True`，會將常見 Shell 標點符號（如 `(`, `)`, `;`, `|`, `&`, `>`, `<`）獨立視為個別 Token 拆分。
- **回傳值**：
    - `shlex.shlex`：語彙剖析器實例物件。

```python
import io
import shlex

# 1. 使用 punctuation_chars=True 保留管線符號與重定向符號
raw_script = "cat input.txt | grep 'pattern' > output.log"
lexer = shlex.shlex(io.StringIO(raw_script), posix=True, punctuation_chars=True)

tokens = list(lexer)
print(tokens)
# 輸出: ['cat', 'input.txt', '|', 'grep', 'pattern', '>', 'output.log']
```

---

##### shlex.shlex.get_token() 讀取下一個語彙單元

- **使用時機**：需要以迭代或輪詢方式逐個讀取語彙單元，適合用於有限狀態機 (State Machine) 或由前向後逐項分析語法樹時。
- **語法**：`lexer.get_token()`
- **參數說明**：
    - 無特定參數。
- **回傳值**：
    - `str`：下一個語彙單元字串。若已讀取至串流末尾（EOF），則回傳空字串 `""` 或等於 `lexer.eof`。

```python
import io
import shlex

text_stream = io.StringIO("HOST=127.0.0.1 PORT=8080")
lexer = shlex.shlex(text_stream, posix=True)

# 逐一取出語彙單元
while True:
    token = lexer.get_token()
    if not token:
        break
    print(f"解析到 Token: {token}")

# 輸出:
# 解析到 Token: HOST
# 解析到 Token: =
# 解析到 Token: 127.0.0.1
# 解析到 Token: PORT
# 解析到 Token: =
# 解析到 Token: 8080
```

---

##### shlex.shlex.push_token() 壓回語彙單元至串流堆疊

- **使用時機**：在語法解析過程中「預讀 (Lookahead)」了一個 Token，發現當前語法規則不匹配時，將該 Token 推回解析器的內部堆疊中，下次呼叫 `get_token()` 時會優先重新取出。
- **語法**：`lexer.push_token(tok)`
- **參數說明**：
    - `tok`：要回推至堆疊的字串單元。
- **回傳值**：
    - `None`。

```python
import io
import shlex

lexer = shlex.shlex(io.StringIO("alpha beta gamma"), posix=True)

# 讀取第一個 Token
t1 = lexer.get_token()
print(t1)  # 輸出: alpha

# 將其重新壓回堆疊
lexer.push_token(t1)

# 再次讀取，依然是剛剛壓回的 Token
t2 = lexer.get_token()
print(t2)  # 輸出: alpha
```

---

# 實戰避坑與核心天條

## 1. Windows 檔案路徑反斜線遭吞噬破壞

> **[核心天條]：在 Windows 平台上處理本機路徑時，切勿直接以預設的 POSIX 模式進行 `shlex.split()`！**

- **錯誤症狀**：
  在 Windows 傳入 `C:\Users\test\file.txt`，經過 `shlex.split()` 處理後，反斜線全部消失，變成了 `C:Userstestfile.txt`。
- **背後原理**：
  `shlex.split()` 預設開啟 `posix=True`。在 POSIX 規範中，反斜線 `\` 是跳脫字元（例如 `\n`、`\ `），後續字母被視為轉義目標，導致反斜線本身被抹除。

```python
import shlex
from pathlib import Path

win_path = r"C:\Users\test\documents\report.pdf"

# [錯誤寫法]：預設 POSIX 模式會吃掉反斜線
wrong_result = shlex.split(win_path)
print(wrong_result)
# 輸出: ['C:Userstestdocumentsreport.pdf']

# [正確寫法一]：將路徑先行轉換為 POSIX 正斜線形式
clean_path = Path(win_path).as_posix()
right_result_1 = shlex.split(clean_path)
print(right_result_1)
# 輸出: ['C:/Users/test/documents/report.pdf']

# [正確寫法二]：顯式指定 posix=False 關閉轉義解析
right_result_2 = shlex.split(win_path, posix=False)
print(right_result_2)
# 輸出: ['C:\\Users\\test\\documents\\report.pdf']
```

---

## 2. 引號未閉合引發 ValueError 致命崩潰

> **[核心天條]：解析不可信的外部使用者命令列字串時，務必使用 `try...except ValueError` 進行例外防禦！**

- **錯誤症狀**：
  使用者輸入了未對齊的單雙引號（例如手滑漏打後引號 `python script.py -m "hello world`），直接執行 `shlex.split()` 會拋出 `ValueError: No closing quotation` 導致服務崩潰中斷。
- **背後原理**：
  `shlex` 嚴格遵守詞法分析閉合性。未閉合的字串在語法上被視為不合法語句，因此會立即拋出 `ValueError`。

```python
import shlex

user_raw_command = 'python -m "unfinished quote'

# [錯誤寫法]：直接解析未經防禦的字串，引發例外
# tokens = shlex.split(user_raw_command)  # 拋出 ValueError: No closing quotation

# [正確寫法]：使用 try-except 進行防禦捕獲並妥善提示
try:
    tokens = shlex.split(user_raw_command)
except ValueError as exc:
    print(f"語法解析失敗：指令中包含未閉合的引號！詳細：{exc}")
    tokens = []
```

---

## 3. 誤信 shlex.quote() 能防禦 Windows cmd.exe 命令注入

> **[核心天條]：`shlex.quote()` 專門針對 POSIX Shell 設計，在 Windows `cmd.exe` 下無法防禦注入！**

- **錯誤症狀**：
  在 Windows 環境下使用 `shlex.quote()` 跳脫使用者參數，再以 `cmd.exe` 或 `shell=True` 執行時，依然遭到 `&` 或 `%VAR%` 等特殊字元注入攻擊。
- **背後原理**：
  `shlex.quote()` 使用單引號 `'...'` 來進行封裝保護。然而，Windows 的 `cmd.exe` 完全不將單引號視為特殊封裝字元，單引號會被當作一般文字處理，惡意運算子依然生效。

```python
import shlex
import subprocess

user_filename = "test.txt & whoami"

# [錯誤寫法]：在 Windows 下對 cmd.exe 使用 shlex.quote 搭配 shell=True
safe_in_unix = shlex.quote(user_filename)  # 結果為 'test.txt & whoami'
# 若在 Windows 執行: subprocess.run(f"type {safe_in_unix}", shell=True)
# Windows cmd 會將 & 解讀為命令連接符，仍會執行 whoami 指令！

# [正確寫法]：絕不使用 shell=True，一律傳遞 List 參數清單給作業系統直調
subprocess.run(["type", user_filename], shell=False)
```

---

## 4. 誤開 comments=True 造成特殊參數截斷

> **[核心天條]：處理常規命令列參數時，切勿隨意啟用 `comments=True`！**

- **錯誤症狀**：
  當指令參數中包含顏色十六進位碼（如 `--color=#FFFFFF`）或標註 Hash 標籤（如 `git commit -m "#123 issue"`）時，`#` 之後的內容無故蒸發失蹤。
- **背後原理**：
  `comments=True` 會將未經引號保護的 `#` 視為 Shell 註解的起始標記，將其後方整行內容直接丟棄。

```python
import shlex

cmd = "set-theme --color=#FFFFFF --dark-mode"

# [錯誤寫法]：誤開 comments=True 導致非預期截斷
wrong = shlex.split(cmd, comments=True)
print(wrong)
# 輸出: ['set-theme', '--color=']  (後方所有參數被當成註解消失了！)

# [正確寫法]：保持預設 comments=False，完整保留各項參數
correct = shlex.split(cmd, comments=False)
print(correct)
# 輸出: ['set-theme', '--color=#FFFFFF', '--dark-mode']
```
