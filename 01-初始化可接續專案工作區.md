# Codex 懶人包 #01：初始化可接續專案工作區

> 版本：v0.1  
> 更新日期：2026-06-18  
> 可選參考模板：https://github.com/chunyifish/codex-project-template  
> 用途：把任何新任務初始化成 Codex 可以長期接手、跨對話接續、可追蹤、可部署的專案工作區。

> 本懶人包是「專案工作區啟動包」。  
> 跑完之後，你會得到一個標準專案資料夾，裡面包含 `README.md`、`AGENTS.md`、`PROGRESS.md`、`TODO.md`、`DECISIONS.md`、`DEPLOYMENT.md` 等文件，讓後續 Codex 對話可以直接接續，不用每次重講背景。
>
> 本懶人包是獨立流程，不依附任何特定 repo。若你想快速開始，可以選擇搭配 `codex-project-template`；若你已有自己的資料夾結構，也可以直接套用這份流程。

---

## 這個懶人包會幫你做什麼？

### 一次性建立

- 建立一個新的專案資料夾。
- 建立或套用一套清楚的專案結構。
- 建立或改寫專案核心文件：
  - `README.md`：給人看的專案說明。
  - `AGENTS.md`：給 Codex 看的工作規則。
  - `PROGRESS.md`：目前進度與交接紀錄。
  - `TODO.md`：下一步與待確認事項。
  - `DECISIONS.md`：重要決策與理由。
  - `DEPLOYMENT.md`：GitHub、部署、Firebase、環境設定狀態。
- 視需要建立 GitHub repo。
- 視需要設定 GitHub Pages、Vercel 或 Firebase。
- 視需要保留原始資料到 `materials/`。

### 之後每次工作的節奏

| 場景 | 你說的話 | Codex 做什麼 |
|---|---|---|
| 初始化新專案 | 「專案初始化」 | 讀 `INIT.md`，先問必要資訊，確認後建立專案骨架 |
| 接續工作 | 「開工」 | 讀核心文件，摘要目前狀態，建議下一步 |
| 結束工作 | 「收工」 | 更新進度、待辦、決策與部署紀錄 |
| 同步 GitHub | 「收工並同步 GitHub」 | 完成收工紀錄後 commit + push |
| 部署 | 「收工並部署」 | 同步 GitHub 後依 `DEPLOYMENT.md` 執行部署檢查 |

---

## 先備條件

- [ ] 已安裝 Codex 桌面版或可使用 Codex 的工作環境。
- [ ] 已安裝 Git。
- [ ] 若要建立 GitHub repo：已登入 GitHub CLI。
- [ ] 若要部署到 GitHub Pages：已有 GitHub 帳號。
- [ ] 若要使用 Firebase：已確認 Firebase project 與要使用的服務。
- [ ] 已想好新專案大致要做什麼。

> 如果只要建立本機專案資料夾，不需要先準備 GitHub、部署或 Firebase。

---

## 請 Codex 幫我執行以下步驟

> 以下內容是給 Codex 讀的操作指令。  
> 你可以把這整份 Markdown 丟給 Codex，請它依序執行。  
> 遇到需要你決定的地方，Codex 應該先暫停詢問，不要自行猜測。

---

## 階段一：確認初始化資訊

請 Codex 先詢問使用者以下資訊，不要直接建檔：

1. 專案名稱是什麼？
2. 專案目標是什麼？
3. 專案類型是哪一種？
   - 文件型專案
   - 靜態網頁專案
   - 互動工具專案
   - 資料庫應用專案
   - 自動化流程專案
4. 專案要放在哪個資料夾？
5. 是否需要 GitHub repo？
6. 是否需要部署？
   - 不部署
   - GitHub Pages
   - Vercel
   - Firebase Hosting
   - 其他
7. 是否需要 Firebase？
   - 不需要
   - Hosting
   - Firestore
   - Storage
   - Auth
   - Functions
8. 是否有原始資料要放進 `materials/`？

確認完後，請 Codex 先提出初始化規劃，等使用者確認後再執行。

---

## 階段二：套用專案模板

請 Codex 建立一個清楚、可接續的專案結構。

若使用者希望快速開始，可以選擇使用這個參考模板：

```text
https://github.com/chunyifish/codex-project-template
```

如果不使用模板，也請建立新專案資料夾，並產生以下建議結構：

