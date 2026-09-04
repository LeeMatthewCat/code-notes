# 概念與命名規範

`pytest` 是 Python 最主流的測試框架，其核心特色之一為「自動搜尋機制 (Test Discovery)」。當你在終端機輸入 `pytest` 命令時，它會自動掃瞄整個專案目錄，尋找符合規範的測試檔與測試函數並自動執行，無須手動撰寫複雜的測試註冊邏輯。

`pytest` 搜尋測試程式碼時嚴格遵循「三鐵律」：

- **資料夾 (Directory)**：通常命名為 `tests/`，放於專案根目錄下，用於將測試程式碼與業務邏輯程式碼清晰區隔。
- **檔案 (File)**：命名必須為 `test_*.py` 或 `*_test.py`（例如 `test_calculator.py`）。只有符合此特徵的檔名才會被 `pytest` 視為測試檔載入，其他一般 `.py` 檔案都會被忽略。
- **函數 (Function)**：命名必須以 `test_` 開頭（例如 `def test_add_success():`）。`pytest` 進入測試檔後，只會將 `test_` 開頭的函數（或測試類別中 `test_` 開頭的方法）當作測試案例執行。

**命名規範的作用與底層機制**

為什麼需要這套命名規範？這是在做什麼？
1. **零配置自動發現 (Zero-Configuration Discovery)**：`pytest` 透過檔名與函數名的特定前綴，可以在不需撰寫任何測試清單或配置檔檔案的情況下，自動掃瞄、匯入並執行所有測試案例。
2. **隔離生產環境程式碼**：防止 `pytest` 在執行測試時，誤將專案中的普通腳本、工具函數或生產環境程式碼當作測試執行。
3. **明確程式碼意圖**：統一團隊規範，開發者一看到 `test_` 前綴就能立刻識別該函數是用於驗證特定業務邏輯的測試程式碼。

**原檔案與測試檔的架構對照**

在實際專案中，測試檔案通常會鏡像（Mirror）原始業務邏輯檔案的目錄結構。原始檔案放置於專案主模組內，而測試檔案則集中放在 `tests/` 資料夾下，並以 `test_` 作為檔名前綴。

專案目錄結構對照範例：

```text
my_project/
├── calculator.py          # 原始業務邏輯檔案（生產程式碼）
└── tests/                 # 測試資料夾
    └── test_calculator.py  # 對應的測試檔案
```

**原函數與測試用函數的運作方式**

- **原函數 (Target Function)**：位於業務邏輯檔案（如 `calculator.py`），負責實作系統實際的功能與邏輯（例如計算加法或除法）。
- **測試用函數 (Test Function)**：位於測試檔案（如 `tests/test_calculator.py`），負責「匯入原函數」、「傳入特定輸入參數」、「呼叫原函數」並「斷言驗證回傳結果是否符合預期」。

以下為原函數與測試用函數互動的完整程式碼範例：

```python
# 1. 【原始檔案】：calculator.py (原函數)
def add(a: float, b: float) -> float:
    """原函數：計算兩數相加"""
    return a + b


def divide(a: float, b: float) -> float:
    """原函數：計算兩數相除，若除數為 0 則拋出 ValueError"""
    if b == 0:
        raise ValueError("除數不能為零")
    return a / b
```

```python
# 2. 【測試檔案】：tests/test_calculator.py (測試用函數)
import pytest
from calculator import add, divide  # 匯入要測試的原函數


def test_add():
    """測試用函數：驗證 add() 原函數是否能正確計算和"""
    result = add(2, 3)  # 調用原函數取得實際結果
    assert result == 5  # 使用 assert 驗證實際結果是否等於預期值 (5)


def test_add_negative_numbers():
    """測試用函數：驗證 add() 原函數處理負數的情況"""
    assert add(-1, -1) == -2
```

## pyproject.toml 專案設定與路徑配置

在現代 Python 專案中，推薦使用 `pyproject.toml` 來集中管理 `pytest` 的設定。這不僅能避免在命令列重複輸入繁瑣的引數，還能徹底解決測試檔匯入業務模組時常發生的 `ModuleNotFoundError`。

**常見問題：`ModuleNotFoundError: No module named '...'`**
當直接執行 `pytest` 時，Python 的 `sys.path` 可能未包含專案根目錄或 `src/` 目錄。透過在 `pyproject.toml` 中指定 `pythonpath`，即可優雅解決路徑載入問題。

```toml
[tool.pytest.ini_options]
# 1. 測試檔案搜尋目錄
testpaths = ["tests"]

# 2. 自動將專案根目錄或 src 加入 sys.path (解決 ModuleNotFoundError)
pythonpath = ["."]

# 3. 預設執行的引數 (詳細輸出、嚴格標籤檢查、顯示簡短錯誤堆疊)
addopts = "-v --strict-markers --tb=short"

# 4. 註冊自訂測試標籤 (避免標記時跳出未註冊警告)
markers = [
    "slow: 標記執行時間較長的測試案例",
    "integration: 標記與外部服務互動的整合測試",
    "api: 標記 API 相關測試",
]
```

---

# assert 斷言與標籤控制

## assert 斷言基礎

`assert` 是 Python 原生關鍵字，用於聲明「某個條件必須為 True」。

當條件為 `False` 時，Python 會拋出 `AssertionError`。`pytest` 透過重寫 AST (Abstract Syntax Tree)，大幅增強了 `assert` 的回報能力：當測試失敗時，`pytest` 會自動印出表達式中各個變數的實際數值與差異細節，無需使用傳統測試庫繁瑣的 `assertEqual` 語法。

**常見斷言情境與實作範例**：

