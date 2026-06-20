# Codex 連接 Google NotebookLM：跨電腦執行版

> 適用對象：把本文件交給 Codex，讓 Codex 協助在新電腦完成安裝、登入、串接與驗證。  
> 主要環境：Windows PowerShell。macOS/Linux 差異請見文末附錄。  
> 文件查核日期：2026-06-20。

## 完成條件

只有下列項目全部通過，才可回報「NotebookLM 已成功連接 Codex」：

- `uv`、`nlm` 與 `notebooklm-mcp` 指令可用。
- 使用者已在這台電腦完成 Google 登入。
- `nlm notebook list` 能正常回應；清單為空也算連線成功。
- Codex 已載入 `notebooklm-mcp`，並能呼叫 `notebook_list`。
- 若使用者同意寫入測試，測試 notebook 已建立、核對 ID 並刪除。

## 重要限制與安全提醒

- `notebooklm-mcp-cli` 是第三方開源工具，不是 Google 官方 API。
- 此工具使用 NotebookLM 的內部 API；介面可能在未預告的情況下改變。
- 登入流程會從專用瀏覽器工作階段取得 Cookie 與認證資訊，並儲存在本機使用者目錄。
- 不得從其他電腦複製 Cookie、瀏覽器設定檔或認證資料。每台電腦都要重新登入。
- 不得在畫面、對話、日誌或 Git 儲存庫中輸出 Cookie、CSRF token 或其他登入憑證。
- 建立、刪除 notebook 前必須取得使用者同意；只可刪除本次建立且 ID 已核對的測試 notebook。
- 任何必要指令失敗時，先停止並回報錯誤，不要跳過驗證後宣稱完成。

## 交給 Codex 的執行規則

1. 先執行唯讀檢查，再決定是否需要安裝或修改設定。
2. 每完成一個階段，都要回報「完成項目、驗證結果、下一步」。
3. 需要使用者登入 Google、關閉瀏覽器、重開 Codex 或同意寫入測試時，暫停並清楚提示。
4. 不要安裝 Git；本流程從 PyPI 安裝套件，不需要 Git。
5. 不要複製其他電腦的 NotebookLM 認證資料。

## Windows 主流程

### 1. 環境預檢

在 PowerShell 執行：

```powershell
$PSVersionTable.PSVersion
[System.Environment]::OSVersion.VersionString
Get-Command codex -ErrorAction SilentlyContinue | Format-List Name,Source
Get-Command uv -ErrorAction SilentlyContinue | Format-List Name,Source
Get-Command nlm -ErrorAction SilentlyContinue | Format-List Name,Source
```

同時確認：

- 電腦可以連線至網路。
- 已安裝 Codex App 或 Codex CLI。
- 使用者能登入 `https://notebooklm.google.com/`。
- 已安裝 Chrome、Edge、Brave 或其他受支援的 Chromium 瀏覽器。

**通過條件：** 作業系統與 Codex 使用方式已確認；缺少 `uv` 或 `nlm` 可以進入後續安裝步驟。

### 2. 安裝 uv

若 `uv --version` 已成功，保留現有安裝並前往下一步。否則執行 uv 官方安裝指令：

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

安裝後重新開啟 PowerShell，再驗證：

```powershell
uv --version
```

若新開的 PowerShell 仍找不到 `uv`，先檢查預設安裝位置：

```powershell
Test-Path "$HOME\.local\bin\uv.exe"
```

若檔案存在，可只在目前工作階段補上路徑後再驗證：

```powershell
$env:Path = "$HOME\.local\bin;$env:Path"
uv --version
```

**停止條件：** `uv --version` 仍失敗。請回報完整錯誤，不要繼續安裝 NotebookLM 工具。

### 3. 安裝或更新 NotebookLM CLI 與 MCP Server

先檢查：

```powershell
uv tool list
```

若尚未安裝：

```powershell
uv tool install notebooklm-mcp-cli
```

若已安裝，更新至目前可取得的版本：

```powershell
uv tool upgrade notebooklm-mcp-cli
```

驗證兩個入口：

```powershell
nlm --version
notebooklm-mcp --help
```

若找不到指令，重新開啟 PowerShell；仍失敗時，可在目前工作階段補上：

```powershell
$env:Path = "$HOME\.local\bin;$env:Path"
nlm --version
```

**通過條件：** `nlm --version` 與 `notebooklm-mcp --help` 都能正常回應。不要用固定版號判定成功。

