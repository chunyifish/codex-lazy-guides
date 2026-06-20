# Codex 環境建置懶人包（Windows 主版）

> 版本：v1.0  
> 更新日期：2026-06-20  
> 適用對象：第一次在 Windows 電腦使用 Codex，或更換電腦後需要重新確認工作環境的人

## 這份懶人包要做什麼？

這份懶人包協助你先檢查電腦環境，再依實際需求安裝工具。它不會要求每個人把所有工具都裝齊。

| 工具 | 用途 | 是否必要 |
| --- | --- | --- |
| Codex 桌面版 | 讓 Codex 在本機專案中讀取、編輯與執行工作 | 必要；你正在使用即可 |
| Git | 記錄檔案版本，方便比較、復原及發布到 GitHub | 本流程必要 |
| GitHub CLI（`gh`） | 從終端機登入及操作 GitHub | 選用；要用指令發布時再安裝 |
| uv | 管理 Python、Python 套件與工具 | 選用；Python 或 MCP 工作需要時再安裝 |

本文件只處理環境檢查與工具準備。GitHub 登入、建立儲存庫及上傳檔案，應在後續發布流程處理。

## 開始前

- [ ] Codex 桌面版可以正常開啟
- [ ] 已在 Codex 中選擇要工作的專案資料夾
- [ ] 電腦可以連上網路
- [ ] 知道安裝軟體或修改系統設定前，Codex 會要求授權

Codex 桌面版與 Codex CLI 是不同的使用介面。正在使用桌面版時，不需要另外安裝或檢查 CLI。

## 一、快速檢查

在 PowerShell 執行：

```powershell
[System.Environment]::OSVersion.VersionString
Get-Location
Test-NetConnection github.com -Port 443 -InformationLevel Quiet
Get-Command winget -ErrorAction SilentlyContinue
git --version
gh --version
uv --version
git config --global --get user.name
git config --global --get user.email
```

看到「找不到指令」不代表整個流程失敗，只表示該工具尚未安裝或不在 `PATH`。其中 `gh` 與 `uv` 是選用工具，可以保持未安裝。

請把結果整理成以下狀態：

| 項目 | 可用狀態 |
| --- | --- |
| Codex 桌面版 | 可用／不可用／無法確認 |
| 網路 | 可用／受限／無法確認 |
| WinGet | 可用／不可用 |
| Git | 已安裝／未安裝／無法確認 |
| GitHub CLI | 已安裝／未安裝／不需要／無法確認 |
| uv | 已安裝／未安裝／不需要／無法確認 |
| Git 使用者資訊 | 已設定／未設定／暫時不需要 |

> `Test-NetConnection` 失敗也可能是沙盒、防火牆或學校／公司網路限制。若瀏覽器能開啟 GitHub，仍應回報「檢查受限」，不要直接判定沒有網路。

## 二、依需求安裝

只處理檢查後確定缺少、而且本次工作需要的工具。下載、安裝或修改系統設定前，先說明影響並取得使用者授權。

### 1. 安裝 Git（必要）

WinGet 可用時：

```powershell
winget install --id Git.Git -e --source winget
```