1. **基礎數值與布林斷言**：直接比較兩數是否相等或滿足條件。
2. **浮點數比較 (`pytest.approx`)**：由於電腦浮點數計算的精度誤差（如 `0.1 + 0.2 != 0.3`），比較浮點數時應使用 `pytest.approx()`。
3. **容器與字串包含斷言**：使用 `in` 驗證元素是否存在於清單、字典或字串中。
4. **例外拋出斷言 (`pytest.raises`)**：使用 `with pytest.raises(ExceptionType)` 驗證原函數在異常輸入下是否正確拋出指定的 Exception，並可檢視錯誤訊息細節。

```python
import pytest
from calculator import divide


def test_assert_examples():
    # 1. 基礎數值與布林斷言
    assert divide(6, 2) == 3.0
    assert divide(5, 2) != 2.0
    assert (1 + 1 == 2) is True

    # 2. 浮點數斷言 (避免浮點數精確度誤差)
    assert 0.1 + 0.2 == pytest.approx(0.3)

    # 3. 容器與字串斷言
    fruits = ["apple", "banana", "cherry"]
    assert "apple" in fruits
    assert len(fruits) == 3

    # 4. 例外報錯斷言 (pytest.raises())
    with pytest.raises(ValueError) as exc_info:
        divide(10, 0)
    
    # 驗證例外訊息內容
    assert "除數不能為零" in str(exc_info.value)
```

## 測試標籤 (Markers) 與條件執行

`pytest` 提供豐富的裝飾器用來標記測試案例，以便在不同環境下控制測試的執行、跳過或預期失敗。

### 1. 條件跳過 (@pytest.mark.skip / skipif)

當測試案例在特定作業系統、Python 版本或缺乏環境變數時無法執行時，可以使用 `@pytest.mark.skip` 或 `@pytest.mark.skipif`。

```python
import sys
import os
import pytest


# 無條件跳過測試
@pytest.mark.skip(reason="功能尚未實作完成，暫時跳過")
def test_feature_in_progress():
    pass


# 滿足條件時才跳過測試
@pytest.mark.skipif(
    sys.platform != "win32",
    reason="僅適用於 Windows 作業系統"
)
def test_windows_only_feature():
    assert True


@pytest.mark.skipif(
    os.getenv("CI") is None,
    reason="僅在 CI/CD 環境中執行"
)
def test_ci_pipeline_only():
    assert True
```

### 2. 預期失敗 (@pytest.mark.xfail)

當你已知某個測試目前因為 Bug 未修復或功能未完成而會失敗時，使用 `@pytest.mark.xfail`。`pytest` 不會將此失敗視為測試中斷。

- **XFAIL (Expected Fail)**：測試如預期失敗（通過驗證）。
- **XPASS (Unexpected Pass)**：測試竟然成功了（若設定 `strict=True` 則會被視為失敗以提醒 Bug 已修復）。

```python
@pytest.mark.xfail(reason="已知的 Bug #1024，等待修復", strict=True)
def test_known_bug():
    # 當 strict=True 時，若此測試意外 PASS，pytest 會將其視為整體測試失敗
    assert 1 / 0 == 1
```

### 3. 自訂標籤與篩選執行 (@pytest.mark.custom)

可以為測試案例加上自訂標記（如 `@pytest.mark.slow`、`@pytest.mark.integration`），並在終端機中使用 `-m` 參數選擇性執行。

```python
@pytest.mark.slow
@pytest.mark.integration
def test_heavy_database_migration():
    # 耗時長或需要連線資料庫的測試
    assert True
```

**終端機執行命令範例**：

```shell
# 僅執行標記為 integration 的測試
pytest -m integration

# 排除標記為 slow 的測試 (常用於快速本機回饋)
pytest -m "not slow"

# 組合條件：執行 api 且非 slow 的測試
pytest -m "api and not slow"
```

---

# pytest.mark.parametrize() 批量測試

使用 `@pytest.mark.parametrize()` 裝飾器，可以將多組測試數據傳入同一個測試函數中，避免為不同的輸入輸出編寫重複的 `test_*` 函數，實現高效批量驗證。

**核心機制與優勢**：
- **獨立執行每個案例**：`pytest` 會將傳入的每組數據當作獨立的測試案例執行。若其中一組數據測試失敗，不會影響其他數據的驗證。
- **自訂案例標籤 (`ids`)**：可透過 `ids` 參數為每一組測試數據命名，使測試報告的可讀性更高。
- **笛卡兒積組合測試 (Stacking)**：可同時堆疊多個 `@pytest.mark.parametrize()`，自動組合出所有參數組合（Cartesian Product）。

**參數說明**：
- **第一個參數**：以逗點分隔的參數名稱字串（如 `"a, b, expected"`）。
- **第二個參數**：包含多組測試資料元組 (Tuple) 的列表（如 `[(1, 2, 3), (0, 0, 0)]`）。

```python
import pytest
from calculator import add, divide


# 1. 基礎參數化測試 (指定 ids 提升報告可讀性)
@pytest.mark.parametrize(
    "a, b, expected",
    [
        (1, 2, 3),          # 正數相加
        (0, 0, 0),          # 零與零相加
        (-1, 1, 0),         # 負數與正數相加
        (10.5, 4.5, 15.0)   # 浮點數相加
    ],
    ids=["positive_sum", "zero_sum", "mixed_sign_sum", "float_sum"]
)
def test_add_parametrize(a: float, b: float, expected: float):
    # 測試函數將自動執行 4 次，每次帶入一組 (a, b, expected)
    assert add(a, b) == expected


# 2. 批量驗證例外拋出
@pytest.mark.parametrize(
    "a, b",
    [
        (10, 0),
        (-5, 0),
        (0, 0)
    ],
    ids=["pos_by_zero", "neg_by_zero", "zero_by_zero"]
)
def test_divide_by_zero_parametrize(a: float, b: float):
    with pytest.raises(ValueError):
        divide(a, b)


# 3. 堆疊裝飾器：產生參數組合測試 (2x2 = 4 次執行)
@pytest.mark.parametrize("x", [1, 2])
@pytest.mark.parametrize("y", [10, 20])
def test_cartesian_product(x: int, y: int):
    # 自動測試 (x=1, y=10), (x=1, y=20), (x=2, y=10), (x=2, y=20)
    assert x + y > 5
```