```text
PROJECT_NAME/
├─ AGENTS.md
├─ README.md
├─ INIT.md
├─ PROGRESS.md
├─ TODO.md
├─ DECISIONS.md
├─ DEPLOYMENT.md
├─ .gitignore
├─ .env.example
├─ materials/
├─ working/
├─ outputs/
├─ assets/
├─ references/
├─ scripts/
├─ app/
├─ firebase/
└─ archive/
```

請依專案類型保留適合的資料夾。若某些資料夾暫時用不到，也可以先保留空資料夾，方便後續接手。

---

## 階段三：改寫核心文件

### 1. 改寫 README.md

`README.md` 要給人看，請包含：

- 專案名稱
- 專案目標
- 使用對象
- 專案結構
- 目前狀態
- 如何開始使用或接續工作

### 2. 改寫 AGENTS.md

`AGENTS.md` 要給 Codex 看，請包含：

- 專案目標
- 專案類型
- 工作原則
- 檔案放置規則
- 開工規則
- 收工規則
- GitHub 與部署規則
- Firebase 規則（若不使用 Firebase，也要明確寫不使用）
- 完成回報格式

### 3. 初始化 PROGRESS.md

`PROGRESS.md` 請記錄：

- 初始化日期
- 已建立哪些文件與資料夾
- 使用者已確認哪些設定
- 目前狀態
- 下一步建議

### 4. 初始化 TODO.md

`TODO.md` 請分成：

- 下一步
- 下次優先
- 待確認

### 5. 初始化 DECISIONS.md

`DECISIONS.md` 請記錄已確定的重要選擇，例如：

- 專案類型
- 是否使用 GitHub
- 是否部署
- 是否使用 Firebase
- 為什麼這樣選

### 6. 初始化 DEPLOYMENT.md

`DEPLOYMENT.md` 請記錄：

- GitHub repo 狀態
- Branch
- Remote
- 部署目標
- 部署方式
- Firebase 狀態
- 環境變數需求

若目前不部署，也請明確寫「目前不部署」。

---

## 階段四：選配 GitHub

只有在使用者明確說需要 GitHub repo 時，才執行 GitHub 步驟。

請 Codex 先確認：

- repo 名稱
- public 或 private
- 是否要推送到 GitHub

確認後再執行：

```bash
git init
git status --short
git add AGENTS.md README.md INIT.md PROGRESS.md TODO.md DECISIONS.md DEPLOYMENT.md .gitignore .env.example
git commit -m "Initialize Codex project workspace"
gh repo create <repo-name> --public --source=. --push
```

完成後，把 GitHub repo URL 寫入 `DEPLOYMENT.md`。

> 注意：不要提交 `.env`、token、金鑰、本機設定或任何敏感資料。

---

## 階段五：選配部署

只有使用者明確要求部署時，才設定部署。

### GitHub Pages

適合：

- 靜態網頁
- HTML/CSS/JavaScript
- 不需要後端服務

請 Codex 記錄：

- 發布來源
- 發布資料夾
- Pages URL
- workflow 檔案位置

### Vercel

適合：

- Next.js
- React app
- 需要預覽部署
- 需要 serverless API

請 Codex 記錄：

- Vercel project
- framework
- build command
- output directory

### Firebase

只有專案需要資料庫、登入、排行榜、表單提交、後台或即時同步時才使用 Firebase。

請 Codex 先確認：

- Firebase project id
- 使用哪些服務
- 是否需要 Firestore rules
- 是否需要 Storage rules
- 是否需要 `.env.example`

未確認前不要 deploy。

---

## 階段六：第一次開工驗證

初始化完成後，請 Codex 執行一次「開工檢查」：

1. 讀取 `README.md`。
2. 讀取 `PROGRESS.md`。
3. 讀取 `TODO.md`。
4. 讀取 `DECISIONS.md`。
5. 讀取 `DEPLOYMENT.md`。
6. 檢查 Git 狀態。
7. 回報目前狀態與下一步。

回報格式：

```text
📂 專案：<專案名稱>
🎯 目標：<一句話摘要>
📌 目前狀態：<目前完成到哪>
🔧 Git：<clean / 有未提交變更 / 尚未初始化>
🌐 部署：<未部署 / GitHub Pages / Vercel / Firebase>
➡️ 建議下一步：
1. <下一步 1>
2. <下一步 2>
```

---

