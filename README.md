# 開發者技術筆記與知識庫 (Developer Notes & Knowledge Base)

本知識庫收錄了從程式語言基礎、現代 Python 開發、大語言模型 (LLM / Agent) 調用到機器學習與深度學習核心架構的系統化技術筆記。採用 Markdown 格式撰寫，相容於 Obsidian 與 GitHub 渲染。

---

## 📚 目錄結構導覽

### 1. 基礎語言與核心工具
* **[C++.md](./C%2B%2B.md)**：現代 C++ 核心語法、記憶體管理、物件導向、指針與引用詳解。
* **[Git.md](./Git.md)**：Git 版本控制全流程指南（基礎指令、分支合併策略、衝突排解、進階技巧）。
* **[SQL.md](./SQL.md)**：關係型資料庫查詢、表格設計、聚合運算與常用 CRUD 操作。
* **[終端指令.md](./終端指令.md)**：macOS / Linux 常用命令列工具、檔案操作、權限管理與管線重定向實戰。
* **[Obsidian 筆記撰寫規範.md](./Obsidian%20筆記撰寫規範.md)**：結構化技術筆記的撰寫原則、格式規範與版面美化指引。

---

### 2. Python 現代化開發與工程實踐 (`Python/`)
* **[Python.md](./Python/Python.md)**：Python 核心語法、高階特徵、型別提示、裝飾器與內建常用函式。
* **[專案準備.md](./Python/專案準備.md)**：現代 Python 專案結構、`uv` 虛擬環境管理、`pyproject.toml` 與環境變數配置規範。
* **[Python 專案測試指南.md](./Python/Python%20專案測試指南.md)**：Pytest 單元測試、Mock 隔離測試與自動化測試最佳實踐。
* **[SQLite.md](./Python/SQLite.md)**：Python 內建輕量化關聯資料庫的使用與操作實務。
* **[Regex.md](./Python/Regex.md)**：正則表達式語法、模式匹配、文字清洗與實戰應用。
* **核心模組詳解**：
  * [os.md](./Python/路徑相關/os.md) / [Pathlib.md](./Python/路徑相關/Pathlib.md)：現代檔案路徑處理與系統互動。
  * [concurrent.futures.md](./Python/多線程相關/concurrent.futures.md) / [threading.md](./Python/多線程相關/threading.md) / [Subprocess.md](./Python/多線程相關/Subprocess.md) / [queue.md](./Python/多線程相關/queue.md)：並行、多執行緒與子行程控制。
  * [HTTPX.md](./Python/網頁相關/HTTPX.md) / [readability.md](./Python/網頁相關/readability.md) / [html2text.md](./Python/html2text.md)：現代非同步網路請求與網頁內容擷取。
  * [Rich.md](./Python/終端美化/Rich.md) / [Questionary.md](./Python/終端美化/Questionary.md) / [prompt_toolkit.md](./Python/終端美化/prompt_toolkit.md) / [Live.md](./Python/終端美化/Live.md)：互動式終端 UI 與控制台美化。
  * [Pydantic.md](./Python/JSON%20格式相關/Pydantic.md) / [JSON Schema.md](./Python/JSON%20格式相關/JSON%20Schema.md) / [JSON.md](./Python/JSON%20格式相關/JSON.md)：資料驗證與結構化 Schema 解析。
  * [sys.md](./Python/sys.md) / [logging.md](./Python/logging.md) / [Time.md](./Python/Time.md) / [inspect.md](./Python/inspect.md) / [platform.md](./Python/platform.md) / [textwrap.md](./Python/textwrap.md)。

---

### 3. AI 模型調用與 Agent 架構 (`Python/模型 api 調用/`)
* **[Gemini API.md](./Python/模型%20api%20調用/Gemini%20API.md)**：Google Gemini 模型調用、Tool Calling、Function Calling、Thinking Budget 與狀態封裝。
* **[Ollama.md](./Python/模型%20api%20調用/Ollama.md)**：本地開源大模型離線部署、隱私保護與 OpenAI 相容介面調用。
* **[MCP.md](./Python/模型%20api%20調用/MCP.md)**：Model Context Protocol (MCP) 架構、Server/Client 雙向通訊、資源掛載與工具擴展。
* **[Tool Registry.md](./Python/模型%20api%20調用/Tool%20Registry.md)**：動態外掛註冊表架構、型別推導與自動 Tool Calling 整合。

---

### 4. 機器學習與深度學習 (`Python/機器學習相關/`)
* **[人工智慧與機器學習.md](./Python/機器學習相關/人工智慧與機器學習.md)**：
  * 深度學習基礎原理（梯度下降、反向傳播、激活函數 ReLU/Sigmoid/Tanh）。
  * Transformer 架構、Self-Attention、Multi-Head Attention 機制。
  * 現代 LLM 推理增強（CoT 思維鏈、Process Verifier、Journey Learning）與 MoE (Mixture of Experts) 架構。
* **[Transformer 架構.md](./Python/機器學習相關/Transformer%20架構.md)** / **[Transformer 架構 1.md](./Python/機器學習相關/Transformer%20架構%201.md)**：編碼器與解碼器細節解析。
* **[PyTorch.md](./Python/機器學習相關/PyTorch.md)**：張量運算、自動微分、自定義神經網路層與模型訓練管線。
* **[NumPy.md](./Python/機器學習相關/NumPy.md)** / **[Pandas.md](./Python/機器學習相關/Pandas.md)** / **[Matplotlib.md](./Python/機器學習相關/Matplotlib.md)** / **[SciPy.md](./Python/機器學習相關/SciPy.md)**：科學計算與資料視覺化基礎堆疊。
* **[transformers.md](./Python/機器學習相關/transformers.md)** / **[Datasets.md](./Python/機器學習相關/Datasets.md)** / **[sklearn.md](./Python/機器學習相關/sklearn.md)** / **[SwanLab.md](./Python/機器學習相關/SwanLab.md)** / **[多模態模型.md](./Python/機器學習相關/多模態模型.md)**。

---

## 📌 版權與引用聲明
* 本知識庫為個人學習、技術研究與工程實踐之記錄。
* 機器學習與深度學習章節之部分圖表與理論架構取材自國立臺灣大學李宏毅教授之公開教學課程投影片及相關經典學術論文，著作權歸原作者所有。