---

# 測試資料準備與環境隔離

## pytest.fixture() 測試資料預備

### pytest.fixture() 基礎

`pytest.fixture()` 是 pytest 框架中最核心的靈魂機制，用於預備測試所需的 **context 環境、資料庫連線、Mock 物件**（**模擬物件**，即用來替換真實物件並記錄呼叫行為的「替身物件」）或測試數據。相較於傳統 `unittest` 的 `setUp/tearDown` 繼承模式，`fixture` 採用高彈性的「**依賴注入 (Dependency Injection)**」模式與模組化設計。

> **💡 什麼是「依賴注入 (Dependency Injection, DI)」？**
> 
> 依賴注入是一種軟體設計模式。在測試框架中，當測試案例需要某個物件或測試資料（即「依賴」）時，**測試案例不需要自己去建立或初始化它**，而是由 `pytest` 在執行測試前自動建立好，並以函數參數的形式「注入」傳給測試案例。
> 
> **主要優勢**：
> 1. **解耦（降低程式碼或系統元件之間的相依程度）與高重用性**：將測試資料的「建立／清理邏輯」與「測試驗證邏輯」完全分離，多個測試案例可重複調用同一個 fixture。
> 2. **自動生命週期管理**：測試案例只需表達「我需要這個資源」，由框架負責資源的 Setup（初始化）與 Teardown（自動清理銷毀）。

**1. 生命週期管理與 Setup/Teardown 雙模式**

Fixture 透過 `yield` 或 `request.addfinalizer()` 區分 **Setup 準備階段**與 **Teardown 清理階段**：

- **`yield` 模式（推薦主流）**：`yield` 之前的程式碼在測試案例執行前運行（Setup）；`yield` 之後的程式碼在測試案例結束後自動運行（Teardown）。
- **`finalizer` 模式（進階）**：透過注入 `request` fixture 註冊清理函數，優點是即使 Setup 過程中發生例外拋出，已註冊的 finalizer 仍會執行。

```python
import pytest


# 模式 A：yield 模式 (最直觀且常用)
@pytest.fixture
def managed_resource():
    # [Setup] 建立資源
    resource = {"status": "active", "id": 101}
    print("
[Setup] 資源已創建")
    
    yield resource  # 提供給測試函數使用
    
    # [Teardown] 清理資源
    resource["status"] = "destroyed"
    print("
[Teardown] 資源已釋放")


# 模式 B：request.addfinalizer 模式
@pytest.fixture
def finalizer_resource(request):
    resource = {"status": "active"}
    
    def cleanup():
        resource["status"] = "destroyed"
        print("
[Finalizer] 資源已透過終結器清理")
        
    request.addfinalizer(cleanup)  # 註冊清理回調
    return resource


# 3. 【與被測試函數連接範例】：直接將 Fixture 名稱放進測試函數的參數中
def test_using_managed_resource(managed_resource: dict):
    # pytest 自動執行 managed_resource 的 Setup -> yield 資源傳給引數 managed_resource
    assert managed_resource["status"] == "active"
    assert managed_resource["id"] == 101
    # 函數執行結束後，pytest 會回到 managed_resource 繼續執行 yield 之後的 Teardown 清理


def test_using_finalizer_resource(finalizer_resource: dict):
    # pytest 自動執行 finalizer_resource -> 回傳資源 -> 測試結束後觸發 cleanup 回調
    assert finalizer_resource["status"] == "active"
```

**2. 作用域層級 (Scope)**

透過 `@pytest.fixture(scope=...)` 可控制資源的建立頻率與生命週期：

- **`function`（預設）**：每個測試函數執行前建立一次，執行完後銷毀（隔離性最高）。
- **`class`**：每個測試類別建立一次，類別內所有測試方法共享。
- **`module`**：每個測試檔 (`.py`) 建立一次，適合檔案級別的連線或測試檔共用數據。
- **`package`**：每個測試套件（資料夾）建立一次（pytest 7+ 支援）。
- **`session`**：整個測試會話（全域）僅建立一次，適合極昂貴的資源開銷（如啟動 Docker 容器或測試資料庫）。

**3. Fixture 互相鏈結與依賴組合 (Fixture Chaining)**

Fixture 不僅能注入測試函數，**Fixture 本身也可以注入其他 Fixture**。`pytest` 會自動解析依賴關係圖 (DAG) 並按正確順序執行。

**💡 這樣做的核心意義與優勢是什麼？**
1. **職責分離與模組化 (Single Responsibility)**：每個 Fixture 只專注做好一件事（例如一個負責載入設定檔、一個負責建立資料庫連線、一個負責預載測試資料），避免單一 Fixture 變得過於龐大複雜。
2. **差異化 Scope 生命週期優化（大幅提升測試速度）**：
   - 可以將開銷極昂貴的「基礎設施」（如連線池或設定檔）設為 `scope="session"`（整個測試過程只執行 1 次）。
   - 將需要頻繁重置的「測試數據」設為 `scope="function"`（每個測試前獨立建立）。
   - 透過鏈結，`function` 層級 of Fixture 可以直接繼承並重用 `session` 層級的基礎連線，兼顧測試隔離與執行效率！