WinGet 不可用時，從 [Git for Windows 官方下載頁](https://git-scm.com/downloads/win) 安裝。

安裝後關閉並重新開啟 PowerShell 或 Codex，再檢查：

```powershell
git --version
```

### 2. 安裝 GitHub CLI（選用）

只有需要用終端機操作 GitHub 時才安裝。

```powershell
winget install --id GitHub.cli --source winget
```

安裝程式會調整 `PATH`。安裝後請開啟新的終端機視窗，再檢查：

```powershell
gh --version
```

如果不能使用 WinGet，請從 [GitHub CLI 最新版本頁](https://github.com/cli/cli/releases/latest) 選擇符合電腦架構的安裝檔。不要固定使用舊版版本號，也不要在未確認架構前假設電腦一定是 `amd64`。

GitHub CLI 的登入留到發布流程：

```powershell
gh auth login
```

### 3. 安裝 uv（選用）

只有 Python、Python 工具或特定 MCP 工作需要時才安裝。官方 Windows PowerShell 安裝指令為：

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

若要先查看安裝腳本內容：

```powershell
powershell -c "irm https://astral.sh/uv/install.ps1 | more"
```

安裝後開啟新的 PowerShell，再檢查：

```powershell
uv --version
```

## 三、設定 Git 使用者資訊（準備提交時才需要）

Git 使用者名稱與 Email 會寫進 commit 紀錄，必須詢問使用者，不可自行猜測或沿用其他人的資料。

個人電腦需要設定所有專案的預設值時：

```powershell
git config --global user.name "你的名稱"
git config --global user.email "你的 Email"
```

共用電腦或只想套用目前專案時，移除 `--global` 並在專案資料夾執行：

```powershell
git config user.name "你的名稱"
git config user.email "你的 Email"
```

確認目前專案實際採用的設定：

```powershell
git config --get user.name
git config --get user.email
```

## 四、直接交給 Codex 的執行提示詞

將以下內容貼給 Codex：

```text
請檢查這台 Windows 電腦是否具備目前工作需要的 Codex 環境。

成功條件：
1. 確認目前作業系統、工作資料夾與網路狀態。
2. 確認 Git 是否可用；Git 是本次必要工具。
3. 詢問我是否需要用指令操作 GitHub，再決定是否檢查或安裝 GitHub CLI。
4. 詢問本次是否涉及 Python 或 MCP，再決定是否檢查或安裝 uv。
5. 如果後續要建立 commit，檢查 Git 使用者名稱與 Email；未設定時先詢問我，不得猜測。

請先執行唯讀檢查並用表格回報「已安裝、未安裝、不需要、無法確認」。
不要把選用工具未安裝視為整體失敗。
若需要下載、安裝、修改 PATH 或其他系統設定，先說明原因與影響，取得我的授權後再執行。
遇到權限、網路或沙盒限制時，請明確指出受限環節並提供手動替代方案。
完成後重新驗證，列出已完成、未完成及需要我處理的項目。
```

## 五、最終驗證

必要項目：

```powershell
git --version
```

只有已選擇安裝的工具才驗證：

```powershell
gh --version
uv --version
```

若要建立 commit，再驗證：

```powershell
git config --get user.name
git config --get user.email
```

完成回報應採用以下格式：

```text
Codex 基礎環境檢查完成。

- 已完成：
- 選用且未安裝：
- 尚未完成：
- 無法確認：
- 需要使用者處理：
```

只要 Git 可用，而且本次不需要其他選用工具，就可以回報基礎環境已完成。

## 六、常見問題

| 問題 | 處理方式 |
| --- | --- |
| `winget` 找不到 | 改用各工具的官方下載頁；不要為了安裝一個工具而擅自加入其他套件管理器 |
| 安裝後仍找不到指令 | 完全關閉並重開 PowerShell 或 Codex，再檢查 `PATH` |
| 網路檢查失敗 | 區分無網路、沙盒限制、防火牆與公司／學校網路限制，再提供手動下載方案 |
| 權限不足 | 說明需要哪項權限；由使用者授權，或改成使用者手動安裝 |
| Git 可用，但 `gh` 或 `uv` 不可用 | 若本次不需要選用工具，基礎環境仍可視為完成 |
| `codex --version` 失敗 | 桌面版可正常使用時，不影響本文件流程；不要把 CLI 當成桌面版的必要檢查 |

## 七、macOS／Linux 簡要說明

本懶人包的可執行指令以 Windows PowerShell 為準。macOS 或 Linux 請依官方文件選擇符合系統的方式，不要直接執行本文的 PowerShell 指令：

- [Git 官方下載](https://git-scm.com/downloads)
- [GitHub CLI macOS 安裝說明](https://github.com/cli/cli/blob/trunk/docs/install_macos.md)
- [GitHub CLI Linux 安裝說明](https://github.com/cli/cli/blob/trunk/docs/install_linux.md)
- [uv 官方安裝說明](https://docs.astral.sh/uv/getting-started/installation/)
- [Codex 官方快速入門](https://developers.openai.com/codex/quickstart/)

## 官方參考資料

- [Codex Windows 桌面版](https://developers.openai.com/codex/app/windows/)
- [Git for Windows](https://git-scm.com/downloads/win)
- [GitHub CLI Windows 安裝說明](https://github.com/cli/cli/blob/trunk/docs/install_windows.md)
- [GitHub CLI 快速入門](https://docs.github.com/en/github-cli/github-cli/quickstart)
- [uv 安裝說明](https://docs.astral.sh/uv/getting-started/installation/)
