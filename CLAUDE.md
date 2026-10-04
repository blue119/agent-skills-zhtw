# agent-skills

這是代理技能專案——為 AI 編碼代理收集的生產級工程技能。

> **範圍：** 此檔案配置的是在 [`addyosmani/agent-skills`](https://github.com/addyosmani/agent-skills) 儲存庫上工作的代理，而不是其他專案。不要將其複製到另一個專案或全局代理配置中；可重用的資產在 `skills/` 中。

## 專案結構

```
skills/       → 核心技能 (每個目錄下有 SKILL.md)
agents/       → 可重用的代理角色 (code-reviewer, test-engineer, security-auditor, web-performance-auditor)
hooks/        → 會話生命週期鉤子 (Session lifecycle hooks)
.claude/commands/ → 斜線指令 (/spec, /plan, /build, /test, /review, /code-simplify, /ship; 以及 /webperf specialist audit)
references/   → 補充檢查清單 (測試、效能、安全、無障礙性、可觀察性)
evals/        → 技能評估案例 + 框架 (請參閱 evals/README.md)
docs/         → 不同工具的設定指南
```

## 依階段劃分的技能

**定義 (Define):** interview-me, idea-refine, spec-driven-development
**規劃 (Plan):** planning-and-task-breakdown
**建構 (Build):** incremental-implementation, test-driven-development, context-engineering, source-driven-development, doubt-driven-development, frontend-ui-engineering, api-and-interface-design
**驗證 (Verify):** browser-testing-with-devtools, debugging-and-error-recovery
**審查 (Review):** code-review-and-quality, code-simplification, security-and-hardening, performance-optimization
**發布 (Ship):** git-workflow-and-versioning, ci-cd-and-automation, deprecation-and-migration, documentation-and-adrs, observability-and-instrumentation, shipping-and-launch

## 約定規範

- 每個技能都位於 `skills/<name>/SKILL.md`
- YAML 前置內容包含 `name` 和 `description` 欄位
- 描述以技能的功能（第三人稱）開始，接著是觸發條件（"使用時..."）
- 每個技能都包含：概述 (Overview)、使用時機 (When to Use)、流程 (Process)、常見理由 (Common Rationalizations)、警示標誌 (Red Flags)、驗證 (Verification)
- 共享的參考資訊在根目錄的 `references/` 目錄中；對於自包含、可分發的技能，演進的規範是將技能自身的參考資訊放在 `skills/<name>/references/` 中。
- 只有當內容超過 100 行時才創建支援檔案。

## 貢獻

在添加新技能或大幅修改現有技能之前，請在 [CONTRIBUTING.md](CONTRIBUTING.md#before-proposing-a-new-skill) 中執行預檢檢查：搜尋目錄、檢查已開啟的 PR、確認想法符合 [docs/skill-anatomy.md](docs/skill-anatomy.md)，並證明差距。優先擴展現有技能而非添加幾乎重複的內容。CONTRIBUTING.md 是此工作流程的唯一真實來源；請勿在此處或其他地方重複其檢查清單，請連結到它。

## 指令

- `npm test` — 不適用（這是一個文件專案）
- 驗證：檢查所有 `SKILL.md` 檔案是否具有包含 `name` 和 `description` 的有效 YAML 前置內容
- 評估 (Evals)：`node scripts/run-evals.js` — 為每個技能觸發/路由評估 (CI)；使用 `--behavioral <skill>` 進行分級運行。

## 拉取請求 (Pull Requests)

PRs 針對上游儲存庫的預設分支。在典型的分叉設置中，上游遠端是 `upstream`，您的分叉是 `origin`，但確切的遠端名稱在此處並不重要。

- 在開啟 PR 之前，請搜尋上游儲存庫中已開啟的 PR 和問題，以查找涉及相同檔案或規則的工作。如果存在重疊，請協調（基於它進行構建、使您的規則與之一致，或在它合併後進行重新基礎），而不是開啟衝突的 PR。
- 優先使用小而集中的 PR，而不是對廣泛共享的檔案（例如 `scripts/` 下的檔案）進行大規模重構，因為後者更有可能與進行中的工作產生衝突。

## 界限

- 永遠：在創建新技能目錄之前執行 `CONTRIBUTING.md` 的預檢檢查
- 永遠：遵循 `skill-anatomy.md` 的格式來建立新技能
- 永遠：在開啟新 PR 之前，檢查上游儲存庫中已開啟的 PR 和問題是否存在重疊
- 絕不：添加模糊的建議而不是可執行的流程作為技能
- 絕不：在技能之間重複內容——請參考其他技能