3. **組裝式高重用性 (Composable Reusability)**：不同的測試案例可以根據需求，隨心組合不同層級的 Fixture，無需重複撰寫初始化邏輯。

```python
import pytest


@pytest.fixture(scope="session")
def db_config():
    return {"host": "localhost", "port": 5432}


@pytest.fixture(scope="session")
def db_connection(db_config):
    # db_connection 依賴 db_config
    conn = f"Connected to {db_config['host']}:{db_config['port']}"
    return conn


@pytest.fixture
def user_data(db_connection):
    # user_data 依賴 db_connection
    return {"db": db_connection, "user": "matthew"}


def test_user_creation(user_data):
    # 自動依序載入 db_config -> db_connection -> user_data
    assert "localhost:5432" in user_data["db"]
    assert user_data["user"] == "matthew"
```

**4. Fixture 參數化 (Parametrized Fixtures)**

不僅測試函數可以參數化，Fixture 本身也可以透過 `params` 參數進行參數化！當 Fixture 被參數化時，所有依賴該 Fixture 的測試案例都會自動針對每組參數重複執行。

```python
import pytest


# 透過 params 傳入多種資料庫驅動設定
@pytest.fixture(params=["sqlite", "postgresql", "mysql"])
def db_engine(request):
    # 透過 request.param 取出目前的參數值
    engine_type = request.param
    return f"Engine({engine_type})"


def test_db_operations(db_engine):
    # 此測試案例會自動執行 3 次：
    # 第一次執行：db_engine 為 "Engine(sqlite)"     -> 驗證 db_engine 開頭為 "Engine(" 且包含 "sqlite"
    # 第二次執行：db_engine 為 "Engine(postgresql)" -> 驗證 db_engine 開頭為 "Engine(" 且包含 "postgresql"
    # 第三次執行：db_engine 為 "Engine(mysql)"      -> 驗證 db_engine 開頭為 "Engine(" 且包含 "mysql"
    assert db_engine.startswith("Engine(")
```

**5. 跨檔案共享機制與內建 Fixtures**

- **`conftest.py` 共享機制**：在 `tests/conftest.py` 中定義的 Fixture，無需任何 `import` 即可被該目錄及子目錄下的所有測試檔自動使用。
- **自動執行 (`autouse=True`)**：標註 `autouse=True` 的 Fixture 會自動在對應作用域內的所有測試前執行（適合用於開啟全域 Timer、環境變數初始化或日誌記錄）。
- **常用內建 Fixtures**：
  - `capsys`：捕捉標準輸出/標準錯誤 (`sys.stdout` / `sys.stderr`)。
  - `tmp_path`：提供自動建立且測試後會自動清理的臨時目錄 (`pathlib.Path` 物件)。
  - `monkeypatch`：安全地動態修改環境變數、字典或物件屬性，測試結束自動還原。

---

### capsys 捕捉標準輸出與標準錯誤

`capsys` 是 `pytest` 內建的 Fixture。它在測試執行期間，會自動攔截並覆蓋 Python 底層的 `sys.stdout`（標準輸出）與 `sys.stderr`（標準錯誤）。

這代表它能捕捉所有透過 `print()` 函數、`sys.stdout.write()` 或 `sys.stderr.write()` 輸出的文字串流。

被攔截的輸出不會直接印在終端機畫面上，而是被暫存於記憶體中的虛擬緩衝區，以供測試進行斷言驗證。

#### capsys 具體捕捉的對象

**核心定義**：
`capsys` 具體捕捉的是 Python 執行期間寫入標準輸出與標準錯誤流的字串資料。在底層，它攔截了原本指向系統終端機的資料流，並將其重新導向至記憶體中的虛擬緩衝區。

**箇條說明**：
- **捕捉 `sys.stdout` 標準輸出**
    - 包含所有使用 `print()` 函數輸出的文字。
    - 包含直接呼叫 `sys.stdout.write()` 所寫入的資料。
    - 用於驗證程式的一般命令列輸出是否符合預期。
- **捕捉 `sys.stderr` 標準錯誤**
    - 包含透過 `print(..., file=sys.stderr)` 輸出的錯誤訊息。
    - 包含直接對 `sys.stderr.write()` 寫入的診斷或警告資訊。
    - 用於檢查非致命性的錯誤提示與系統日誌輸出。

**程式碼證明**：
以下為說明 `capsys` 捕捉對象的具體測試程式碼：

```python
import sys


def output_to_both():
    # 1. 寫入標準輸出 (stdout)
    print("這是標準輸出訊息")
    sys.stdout.write("這是 stdout.write 寫入的內容
")

    # 2. 寫入標準錯誤 (stderr)
    print("這是標準錯誤訊息", file=sys.stderr)
    sys.stderr.write("這是 stderr.write 寫入的內容
")


def test_capsys_capture_detail(capsys):
    # 呼叫產生輸出的業務函數
    output_to_both()

    # 讀取並重設快取
    captured = capsys.readouterr()

    # 3. 驗證標準輸出捕捉內容
    assert "這是標準輸出訊息" in captured.out
    assert "這是 stdout.write 寫入的內容" in captured.out

    # 4. 驗證標準錯誤捕捉內容
    assert "這是標準錯誤訊息" in captured.err
    assert "這是 stderr.write 寫入的內容" in captured.err
```

**陷阱提醒**：
> **💡 小提醒**：`capsys` 只能捕捉由 Python 進程內部直接產生的輸出。如果是透過 Python 呼叫外部 C 擴充套件，或者呼叫系統指令（例如使用 `os.system()` 或 `subprocess`）所產生的 C 等級輸出，`capsys` 將無法攔截！此時必須改用 `capfd` 才能成功捕捉。
> 
> **💡 小提醒**：一旦呼叫了 `capsys.readouterr()`，內部的輸出快取便會被立刻重設（清空）。如果需要對同一批輸出做多次斷言，請務必先將回傳值指派給變數，再對該變數進行多次驗證。

