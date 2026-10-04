# AGENTS.md

此文件為 AI 編碼代理（Claude Code、Cursor、Copilot、Antigravity 等）在處理本儲存庫中的程式碼時的指導。

> **範圍：** 此文件配置了在 [`addyosmani/agent-skills`](https://github.com/addyosmani/agent-skills) 儲存庫上工作的代理。它不應複製到其他專案或全局代理配置中；可重用的資產是 `skills/` 中的技能，而不是此文件。

## 儲存庫概覽

為 Claude.ai 和 Claude Code 的資深軟體工程師提供的一系列技能。技能是擴展 Claude 和您編碼代理能力的指導和腳本。

## OpenCode 整合

OpenCode 使用**技能驅動的執行模型**，由 `skill` 工具和本儲存庫的 `/skills` 目錄提供支持。

### 核心規則

- 如果任務與某個技能匹配，您**必須**呼叫它。
- 技能位於 `skills/<skill-name>/SKILL.md`。
- 如果有技能適用，**絕不**直接實作。
- 始終嚴格遵循技能指示（不要部分應用）。

### 意圖 → 技能對應

代理應自動將使用者意圖映射到技能：

- 功能/新功能 → `spec-driven-development`，然後是 `incremental-implementation`，`test-driven-development`
- 規劃/分解 → `planning-and-task-breakdown`
- Bug/失敗/意外行為 → `debugging-and-error-recovery`
- 程式碼審查 → `code-review-and-quality`
- 重構/簡化 → `code-simplification`
- API 或介面設計 → `api-and-interface-design`
- UI 工作 → `frontend-ui-engineering`

### 生命週期對應（隱式指令）

OpenCode 不支援如 `/spec` 或 `/plan` 等斜線指令。

相反，代理必須在內部遵循此生命週期：

- 定義 (DEFINE) → `spec-driven-development`
- 規劃 (PLAN) → `planning-and-task-breakdown`
- 建置 (BUILD) → `incremental-implementation` + `test-driven-development`
- 驗證 (VERIFY) → `debugging-and-error-recovery`
- 審查 (REVIEW) → `code-review-and-quality`
- 發布 (SHIP) → `shipping-and-launch`

### 執行模型

對於每一個請求：

1. 確定是否有任何技能適用（即使只有 1% 的機率）。
2. 使用 `skill` 工具呼叫適當的技能。
3. 嚴格遵循技能工作流程。
4. 僅在所需步驟（規範、計畫等）完成後才進行實作。

### 反合理化

以下想法是錯誤的，必須忽略：

-「這對一個技能來說太小了」
-「我可以直接快速實作這個」
-「我會先收集上下文」

正確行為：

- 始終先檢查並使用技能。

這確保 OpenCode 的行為與 Claude Code 類似，具有完整的流程強制執行。

## 編排：角色、技能和指令

本儲存庫有三個可組合的層次。它們有不同的職責，不應混淆：

- **技能** (`skills/<name>/SKILL.md`) — 具有步驟和退出標準的工作流程。**如何做**。當意圖匹配時的強制跳轉。
- **角色** (`agents/<role>.md`) — 具有視角和輸出格式的角色。**誰**。
- **斜線指令** (`.claude/commands/*.md`) — 面向使用者的入口點。**何時**。編排層。

組合規則：**使用者（或斜線指令）是編排者。角色不呼叫其他角色。** 一個角色可以呼叫技能。

本儲存庫唯一支持的多角色編排模式是**平行分發與合併步驟**——由 `/ship` 用於同時運行 `code-reviewer`、`security-auditor` 和 `test-engineer` 並綜合它們的報告。不要建立一個決定呼叫哪個其他角色的「路由器」角色；這是斜線指令和意圖映射的工作。

請參閱 [docs/agents.md](docs/agents.md) 以查看決策矩陣，以及 [references/orchestration-patterns.md](references/orchestration-patterns.md) 以查看完整的模式目錄。

**Claude Code 互操作性：** `agents/` 中的角色作為 Claude Code 的子代理（從此插件的 `agents/` 目錄自動發現）以及代理團隊的隊友（在生成時按名稱引用）。兩個平台約束與我們的規則一致：子代理不能生成其他子代理，團隊不能嵌套。插件代理會靜默忽略 `hooks`、`mcpServers` 和 `permissionMode` 的前置元數據欄位。

## 建立新技能

> **在開始之前：** 在 [CONTRIBUTING.md](CONTRIBUTING.md#before-proposing-a-new-skill) 中運行預先檢查，搜尋目錄，檢查已開的 PR (`gh pr list --state open`)，確認想法符合 [docs/skill-anatomy.md](docs/skill-anatomy.md)，並在 PR 描述中說明差距。大多數新技能想法與現有技能或已開的 PR 重疊；最好是擴展現有技能而不是添加幾乎重複的內容。`CONTRIBUTING.md` 是此工作流程的單一事實來源。

本儲存庫中的技能採用 Markdown 為首：每個技能位於 `skills/<kebab-case-name>/SKILL.md`，具有 YAML 前置元數據（`name`，`description`），並遵循區塊結構（概覽、何時使用、流程、常見合理化、危險信號、驗證）。只有在技能發布可執行輔助程序時才添加 `scripts/` 目錄；大多數技能僅為 Markdown，且沒有每個技能的 zip 套件。

要獲取完整的格式、命名約定、前置元數據規則、支持文件閾值和寫作原則，請參閱 [docs/skill-anatomy.md](docs/skill-anatomy.md)，這是技能結構的單一事實來源。請勿在此處重複該指導，請連結到它。
