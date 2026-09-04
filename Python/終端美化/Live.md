mseeages=history_messages# 概念與原理

## 什麼是 Rich Live？

Python 最強終端美化庫 `rich` 中的**動態畫面即時渲染器 (Dynamic Live Display)**。

在傳統控制台程式中，使用 `print()` 輸出資訊就像是**「傳統紙張列印機」**，資料只能不斷往下追加滾動，舊文字一旦印出就無法修改。

`rich.live.Live` 將終端機區域升級為**「液晶電視螢幕 (Dynamic Canvas)」**。它允許你在控制台的固定區域內，以極高的影格率 (FPS) **即時重繪、動態刷新與更新任何 Rich 元件**（如動態表格 [[Rich|Table]]、即時狀態、進度條與 Dashboard 儀表板），完全不會造成畫面閃爍或滾動雜亂！

> **🍿 生動白話比喻**：  
> - 傳統 `print()`：像是**傳真機印紙**，每秒吐出一行新紙，想更新資訊只能再吐出一行新紙（舊資料留在那裡淹沒螢幕）。  
> - `rich.live.Live`：像是**控制台裡的 iPad 螢幕**！你可以在螢幕上放一張表格或跑馬燈，畫面上的數字每秒動態跳動（`1%... 50%... 100%`），但紙張完全不會往下滾動！

---

##### Live() 建立動態刷新上下文

- **使用時機**：當你需要建立一個即時動態畫面的情境管理器，讓畫面可以在固定區域內持續更新時。
- **語法**：`Live(renderable=None, console=None, auto_refresh=True, refresh_per_second=4, transient=False, vertical_overflow="crop", ...)`
- **參數說明**：

| 參數名稱 | 資料型別 | 預設值 | 說明 |
| :--- | :--- | :--- | :--- |
| **`renderable`** | `Renderable` / `str` | `None` | **初始渲染物件**。可傳入 Table、Panel、Group 或文字。 |
| **`console`** | `Console` | `None` | 指定使用的 Console 實例（未設定自動使用全域 Console）。 |
| **`auto_refresh`** | `bool` | `True` | **是否開啟自動定期刷新**。若設為 `False` 需手動呼叫 `.refresh()`。 |
| **`refresh_per_second`** | `float` / `int` | `4` | **每秒畫面刷新次數 (FPS)**。預設每秒刷新 4 次，最高可調高（如 10 或 20）。 |
| **`transient`** | `bool` | `False` | **離開時是否隱藏畫面**。若設為 `True`，`with` 結束時會清除終端機上的動態畫面。 |
| **`vertical_overflow`** | `str` | `"crop"` | **當內容超出高度時的處理**（`"crop"` 裁切 / `"visible"` 顯示 / `"ellipsis"` 省略號）。 |

- **回傳值**：
  - `Live`：返回一個 Live 上下文物件，通常命名為 `live`。

> **💡 transient=True 離開清除技巧**：  
> 如果你希望動畫或即時進度結束後，**不要在控制台上留下痕跡**（例如只是臨時過場動畫），可以設定 `Live(..., transient=True)`。當 `with` 區塊結束時，控制台會自動擦除該動態畫面，恢復乾淨！

```python
import time
from rich.live import Live

# 專注展示基礎調用
with Live(refresh_per_second=10) as live:
    for count in range(1, 11):
        time.sleep(0.3)
        pass # 在這區間，畫面會以 10 FPS 持續刷新
```

---

##### live.update() 原地覆蓋畫面

- **使用時機**：當你需要將最新的文字、表格或任何 Rich 元件，重新繪製到目前的動態畫面上時。
- **語法**：`live.update(renderable, refresh=False)`
- **參數說明**：
  - `renderable`：要覆蓋上去的最新內容（可為字串、Table、Panel 等）。
  - `refresh`：布林值，設定為 `True` 時將強制立刻刷新畫面（無視 `refresh_per_second` 的計時）。
- **回傳值**：
  - 無。

> **💡 原地覆蓋特性**：  
> 當多次呼叫 `live.update()` 時，**最新的內容會直接原地覆蓋 (Overwrite) 上一次的舊畫面**，完全不會在控制台上堆疊或產生多行舊輸出！

> **⚠️ 避免在 Live 內部使用 print() 衝突**：  
> 在 `with Live(...)` 區塊內部，**嚴禁混用傳統的 `print()` 或 `console.print()`**！手動列印會打亂 Live 的游標位址，導致畫面撕裂與重複列印。若需輸出日誌，應使用 `live.console.print()`！

```python
import time
import random
from rich.live import Live
from rich.table import Table

def generate_table() -> Table:
    """動態產生最新的伺服器狀態表格"""
    table = Table(title="🖥️ 本地伺服器監控", border_style="cyan")
    table.add_column("伺服器", style="bold white")
    table.add_column("CPU 使用率", justify="right")
    
    cpu = random.randint(10, 99)
    table.add_row("Server-Alpha", f"{cpu}%")
    return table

with Live(generate_table(), refresh_per_second=4) as live:
    for _ in range(10):
        time.sleep(0.5)
        # 重新產生並替換整張 Table 畫面
        live.update(generate_table())
```

---

##### live.stop() 與 live.start() 暫停與恢復

- **使用時機**：當你在動態畫面運作途中，需要彈出互動式選單（如 Questionary）或暫時接管終端機，避免兩邊畫面打架時使用。
- **語法**：
  - `live.stop()`：暫停刷新並交出控制權。
  - `live.start()`：重新喚醒並接管畫面刷新。
- **參數說明**：無。
- **回傳值**：無。

> **💡 為什麼要手動暫停？ (Questionary 衝突問題)**：
> 像 `questionary` 這種互動式選單套件，也需要**完全控制終端機的游標與畫面**來繪製選項。
> 如果你讓 `Live` 繼續在背景每秒瘋狂刷新（爭搶畫面控制權），當你一呼叫 `questionary` 彈出選項，兩邊就會在終端機上打架，導致畫面瘋狂閃爍、錯位甚至崩潰！
> **解法**：在呼叫互動選單前，先 `live.stop()` 暫停，等使用者選完後再 `live.start()` 喚醒。

> **⚠️ 致命陷阱：未恢復畫面導致終端機殘廢**：
> 當你呼叫 `live.stop()` 後，如果程式因為例外錯誤崩潰，或是使用者按下了 `Ctrl+C` 中斷，導致沒有成功執行到後面的 `live.start()`，你的終端機會處於**「游標隱藏、排版錯亂」**的半殘廢狀態。
> **防呆對策**：務必使用 `try...finally`，確保無論互動選單有沒有出錯，最後都會乖乖呼叫 `live.start()`，讓外層的 `with` 能正常收尾並還原終端機狀態！

```python
import time
import questionary
from rich.live import Live

with Live("🔄 系統運作中...", refresh_per_second=4) as live:
    time.sleep(1)
    
    # ⚠️ 準備彈出 Questionary 互動選單，必須先暫停 Live 避免畫面打架！
    live.stop()
    
    try:
        # 現在終端機控制權交還，可以安全地彈出選單了
        answer = questionary.confirm("要中斷任務嗎？").ask()
        
    finally:
        # 🛡️ 鐵律：無論是正常選完，還是按 Ctrl+C 強制中斷，
        # 都要保證把 Live 重新喚醒，避免終端機游標消失或排版徹底壞掉！
        live.start()
        
    if not answer:
        # 使用者不想中斷，就繼續執行後續任務
        time.sleep(2)
```