## 階段七：收工流程

使用者說「收工」時，請 Codex：

1. 整理本輪完成內容。
2. 更新 `PROGRESS.md`。
3. 更新 `TODO.md`。
4. 若有重要決策，更新 `DECISIONS.md`。
5. 若有 GitHub、部署、Firebase 或環境設定變更，更新 `DEPLOYMENT.md`。
6. 檢查 Git 狀態並回報。

使用者說「收工並同步 GitHub」時，請 Codex：

1. 先完成普通收工。
2. 只提交本輪相關變更。
3. 寫具體 commit message。
4. push 到目前 branch。

使用者說「收工並部署」時，請 Codex：

1. 先完成普通收工。
2. commit + push。
3. 依 `DEPLOYMENT.md` 執行部署檢查。
4. Firebase deploy 前必須再次確認 project 與 deploy 範圍。

---

## 常用提示詞

### 初始化新專案

```text
專案初始化。請依這份懶人包建立一個可接續的 Codex 專案工作區。先問我必要資訊，確認後再建立專案。
```

### 開工

```text
開工。請先讀 AGENTS.md、README.md、PROGRESS.md、TODO.md、DECISIONS.md、DEPLOYMENT.md，整理目前狀態，檢查這台電腦還缺哪些環境設定。先不要改檔。
```

### 收工

```text
收工。請整理本次完成內容，更新 PROGRESS.md、TODO.md、DECISIONS.md。若有部署相關變更，也更新 DEPLOYMENT.md。最後檢查 Git 狀態並回報。
```

### 收工並同步 GitHub

```text
收工並同步 GitHub。請先更新專案紀錄，再提交本輪相關變更並 push。
```

### 收工並部署

```text
收工並部署。請先更新專案紀錄、commit、push，再依 DEPLOYMENT.md 執行部署檢查。
```

---

## FAQ

| 問題 | 解法 |
|---|---|
| 我只是想做一份講義，也需要整套模板嗎？ | 可以用，但不需要啟用 GitHub、部署或 Firebase。 |
| 什麼時候需要 GitHub？ | 需要跨電腦、長期維護、版本控管或公開分享時。 |
| 什麼時候需要 GitHub Pages？ | 專案是靜態網頁，而且想公開給別人使用時。 |
| 什麼時候需要 Firebase？ | 需要資料庫、登入、排行榜、表單提交、後台或即時同步時。 |
| Codex 可以直接幫我建 repo 嗎？ | 可以，但要先確認 repo 名稱、公開狀態與是否要 push。 |
| 可以把這個模板用在非程式專案嗎？ | 可以，文件型專案也適用，只要把部署與 Firebase 標成不使用。 |
| 為什麼要有 `DECISIONS.md`？ | 讓之後接手的人知道當時為什麼這樣選，不用重複討論。 |
| 為什麼要有 `DEPLOYMENT.md`？ | 避免忘記 repo、網址、branch、部署來源與環境設定。 |

---

## 安全與隱私

- 不提交 `.env`。
- 不提交 token、API key、私鑰或帳密。
- 不把本機個人設定提交到 GitHub。
- 使用 Firebase 時，學生資料應去識別化。
- 使用公開 repo 時，先確認資料是否可公開。
- 部署前確認 `DEPLOYMENT.md` 的目標與環境設定。

---

## 完成判準

跑完本懶人包後，應該得到：

- [ ] 新專案資料夾已建立。
- [ ] 核心文件已依專案內容改寫。
- [ ] `PROGRESS.md` 有目前狀態。
- [ ] `TODO.md` 有下一步。
- [ ] `DECISIONS.md` 有重要決策。
- [ ] `DEPLOYMENT.md` 有 GitHub 與部署狀態。
- [ ] 若需要 GitHub，repo URL 已記錄。
- [ ] 若需要部署，公開網址或部署狀態已記錄。
- [ ] 使用者可以對 Codex 說「開工」並接續工作。

---

## 更新紀錄

| 日期 | 版本 | 更新內容 |
|---|---|---|
| 2026-06-18 | v0.1 | 初版，建立可直接交給 Codex 執行的獨立專案初始化懶人包。 |

---

## 相關連結

- 可選參考模板：https://github.com/chunyifish/codex-project-template
- 建議用途：Codex 專案初始化、教學專案、靜態網頁、互動工具、文件型專案、自動化流程專案