#### capsys.readouterr() 讀取並重設輸出快取

用於讀取自測試開始或上一次呼叫以來累積的標準輸出與標準錯誤內容，呼叫後會自動清空快取。

- **語法**：`capsys.readouterr()`
- **說明**：無須傳入參數。呼叫後回傳包含 `out` 與 `err` 兩個字串屬性的元組物件。
- **回傳值**：回傳 `CaptureResult(out: str, err: str)` 具名元組，`out` 為 `sys.stdout` 的捕捉內容，`err` 為 `sys.stderr` 的捕捉內容。

```python
import sys


def print_hello():
    print("Hello, stdout!")
    print("Error message!", file=sys.stderr)


def test_capsys_example(capsys):
    print_hello()

    # 讀取並重設快取
    captured = capsys.readouterr()

    assert "Hello, stdout!" in captured.out
    assert "Error message!" in captured.err
```

#### capsys.disabled() 暫時停用輸出捕捉

用於在 `with` 上下文區塊內暫時停用 `capsys` 的捕捉功能，使印出訊息能直接顯示於終端機螢幕上。

- **語法**：`capsys.disabled()`
- **說明**：作為上下文管理器 (`with`) 使用，無須傳入參數。
- **回傳值**：回傳上下文管理器物件。

```python
def test_capsys_disabled_example(capsys):
    print("這段輸出會被 capsys 捕捉")

    with capsys.disabled():
        print("這段輸出會直接印在終端機畫面上，不會被 capsys 捕捉")

    captured = capsys.readouterr()
    assert "這段輸出會被 capsys 捕捉" in captured.out
    assert "直接印在終端機畫面上" not in captured.out
```

---

### tmp_path 臨時目錄管理

`tmp_path` 是 `pytest` 內建的 Fixture，為每個測試案例提供一個獨立且自動清理的臨時目錄物件（採用 Python 原生 `pathlib.Path` 型別）。

#### tmp_path 建立與操作臨時檔案

用於在測試期間建立臨時檔案或資料夾，測試結束後 `pytest` 會自動清除該資料夾。

- **語法**：`tmp_path` (作為測試函數參數注入，型別為 `pathlib.Path`)
- **說明**：`tmp_path` 為 `pathlib.Path` 實體，可直接使用 `/` 運算子串接子目錄或檔名（如 `tmp_path / "sub" / "file.txt"`）。
- **參數與屬性**：`tmp_path` 為 `pathlib.Path` 物件，指向目前測試案例專屬的臨時目錄。
- **回傳值**：回傳指向臨時目錄的 `pathlib.Path` 物件。

```python
from pathlib import Path


def write_config(filepath: Path, content: str):
    """業務函數：將內容寫入指定檔案"""
    filepath.write_text(content, encoding="utf-8")


def test_tmp_path_example(tmp_path):
    # 建立臨時子目錄與檔案路徑
    test_file = tmp_path / "config.json"

    # 呼叫業務函數寫入檔案
    write_config(test_file, '{"env": "test"}')

    # 斷言檔案確實存在且內容正確
    assert test_file.exists()
    assert test_file.read_text(encoding="utf-8") == '{"env": "test"}'
```

---

### monkeypatch 測試環境隔離與動態替換

`monkeypatch` 是 `pytest` 內建的核心 Fixture，專門用來在單個測試執行期間**動態修改或替換環境變數、物件屬性、模組函數或字典**。

**核心優勢與底層機制**：
若直接使用 `os.environ["KEY"] = "VALUE"` 或直接改寫模組屬性，修改會永久保留在 Python 行程中，污染後續其他測試案例。`monkeypatch` 會在測試前記錄原始狀態，**測試結束時（無論成功或拋錯）自動還原所有修改**，確保測試案例之間的絕對隔離。

#### monkeypatch.setenv() 設定環境變數

用於在單個測試期間設定或覆蓋系統環境變數 (`os.environ`)。

- **語法**：`monkeypatch.setenv(name: str, value: str, prepend: str | None = None)`
- **說明**：`name` 為變數名，`value` 為值（必須是字串）。若傳入 `prepend=":"` 可將新路徑插在路徑變數（如 `PATH`）最前面。
- **回傳值**：`None`。

```python
import os


def test_setenv_example(monkeypatch):
    # 安全設定 APP_ENV，測試結束後自動還原
    monkeypatch.setenv("APP_ENV", "testing")
    assert os.getenv("APP_ENV") == "testing"
```

#### monkeypatch.delenv() 刪除環境變數

用於在單個測試期間暫時刪除某個環境變數，驗證系統在缺乏該環境變數時的預設行為或報錯機制。

- **語法**：`monkeypatch.delenv(name: str, raising: bool = True)`
- **說明**：`name` 為變數名，`raising=False` 可防止在變數原本就不存在時拋出 `KeyError`。
- **回傳值**：`None`。

```python
import os
import pytest


def test_delenv_example(monkeypatch):
    monkeypatch.setenv("DB_PASSWORD", "secret123")
    
    # 暫時刪除 DB_PASSWORD
    monkeypatch.delenv("DB_PASSWORD", raising=False)
    assert os.getenv("DB_PASSWORD") is None
```

#### monkeypatch.setattr() 替換物件與模組屬性

用於在單個測試期間暫時替換類別、物件或模組中的函數或屬性（例如跳過耗時的 `time.sleep` 或替換回傳值）。

- **語法**：`monkeypatch.setattr(target, name, value, raising: bool = True)`
- **說明**：可直接傳入物件 `target` 與屬性名 `name`，或是傳入字串路徑如 `"module.path.func"`。
- **回傳值**：`None`。

