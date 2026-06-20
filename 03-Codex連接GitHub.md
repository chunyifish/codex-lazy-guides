# Codex 連接 GitHub 懶人包（Windows 主版）

> 適用對象：第一次讓 Codex 連接 GitHub，或更換 Windows 電腦後需要重新設定的人  
> 主要環境：Codex 桌面版、Windows PowerShell  
> 文件查核日期：2026-06-20

## 完成條件

只有以下項目全部通過，才可回報「Codex 已成功連接 GitHub」：

- Git 與 GitHub CLI（`gh`）可以執行。
- 使用者已親自在這台電腦完成 GitHub 授權。
- Git commit 使用者名稱與 Email 已依使用者選擇設定。
- 測試 repo 已建立並成功推送 `index.html`。
- GitHub Pages 已建置完成，而且實際網址可以開啟。
- 已詢問使用者要保留或刪除測試 repo；刪除前已再次確認完整名稱。

## 安全規則

- 每台電腦都要重新檢查 Git、`gh`、登入狀態、Git 使用者資訊、網路與工作資料夾，不沿用其他電腦的結果。
- 安裝軟體、登入 GitHub、修改全域 Git 設定、建立公開 repo、開啟 Pages 或刪除 repo 前，先說明影響並取得使用者同意。
- 不要求使用者在對話中提供 GitHub 密碼、驗證碼、token 或 Cookie。
- 不把憑證、私人資料或測試用祕密寫入檔案、commit 或終端機紀錄。
- 必要指令失敗時停止目前階段並回報錯誤，不可跳過驗證後宣稱完成。
- `safe.directory` 只能加入 Git 錯誤指出、且已核對的當前 repo 路徑；不可使用萬用值或其他電腦的固定路徑。

## Windows PowerShell 主流程

### 1. 唯讀環境預檢

```powershell
$PSVersionTable.PSVersion
[System.Environment]::OSVersion.VersionString
Get-Location
Test-NetConnection github.com -Port 443 -InformationLevel Quiet
Get-Command git -ErrorAction SilentlyContinue | Format-List Name,Source
Get-Command gh -ErrorAction SilentlyContinue | Format-List Name,Source
git --version
gh --version
gh auth status
git config --get user.name
git config --get user.email
```

若 `git` 或 `gh` 尚未安裝，相關指令失敗是預期結果。先整理狀態，再處理缺少的項目：

| 項目 | 狀態 |
| --- | --- |
| 工作資料夾 | 已確認／需使用者指定 |
| 網路 | 可用／受限／無法確認 |
| Git | 已安裝／未安裝／無法確認 |
| GitHub CLI | 已安裝／未安裝／無法確認 |
| GitHub CLI 登入 | 已登入／未登入／無法確認 |
| Git 使用者名稱 | 已設定／未設定 |
| Git 使用者 Email | 已設定／未設定 |

> `Test-NetConnection` 失敗也可能是 Codex 沙盒、防火牆或學校／公司網路限制。若瀏覽器仍能開啟 GitHub，應回報「檢查受限」，不要直接判定完全沒有網路。

**通過條件：** 作業系統、工作資料夾及各工具狀態已確認。缺少工具可以進入安裝階段。

### 2. 安裝缺少的工具

只安裝預檢確認缺少的工具，並在安裝前取得使用者同意。

```powershell
winget install --id Git.Git -e --source winget
winget install --id GitHub.cli -e --source winget
```