### 4. 登入 Google NotebookLM

先請使用者完全關閉所有 Chromium 瀏覽器，再執行：

```powershell
nlm login
```

此步驟會開啟專用瀏覽器工作階段。請使用者親自完成 Google 登入；Codex 不得要求或代填密碼、驗證碼。

登入完成後執行：

```powershell
nlm login --check
nlm doctor
```

如果瀏覽器未自動開啟，可改用：

```powershell
nlm login --manual
```

`--manual` 模式可能要求使用者提供 Cookie 檔案。不得把該檔案加入專案、上傳或貼進對話。

**通過條件：** 認證檢查成功，且不是 `not_configured` 或 `stale`。若顯示 `unverified`，先檢查網路並實際執行下一階段的唯讀清單測試，不要直接判定認證失效。

### 5. 設定 Codex MCP

優先使用工具提供的 Codex 自動設定：

```powershell
nlm setup add codex
codex mcp get notebooklm-mcp
```

如果自動設定失敗，先找出 MCP Server 的完整路徑：

```powershell
$mcpCommand = (Get-Command notebooklm-mcp -ErrorAction Stop).Source
$mcpCommand
```

再使用 Codex 官方 MCP 指令新增：

```powershell
codex mcp add notebooklm-mcp -- "$mcpCommand"
codex mcp get notebooklm-mcp
```

若 Codex CLI 無法使用，可在 Codex App 開啟「Settings → Configuration → Open config.toml」，把下列設定加入使用者層級的 `~/.codex/config.toml`：

```toml
[mcp_servers.notebooklm-mcp]
command = 'C:\Users\你的使用者名稱\.local\bin\notebooklm-mcp.exe'
```

請把範例路徑換成 `$mcpCommand` 顯示的完整路徑；不要把 `$HOME` 原樣寫入 TOML。

**通過條件：** `codex mcp get notebooklm-mcp` 顯示已啟用的 STDIO Server，且啟動命令指向實際存在的 `notebooklm-mcp`。

### 6. 驗證 NotebookLM CLI

先執行不會改動資料的清單測試：

```powershell
nlm notebook list
```

清單有內容或為空都可以；只要指令正常完成，就代表 CLI 已能連接 NotebookLM。

#### 可選：建立與刪除測試

只有使用者明確同意後才能執行：

```powershell
nlm notebook create "Codex NotebookLM 連線測試"
```

1. 記下指令回傳的 notebook ID。
2. 使用下列指令核對標題與 ID：

```powershell
nlm notebook get <剛剛建立的-notebook-ID>
```

3. 只有標題與 ID 都吻合時，才能刪除：

```powershell
nlm notebook delete <剛剛建立的-notebook-ID> --confirm
```

4. 再次執行 `nlm notebook list`，確認測試 notebook 已不存在。

**禁止事項：** 不可依清單位置、相似名稱或猜測的 ID 刪除 notebook。

### 7. 完全重開 Codex

完成 MCP 設定後：

1. 請使用者完全關閉 Codex App 或結束目前 CLI 工作階段。
2. 重新開啟 Codex。
3. 建立新對話，避免沿用尚未載入新 MCP 工具的舊對話。

### 8. 驗證 Codex MCP

在新對話要求 Codex：

1. 呼叫 `server_info`（若目前版本提供此工具）。
2. 呼叫 `notebook_list`。
3. 回報 MCP Server 版本、認證狀態與筆記本清單結果，但不要輸出任何 Cookie 或 token。

**通過條件：** `notebook_list` 回傳成功。CLI 測試成功但 MCP 測試失敗時，不可回報串接完成，應回到「設定 Codex MCP」檢查命令路徑與設定檔。

## 完成回報範本

全部必要驗證成功後，使用以下格式回報：

```text
NotebookLM 已成功連接 Codex。

- CLI 驗證：nlm notebook list 成功
- MCP 驗證：notebook_list 成功
- 寫入測試：未執行／已建立並刪除測試 notebook
- 尚未驗證：無（若有，請逐項列出）
```

## 常見問題