```python
import time


def slow_function():
    time.sleep(10)  # 耗時 10 秒
    return "completed"


def test_setattr_example(monkeypatch):
    # 替換 time.sleep 假函數，讓它立刻回傳 0，無須等待 10 秒
    monkeypatch.setattr(time, "sleep", lambda x: 0)
    assert slow_function() == "completed"
```

> **💡 為什麼使用 `lambda x: 0` 而不直接傳入 `0` 或 `time.sleep(0)`？**
> 
> 1. **不能直接傳入 `0`**：
>    因為業務程式碼中會呼叫 `time.sleep(10)`，這代表被替換後的屬性必須是一個可以接收參數並被呼叫的**函數 (Callable)**。若直接傳入整數 `0`，執行時會等同於執行 `0(10)`，這將拋出 `TypeError: 'int' object is not callable` 錯誤。
> 
> 2. **不能寫成 `time.sleep(0)`**：
>    如果寫成 `monkeypatch.setattr(time, "sleep", time.sleep(0))`，Python 會在設定的當下**立即執行** `time.sleep(0)`。由於該函數執行完回傳的是 `None`，這等同於把 `time.sleep` 替換成 `None`。當業務程式碼呼叫 `time.sleep(10)` 時，便會等同於執行 `None(10)`，從而拋出 `TypeError: 'NoneType' object is not callable` 錯誤。

#### monkeypatch.delattr() 刪除物件與模組屬性

用於在單個測試期間暫時刪除物件或模組的屬性，測試結束自動恢復。

- **語法**：`monkeypatch.delattr(target, name, raising: bool = True)`
- **說明**：`target` 為目標物件或模組，`name` 為屬性名稱。
- **回傳值**：`None`。

```python
class Config:
    debug = True


def test_delattr_example(monkeypatch):
    # 暫時刪除 Config 的 debug 屬性
    monkeypatch.delattr(Config, "debug", raising=False)
    assert not hasattr(Config, "debug")
```

#### monkeypatch.setitem() 修改字典與容器項目

用於在單個測試期間暫時修改字典或類字典物件的鍵值對（例如修改 `os.environ` 字典或配置檔字典）。

- **語法**：`monkeypatch.setitem(dic: dict, name, value)`
- **說明**：`dic` 為目標字典，`name` 為鍵名，`value` 為欲設定的值。
- **回傳值**：`None`。

```python
app_config = {"theme": "light", "max_users": 100}


def test_setitem_example(monkeypatch):
    # 暫時修改字典配置
    monkeypatch.setitem(app_config, "theme", "dark")
    assert app_config["theme"] == "dark"
```

#### monkeypatch.delitem() 刪除字典與容器項目

用於在單個測試期間暫時刪除字典中的特定鍵值對。

- **語法**：`monkeypatch.delitem(dic: dict, name, raising: bool = True)`
- **說明**：`dic` 為目標字典，`name` 為要刪除的鍵名。
- **回傳值**：`None`。

```python
user_session = {"user_id": 42, "token": "xyz"}


def test_delitem_example(monkeypatch):
    # 暫時刪除 token 鍵
    monkeypatch.delitem(user_session, "token", raising=False)
    assert "token" not in user_session
```

#### monkeypatch.syspath_prepend() 擴充 Python 模組搜尋路徑

用於在單個測試期間暫時將特定目錄插到 Python `sys.path` 的最前方，方便動態載入特定測試模組。

- **語法**：`monkeypatch.syspath_prepend(path)`
- **說明**：`path` 為欲加入 `sys.path` 最前方的字串或 `Path` 路徑。
- **回傳值**：`None`。

```python
import sys


def test_syspath_example(monkeypatch, tmp_path):
    # 將臨時目錄加入 sys.path
    monkeypatch.syspath_prepend(str(tmp_path))
    assert str(tmp_path) in sys.path
```

---

## 時間模擬與狀態隔離 (Freezing Time)

許多業務邏輯會依賴 `datetime.now()` 或時間戳記（例如 Token 過期、優惠券計時、訂單建立時間）。若直接測試真實時間，會導致測試結果不可預測。

可以使用 `freezegun` 或 `time-machine` 庫來「凍結時間」。

```shell
pip install freezegun
```

```python
from datetime import datetime
from freezegun import freeze_time


def is_discount_active() -> bool:
    """業務函數：判斷目前時間是否在雙十一優惠期間 (2026-11-11)"""
    now = datetime.now()
    return now.year == 2026 and now.month == 11 and now.day == 11


# 使用裝飾器將 datetime.now() 鎖定在指定時間點
@freeze_time("2026-11-11 12:00:00")
def test_is_discount_active_during_promo():
    assert is_discount_active() is True


@freeze_time("2026-11-12 00:00:01")
def test_is_discount_active_after_promo():
    assert is_discount_active() is False
```

---

# unittest.mock 隔離外部依賴

單元測試必須保持獨立、穩定與快速。
當業務邏輯依賴網路 API、第三方服務或資料庫時，必須進行隔離。

此時應使用 Mock 物件替換真實呼叫。
確保測試不會因為網路中斷、第三方異常或資料庫污染而失敗。

## Mock 與 MagicMock 的核心概念

在 Python 中，`Mock` 與 `MagicMock` 是建構模擬物件的兩大主力。
`MagicMock` 繼承自 `Mock`，預先實作了絕大多數 Python 的魔術方法。

#### unittest.mock.Mock() 建立基礎模擬物件

用於模擬一般的 Python 物件、屬性或函數，記錄調用歷史與參數。

