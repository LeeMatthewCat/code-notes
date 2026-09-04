# 概念與原理

## Git 核心架構與四個工作區域

Git 是一個**分散式版本控制系統 (Distributed Version Control System)**

在 Git 的運作體系中，程式碼的狀態被劃分為四個核心區域，理解這四個區域之間的資料流動是掌握 Git 的最大關鍵

1. **工作區 (Working Directory)**：你目前在編輯器（如 VS Code）中實體看到與修改檔案的地方，這裡的變更尚未經過 Git 追蹤或記錄
2. **暫存區 (Staging Area / Index)**：準備要提交進版本庫的**預備快照暫存區**，就像出貨前的購物車或打包箱
3. **本地儲存庫 (Local Repository)**：放在你電腦本機（`.git` 資料夾）中的完整版本資料庫，紀錄了專案從出生到現在的所有 commit 歷史快照
4. **遠端儲存庫 (Remote Repository)**：託管在雲端伺服器（如 GitHub、GitLab）上的共用版本庫，供團隊成員同步與備份程式碼

> **生動白話比喻**：  
> - **工作區** 像你在桌上**寫草稿**  
> - **暫存區** 像把寫好的草稿**放入包裝盒打包**（[[#git add|git add]]）  
> - **本地儲存庫** 像把包裝盒**蓋印章放進自家地下室倉庫**保存（[[#git commit|git commit]]）  
> - **遠端儲存庫** 像把自家倉庫的複本**郵寄上傳到海外總公司大樓**共用（[[#git push|git push]]）

---
## 快照機制與版本庫結構

Git 與傳統版本控制系統（如 SVN）最大的區別在於**資料儲存方式**

傳統系統儲存的是「差異清單 (Delta-based)」，即紀錄檔案 A 從第 1 版到第 2 版修改了哪幾行文字

而 Git 紀錄的是**檔案系統的全景快照 (Snapshots)**，每一次你提交（[[#git commit|commit]]）時，Git 就會幫當時所有的檔案拍一張全景照片紀錄下來，如果某個檔案沒有修改，Git 不會重新複製一份，而是直接用指標指向之前的舊快照，極大化節省了空間與提升效率

> Git 中的每一個 commit 快照都會經過 SHA-1 雜湊演算法計算，生成一串長度為 40 個字元的唯一識別碼（如 `a1b2c3d...`）  
> 只要檔案內容或提交時間有微小改變，生成的雜湊值就完全不同，這保證了版本庫資料絕不可能被偷偷篡改

---
## HEAD 指針

HEAD 是 Git 中用來標記**「你現在身處在哪個位置」**的動態導覽游標與指標

把 HEAD 想像成唱片機上的**讀取針頭**或影片播放器上的**時間軸游標**

在絕大多數情況下，HEAD 會指向你目前所在分支的最新 [[#git commit|commit]] 快照（例如指向 `main` 分支的最前端）

當你切換分支或退回舊版本時（[[#git checkout 與 git switch|checkout / switch]]），HEAD 指針就會跟著移動到目標位置，工作區的檔案內容也會隨之瞬間變換成該位置的歷史樣貌

> **分離 HEAD 狀態 (Detached HEAD)**：  
> 當 HEAD 直接指向某個特定的 commit 哈希值，而不是指向分支名稱時，就稱為「分離 HEAD 狀態」，此時進行的任何新提交都不屬於任何分支，需要特別小心處理

---
## 分支 Branch 原理

分支是 Git 用來實現**多軌並列開發、平行宇宙**的核心機制

在許多傳統版控系統中，建立分支意味著要把整套程式碼完整複製一份，既慢又佔用空間

而在 Git 中，**分支本質上只是一個指向某個 commit 快照的輕量指標貼紙（僅佔用 41 位元組）**

因此在 Git 中建立、切換或刪除分支幾乎是**瞬間完成**的，完全不影響效能

> **生動白話比喻**：  
> 主線分支 `main` 是故事的主線情節，當你想嘗試開發一個新功能時，隨手貼一張 `feature` 貼紙開闢平行宇宙，即使新功能寫砸了，直接把 `feature` 貼紙撕掉刪除即可，主線故事完全不受影響

---
## 合併策略 Merge vs Rebase

當多條分支開發完畢需要整合時，Git 提供兩種主要策略：

### 1. Merge (合併歷史)
將兩條分支的最新快照進行三方合併，並自動生成一個新的**「合併 commit (Merge Commit)」**

- **優點**：完整保留歷史的真實發展軌跡，能夠清楚看出哪條分支在什麼時間點切出與合回
- **缺點**：如果團隊成員频繁 merge， commit 歷史圖表會出現大量的分叉與交織網狀，顯得較為雜亂

### 2. Rebase (重新定位 / 基底重置)
將當前分支上的 commit 「剪下」，並重新粘貼接到目標分支的最前端

- **優點**：創造出一條**完全線性、乾淨無分叉**的 commit 歷史紀錄，極度美觀
- **缺點**：會改變 commit 的哈希值與歷史順序，**絕對不能在已經推送到遠端公用分支上執行 rebase**，否則會造成其他協作者的歷史衝突

> **生動白話比喻**：  
> - **Merge** 像兩條河流匯入交界處，留下一個明確的會流節點  
> - **Rebase** 像把自己在平行宇宙寫好的草稿，剪下來重新順序貼到主線的故事最尾端

---
## 衝突 Conflict 原理與解決策略

當兩條分支修改了**同一個檔案的同一行程式碼**，且兩邊的修改內容不一致時，Git 在合併時無法自動判斷哪一個版本才是正確的

此時 Git 會暫停合併過程，拋出**衝突 (Conflict)** 提示，並在衝突檔案中留下顯眼的標記：

```text
<<<<<<< HEAD (當前分支的內容)
print("Hello from Main")
=======
print("Hello from Feature Branch")
>>>>>>> feature-branch (準備合進來的分支內容)
```

### 解決衝突三步驟：
1. 開啟衝突檔案，搜尋 `<<<<<<<` 標記
2. 人工評估討論後，手動修改保留正確的程式碼，並刪除 Git 自動產生的標記符號
3. 執行 `git add 檔案名` 將解決後的檔案加入暫存區，最後執行 `git commit` 完成合併

---
## .gitignore 檔案黑名單機制與語法詳解

專案開發過程中，許多檔案是不應該提交到 Git 版本庫中的

例如：敏感的 API 金鑰與密碼檔 (`.env`)、編譯產生的暫存檔 (`__pycache__/`, `build/`)、套件下載目錄 (`node_modules/`, `venv/`)、作業系統產生的隱藏檔 (`.DS_Store`)

在專案根目錄下建立 `.gitignore` 檔案，將不需追蹤的檔案名稱或路徑寫入其中，Git 就會自動忽略這些檔案

### .gitignore 匹配語法 (Glob Patterns) 完整解析

`.gitignore` 使用的是 Shell 的 **Glob 匹配模式**，常用語法符號如下：

#### 1. 基本檔名與萬用字元 `*` (Asterisk)
- `secret.txt`：無視任何目錄底下名為 `secret.txt` 的檔案
- `*.log`：忽略所有以 `.log` 結尾的日誌檔（`*` 代表 0 個或多個任意字元）
- `*.tmp`：忽略所有暫存檔

#### 2. 斜線 `/` 的三種關鍵位置差異
斜線 `/` 在 `.gitignore` 中是用來指定**路徑層級與類型**的極重要符號：

- **開頭有斜線 `/`（錨定根目錄）**：
  - `/config.json`：只忽略專案**最外層根目錄**下的 `config.json`，如果子目錄裡有 `app/config.json` 則**不會**被忽略
- **結尾有斜線 `/`（限定為資料夾）**：
  - `build/`：只忽略名為 `build` 的**資料夾及其底下所有內容**，如果剛好有一個普通檔案叫 `build` 則**不會**被忽略
- **中間有斜線 `/`（指定相對路徑）**：
  - `docs/notes.txt`：只忽略 `docs` 資料夾正底下的 `notes.txt`

#### 3. 跨目錄雙星號 `**` (Recursive Wildcard)
雙星號 `**` 用來匹配**任意層級的子目錄**：

- `**/logs`：忽略任何地方、任何層級目錄下的 `logs` 資料夾（如 `logs`, `app/logs`, `src/utils/logs`）
- `logs/**`：忽略 `logs` 資料夾底下所有的檔案與子目錄
- `foo/**/bar`：匹配 `foo/bar`、`foo/a/bar`、`foo/a/b/c/bar` 等跨越任意層級的結構

#### 4. 單一字元 `?` 與字元範圍 `[]`
- `file?.txt`：匹配 `file1.txt`、`fileA.txt`，但**不匹配** `file12.txt`（`?` 剛好代表 1 個任意字元）
- `[0-9].log`：匹配 `0.log` 到 `9.log`
- `[ab].log`：匹配 `a.log` 或 `b.log`

#### 5. 反向例外白名單 `!` (Negation)
驚嘆號 `!` 用來指定**例外情況**，意思是「不要忽略此檔案」：

```gitignore
# 忽略所有的 .log 日誌檔
*.log

# 但不要忽略 important.log 檔案
!important.log
```

> **例外規則的重大陷阱**：  
> 如果父資料夾已經被忽略了（例如寫了 `logs/`），Git 為了效能不會再去讀取該資料夾內部，此時你就算寫 `!logs/important.log` 也**無法讓子檔案復活**  
> 解決方法：必須先讓父資料夾不被忽略（`!logs/`），再個別忽略不需要的子項目

#### 6. 註解與空行
- 以 `#` 開頭的行會被視為註解說明
- 空白行會被 Git 自動忽略，可用來分隔不同邏輯區域

---
### 常用的 .gitignore 設定範例

```gitignore
# 忽略作業系統產生的垃圾檔
.DS_Store
Thumbs.db

# 忽略敏感環境變數與金鑰檔
.env
.env.local
*.pem

# 忽略 Python 編譯檔與虛擬環境
__pycache__/
*.py[cod]
.venv/
venv/

# 忽略 Node.js 套件目錄與構建產物
node_modules/
dist/
build/

# 忽略所有 log 檔，但保留專案說明 log
*.log
!README-log.txt
```

---
### .gitignore 無效/無回應時的除錯技巧

#### 為什麼加了 .gitignore 檔案還是持續被 Git 追蹤？
**核心原因**：`.gitignore` **只對尚未被 Git 追蹤 (Untracked)** 的新檔案生效  
如果某個檔案在加入 `.gitignore` 之前，就已經執行過 [[#git add|git add]] 或 commit 提交過，Git 已經將其納入版本控制中，此時光改 `.gitignore` 是不會自動取消追蹤的

#### 解決方法：將檔案從 Git 暫存快照中抽離 (保留實體檔案)
```shell
# 1. 將特定檔案從 Git 追蹤快照中移除 (不會刪除硬碟上的實體檔案)
git rm --cached 檔案名

# 2. 如果是整塊資料夾被誤追蹤 (例如 node_modules/ 或 venv/)
git rm -r --cached 資料夾名/

# 3. 重新提交 commit 完成解鎖
git commit -m "Fix: 移除被誤追蹤的忽略檔案"
```

#### 如何檢查某個檔案是被哪一行 .gitignore 規則忽略的？
使用 `git check-ignore` 指令進行診斷：

```shell
git check-ignore -v 檔案路徑
```

> 終端會精確顯示是哪一份 `.gitignore` 檔案的第幾行規則觸發了忽略，除錯極度方便

---
## 基礎設定與初始化

## git config 設定使用者資訊與全域偏好

第一次安裝 Git 或使用新電腦時，必須設定你的身分標誌（姓名與 Email），這些資訊會被永久寫入每一個 [[#git commit|commit]] 的[[#快照機制與版本庫結構|元數據]]中

```shell
# 設定全域使用者名稱
git config --global user.name "你的名字"

# 設定全域 Email
git config --global user.email "your_email@example.com"

# 查看目前所有的 Git 設定項目
git config --list
```

> `--global` 表示全域設定，修改後會套用到這台電腦上的所有專案，若特定專案需要用不同身分，可去掉 `--global` 在該專案目錄下單獨設定

---
## git init 初始化本地倉庫

將一個普通的資料夾轉化為受到 Git 追蹤管理 Versions 倉庫

執行此指令後，Git 會在該資料夾最外側默默建立一個隱藏的 `.git` 目錄，用來存放所有的版本歷史與配置

```shell
git init
```

> `git init` 會把當前所在的資料夾作為倉庫根目錄  
> 如果想建立新資料夾並直接初始化，可以執行 `git init 專案名稱`

---
## git status 查看目前倉庫狀態

開發過程中最頻繁使用的指令，用來查詢目前[[#Git 核心架構與四個工作區域|工作區與暫存區]]的檔案狀態

```shell
git status
```

Git 會列出：
- 目前身處在哪個[[#分支 Branch 原理|分支]]（HEAD 位置）
- 有哪些檔案被修改了但尚未加入暫存區 (Changes not staged for commit)
- 有哪些檔案已經放入暫存區準備提交 (Changes to be committed)
- 有哪些新檔案尚未被 Git 追蹤 (Untracked files)

> > **提示**：可加入 `-s` 參數（`git status -s`）以極簡短格式印出狀態資訊

---
## 基礎版控流程

## git add 將變更加入暫存區

將工作區中已修改或新建的檔案，添加到[[#Git 核心架構與四個工作區域|暫存區 (Staging Area)]]，準備封箱打包

```shell
# 將特定檔案加入暫存區
git add index.html script.js

# 將當前目錄下的所有變更（新增、修改、刪除）一次性全部加入暫存區
git add .
```

> `git add` 並不會真正建立版本歷史快照，它只是把檔案狀態標記為「準備好了」

---
## git commit 提交快照至版本庫

將暫存區中的所有檔案狀態，正式拍下快照並永久寫入[[#Git 核心架構與四個工作區域|本地儲存庫]]中，生成一個全新的 commit 節點

```shell
# 提交暫存區變更並附帶說明訊息
git commit -m "完成使用者登入功能"
```

> **commit 訊息撰寫最佳實踐**：  
> - 訊息應簡明扼要說明「這次修改了什麼與為什麼修改」  
> - 常用前綴動詞：`Feat:` (新功能)、`Fix:` (修復 Bug)、`Docs:` (文件變更)、`Refactor:` (重構程式碼)

---
## git diff 比對變更差異

在提交前檢查程式碼到底修改了哪些具體的行數與文字

```shell
# 比對「工作區」與「暫存區」之間的程式碼差異
git diff

# 比對「暫存區」與「最新一次 commit (HEAD)」之間的差異
git diff --staged

# 比對兩條不同分支之間的差異
git diff main feature-login
```

> 綠色加號 `+` 代表新增的程式碼行，紅色減號 `-` 代表被刪除或替換掉的程式碼行

---
## 分支管理與操作

## git branch 查看與建立分支

管理專案中的平行開發[[#分支 Branch 原理|分支]]

```shell
# 查看本地所有分支（當前所在分支前會有 * 號加亮）
git branch

# 查看本地與遠端的所有分支
git branch -a

# 建立名為 feature-user 的新分支（建立後 HEAD 仍停留在原處）
git branch feature-user

# 刪除已合併的指定分支
git branch -d feature-user

# 強制刪除尚未合併的分支（需謹慎使用）
git branch -D feature-user
```

> > `b -> branch 分支`

---
## git checkout 與 git switch 切換分支與修復

切換當前工作目錄至指定分支，或復原檔案狀態

```shell
# 切換到指定分支 (傳統指令)
git checkout feature-user

# 切換到指定分支 (現代建議指令)
git switch feature-user

# 建立新分支並「立刻切換」過去 (極常用)
git checkout -b feature/refactor-providers

# 現代建議寫法 (使用 switch -c)：
git switch -c feature/refactor-providers
```

> **💡 指令拆解與語法說明 (`git checkout -b feature/refactor-providers`)**：  
> - **`checkout`**：切換工作區的 [[#HEAD 指針|HEAD]] 動態導覽游標。  
> - **`-b` 參數**：代表 **Create & Switch**，指示 Git 先建立新分支並立即切換過去（等同先執行 `git branch` 再執行 `git checkout`）。  
> - **`feature/refactor-providers`**：分支命名格式。`feature/` 為類別前綴（表示新功能開發或重構），`refactor-providers` 為具體主題。

> **💡 團隊分支命名規範與前綴慣例 (Branch Naming Conventions)**：  
> 採用 **`前綴/名稱`** (`prefix/name`) 命名規範能讓分支語意明確，並使 Git GUI 工具（如 GitKraken, SourceTree）自動歸類為資料夾目錄結構：  
>  
> | 前綴名稱 | 適用情境與說明 | 命名範例 |
> | :--- | :--- | :--- |
> | **`feature/`** (或 `feat/`) | **新功能開發與模組重構** | `feature/login-page`<br>`feature/refactor-providers` |
> | **`bugfix/`** (或 `fix/`) | **修復一般性 Bug 或缺陷** | `bugfix/cart-calculation`<br>`fix/user-avatar-upload` |
> | **`hotfix/`** | **線上緊急修復**（直切自 `main`） | `hotfix/payment-gateway-500` |
> | **`refactor/`** | **純程式碼架構重構** | `refactor/database-orm-v2` |
> | **`release/`** | **版本預備發布** | `release/v1.2.0` |
> | **`chore/`** / **`ci/`** | **日常維護、腳本與 CI/CD 配置** | `chore/update-deps`<br>`ci/github-actions` |
> | **`docs/`** | **僅修改說明文件** | `docs/update-readme` |
> | **`experiment/`** | **實驗性概念驗證 (PoC)** | `experiment/try-wasm` |

> 早期 `git checkout` 既能切換分支又能復原檔案，功能過於雜亂  
> Git 官方在 2.23 版本後推出了 `git switch` (專門切換分支) 與 `git restore` (專門復原檔案) 來明確分工

---
## git merge 合併分支

將指定分支的變更[[#合併策略 Merge vs Rebase|合併]]到當前所在的分支中

```shell
# 步驟 1：先切換回目標主分支 (例如 main)
git switch main

# 步驟 2：將 feature-user 分支的成果合併進 main
git merge feature-user
```

> 如果合併過程順利且沒有程式碼重疊，Git 會自動完成合併  
> 若出現程式碼衝突，需參考[[#衝突 Conflict 原理與解決策略|衝突解決步驟]]手動處理後提交

---
## 歷史紀錄與復原撤銷

## git log 查看版本歷史紀錄

瀏覽倉庫中過往所有 [[#git commit|commit]] 提交的時間線與詳細資訊

```shell
# 查看詳細歷史紀錄 (包含作者、時間、完整 SHA-1 哈希值與 commit 訊息)
git log

# 以單行簡短格式印出歷史紀錄 (極美觀且利於快速檢視)
git log --oneline

# 以圖表樹狀結構印出分支分叉與合併軌跡 (開發必備美化指令)
git log --oneline --graph --all
```

> 退出 `git log` 瀏覽畫面請按鍵盤上的字母 `q`

---
## git reset 版本撤銷與退回

將當前的 HEAD 指針強制移動到指定的舊 commit 位置，實現版本倒退

`git reset` 提供三種核心模式，差別在於對[[#Git 核心架構與四個工作區域|工作區與暫存區]]資料的處理方式：

```shell
# 1. Soft 模式：只移動 HEAD 指針，工作區與暫存區的程式碼完全保留 (退回狀態為準備 commit)
git reset --soft HEAD~1

# 2. Mixed 模式 (預設模式)：移動 HEAD 並清空暫存區，但工作區修改的程式碼依然保留
git reset --mixed HEAD~1

# 3. Hard 模式 (極危險)：移動 HEAD 並且「徹底抹殺」工作區與暫存區的所有修改，完全還原到該 commit 狀態
git reset --hard commit_id
```

> `HEAD~1` 代表退回上一個 commit，`HEAD~2` 代表退回上上個 commit  
> **警告**：`git reset --hard` 會直接丟棄未提交的程式碼，執行前請務必三思

---
## git revert 安全撤銷已發布的提交

建立一個全新的 commit 來**反向抵銷**指定 commit 所做的變更

```shell
# 抵銷指定 commit 的變更，並自動生成一個新的抵銷 commit
git revert commit_id
```

> **reset vs revert 的核心區別**：  
> - [[#git reset 版本撤銷與退回|reset]] 是「搭乘時光機直接抹滅歷史」，歷史線會變短  
> - `revert` 是「向前看，用補改方式覆蓋舊錯」，歷史線繼續向前延伸  
> 若 commit 已經 [[#git push|push]] 到遠端共用倉庫，必須使用 `revert` 而絕對不能使用 `reset`，否則會破壞團隊其他人的歷史紀錄

---
## git restore 復原工作區與暫存區檔案

取消檔案的修改或將檔案從暫存區抽離

```shell
# 丟棄工作區中某個檔案的所有未提交修改 (還原成最後一次 commit 的樣子)
git restore 檔案名

# 將檔案從「暫存區」撤回至「工作區」(取消 git add)
git restore --staged 檔案名
```

---
## 遠端倉庫協作

## git remote 管理遠端倉庫連結

將本地 Git 倉庫與遠端託管平台（如 GitHub、GitLab）上的倉庫建立關聯

```shell
# 查看目前關聯的遠端倉庫名稱與 URL
git remote -v

# 新增遠端倉庫連結 (慣例將主遠端倉庫命名為 origin)
git remote add origin https://github.com/username/repository.git

# 移除指定的遠端倉庫連結
git remote remove origin
```

---
## git push 將本地提交推送到遠端

將本地倉庫的分支歷史與 [[#git commit|commit]] 快照上傳至遠端倉庫

```shell
# 將本地的 main 分支推送到遠端 origin 倉庫
git push origin main

# 第一次推送時加上 -u 參數，設定預設追蹤分支，後續只需直接打 git push
git push -u origin main
```

---
## git fetch 與 git pull 抓取與同步遠端變更

獲取遠端倉庫的最新動態

```shell
# 1. git fetch：僅下載遠端最新的 commit 歷史與分支資訊，但「完全不修改」本地工作區程式碼 (安全檢查)
git fetch origin

# 2. git pull：下載遠端最新內容並「自動與當前本地分支進行 merge 合併」
git pull origin main
```

> `git pull` 的底層本質等於執行了 `git fetch` + `git merge`  
> 為了避免自動 merge 產生意外衝突，許多資深工程師習慣先 `git fetch` 檢查後再手動合併

---
## 高級與實用開發技巧

## git stash 暫存當前工作進度

當你在某分支開發到一半，程式碼還處於草稿階段無法 commit，卻突然需要緊急切換到其他分支修 Bug 時的救星

`git stash` 會將工作區與暫存區未完成的修改「抽離並藏起來」，讓工作區瞬間變回乾淨狀態

```shell
# 暫存當前未提交的進度 (可附帶說明備註)
git stash save "登入頁面草稿版"

# 查看目前藏起來的所有進度清單
git stash list

# 還原最新一次暫存的進度，並將該暫存紀錄從 stash 清單中移除
git stash pop

# 清空所有的 stash 暫存紀錄
git stash clear
```

> > `stash -> 隱藏/暫存`

---
## git rebase 整理與重構提交歷史

除了用於合併分支外，`git rebase -i` (互動式 rebase) 是整理本地散亂 commit 的極致神器

```shell
# 互動式整理最近 3 次 commit 歷史
git rebase -i HEAD~3
```

執行後會開啟編輯器，你可以對近期的 commit 進行：
- **squash (s)**：將多個小 commit 合併壓縮成一個大 commit
- **reword (r)**：修改過去某次 commit 的說明文字
- **drop (d)**：直接刪除某次特定的 commit

> 整理歷史能讓推送到團隊遠端倉庫的 commit 線條極度清晰有條理

---
## git cherry-pick 採摘指定提交

從其他分支中，精準挑選某一個特定的 [[#git commit|commit]] 快照，「複製並套用」到當前所在的分支

```shell
# 切換到目標分支後，指定要採摘的 commit 雜湊值
git cherry-pick a1b2c3d
```

> **經典應用場景**：  
> 你在 `experiment` 實驗分支上記錄了 10 個 commit，其中只有第 3 個 commit 修復了一個關鍵 Bug，此時你不需要將整條分支 merge 過來，只需在 `main` 分支執行 `cherry-pick` 挑選該 commit 即可
