# 使用代理技能與 Gemini CLI

## 設定

### 選項 1：安裝為技能（推薦）

Gemini CLI 具有內建的技能系統，它會自動在 `.gemini/skills/` 或 `.agents/skills/` 目錄中搜尋 `SKILL.md` 文件。每個技能在符合您的任務時會按需啟用。

**從儲存庫安裝：**

```bash
gemini skills install https://github.com/addyosmani/agent-skills.git --path skills
```

**或從本地克隆安裝：**

```bash
git clone https://github.com/addyosmani/agent-skills.git
gemini skills install /path/to/agent-skills/skills/
```

**僅為特定工作區安裝：**

```bash
gemini skills install /path/to/agent-skills/skills/ --scope workspace
```

安裝在工作區範圍內的技能會存入 `.gemini/skills/`（或 `.agents/skills/`）。使用者層級的技能會存入 `~/.gemini/skills/`。

安裝後，請使用以下命令驗證：

```
/skills list
```

Gemini CLI 會自動將技能名稱和描述注入到提示中。當它識別到匹配的任務時，它會請求權限以啟用該技能，然後才載入其完整指令。

### 選項 2：GEMINI.md（持久上下文）

對於您希望始終作為持久專案上下文（而非按需啟用）的技能，請將它們添加到專案的 `GEMINI.md` 中：

```bash
# 將核心技能作為持久上下文建立 GEMINI.md
cat /path/to/agent-skills/skills/incremental-implementation/SKILL.md > GEMINI.md
echo -e "\n---\n" >> GEMINI.md
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> GEMINI.md
```

您也可以透過匯入單獨的文件進行模組化：

```markdown
# 專案指令

@skills/test-driven-development/SKILL.md
@skills/incremental-implementation/SKILL.md
```

使用 `/memory show` 來驗證載入的上下文，使用 `/memory reload` 在更改後刷新。

> **技能與 GEMINI.md 的區別：** 技能是按需啟用的專業知識，僅在相關時才啟用，保持您的上下文視窗的乾淨。GEMINI.md 提供為每個提示載入的持久上下文。請使用技能進行階段特定的工作流程，並使用 GEMINI.md 進行全時專案慣例。

## 推薦的設定

### 全時開啟（GEMINI.md）

將以下內容作為每個會話的持久上下文：

- `incremental-implementation` — 以小塊可驗證的區塊進行建構
- `code-review-and-quality` — 五軸審查

### 按需啟用（技能）

將這些技能安裝為技能，以便它們僅在相關時啟用：

- `test-driven-development` — 在實作邏輯或修復錯誤時啟用
- `spec-driven-development` — 在開始新專案或功能時啟用
- `frontend-ui-engineering` — 在建構使用者介面時啟用
- `security-and-hardening` — 在安全審查期間啟用
- `performance-optimization` — 在效能工作中啟用

## 進階設定

### MCP 整合

此套件中的許多技能利用 [模型上下文協議 (MCP)](https://modelcontextprotocol.io/) 工具與環境互動。例如：

- `browser-testing-with-devtools` 使用 `chrome-devtools` MCP 擴充功能。
- `performance-optimization` 可以受益於與效能相關的 MCP 工具。

要啟用這些功能，請確保您已在 Gemini CLI 設定 (`~/.gemini/config.json`) 中安裝了相關的 MCP 擴充功能。

### 會話掛鉤 (Session Hooks)

Gemini CLI 支援會話生命週期掛鉤。您可以使用它們在會話開始時自動注入上下文或運行驗證腳本。

要複製其他工具中的 `agent-skills` 體驗，您可以配置一個 `SessionStart` 掛鉤，提醒您可用的技能或載入一個元技能。此儲存庫中的 `hooks/session-start.sh` 是一個現成的腳本：它將 `using-agent-skills` 元技能作為標準的 `SessionStart` JSON 包輸出到標準輸出。

### 明確的上下文載入

您可以透過在提示中使用 `@` 符引用任何技能，將其明確載入到當前會話中：

```markdown
使用 @skills/test-driven-development/SKILL.md 技能來實作這個修復。
```

當您希望確保遵循特定工作流程而無需等待自動發現時，這非常有用。

## 斜線指令 (Slash Commands)

該儲存庫在 `.gemini/commands/` 下提供了 9 個斜線指令：8 個生命週期指令以及 `/webperf` 專家的審核。當您從專案根目錄運行時，Gemini CLI 會自動發現它們。

| 指令 | 功能 |
|---------|--------------|
| `/spec` | 在編寫程式碼前撰寫結構化規格 |
| `/constraints` | 定義和強制專案的品質標準 |
| `/planning` | 將工作分解為小而可驗證的任務 |
| `/build` | 逐步實作下一個任務 |
| `/test` | 運行 TDD 工作流程 — 紅、綠、重構 |
| `/review` | 五軸程式碼審查 |
| `/code-simplify` | 在不改變行為的情況下減少複雜度 |
| `/ship` | 透過平行角色分流進行預發布檢查清單 |
| `/webperf` | 對瀏覽器相關應用程式進行核心網路指標和效能問題審查 |

每個指令都會自動呼叫對應的技能 — 無需手動載入技能。

> **注意：** 請使用 `/planning` 而非 `/plan` — `/plan` 與 Gemini CLI 內部指令名稱衝突。

## 使用提示

1. **優先使用技能而非 GEMINI.md** — 技能按需啟用並保持您的上下文視窗的專注。只有當您希望它們始終載入時，才將技能放入 GEMINI.md 中。
2. **技能描述很重要** — 每個 `SKILL.md` 都有一個前置元數據中的 `description` 欄位，用於告知代理何時應啟用它。此儲存庫中的描述針對所有支援的工具（Claude Code、Gemini CLI 等）的自動發現進行了優化，它清楚地說明了技能是**做什麼**以及**何時**應該被觸發。
3. **使用代理進行審查** — 在要求結構化的程式碼審查時，複製 `agents/code-reviewer.md` 的內容。
4. **與參考結合** — 在處理測試或效能等特定品質領域時，引用 `references/` 中的檢查清單。