- **語法**：`mock_obj = Mock(return_value=None, side_effect=None, spec=None)`
- **說明**：
  - `return_value`：設定調用模擬物件時要回傳的假結果。
  - `side_effect`：傳入例外時會拋出異常；傳入列表時會依序回傳值；傳入函式時會動態計算值。
  - 可使用各種斷言方法驗證調用，如 `assert_called_once_with()`。
- **回傳值**：回傳一個新的 `Mock` 實例。

```python
from unittest.mock import Mock

# 建立模擬函數，設定回傳值
add_mock = Mock(return_value=10)
assert add_mock(2, 3) == 10

# 驗證該模擬函數確實被呼叫過
add_mock.assert_called_once_with(2, 3)
```

#### unittest.mock.MagicMock() 建立魔術方法模擬物件

繼承自 `Mock`，預先實作了 Python 所有魔術方法，常用於模擬上下文管理器、容器或疊代器。

- **語法**：`magic_mock_obj = MagicMock(return_value=None, side_effect=None, spec=None)`
- **說明**：
  - 預設支援如 `__len__`、`__iter__`、`__enter__`/`__exit__` 等魔術方法。
  - 適用於需要模擬 `with` 上下文、`len()` 長度計算或 `for` 迴圈遍歷的物件。
- **回傳值**：回傳一個新的 `MagicMock` 實例。

```python
from unittest.mock import MagicMock

# 模擬一個上下文管理器與長度計算
mock_conn = MagicMock()
mock_conn.__len__.return_value = 5

assert len(mock_conn) == 5
```

## 動態替換外部依賴

在測試過程中，我們需要將現有的模組函數或類別動態替換為 Mock 物件。
此時可以使用 `patch()` 工具。

> **⚠️ patch() 核心黃金法則：Patch 在哪裡被使用，而不是在哪裡被定義！**
> 
> 假設 `services.py` 內有 `from utils import send_email`：
> - ❌ **錯誤寫法**：`@patch("utils.send_email")`（因為 `services.py` 已經匯入自己的命名空間）。
> - ✅ **強烈建議**：`@patch("services.send_email")`（直接替換 `services` 模組正在使用的那個引用）。

#### unittest.mock.patch() 動態替換目標

在測試期間，將指定路徑的模組屬性、類別或函數動態替換為模擬物件，並在測試結束後自動還原。

- **語法**：`patch(target, new=DEFAULT, spec=None, create=False)`
- **說明**：
  - 可作為裝飾器 `@patch("module.target")` 注入測試函數。
  - 可作為上下文管理器 `with patch("module.target") as mock_obj:` 限制替換範圍。
  - 測試結束後，`patch` 會自動清理並還原原本的函數或類別，避免污染其他測試。
- **回傳值**：回傳一個上下文管理器或裝飾器，會將建立的 Mock 物件作為參數傳入。

```python
import requests
import pytest
from unittest.mock import patch, MagicMock


def get_user_profile(user_id: int) -> dict:
    """業務函數：發送 HTTP 請求取得使用者資料"""
    response = requests.get(f"https://api.example.com/users/{user_id}", timeout=5)
    if response.status_code == 200:
        return response.json()
    raise RuntimeError("API 呼叫失敗")


# 1. 裝飾器形態 + return_value 設定
@patch("requests.get")
def test_get_user_profile_success(mock_get):
    # 配置回傳值
    mock_get.return_value.status_code = 200
    mock_get.return_value.json.return_value = {"id": 1, "username": "matthew"}

    # 執行測試
    profile = get_user_profile(1)
    
    # 斷言驗證
    assert profile["username"] == "matthew"
    mock_get.assert_called_once_with("https://api.example.com/users/1", timeout=5)


# 2. side_effect 拋出例外情境驗證
@patch("requests.get")
def test_get_user_profile_timeout(mock_get):
    # 設定 side_effect 為例外
    mock_get.side_effect = requests.exceptions.Timeout("連線超時")

    # 驗證是否拋出預期例外
    with pytest.raises(requests.exceptions.Timeout):
        get_user_profile(1)
```

## 異步與規格化模擬

在現代 Python 專案中，異步協程 (Asyncio) 與程式碼健壯性是非常核心的議題。
因此需要特別的 Mock 機制進行處理。

#### unittest.mock.AsyncMock() 模擬異步函數

專門用來模擬異步函數 (`async def`) 的物件，支援 `await` 呼叫並提供非同步斷言。

- **語法**：`async_mock = AsyncMock(spec=None, **kwargs)`
- **說明**：
  - 解決一般 `Mock` 無法被 `await` 的問題。
  - 提供專屬非同步驗證方法，如 `assert_awaited_once_with()` 等。
- **回傳值**：回傳一個新的 `AsyncMock` 實例。

```python
import pytest
from unittest.mock import AsyncMock


class AsyncHttpClient:
    async def fetch_json(self, url: str) -> dict:
        # 實際非同步 HTTP 請求
        pass


async def get_dashboard_data(client: AsyncHttpClient) -> dict:
    """業務函數：依賴非同步 HTTP 用戶端"""
    data = await client.fetch_json("https://api.example.com/dashboard")
    return {"status": "ok", "payload": data}


@pytest.mark.asyncio
async def test_get_dashboard_data():
    # 建立 AsyncMock
    mock_client = AsyncMock(spec=AsyncHttpClient)
    mock_client.fetch_json.return_value = {"visitors": 1050}

    # 呼叫包含 await 的業務函數
    result = await get_dashboard_data(mock_client)

    # 驗證回傳結果與非同步呼叫斷言
    assert result["payload"]["visitors"] == 1050
    mock_client.fetch_json.assert_awaited_once_with("https://api.example.com/dashboard")
```

#### unittest.mock.create_autospec() 建立規格化模擬

根據真實類別或函數的簽名 (Signature)，自動建立具有嚴格參數校驗的模擬物件。

