# OpenCode 設定

本指南解釋如何使用 OpenCode 中的 Agent Skills。可重複使用的資產是 `skills/` 目錄中的 markdown 技能；此儲存庫的根目錄 `AGENTS.md` 是針對儲存庫範圍的，不應複製到其他專案中。

## 概述

OpenCode 從多個位置發現技能。Agent Skills 提供兩種可選的使用風格：

- **Agent 驅動工作流程：** 技能透過內建的 `skill` 工具和您為自己儲存庫撰寫的專案本地 `AGENTS.md` 自動選擇。
- **指令驅動工作流程：** 手動使用 `.opencode/commands/`（可選）來呼叫生命週期指令。

## 安裝

有兩種方法將技能導入您的專案：

1. 使用 `skills` CLI 安裝（最快）。
2. 克隆此儲存庫並手動複製技能目錄。

在完成任一步驟後，建立您自己的專案本地 `AGENTS.md`，如果您需要，請複製 `.opencode/commands/*.md` 文件。

### 選項 1：使用 `npx skills` 安裝

最快的路徑是使用公開的 [`skills` CLI](https://github.com/vercel-labs/skills)：

```bash
npx skills add addyosmani/agent-skills            # 安裝選定的技能
npx skills add addyosmani/agent-skills --list     # 安裝前瀏覽
```

安裝單個技能：

```bash
npx skills add addyosmani/agent-skills --skill spec-driven-development
```

預設情況下，`npx skills` 會安裝到特定工具的目錄中（通常是 `.claude/skills/` 或共用位置）。OpenCode 會發現這些技能，因為它會讀取 `.claude/skills/<name>/SKILL.md` 和通用的 `.agents/skills/<name>/SKILL.md` 路徑。

如果技能被放置在 OpenCode 不掃描的某處，請將它們複製或建立符連結到以下發現路徑之一，例如：

```bash
mkdir -p .opencode/skills
cp -r .claude/skills/<skill-name> .opencode/skills/
```

> **注意：** 每個技能的安裝僅複製技能目錄本身。如果一個技能引用 `references/` 下的共用文件，您可能需要將它們複製到已安裝的技能目錄中，或安裝整個套件。請參閱 [#361](https://github.com/addyosmani/agent-skills/issues/361) 以了解背景資訊。

### 選項 2：克隆此儲存庫

1. 克隆儲存庫：

```bash
git clone https://github.com/addyosmani/agent-skills.git
```

2. 將所需的技能複製到 OpenCode 的技能發現路徑之一。

#### 專案本地安裝

```bash
mkdir -p .opencode/skills
cp -r /path/to/agent-skills/skills/<skill-name> .opencode/skills/
```

例如，要安裝 `spec-driven-development` 和 `incremental-implementation`：

```bash
mkdir -p .opencode/skills
cp -r /path/to/agent-skills/skills/spec-driven-development .opencode/skills/
cp -r /path/to/agent-skills/skills/incremental-implementation .opencode/skills/
```

#### 全域安裝

```bash
mkdir -p ~/.config/opencode/skills
cp -r /path/to/agent-skills/skills/<skill-name> ~/.config/opencode/skills/
```

#### 跨相容路徑

OpenCode 還會發現放置在 Claude 相容或通用代理路徑中的技能：

- `.claude/skills/<name>/SKILL.md`
- `~/.claude/skills/<name>/SKILL.md`
- `.agents/skills/<name>/SKILL.md`
- `~/.agents/skills/<name>/SKILL.md`

如果您已經在 Claude Code 和 OpenCode 之間共享技能，則以上任何位置都適用。

### 應複製的內容

複製 `skills/` 下的目錄（例如 `skills/spec-driven-development/`）。每個目錄必須包含一個 `SKILL.md` 文件。請不要複製儲存庫的根目錄 `AGENTS.md` 或 `CLAUDE.md`；這些文件配置的是該儲存庫本身的開發。

## 專案 `AGENTS.md`

在**您自己的專案**根目錄中建立一個 `AGENTS.md`。這是告訴 OpenCode 何時以及如何呼叫已安裝的技能的系統提示。與 `addyosmani/agent-skills` 中的儲存庫範圍 `AGENTS.md` 不同，此文件屬於您的專案，應根據您的技術棧進行調整。

以下是一個您可以貼入專案 `AGENTS.md` 的範本：

```markdown
# Agent Skills (OpenCode)

此專案使用安裝在 `.opencode/skills/`（或相容路徑）下的技能。

## 核心規則

- 如果任務與某個技能匹配，則在採取行動前使用 `skill` 工具呼叫它。
- 技能位於 `.opencode/skills/<skill-name>/SKILL.md`。
- 嚴格遵循技能工作流程；不要部分應用它。
- 當技能要求時，絕不跳過必要的步驟，例如規格、規劃或測試。

## 意圖 → 技能對應

自動將使用者的意圖映射到匹配的技能：

- 功能/新功能 → `spec-driven-development`，然後是 `incremental-implementation` 和 `test-driven-development`
- 規劃/分解 → `planning-and-task-breakdown`
- Bug/失敗/意外行為 → `debugging-and-error-recovery`
- 代碼審查 → `code-review-and-quality`
- 重構/簡化 → `code-simplification`
- API 或介面設計 → `api-and-interface-design`
- UI 工作 → `frontend-ui-engineering`

## 執行模型

對於每一個請求：

1. 確定是否有任何技能適用（即使是小概率）。
2. 使用 `skill({ name: "<skill-name>" })` 加載該技能。
3. 精確遵循技能工作流程。
4. 僅在所需步驟完成後才進行實施。
```

將此內容儲存為專案根目錄中的 `AGENTS.md`。OpenCode 將自動載入它。

> **注意：** `addyosmani/agent-skills` 儲存庫中的根 `AGENTS.md` 是供在此儲存庫中工作的貢獻者使用的，不應複製到其他專案中。請參閱 [CONTRIBUTING.md](../CONTRIBUTING.md#repo-scoped-files)。

## 工作原理

### 1. 技能發現

OpenCode 會遍歷以下路徑（優先考慮專案本地，然後是全域）：

- `.opencode/skills/<name>/SKILL.md`
- `~/.config/opencode/skills/<name>/SKILL.md`
- `.claude/skills/<name>/SKILL.md`
- `~/.claude/skills/<name>/SKILL.md`
- `.agents/skills/<name>/SKILL.md`
- `~/.agents/skills/<name>/SKILL.md`

每個技能必須在它的前置元數據中包含一個有效的 `name` 和 `description`。

### 2. 自動技能呼叫

當您專案的 `AGENTS.md` 指示代理使用技能時，代理會評估每一個請求並將其映射到適當的技能。

範例：

- "建立一個功能" → `incremental-implementation` + `test-driven-development`
- "設計一個系統" → `spec-driven-development`
- "修復一個 Bug" → `debugging-and-error-recovery`
- "審查這段代碼" → `code-review-and-quality`

### 3. 生命週期對應（隱式指令）

OpenCode 不需要斜線指令，但如果您偏好使用它們，請參閱下一節。在代理驅動模式下，生命週期是隱式映射的：

- 定義 (DEFINE) → `spec-driven-development`
- 規劃 (PLAN) → `planning-and-task-breakdown`
- 建置 (BUILD) → `incremental-implementation` + `test-driven-development`
- 驗證 (VERIFY) → `debugging-and-error-recovery`
- 審查 (REVIEW) → `code-review-and-quality`
- 發布 (SHIP) → `shipping-and-launch`

### 複製可選的斜線指令

如果您偏好明確的指令，請將此儲存庫中的範例指令文件複製到您的專案中，並將它們調整為呼叫您安裝的技能：

```bash
mkdir -p .opencode/commands
cp /path/to/agent-skills/.opencode/commands/*.md .opencode/commands/
```

> **注意：** 該儲存庫的 `main` 分支目前不包含 `.opencode/commands/*.md`。您可以建立自己的指令文件，或關注 PR #200，該 PR 提議添加它們。一旦它們存在，上述模式就適用。

一個典型的指令文件看起來像這樣：

```markdown
---
description: 將工作分解為可驗證的小任務
---

呼叫 planning-and-task-breakdown 技能。閱讀規格並建立具有驗收標準的任務。
```

將其儲存為 `.opencode/commands/plan.md` 以在 OpenCode 中啟用 `/plan`。

## 使用範例

### 範例 1：功能開發

使用者：
```
為這個應用程式新增身份驗證
```

代理行為：
- 偵測到功能工作
- 呼叫 `spec-driven-development`
- 在編寫代碼前產生規格
- 轉向規劃和實施技能

### 範例 2：修復 Bug

使用者：
```
這個端點回傳 500 錯誤
```

代理行為：
- 呼叫 `debugging-and-error-recovery`
- 重現 → 定位 → 修復 → 添加保護措施

### 範例 3：代碼審查

使用者：
```
審查這個 PR
```

代理行為：
- 呼叫 `code-review-and-quality`
- 應用結構化審查（正確性、設計、可讀性等）

## 代理期望

為了讓 OpenCode 正確運作，代理應該：

- 在採取行動前始終檢查是否有技能適用
- 當技能適用時，使用 `skill` 工具來載入該技能
- 絕不跳過必要的流程（規格、規劃、測試等）
- 不直接跳到實施

這些規則是由您專案的 `AGENTS.md` 執行的，而不是由技能本身。

## 限制

- 技能呼叫取決於模型的相容性。
- OpenCode 不會自動安裝技能；請複製或安裝您需要的目錄。
- 如果一個技能引用 `references/` 下的文件，在手動安裝時您可能需要複製這些文件。

## 總結

1. 安裝您需要的技能，方法是使用 `npx skills add addyosmani/agent-skills`，或從此儲存庫的克隆中複製它們到 `.opencode/skills/`（專案）、`~/.config/opencode/skills/`（全域），或相容的路徑，例如 `.claude/skills/` / `.agents/skills/`。
2. 建立您自己的專案本地 `AGENTS.md`，包含上述規則和意圖映射。
3. OpenCode 發現技能，而您的 `AGENTS.md` 指引代理呼叫它們。
4. 可選地添加 `.opencode/commands/*.md` 以啟用明確的斜線指令。

這將可重複使用的資產（技能）與儲存庫特定的配置（`addyosmani/agent-skills` 的根 `AGENTS.md`）分開。