WinGet 不可用時，改用 [Git for Windows 官方下載頁](https://git-scm.com/downloads/win) 與 [GitHub CLI 最新版本頁](https://github.com/cli/cli/releases/latest)。依實際 Windows 架構選擇安裝檔或壓縮檔，不固定版本號，也不預設一定是 `amd64`。

安裝後完全關閉並重新開啟 PowerShell 或 Codex，再驗證：

```powershell
git --version
gh --version
$GH_CMD = (Get-Command gh -ErrorAction Stop).Source
& $GH_CMD --version
```

若使用解壓縮的可攜版，將 `$GH_CMD` 設為實際 `gh.exe` 完整路徑，後續一律用 `& $GH_CMD` 執行。

**停止條件：** `git --version` 或 `& $GH_CMD --version` 仍失敗。回報完整錯誤，不進入登入階段。

### 3. 登入 GitHub CLI

無論是否執行過安裝步驟，都先確認這次流程使用的 `gh` 路徑：

```powershell
if (-not $GH_CMD) {
    $GH_CMD = (Get-Command gh -ErrorAction Stop).Source
}
& $GH_CMD --version
```

```powershell
& $GH_CMD auth status
```

若尚未登入，取得同意後執行：

```powershell
& $GH_CMD auth login --hostname github.com --web --git-protocol https
```

由使用者親自在瀏覽器完成授權。若瀏覽器沒有自動開啟，手動前往 [GitHub 裝置登入頁](https://github.com/login/device)，輸入終端機顯示的裝置碼。

```powershell
& $GH_CMD auth status
$GITHUB_OWNER = & $GH_CMD api user --jq '.login'
$GITHUB_OWNER
```

**通過條件：** `auth status` 顯示已登入 `github.com`，而且 `$GITHUB_OWNER` 回傳目前帳號名稱。

### 4. 設定 Git 使用者資訊

Git 名稱與 Email 會寫進 commit，不可猜測或沿用其他人的資料。

```powershell
git config --get user.name
git config --get user.email
```

尚未設定時，詢問使用者要使用的值。若只套用本次測試 repo，先記下資料；此時尚未執行 `git init`，不要先執行 `git config`：

```powershell
$GIT_NAME = "使用者指定的名稱"
$GIT_EMAIL = "使用者指定的 Email"
```

只有使用者明確要求所有專案共用時，才設定全域值：

```powershell
git config --global user.name "使用者指定的名稱"
git config --global user.email "使用者指定的 Email"
```

設定後重新執行唯讀檢查，確認實際值正確。

### 5. 建立測試 repo 並推送

這一步會在本機建立資料夾與 commit，並在 GitHub 建立公開 repo。執行前必須確認工作位置、repo 名稱與公開狀態。

```powershell
$WORK_DIR = (Get-Location).Path
$TEST_REPO = 'codex-github-test-' + (Get-Date -Format 'yyyyMMdd-HHmmss')
$REPO_PATH = Join-Path $WORK_DIR $TEST_REPO

& $GH_CMD repo view "$GITHUB_OWNER/$TEST_REPO" 2>$null
if ($LASTEXITCODE -eq 0) {
    throw "GitHub repo 已存在，請產生新的名稱：$TEST_REPO"
}

New-Item -ItemType Directory -Path $REPO_PATH -ErrorAction Stop | Out-Null
Set-Location -LiteralPath $REPO_PATH
git init
git branch -M main

if ($GIT_NAME -and $GIT_EMAIL) {
    git config user.name "$GIT_NAME"
    git config user.email "$GIT_EMAIL"
}

git config --get user.name
git config --get user.email
```

建立測試頁面並提交：

```powershell
@'
<!doctype html>
<html lang="zh-Hant">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Codex GitHub 測試</title>
</head>
<body>
  <h1>Hello! Codex 已成功連接 GitHub。</h1>
</body>
</html>
'@ | Set-Content -LiteralPath '.\index.html' -Encoding UTF8

git add index.html
git commit -m "Add GitHub Pages test page"
```

若 Git 明確回報 `dubious ownership`，先核對錯誤路徑就是 `$REPO_PATH`，取得同意後只加入該路徑：

```powershell
$SAFE_DIR = (Resolve-Path -LiteralPath $REPO_PATH).Path.Replace('\', '/')
git config --global --add safe.directory "$SAFE_DIR"
```

不要設定 `safe.directory '*'`。重新執行原本失敗的 Git 指令，確認 commit 成功後才建立遠端：

```powershell
& $GH_CMD repo create $TEST_REPO --public --source=. --remote=origin --push
& $GH_CMD repo view "$GITHUB_OWNER/$TEST_REPO" --web
```

**通過條件：** `git status` 沒有未提交的 `index.html`，`origin` 指向正確 repo，GitHub 頁面能看到該檔案。

### 6. 開啟並驗證 GitHub Pages

此操作會變更 repo 設定，必須先取得同意。

```powershell
$REPO_FULL_NAME = "$GITHUB_OWNER/$TEST_REPO"

& $GH_CMD api --method POST "repos/$REPO_FULL_NAME/pages" `
    -H 'Accept: application/vnd.github+json' `
    -H 'X-GitHub-Api-Version: 2022-11-28' `
    -F 'source[branch]=main' `
    -F 'source[path]=/'
```

若 API 回報 Pages 已存在，不重複建立，改查狀態：

```powershell
& $GH_CMD api "repos/$REPO_FULL_NAME/pages" `
    -H 'Accept: application/vnd.github+json' `
    -H 'X-GitHub-Api-Version: 2022-11-28' `
    --jq '{status: .status, url: .html_url}'
```

Pages 建置可能需要數分鐘。間隔一段時間重新查詢，直到狀態為 `built` 或出現明確錯誤；不要只等待固定一兩分鐘就判定失敗。

請使用者開啟 API 回傳的網址，確認顯示：

```text
Hello! Codex 已成功連接 GitHub。
```

**通過條件：** API 狀態為 `built`，實際網址可以開啟測試內容。

### 7. 保留或清理測試 repo

測試成功後詢問：

```text
測試 repo「<owner>/<repo>」已建立並上線。你要保留它，還是刪除它？
```

若要保留，到此結束。若要刪除，先再次顯示完整 repo 名稱並取得確認。刪除 repo 需要額外的 `delete_repo` scope：

```powershell
& $GH_CMD auth refresh -h github.com -s delete_repo
& $GH_CMD repo delete "$GITHUB_OWNER/$TEST_REPO" --yes
& $GH_CMD repo view "$GITHUB_OWNER/$TEST_REPO"
```

最後一行應回報 repo 不存在。本機資料夾要另外取得同意後才能刪除；刪除前必須確認解析後的完整路徑等於 `$REPO_PATH`，且位於預期的 `$WORK_DIR` 內。本文件不提供自動刪除指令，避免誤刪其他資料。

## 直接交給 Codex 的執行提示詞

```text
請依這份懶人包協助我在 Windows 電腦完成 Codex 與 GitHub 的連接測試。

成功條件：
1. Git 與 GitHub CLI 可用。
2. GitHub CLI 已由我親自完成授權。
3. Git 使用者名稱與 Email 已依我的選擇設定。
4. 經我同意後建立公開測試 repo、推送 index.html 並開啟 GitHub Pages。
5. Pages 狀態為 built，且實際網址可開啟。
6. 最後詢問我要保留或刪除測試 repo。

請先只做唯讀檢查，用表格回報目前狀態，不要直接安裝或修改設定。
需要安裝、登入、修改全域 Git 設定、建立 repo、開啟 Pages、取得刪除權限或刪除資料時，先說明影響並取得我的同意。
不得要求我在對話中提供密碼、驗證碼、token 或 Cookie。
每個階段完成後，回報「完成項目、驗證結果、下一步」。任何必要驗證失敗時停止，不要跳過後宣稱完成。
```

## 完成回報範本

```text
Codex 與 GitHub 連接流程已完成／尚未完成。

- Git：
- GitHub CLI：
- GitHub 登入：
- Git 使用者資訊：
- 測試 repo：
- push 驗證：
- GitHub Pages 狀態與網址：
- 測試 repo 保留／刪除狀態：
- 尚未驗證或失敗項目：
- 需要使用者處理：
```

只有全部必要項目都通過時，才能寫「已完成」。

## 常見問題

| 症狀 | 處理方式 | 再次驗證 |
| --- | --- | --- |
| 找不到 `git` | 安裝 Git，完全重開 PowerShell 或 Codex | `git --version` |
| 找不到 `gh` | 安裝 GitHub CLI，或指定可攜版 `gh.exe` 完整路徑 | `& $GH_CMD --version` |
| `winget` 找不到 | 改用官方下載頁，不擅自安裝其他套件管理器 | 重新執行版本檢查 |
| 登入時瀏覽器沒開 | 手動開啟裝置登入頁並輸入畫面上的裝置碼 | `& $GH_CMD auth status` |
| `git commit` 缺少身分資訊 | 詢問使用者，優先設定目前 repo | `git config --get user.name` 與 `user.email` |
| Git 顯示 `dubious ownership` | 只把已核對的目前 repo 路徑加入 `safe.directory` | 重試原本失敗的 Git 指令 |
| repo 名稱已存在 | 產生新的秒級時間戳名稱並重新確認 | `gh repo view <owner>/<repo>` 應先顯示不存在 |
| `gh repo create` 失敗 | 檢查登入、網路、repo 名稱、資料夾與 commit | `gh auth status`、`git status` |
| Pages API 回傳 422 | 確認 `main` 已推送，使用 `source[branch]` 與 `source[path]` | 查詢 Pages 狀態 |
| Pages 顯示 404 | 確認 API 狀態；未完成時繼續等待，不重複建立 | 狀態為 `built` 後重開網址 |
| 刪除權限不足 | 經同意後執行 `gh auth refresh -s delete_repo` | 再刪除指定完整名稱的 repo |

## macOS／Linux 附錄

整體階段與安全規則相同，但本文主流程是 Windows PowerShell，不可直接複製其中的 PowerShell 變數、反引號續行或檔案寫入語法。

- [Git 官方下載與安裝](https://git-scm.com/downloads)
- [GitHub CLI macOS 安裝說明](https://github.com/cli/cli/blob/trunk/docs/install_macos.md)
- [GitHub CLI Linux 安裝說明](https://github.com/cli/cli/blob/trunk/docs/install_linux.md)
- [GitHub CLI 手冊](https://cli.github.com/manual/)

## 官方參考資料

- [GitHub CLI：登入](https://cli.github.com/manual/gh_auth_login)
- [GitHub CLI：建立 repo](https://cli.github.com/manual/gh_repo_create)
- [GitHub CLI：刪除 repo](https://cli.github.com/manual/gh_repo_delete)
- [GitHub Pages REST API](https://docs.github.com/en/rest/pages/pages?apiVersion=2022-11-28)
- [GitHub Pages 快速入門](https://docs.github.com/en/pages/quickstart)
- [Git for Windows](https://git-scm.com/downloads/win)

> GitHub CLI 與 API 會持續更新。若本文與實際 `--help` 或官方文件不一致，以當下官方文件為準，並在完成回報中說明差異。