- **語法**：`mock_obj = create_autospec(spec, spec_set=False, instance=False)`
- **說明**：
  - 防止測試中「傳錯參數數量或名稱卻依舊通過」的虛假測試。
  - 若調用時傳入的參數與真實物件不符，會立刻拋出 `TypeError`。
- **回傳值**：回傳一個具有與 `spec` 相同介面與簽名的 Mock 物件。

```python
from unittest.mock import create_autospec


def real_function(a: int, b: str) -> bool:
    return True


def test_with_autospec():
    # 建立嚴格遵循 real_function 簽名的 Mock
    mock_func = create_autospec(real_function, return_value=True)

    # ✅ 正確呼叫
    mock_func(1, "hello")

    # ❌ 傳入錯誤參數數量（例如少傳引數或傳錯引數名），會立刻拋出 TypeError，防止無效測試！
    # mock_func(1)  # 拋出 TypeError: missing a required argument: 'b'
```

---

# 測試執行、除錯與最佳實踐

## 常用 pytest CLI 命令與除錯技巧

掌握終端機命令列引數，能大幅提升編寫與排查測試錯誤的效率。

| CLI 引數 | 說明與適用情境 |
| :--- | :--- |
| `pytest -v` | **詳細模式 (Verbose)**：印出每一個測試案例的完整名稱與結果狀態。 |
| `pytest -s` | **取消輸出捕捉**：允許測試過程中的 `print()` 訊息直接印在終端機螢幕上（除錯常用）。 |
| `pytest -x` | **首錯即停 (Exit on First Failure)**：一旦遇到第一個失敗案例就立刻中斷執行。 |
| `pytest --maxfail=N` | **上限中斷**：累積失敗案例達到 `N` 個時中斷執行（如 `--maxfail=3`）。 |
| `pytest -k "expression"` | **關鍵字篩選**：僅執行函數名稱符合表達式的測試（如 `pytest -k "add or divide"`）。 |
| `pytest --lf` | **僅重跑失敗案例 (Last Failed)**：只執行上一次測試中失敗的案例。 |
| `pytest --ff` | **失敗案例優先 (Failed First)**：執行所有測試，但優先執行上次失敗的案例。 |
| `pytest --pdb` | **失敗時自動進入 PDB**：測試拋錯時自動觸發 Python 原生除錯器 (`pdb`) 進行互動檢查。 |
| `pytest -q` | **精簡模式 (Quiet)**：僅顯示精簡測試摘要資訊。 |

**實用除錯組合命令**：

```shell
# 1. 鎖定單一測試檔，只印 print 訊息並在失敗時開啟 pdb 除錯
pytest tests/test_calculator.py -s --pdb

# 2. 僅重跑上次失敗的案例並在第一個失敗處停止
pytest --lf -x
```

## 測試設計原則與最佳實踐

良好的測試程式碼與良好的生產程式碼同等重要。遵循以下設計原則可確保測試套件易於維護且穩定可靠。

### 1. AAA 模式 (Arrange-Act-Assert)

將每個測試函數的結構明確劃分為三個區段，提升可讀性：

- **Arrange (準備)**：設置測試所需的所有資料、物件與狀態。
- **Act (執行)**：呼叫要測試的目標原函數，並記錄傳回值或狀態變更。
- **Assert (驗證)**：檢查執行結果與預期是否一致。

```python
def test_shopping_cart_checkout():
    # 1. Arrange (準備)
    cart = ShoppingCart()
    item = Product(name="Python 書籍", price=500)
    cart.add_item(item)

    # 2. Act (執行)
    total_price = cart.checkout()

    # 3. Assert (驗證)
    assert total_price == 500
    assert cart.is_empty() is True
```

### 2. FIRST 原則

高質量的單元測試應符合 FIRST 原則：

- **Fast (快速)**：測試執行速度必須極快（毫秒級），以便開發者在編程時頻繁執行。
- **Independent / Isolated (獨立性)**：測試案例之間絕不能互相依賴或有順序限制。每個案例應能獨立單獨執行。
- **Repeatable (可重複性)**：在任何環境（本機、CI/CD server、無網路環境）下執行多回，結果都必須一致。
- **Self-Validating (自我驗證)**：測試結果應為明確的 Pass 或 Fail（由斷言自動判斷），無需人工目視檢查 Log。
- **Timely (及時性)**：測試案例應與業務程式碼同步編寫（甚至採用 TDD 測試驅動開發）。

### 3. 測試金字塔 (Testing Pyramid)

在專案測試規劃中，應按適當比例配置不同層級的測試：

- **單元測試 (Unit Tests)**：佔比最高 (~70%)。執行速度最快，隔離外部依賴，專注驗證單一函數或類別的邏輯。
- **整合測試 (Integration Tests)**：佔比中等 (~20%)。驗證多個模組之間的協同運作（如 Service 與 Database）。
- **端到端測試 (E2E Tests)**：佔比最低 (~10%)。模擬真實使用者完整流程，維護成本與執行開銷最高。

---

# 覆蓋率與 CI/CD 自動化

**1. pytest-cov 測試覆蓋率統計**

驗證測試程式碼覆蓋了多少比例的業務邏輯。

```shell
pip install pytest-cov
```

```shell
# 執行測試並顯示缺少覆蓋的行號
pytest --cov=src --cov-report=term-missing
```

**2. GitHub Actions CI 自動化整合**

在每次 `git push` 時自動執行 `pytest`。設定檔位於 `.github/workflows/test.yml`：

```yaml
name: Run Tests

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up Python
      uses: actions/setup-python@v5
      with:
        python-version: "3.11"

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install pytest pytest-cov pytest-asyncio

    - name: Run pytest
      run: |
        pytest --cov=src --cov-report=xml
```
