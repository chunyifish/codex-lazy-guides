# Codex 懶人包 #04：連接 Firebase Firestore

> 適用環境：Windows、Codex 桌面版、Firebase Cloud Firestore  
> 原稿實測日期：2026-05-24｜原稿版本：v1.0

這份懶人包協助你讓 Codex 連接 Firebase 專案，並為文字雲、投票、IRS、課堂回饋牆等教學工具準備 Firestore 資料庫。

## 完成後，你會得到

- Firebase CLI 的登入狀態
- 專案根目錄中的 Firebase MCP 設定
- 一個已啟用 Firestore 的 Firebase 專案
- `firestore.rules`、`firebase.json`、`.firebaserc` 三個設定檔
- 一組只開放指定集合、其他集合預設封鎖的安全規則

## 開始前先確認

- [ ] 已安裝並登入 Codex 桌面版
- [ ] 已安裝 Node.js
- [ ] 已有可登入 Firebase 的 Google 帳號
- [ ] 電腦可連上網路
- [ ] 可開啟 [Firebase Console](https://console.firebase.google.com/)

> [!IMPORTANT]
> 本文示範的 `wordcloud_words` 集合允許任何人讀寫，只適合不含個資的短期教學示範。不要存放學生姓名、電話、Email、學號或其他可識別資料。正式上線前，應加入 Firebase Authentication、App Check 或更嚴格的安全規則。

## 流程總覽

1. 檢查 Node.js 與 `npx.cmd`
2. 設定 Firebase MCP
3. 登入 Firebase CLI
4. 建立或選擇 Firebase 專案
5. 啟用 Cloud Firestore
6. 建立 Firebase 設定檔
7. 部署並檢查 Firestore 規則

## 1. 檢查執行環境

在 Codex 終端執行：

```powershell
node --version
npx.cmd --version
```

兩行都有顯示版本號即可繼續。

Windows PowerShell 若執行 `npx` 時出現「系統上已停用指令碼執行」，通常是 PowerShell 執行原則擋住 `npx.ps1`，不是 Node.js 損壞。本文統一改用 `npx.cmd`，不需要為了這份教學修改整台電腦的執行原則。

原稿實測版本：

- Node.js：`v24.16.0`
- npx：`11.13.0`
- Firebase CLI：`15.18.0`

你的版本可能不同；只要指令可正常執行即可。

## 2. 設定 Firebase MCP

在專案根目錄的 `.mcp.json` 中加入 Firebase MCP：

```json
{
  "mcpServers": {
    "firebase": {
      "command": "npx.cmd",
      "args": [
        "-y",
        "firebase-tools@latest",
        "mcp"
      ]
    }
  }
}
```

如果 `.mcp.json` 已有其他 MCP，請只把 `firebase` 物件加入原本的 `mcpServers`，不要覆蓋整份檔案。例如：

```json
{
  "mcpServers": {
    "existing-server": {
      "command": "existing-command"
    },
    "firebase": {
      "command": "npx.cmd",
      "args": ["-y", "firebase-tools@latest", "mcp"]
    }
  }
}
```

`@latest` 會在啟動時使用當時的 Firebase CLI 版本。若團隊需要完全可重現的環境，可改成已驗證的固定版本，例如原稿實測的 `firebase-tools@15.18.0`，並在升級前重新測試。

## 3. 登入 Firebase CLI

Codex 的非互動終端可能無法完成瀏覽器登入，並顯示：

```text
Cannot run login in non-interactive mode
```

遇到這個訊息時，請自行開啟 Windows PowerShell，執行：

```powershell
npx.cmd -y firebase-tools@latest login
```

過程中可能出現兩個選項：

```text
Enable Gemini in Firebase features? (Y/n)
```

只做教學資料庫連線時可輸入 `n`。

```text
Allow Firebase to collect CLI and Emulator Suite usage and error reporting information? (Y/n)
```

依你的隱私偏好選擇；輸入 `n` 不影響基本連線。

瀏覽器開啟後，選擇正確的 Google 帳號並完成授權。接著回到終端驗證：

```powershell
npx.cmd -y firebase-tools@latest projects:list
```

能列出 Firebase 專案，即表示登入成功。

## 4. 建立或選擇 Firebase 專案

若帳號中還沒有可用專案，對一般使用者而言，從網頁建立最直觀：

1. 開啟 [Firebase Console](https://console.firebase.google.com/)。
2. 選擇「建立專案」或「Add project」。
3. 輸入簡短且唯一的專案 ID，例如 `codex-teaching-tools`。
4. Google Analytics 可依需求決定是否啟用。
5. 完成建立，記下實際的 **Project ID**。

> [!NOTE]
> 顯示名稱可以重複，但 Project ID 必須唯一，而且建立後不能更改。後續設定要填的是 Project ID，不是 Project Number。

原稿曾遇到 CLI 建立專案後，加入 Firebase 資源時發生權限錯誤。因此本流程把「建立專案」留在網頁完成，Codex 負責本機設定、規則檔與部署。

## 5. 啟用 Cloud Firestore

在 Firebase Console 中：

1. 進入目標專案。
2. 開啟「Firestore Database」。
3. 選擇「建立資料庫」。
4. 選擇 Standard edition。
5. 選擇靠近主要使用者的地區。原稿使用 `asia-east1`（台灣）；若介面沒有此選項，可依實際需求選擇其他鄰近地區。
6. 安全性規則選擇「以正式版模式啟動」。
7. 完成建立。

不要選測試模式作為長期設定。測試模式會暫時開放存取；本流程改從預設封鎖開始，再只開放教學工具需要的集合。

> [!CAUTION]
> Firestore 資料庫位置建立後無法任意更換。若未來要連接其他 Google Cloud 服務，請先確認各服務的地區需求。

## 6. 建立 Firebase 設定檔

請 Codex 在你的專案根目錄建立下列三個檔案。

### `firestore.rules`

```text
rules_version = '2';

service cloud.firestore {
  match /databases/{database}/documents {

    // 教學示範集合：任何人都能讀寫，不可存放個資。
    match /wordcloud_words/{document} {
      allow read, write: if true;
    }

    // 其他未明確開放的集合一律拒絕。
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

這是「集合白名單」做法：只開放 `wordcloud_words`，其他集合維持封鎖。新增投票工具時，應另外加入 `votes` 的明確規則，不要把整個資料庫改成公開讀寫。

### `firebase.json`

```json
{
  "firestore": {
    "rules": "firestore.rules"
  }
}
```

### `.firebaserc`

```json
{
  "projects": {
    "default": "YOUR_FIREBASE_PROJECT_ID"
  }
}
```

把 `YOUR_FIREBASE_PROJECT_ID` 換成 Firebase Console 顯示的實際 Project ID。

## 7. 部署 Firestore 規則

部署會直接改變雲端資料庫的存取權限。執行前請逐項確認：

- [ ] Project ID 指向正確專案
- [ ] 只有預定的集合允許公開讀寫
- [ ] 集合中沒有個資或敏感資料
- [ ] 其他集合仍由 `allow read, write: if false` 封鎖

確認後，在專案根目錄執行：

```powershell
npx.cmd -y firebase-tools@latest deploy --only firestore
```

成功時會看到：

```text
Deploy complete!
```

接著到 Firebase Console 的 Firestore「Rules」頁籤，確認線上的規則與本機 `firestore.rules` 一致。

## 8. 完成檢查

- [ ] `projects:list` 能看到正確專案
- [ ] 專案根目錄已有 `.mcp.json`
- [ ] 專案根目錄已有 `firestore.rules`、`firebase.json`、`.firebaserc`
- [ ] Firestore 已建立在預期地區
- [ ] 規則已成功部署
- [ ] Firebase Console 顯示的線上規則與本機一致
- [ ] 示範集合中不含個資

## 之後可以怎麼請 Codex 協助

| 你可以對 Codex 說 | 預期協助內容 |
|---|---|
| 幫我做一個即時文字雲，連接 Firebase | 建立前端頁面、使用 Firestore 儲存文字並即時更新 |
| 幫我新增投票工具的 `votes` 集合 | 先修改 `firestore.rules`，確認後再部署 |
| 幫我檢查目前的 Firebase 連線 | 檢查 CLI 登入、Project ID、設定檔與 MCP 設定 |
| 幫我部署 Firestore 規則 | 先顯示規則差異與風險，取得確認後再部署 |
| 幫我整理匿名課堂回饋 | 讀取並彙整資料，不處理可識別個資 |

## 常見問題

### `npx` 被 PowerShell 擋住

改用 `npx.cmd`。本文所有 Windows 指令都已採用這個寫法。

### `firebase login` 在 Codex 終端無法完成

開啟 Windows PowerShell，手動執行：

```powershell
npx.cmd -y firebase-tools@latest login
```

### 找不到 Firebase 專案

先確認登入的是正確 Google 帳號，再執行：

```powershell
npx.cmd -y firebase-tools@latest projects:list
```

### 寫入資料時出現 `Permission denied`

檢查集合名稱是否與 `firestore.rules` 完全一致，並確認修改後的規則已部署。

### 部署到錯誤的 Firebase 專案

先停止後續操作，檢查 `.firebaserc` 的 `default` Project ID，再到 Firebase Console 檢查剛才部署的線上規則。不要只依賴專案顯示名稱判斷。

### 不想讓任何人都能公開寫入

不要使用 `allow read, write: if true`。先設計 Firebase Authentication、App Check 或其他存取條件，再部署相對應的規則。

## 會建立或修改的檔案

```text
<你的專案資料夾>\
├─ .mcp.json
├─ .firebaserc
├─ firebase.json
└─ firestore.rules
```

Firebase CLI 可能另外產生 `firebase-debug.log`。它是排查錯誤用的紀錄，不是專案核心檔案；分享或提交前應先檢查內容，避免包含本機路徑或其他不應公開的資訊。

## 官方參考資料

- [Firebase CLI](https://firebase.google.com/docs/cli)
- [Firebase MCP server](https://firebase.google.com/docs/ai-assistance/mcp-server)
- [Get started with Cloud Firestore Security Rules](https://firebase.google.com/docs/firestore/security/get-started)
- [Cloud Firestore locations](https://firebase.google.com/docs/firestore/locations)

## 更新紀錄

| 日期 | 版本 | 更新內容 |
|---|---|---|
| 2026-05-24 | v1.0 | 原稿完成 Codex 實測流程 |
| 2026-06-20 | v1.1 | 重整操作順序、補齊空白章節、移除個人路徑與 Project Number、強化安全警示與完成檢查，並依 Firebase 官方文件核對指令 |