| 症狀 | 常見原因 | 處理方式 | 再次驗證 |
|---|---|---|---|
| 找不到 `uv` | 新安裝路徑尚未載入 | 重開 PowerShell；必要時暫時加入 `$HOME\.local\bin` | `uv --version` |
| 找不到 `nlm` 或 `notebooklm-mcp` | 工具未安裝完成或 PATH 尚未更新 | 執行 `uv tool list`，重開 PowerShell | `nlm --version`、`notebooklm-mcp --help` |
| `nlm login` 沒有開啟瀏覽器 | 瀏覽器未完全關閉、偵測失敗或環境限制 | 完全關閉 Chromium 瀏覽器後重試；必要時用 `nlm login --manual` | `nlm login --check` |
| 認證為 `stale` | Google 工作階段已失效 | 重新執行 `nlm login` | `nlm login --check`、`nlm notebook list` |
| 認證為 `unverified` | 網路、DNS、Proxy 或暫時性連線問題 | 先檢查網路，不要直接清除認證 | `nlm notebook list` |
| `nlm setup add codex` 失敗 | Codex CLI 不在 PATH 或自動設定未成功 | 找出兩個可執行檔的完整路徑，改用 `codex mcp add` 或 `config.toml` | `codex mcp get notebooklm-mcp` |
| Codex 看不到 NotebookLM 工具 | MCP 工具清單尚未重新載入 | 完全關閉並重開 Codex，建立新對話 | 呼叫 `notebook_list` |
| CLI 成功但 MCP 失敗 | MCP 啟動命令錯誤或設定未載入 | 核對 `Get-Command notebooklm-mcp` 與 Codex MCP 設定 | `codex mcp get notebooklm-mcp`、`notebook_list` |

## 移除、重設與重新登入

只有使用者要求重設時才執行，並逐步確認：

### 移除 Codex MCP 設定

```powershell
codex mcp remove notebooklm-mcp
```

若先前直接編輯 `config.toml`，則移除整個 `[mcp_servers.notebooklm-mcp]` 區塊。

### 查看並刪除指定登入 Profile

先列出 Profile：

```powershell
nlm login profile list
```

確認名稱後，只刪除指定 Profile：

```powershell
nlm login profile delete <profile-name>
```

不要直接刪除整個 `~/.notebooklm-mcp-cli/`，除非使用者已確認要清除所有帳號與專用瀏覽器資料。

### 移除工具

```powershell
uv tool uninstall notebooklm-mcp-cli
```

重新安裝時，從本文件「安裝或更新 NotebookLM CLI 與 MCP Server」開始，再重新登入與設定 MCP。

## 可選：建立本地輸出資料夾

這些資料夾不影響連線，可在確定需要下載 NotebookLM 產物時建立：

```powershell
$base = "$HOME\Documents\NotebookLM"
$dirs = "slides","infographics","audio","video","docs","sheets","mindmaps","quizzes"
foreach ($dir in $dirs) {
  New-Item -ItemType Directory -Force -Path (Join-Path $base $dir) | Out-Null
}
Get-ChildItem -LiteralPath $base -Directory | Sort-Object Name
```

## macOS/Linux 附錄

主流程與 Windows 相同，只替換安裝、PATH 與路徑相關指令。

### 安裝 uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

重新開啟終端機；若仍找不到 `uv`，依目前 Shell 載入設定：

```bash
source ~/.zshrc   # zsh
source ~/.bashrc  # bash
uv --version
```

### 安裝與驗證 NotebookLM 工具

```bash
uv tool install notebooklm-mcp-cli
nlm --version
notebooklm-mcp --help
```

已安裝時使用：

```bash
uv tool upgrade notebooklm-mcp-cli
```

### 設定 Codex MCP

```bash
nlm setup add codex
codex mcp get notebooklm-mcp
```

自動設定失敗時：

```bash
codex mcp add notebooklm-mcp -- "$(command -v notebooklm-mcp)"
codex mcp get notebooklm-mcp
```

登入、CLI 驗證、重開 Codex 與 MCP 驗證方式均與 Windows 主流程相同。

## 查核來源

- [NotebookLM MCP CLI 官方 GitHub](https://github.com/jacob-bd/notebooklm-mcp-cli)
- [notebooklm-mcp-cli — PyPI](https://pypi.org/project/notebooklm-mcp-cli/)
- [uv 官方安裝文件](https://docs.astral.sh/uv/getting-started/installation/)
- [Codex 官方 MCP 文件](https://developers.openai.com/codex/mcp)

> 版本會持續更新。若指令與文件不一致，以查核來源的最新版文件及實際 `--help` 輸出為準，並在回報中說明差異。
