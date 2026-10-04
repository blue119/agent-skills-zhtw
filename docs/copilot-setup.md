# 使用代理技能與 GitHub Copilot

本指南涵蓋了 VS Code 中的 Copilot。有關獨立的 `copilot` 命令列工具，請參閱 [copilot-cli-setup.md](copilot-cli-setup.md)。

**安裝實際提供給你的內容：** 技能。每個安裝的技能都會成為一個名為其前置標記 `name` 的斜線命令 — `/spec-driven-development`、`/test-driven-development` 等。執行 `npx skills add addyosmani/agent-skills` 和下方的手動複製，都只安裝了技能。這兩種方式都無法複製此儲存庫的簡短生命週期包裝器（`/spec`、`/plan`、`/build`、`/test`、`/review`、`/ship`）——這些是存在於 `.claude/commands/` 中的 Claude Code 命令。請使用完整的技能名稱，或添加自己的別名 — 請參閱 [Lifecycle workflows](#lifecycle-workflows)。

## 設定

### Copilot 指示

Copilot 支持在您的儲存庫中的 `.github/skills`、`.claude/skills` 或 `.agents/skills` 目錄來建立代理技能。

```bash
mkdir -p .github/skills/test-driven-development .github/skills/code-review-and-quality

# 建立基本技能檔案
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .github/skills/test-driven-development/SKILL.md
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md > .github/skills/code-review-and-quality/SKILL.md
```

無論您使用哪種方式，結果都是在其中一個三種專案路徑下的某個技能目錄中，每個技能都有一個 `SKILL.md`。使用帶有 global/user 標誌的安裝程式會將內容寫到其他地方——請查看其輸出以了解它使用的路徑。在 Copilot Chat 中運行 `/skills` 以打開 **設定技能** 選單並確認已發現的內容。

如需更多詳細資訊，請參閱 [為 GitHub Copilot 建立代理技能](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-skills) 和 VS Code 中關於 [代理技能](https://code.visualstudio.com/docs/agent-customization/agent-skills) 的指南。

### 代理人角色 (*.agent.md)

Copilot 支持專業化的代理人角色。使用 agent-skills 代理人：

> **重要提示：** GitHub Copilot 要求自定義代理檔案命名為 `*.agent.md`。
> 名為 `*.md` 的檔案會被 Copilot 安靜地忽略。
> 詳見 [VS Code 自定義代理人文件結構](https://code.visualstudio.com/docs/copilot/customization/custom-agents#_custom-agent-file-structure) 以獲取詳細資訊。

```bash
# 建立代理人目錄並複製代理人定義
mkdir -p .github/agents
cp /path/to/agent-skills/agents/code-reviewer.md .github/agents/code-reviewer.agent.md
cp /path/to/agent-skills/agents/test-engineer.md .github/agents/test-engineer.agent.md
cp /path/to/agent-skills/agents/security-auditor.md .github/agents/security-auditor.agent.md
```

在 Copilot Chat 中呼叫代理人：
- `@code-reviewer Review this PR`
- `@test-engineer Analyze test coverage for this module`
- `@security-auditor Check this endpoint for vulnerabilities`

### 自定義指令（使用者層級）

對於您希望在所有儲存庫中使用的技能：

1. 打開 VS Code → 設定 → GitHub Copilot → 自定義指令
2. 添加您最常用的技能摘要

## 推薦的設定

### .github/copilot-instructions.md

GitHub Copilot 透過 `.github/copilot-instructions.md` 支持專案級別的指令。

```markdown
# 專案編碼標準

## 測試
- 在程式碼之前撰寫測試（TDD）
- 對於錯誤：首先寫一個失敗的測試，然後修復它（Prove-It 模式）
- 測試層級：單元 > 整合 > 端到端（使用能捕捉行為的最低層級）
- 每次變更後運行 `npm test`

## 代碼品質
- 跨五個維度進行審查：正確性、可讀性、架構、安全性、效能
- 每個 PR 必須通過：程式碼檢查、類型檢查、測試、建置
- 程式碼或版本控制中不得包含密碼

## 實施
- 以小而可驗證的增量進行建置
- 每個增量：實施 → 測試 → 驗證 → 提交
- 絕不將格式變更與行為變更混合

## 邊界
- 始終：在提交之前運行測試，驗證使用者輸入
- 先提問：資料庫綱要變更、新依賴項
- 絕不：提交密碼、移除失敗的測試、跳過驗證
```

### 專業代理人

在 Copilot Chat 中使用代理人進行有針對性的審查工作流程。

## 生命週期工作流程

技能預設是可由使用者呼叫的，一旦被發現，整個生命週期都以技能自己的名稱提供——無需額外配置：

| 工作流程 | Copilot 呼叫 | 備註 |
|----------|--------------------|-------|
| 定義 | `/spec-driven-development` | 在程式碼前撰寫結構化規格 |
| 規劃 | `/planning-and-task-breakdown` | 產生 `tasks/plan.md` 和 `tasks/todo.md` |
| 建置 | `/incremental-implementation` | 與 `/test-driven-development` 配對；一次一塊 |
| 驗證 | `/test-driven-development` | 紅綠重構、錯誤的 Prove-It |
| 審查 | `/code-review-and-quality` | 五維審查 |
| 發布 | `/shipping-and-launch` | 發布就緒度 |

輸入 `/` 以瀏覽此工作區實際載入的內容。

自然語言是備選方案，無論斜線命令是否顯示，它都能在任何 Copilot 介面中工作：

> 使用 `spec-driven-development` 技能為 [該功能] 撰寫規格。

> 使用 `code-review-and-quality` 技能來審查我的暫存變更。

### 可選：簡短的 `/spec` 風格別名

VS Code 的擴充功能主機本地代理人支持位於 `.github/prompts/<name>.prompt.md` 的 [提示檔案](https://code.visualstudio.com/docs/agent-customization/prompt-files)，每個檔案都成為一個 `/<name>` 斜線命令。代理人主機**不**使用提示檔案——請使用上述的技能名稱。

別名只是指向一個技能的簡短捷徑，僅此而已。這些別名不實施此儲存庫中 Claude Code 命令的協調工作——例如 `/build auto` 的單一批准的規劃和實施過程以及 `/ship` 的平行角色擴散。在 Copilot 上重現這些內容是它自己的工作，而不是單一檔案別名所涵蓋的內容。請優先使用技能名稱作為預設；只有當簡短形式值得額外的檔案時，才尋求別名。

建立一個，不要覆蓋現有檔案：

```bash
mkdir -p .github/prompts
[ -e .github/prompts/spec.prompt.md ] || cat > .github/prompts/spec.prompt.md <<'EOF'
---
description: 在編寫程式碼前撰寫結構化規格
---

使用 spec-driven-development 技能。

就目標和目標使用者、核心功能和接受標準、堆疊偏好和約束以及已知邊界提出澄清問題。然後撰寫涵蓋目標、命令、專案結構、程式碼風格、測試策略和邊界的規格。將其儲存為專案根目錄中的 SPEC.md，並在任何程式碼寫入之前與我確認。
EOF
```

重新載入視窗，然後輸入 `/spec`。

對於其餘部分，運行相同的 `mkdir`/`cat` 區塊，但交換檔名（檔名*就是*命令名稱），並從此表中獲取 `description` 和內文。**請替換 `---` 後面的整個內文**，而不僅僅是技能名稱——規格內文的 SPEC.md 和澄清問題指令只屬於定義，將它們留下會導致 `/plan` 或 `/test` 寫入規格。

| 別名檔案 | `description` | 完整內文 — 前置標記之後的所有內容 |
|------------|---------------|--------------------------------------------------|
| `.github/prompts/plan.prompt.md` | 將已批准的規格分解為有序、可驗證的任務 | 使用 planning-and-task-breakdown 技能。閱讀規格，然後將工作分解為小而獨立可驗證的任務，每個任務都有接受標準和明確的依賴順序。將結果儲存為 `tasks/plan.md` 和 `tasks/todo.md`。不要寫產品程式碼——給我計畫並等待我的批准。 |
| `.github/prompts/build.prompt.md` | 實施下一個規劃的任務，優先測試 | 使用 incremental-implementation 和 test-driven-development 技能。閱讀 `tasks/plan.md` 和 `tasks/todo.md`，然後只取下一個未檢查的任務。首先寫一個失敗的測試，使其通過，重構，運行套件，並完成該任務。在此停止並報告變更。 |
| `.github/prompts/test.prompt.md` | 在滿足其要求的程式碼之前撰寫測試 | 使用 test-driven-development 技能。對於新行為，在任何實施之前寫一個捕獲它的失敗測試。對於錯誤，首先用一個失敗的測試重現它，然後修復它。在每一步後運行套件並向我展示紅色的和綠色的輸出。 |

請自行撰寫內文，或從此表中獲取，而不是逐字複製 `.claude/commands/*.md`：這些檔案將技能引用為 `agent-skills:<name>`，這是一個 Claude Code 外掛命名空間，對 Copilot 來說沒有任何意義。

### 如果技能斜線命令不出現

請依序處理這些問題——`/spec-driven-development` 遺失和 `/spec` 遺失有不同的原因：

1. **檢查技能的位置。** 每個技能都需要自己的目錄，其中包含一個 `SKILL.md`。本指南使用的專案級別路徑是 `.github/skills/`、`.claude/skills/` 和 `.agents/skills/`——但使用 global/user 標誌運行的安裝程式會將內容寫到工作區之外（`~/.agents/skills/` 等），因此一個不在這三者中的技能可能只是個人安裝的。請根據 [VS Code 代理人技能文件](https://code.visualstudio.com/docs/agent-customization/agent-skills) 確認實際路徑，然後如果需要專案範圍，請重新安裝到專案中。
2. **檢查前置標記。** `name` 必須存在且有效——它*就是*斜線命令。如果一個技能的前置標記選擇退出使用者呼叫，它就不會出現。
3. **檢查設定技能選單。** 運行 `/skills` 並在此處確認技能是否已啟用。
4. **開始一個新的會話。** 新添加的技能不一定會在對話期間被拾取到。
5. **檢查主機。** 如果 `/spec-driven-development` 可以工作但您的 `/spec` 別名不行，您很可能在 Agent Host 上，它不讀取提示檔案。
6. **檢查版本。** VS Code（**說明 → 關於**）和 Copilot Chat 擴充功能；技能作為斜線命令是最近的，因此如果其中任何一個落後，請更新。

## 使用提示

1. **保持指令簡潔**——Copilot 指示在專注時效果最佳。總結關鍵規則，而不是包含完整的技能檔案。
2. **使用代理人進行審查**——程式碼審查員、測試工程師和安全審計員代理人是為 Copilot 的代理人模型而設計的。
3. **在聊天中引用**——在處理特定階段時，將相關的技能內容貼到 Copilot Chat 中以獲得上下文。
4. **與 PR 審查結合**——設定 Copilot 使用程式碼審查員代理人角色來審查 PR。
