# Agent Skills

**用於 AI 編碼代理的生產級工程技能。**

這些技能編碼了資深工程師在構建軟體時使用的工作流程、品質門檻和最佳實踐。它們被封裝起來，以便 AI 代理在開發的每個階段都能一致地遵循它們。

<a href="https://trendshift.io/repositories/25200" target="_blank"><img src="https://trendshift.io/api/badge/repositories/25200" alt="addyosmani%2Fagent-skills | Trendshift" style="width: 250px; height: 55px;" width="250" height="55"/></a

![Addy's Agent Skills](https://addyosmani.com/assets/images/addys-agent-skills.jpg)

```
  DEFINE          PLAN           BUILD          VERIFY         REVIEW          SHIP
 ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐
 │ Idea │ ───▶ │ Spec │ ───▶ │ Code │ ───▶ │ Test │ ───▶ │  QA  │ ───▶ │  Go  │
 │Refine│      │  PRD │      │ Impl │      │Debug │      │ Gate │      │ Live │
 └──────┘      └──────┘      └──────┘      └──────┘      └──────┘      └──────┘
  /spec          /plan          /build        /test         /review       /ship
```

---

## 命令

9 個映射到開發生命週期的斜線命令。每個命令都會自動激活正確的技能。

| 您正在做什麼 | 命令 | 關鍵原則 |
|-------------------|---------|---------------|
| 定義要建什麼 | `/spec` | 在程式碼之前定義規格 |
| 規劃如何建 | `/plan` | 小的、原子化的任務 |
| 增量建構 | `/build` | 一個區塊一次 |
| 證明它有效 | `/test` | 測試即是證明 |
| 設定品質標準 | `/constraints` | 一次決定，在各處強制執行 |
| 合併前審查 | `/review` | 改善程式碼健康度 |
| 審核網路效能 | `/webperf` | 在優化之前衡量 |
| 簡化程式碼 | `/code-simplify` | 清晰度勝過巧妙 |
| 發佈到生產 | `/ship` | 越快越安全 |

希望在規格存在後減少手動步驟？**`/build auto`** 會在一次批准的通過中生成規劃並實施所有任務——您只需一次批准規劃，然後它就能自主運行。它消除了任務之間的*中間*步驟，而不是驗證：每個任務仍然是測試驅動的並單獨提交，並且它會在失敗或有風險的步驟時暫停。

技能也會根據您正在做的事情自動激活——設計 API 會觸發 `api-and-interface-design`，建構 UI 會觸發 `frontend-ui-engineering`，依此類推。

---

## 快速開始

**最快的路徑——任何代理，一個命令。** 開放的 [skills CLI](https://github.com/vercel-labs/skills) 安裝到 70 多個代理中（Claude Code、Cursor、Codex、Copilot、Cline 等）：

```bash
npx skills add addyosmani/agent-skills            # 安裝所有 25 個技能
npx skills add addyosmani/agent-skills --list     # 安裝前瀏覽
```

或者獲取單個技能：

```bash
npx skills add addyosmani/agent-skills --skill code-review-and-quality   # 合併前的五軸審查
npx skills add addyosmani/agent-skills --skill interview-me              # 需求質詢，一次一個問題
npx skills add addyosmani/agent-skills --skill test-driven-development   # 測試驅動開發，強制執行
```

> **安裝一個技能？** 每個技能的 `npx` 安裝只複製
> `skills/<name>/`，而不是儲存庫級別的 `references/` 目錄。該技能仍然有效，但無法存取補充共享檢查清單的路徑。請使用整個儲存庫整合，複製儲存庫到一個安裝的技能中的 `references/` 目錄。這種可攜性差距在 [#361](https://github.com/addyosmani/agent-skills/issues/361) 中被追蹤。

偏好原生整合？請選擇以下工具。

<details>
<summary><b>Claude Code (推薦)</b></summary>

**市場安裝：**

```
/plugin marketplace add addyosmani/agent-skills
/plugin install agent-skills@addy-agent-skills
```

> **SSH 錯誤？** 市場會通過 SSH 克隆儲存庫。如果您在 GitHub 上沒有設置 SSH 金鑰，請[添加您的 SSH 金鑰](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)或使用完整的 HTTPS URL 來強制在市場添加步驟中進行 HTTPS 克隆：
> ```bash
> /plugin marketplace add https://github.com/addyosmani/agent-skills.git
> /plugin install agent-skills@addy-agent-skills
> ```
>
> 如果 `/plugin install` 在 Windows 或 macOS 上仍然以 `git@github.com: Permission denied (publickey)` 失敗，推薦的解決方案是配置 Git 一次，將 GitHub SSH URL 重寫為 HTTPS 以便子程序克隆：
> ```bash
> git config --global url."https://github.com/".insteadOf git@github.com:
> ```

**本地/開發：**

```bash
git clone https://github.com/addyosmani/agent-skills.git
claude --plugin-dir /path/to/agent-skills
```

</details>

<details>
<summary><b>Cursor</b></summary>

將工作流程技能放在 `.cursor/skills/` 下（同步自 `agent-skills/skills/`）並在 `.cursor/rules/*.mdc` 中設置簡短的政策——不要將完整的技能貼入規則中。請參閱 [docs/cursor-setup.md](docs/cursor-setup.md)。

</details>

<details>
<summary><b>Antigravity CLI</b></summary>

作為技能和子代理的原生插件安裝。在受影響的 Antigravity CLI 版本中，報告稱舊的 TOML 命令被轉換，但它們的包裝命令無法被發現；直接調用底層命名空間技能。請參閱 [docs/antigravity-setup.md](docs/antigravity-setup.md#lifecycle-workflows-and-command-compatibility)。

**從儲存庫安裝：**

```bash
agy plugin install https://github.com/addyosmani/agent-skills.git
```

**從本地克隆安裝：**

```bash
git clone https://github.com/addyosmani/agent-skills.git
agy plugin install ./agent-skills
```

</details>

<details>
<summary><b>Gemini CLI</b></summary>

作為原生技能安裝以實現自動發現，或添加到 `GEMINI.md` 以保持持久上下文。請參閱 [docs/gemini-cli-setup.md](docs/gemini-cli-setup.md)。

**從儲存庫安裝：**

```bash
gemini skills install https://github.com/addyosmani/agent-skills.git --path skills
```

**從本地克隆安裝：**

```bash
gemini skills install ./agent-skills/skills/
```

</details>

<details>
<summary><b>Windsurf</b></summary>

將技能內容添加到您的 Windsurf 規則配置中。請參閱 [docs/windsurf-setup.md](docs/windsurf-setup.md)。

</details>

<details>
<summary><b>OpenCode</b></summary>

將技能複製到 `.opencode/skills/`（或 `~/.config/opencode/skills/`），添加一個專案級別的 `AGENTS.md`，並使用內建的 `skill` 工具進行代理驅動的執行。可以添加到 `.opencode/commands/` 下的斜線命令。

請參閱 [docs/opencode-setup.md](docs/opencode-setup.md)。

</details>

<details>
<summary><b>GitHub Copilot</b></summary>

使用 `agents/` 中的代理定義作為 Copilot 人格，並在 `.github/copilot-instructions.md` 中設置技能內容。請參閱 [docs/copilot-setup.md](docs/copilot-setup.md)。

使用獨立的 `copilot` CLI？將其作為插件安裝——請參閱 [docs/copilot-cli-setup.md](docs/copilot-cli-setup.md)。

</details>

<details>
  <summary><b>Kiro IDE & CLI </b></summary>
  Kiro 的技能位於 `.kiro/skills/` 下，並可以存儲在專案或全局級別。Kiro 也支持 Agents.md。請參閱 Kiro 文件 https://kiro.dev/docs/skills/
</details>

<details>
<summary><b>Codex</b></summary>

作為原生 Codex 插件安裝（Codex CLI v0.122+）：

```bash
codex plugin marketplace add addyosmani/agent-skills
codex plugin add agent-skills@agent-skills
```

第一個命令註冊了市場；第二個命令安裝了插件。Codex 直接通過 `.codex-plugin/plugin.json` 讀取根目錄的 `skills/` 目錄。安裝後，使用 `@` 在聊天中調用技能（例如，`@spec-driven-development`）。請參閱 [docs/codex-setup.md](docs/codex-setup.md) 以進行本地安裝和故障排除。

</details>

<details>
<summary><b>Command Code</b></summary>

使用內建的 `cmd skills` 命令原生安裝。Command Code 克隆儲存庫，發現每個 `SKILL.md`，並將其安裝到 `.commandcode/skills/`：

```bash
cmd skills add addyosmani/agent-skills            # 選擇要安裝的技能（專案）
cmd skills add addyosmani/agent-skills --global   # 為所有專案安裝（~/.commandcode/skills/）
cmd skills add addyosmani/agent-skills -s spec-driven-development  # 安裝特定的技能
```

已安裝的技能會顯示在 TUI 斜線選單中，例如 `/spec-driven-development`。請參閱 [docs/commandcode-setup.md](docs/commandcode-setup.md)。

</details>

<details>
<summary><b>其他代理</b></summary>

技能是純 Markdown——它們可以與任何接受系統提示或指令文件的代理一起工作。請參閱 [docs/getting-started.md](docs/getting-started.md)。安裝了該套件但未列在上述列表中的主機位於 [docs/other-hosts.md](docs/other-hosts.md)。

</details>

---

## 採用

已經安裝？如何推出該套件取決於您的程式碼庫。**[採用指南](docs/adoption-guide.md)** 涵蓋了兩種路徑：對於一個全新的專案，從第一天開始的完整生命週期，或對於一個已建立的程式碼庫，增量式、以驗證為先的推出。

---

## 所有 25 個技能

上述命令是入口點。該套件總共包含 25 個技能——24 個生命週期技能加上 `using-agent-skills` 元技能。每個技能都是一個具有步驟、驗證門檻和反合理化表的結構化工作流程。您也可以直接引用任何技能。

### 元技能 - 發現適用的技能

| 技能 | 它做什麼 | 何時使用 |
|-------|-------------|----------|
| [using-agent-skills](skills/using-agent-skills/SKILL.md) | 將輸入的工作映射到正確的技能工作流程並定義共享的操作規則 | 開始會話或決定適用哪個技能 |

### 定義 - 明確要建什麼

| 技能 | 它做什麼 | 何時使用 |
|-------|-------------|----------|
| [interview-me](skills/interview-me/SKILL.md) | 一個時間為一問的訪談，提取用戶實際想要的東西而不是他們認為他們應該想要的東西，直到約 95% 的信心 | 要求不夠明確，或用戶調用「interview me」/「grill me」 |
| [idea-refine](skills/idea-refine/SKILL.md) | 結構化的發散/收斂思考，將模糊的想法轉變為具體的提案 | 您有一個需要探索的粗略概念 |
| [spec-driven-development](skills/spec-driven-development/SKILL.md) | 在任何程式碼之前撰寫涵蓋目標、命令、結構、程式碼風格、測試和邊界的 PRD | 開始一個新專案、功能或重大更改 |
| [constraint-driven-development](skills/constraint-driven-development/SKILL.md) | 為品質標準進行訪談，使用合理的預設閾值，撰寫 CONSTRAINTS.md，按成本放置每個檢查，並捕捉代理為了獲得綠色而壓制檢查或跳過測試 | 沒有寫下標準，或代理產生的內容比任何人閱讀的都多 |

### 規劃 - 分解

| 技能 | 它做什麼 | 何時使用 |
|-------|-------------|----------|
| [planning-and-task-breakdown](skills/planning-and-task-breakdown/SKILL.md) | 將規格分解為具有可接受標準和依賴順序的、小的、可驗證的任務 | 您有一個規格需要實施的單元 |

### 建構 - 編寫程式碼

| 技能 | 它做什麼 | 何時使用 |
|-------|-------------|----------|
| [incremental-implementation](skills/incremental-implementation/SKILL.md) | 細長的垂直切片——實施、測試、驗證、提交。功能標誌、安全預設值、可回滾的更改 | 任何涉及多於一個檔案的更改 |
| [test-driven-development](skills/test-driven-development/SKILL.md) | 紅綠重構、測試金字塔（80/15/5）、測試大小、DAMP 優於 DRY、Beyonce 法則、瀏覽器測試 | 實施邏輯、修復錯誤或更改行為 |
| [context-engineering](skills/context-engineering/SKILL.md) | 在正確的時間提供正確的資訊給代理——規則檔案、上下文打包、MCP 整合 | 開始會話、切換任務或當輸出品質下降時 |
| [source-driven-development](skills/source-driven-development/SKILL.md) | 將每個框架決策都建立在官方文檔上——驗證、引用來源、標記未驗證的部分 | 您想要任何框架或函式庫的權威、有來源引用的程式碼 |
| [doubt-driven-development](skills/doubt-driven-development/SKILL.md) | 對進行中的每個非平凡決策進行對抗性新上下文審查——主張 → 提取 → 質疑 → 調和 → 停止，可選的用戶授權跨模型升級 | 風險很高（生產、安全、不可逆）、在不熟悉代碼中工作，或現在驗證一個確定的輸出比稍後除錯更便宜 |
| [frontend-ui-engineering](skills/frontend-ui-engineering/SKILL.md) | 組件架構、設計系統、狀態管理、響應式設計、WCAG 2.1 AA 可訪問性 | 建構或修改用戶界面 |
| [api-and-interface-design](skills/api-and-interface-design/SKILL.md) | 合約優先設計、Hyrum 法則、單一版本規則、錯誤語義、邊界驗證 | 設計 API、模組邊界或公共介面 |

### 驗證 - 證明它有效

| 技能 | 它做什麼 | 何時使用 |
|-------|-------------|----------|
| [browser-testing-with-devtools](skills/browser-testing-with-devtools/SKILL.md) | Chrome DevTools MCP 用於即時運行數據——DOM 檢查、控制台日誌、網路追蹤、效能分析 | 建構或除錯在瀏覽器中運行的任何東西 |
| [debugging-and-error-recovery](skills/debugging-and-error-recovery/SKILL.md) | 五步分類：重現、定位、減少、修復、保護。一行停止規則、安全備用方案 | 測試失敗、建構出錯或行為意外 |

### 審查 - 合併前的品質門檻

| 技能 | 它做什麼 | 何時使用 |
|-------|-------------|----------|
| [code-review-and-quality](skills/code-review-and-quality/SKILL.md) | 五軸審查、更改大小（約 100 行）、嚴重性標籤（Nit/Optional/FYI）、審查速度規範、分割策略 | 在合併任何更改之前 |
| [code-simplification](skills/code-simplification/SKILL.md) | Chesterton 的圍欄、500 法則、在保持確切行為的同時減少複雜性 | 程式碼可以運行，但比應該的更難閱讀或維護 |
| [security-and-hardening](skills/security-and-hardening/SKILL.md) | OWASP Top 10 防範、認證模式、密碼管理、依賴項審計、三層邊界系統 | 處理用戶輸入、認證、數據存儲或外部整合 |
| [performance-optimization](skills/performance-optimization/SKILL.md) | 衡量優先方法——核心 Web 指標目標、分析工作流程、捆綁包分析、反模式檢測 | 存在效能要求或您懷疑有退步 |

### 發佈 - 帶著信心部署

| 技能 | 它做什麼 | 何時使用 |
|-------|-------------|----------|
| [git-workflow-and-versioning](skills/git-workflow-and-versioning/SKILL.md) | 分支基礎開發、原子提交、更改大小（約 100 行）、提交即保存點模式 | 對任何程式碼更改（總是） |
| [ci-cd-and-automation](skills/ci-cd-and-automation/SKILL.md) | 左移、越快越安全、功能標誌、品質門檻管道、失敗反饋迴路 | 設定或修改建構和部署管道 |
| [deprecation-and-migration](skills/deprecation-and-migration/SKILL.md) | 將程式碼視為負擔的心態、強制性與建議性的棄用、遷移模式、移除殭屍程式碼 | 移除舊系統、遷移用戶或淘汰功能 |
| [documentation-and-adrs](skills/documentation-and-adrs/SKILL.md) | 架構決策記錄、API 文件、內聯文檔標準——記錄*為什麼* | 做出架構決策、更改 API 或發佈功能 |
| [observability-and-instrumentation](skills/observability-and-instrumentation/SKILL.md) | 結構化日誌、RED 指標、OpenTelemetry 追蹤、症狀式警報——在建構時進行儀表化 | 添加遙測，或發佈任何在生產環境中運行之物 |
| [shipping-and-launch](skills/shipping-and-launch/SKILL.md) | 發佈前檢查清單、功能標誌生命週期、分階段推出、回滾程序、監控設置 | 準備部署到生產環境 |

---

## 代理人角色

預配置的專家角色用於目標審查：

| 代理人 | 角色 | 視角 |
|-------|------|-------------|
| [code-reviewer](agents/code-reviewer.md) | 資深技術工程師 | 具有「資深工程師會批准這個嗎？」標準的五軸程式碼審查 |
| [test-engineer](agents/test-engineer.md) | QA 專家 | 測試策略、覆蓋率分析和證明模式 |
| [security-auditor](agents/security-auditor.md) | 安全工程師 | 漏洞檢測、威脅建模、OWASP 評估 |
| [web-performance-auditor](agents/web-performance-auditor.md) | 網頁效能工程師 | 核心 Web 指標審查，具有快速/深度模式和度量誠實規則；通過 `/webperf` 運行它 |

請參閱 [docs/agents.md](docs/agents.md) 以查看決策矩陣、調度規則以及角色如何與技能和斜線命令組合。

---

## 參考檢查清單

技能在需要時會拉取的快速參考材料：

| 參考資料 | 涵蓋內容 |
|-----------|--------|
| [definition-of-done.md](references/definition-of-done.md) | 每個更改都通過的專案級標準，與每次任務的接受標準相比 |
| [testing-patterns.md](references/testing-patterns.md) | 測試結構、命名、模擬、React/API/E2E 範例、反模式（JavaScript/TypeScript） |
| [security-checklist.md](references/security-checklist.md) | 提交前檢查、認證、輸入驗證、標頭、CORS、OWASP Top 10 |
| [performance-checklist.md](references/performance-checklist.md) | 核心 Web 指標目標、前端/後端檢查清單、度量命令 |
| [accessibility-checklist.md](references/accessibility-checklist.md) | 鍵盤導航、螢幕閱讀器、視覺設計、ARIA、測試工具 |
| [observability-checklist.md](references/observability-checklist.md) | 值班問題、結構化日誌、RED/USE 指標、追蹤、症狀式警報、發佈門檻 |
| [orchestration-patterns.md](references/orchestration-patterns.md) | 經認可的多角色調度模式、反模式和「角色不調用角色」規則 |

---

## 技能如何運作

每個技能都遵循一致的結構：

```
┌─────────────────────────────────────────────────┐
│  SKILL.md                                       │
│                                                 │
│  ┌─ Frontmatter ─────────────────────────────┐  │
│  │ name: lowercase-hyphen-name               │  │
│  │ description: 指引代理通過 [任務]。│  │
│  │              使用時…                    │  │
│  └───────────────────────────────────────────┘  │                                                                                                
│  Overview         → 此技能做什麼        │
│  When to Use      → 觸發條件       │
│  Process          → 逐步工作流程       │
│  Rationalizations → 藉口 + 反駁         │
│  Red Flags        → 某事有問題的跡象     │
│  Verification     → 證據要求       │
└─────────────────────────────────────────────────┘
```

**關鍵設計選擇：**

- **流程，而非散文。** 技能是代理遵循的工作流程，而不是它們閱讀的參考文檔。每個技能都有步驟、檢查點和退出標準。
- **反合理化。** 每個技能都包含一個代理用來跳過步驟的常見藉口表（例如，「我稍後會添加測試」）以及記錄的反駁。
- **驗證是不可協商的。** 每個技能都以證據要求結束——通過的測試、建構輸出、運行時數據。一句「看起來是對的」永遠是不夠的。
- **漸進式披露。** `SKILL.md` 是入口點。支持的參考資料僅在需要時加載，保持 Token 使用量最小。

---

## 專案結構

可攜性的核心保留在共享目錄中。特定主機的路徑是原生的發現慣例，而不是品牌別名；更改或合併它們會破壞掃描這些確切位置的工具。

| 層級/消費者 | 儲存庫路徑 | 用途 |
|---|---|---|
| 共享工作流程核心 | `skills/` (25 個技能) | 每個整合使用的可攜式 `SKILL.md` 工作流程 |
| 共享審查材料 | `agents/` (4 個角色)、`references/` (7 個檢查清單) | 專門審查員和整個儲存庫安裝帶來的套件級別檢查清單 |
| Claude Code 适配器 | `.claude/commands/` (9 個命令)、`.claude-plugin/`、`hooks/` | 斜線命令包裝、市場元數據和生命週期掛鉤 |
| Gemini CLI 适配器 | `.gemini/commands/` (9 個命令) | Gemini 原生的 TOML 命令包裝 |
| Antigravity CLI 适配器 | `commands/` (9 個命令)、`plugin.json` | 遺留的 TOML 包裝和根插件標記；參見 [docs/antigravity-setup.md#lifecycle-workflows-and-command-compatibility](docs/antigravity-setup.md#lifecycle-workflows-and-command-compatibility) |
| Codex 适配器 | `.codex-plugin/`、`.agents/plugins/` | Codex 插件元數據和市場註冊；Codex 直接消費 `skills/` |
| GitHub Copilot CLI 适配器 | `plugin.json` | 根插件元數據；Copilot CLI 按照慣例發現 `skills/` 並不註冊生命週期包裝 |
| 貢獻工具 | `scripts/` (13 個腳本)、`evals/` (25 個案例檔案)、`.github/workflows/` | 驗證、路由評估和 CI |
| 文件 | `docs/` | 通用的指導和每工具的設置指南 |

沒有提交的适配器目錄的工具將共享的 `skills/` 核心安裝或複製到它們自己的原生位置。 [快速開始](#quick-start) 連結到每個受支持主機的設置指南。

---

## 為什麼需要代理技能？

AI 編碼代理預設選擇最短路徑——這通常意味著跳過規格、測試、安全審查以及使軟體可靠的實踐。代理技能為代理提供結構化的工作流程，以強制執行資深工程師在生產程式碼中帶來的相同紀律。

每個技能都編碼了艱難獲得的工程判斷：*何時*寫規格、*要*測試、*如何*審查以及*何時*發佈。這些不是通用的提示——它們是區分生產級品質工作和原型級品質工作的、有明確觀點的、流程驅動的工作流程。

技能將 Google 工程文化中的最佳實踐內化——包括來自 [Google 的軟體工程](https://abseil.io/resources/swe-book) 和 Google 的 [工程實踐指南](https://google.github.io/eng-practices/) 中的概念。您會在 API 設計中找到 Hyrum 的法則，在測試中找到 Beyoncé 法則和測試金字塔，在程式碼審查中找到更改大小和審查速度規範，在簡化中找到 Chesterton 的圍欄，在 Git 工作流程中找到分支基礎開發，在 CI/CD 中找到左移和功能標誌，以及一個專門的棄用技能將程式碼視為負擔。這些不是抽象的原則——它們直接嵌入到代理遵循的逐步工作流程中。

---

## 比較

想知道這與 [Superpowers](https://github.com/obra/superpowers) 或 [Matt Pocock 的技能](https://github.com/mattpocock/skills) 相比如何？請參閱 **[docs/comparison.md](docs/comparison.md)** 以獲得關於這三者如何不同以及在何時應選擇它們的誠實、並排的比較——包括一個指向受控的 [頭對頭實驗](https://www.linkedin.com/pulse/superpowers-vs-agent-skills-faster-shipping-safer-reasoning-om-mishra-dzakf/) 的連結。

---

## 貢獻

技能應該是**具體**的（可操作的步驟，而不是模糊的建議）、**可驗證**的（具有證據要求的明確退出標準）、**經過實戰檢驗**的（基於真實的工作流程）和**最少**的（僅包含引導代理所需的部分）。

請參閱 [docs/skill-anatomy.md](docs/skill-anatomy.md) 以了解格式規範，[CONTRIBUTING.md](CONTRIBUTING.md) 以了解指南，以及 [docs/advanced-per-agent-configuration.md](docs/advanced-per-agent-configuration.md) 以應用模型路由、工具限制和其他每代理運行時控制，而不會破壞可攜的 `SKILL.md` 前置信息。

---

## 團隊

agent-skills 由以下人員構建和維護：

| | 名稱 | GitHub | 角色 |
|---|------|--------|------|
| <img src="https://github.com/addyosmani.png?size=120" width="60" height="60" alt="Addy Osmani"> | **Addy Osmani** | [@addyosmani](https://github.com/addyosmani) | 創作者 |
| <img src="https://github.com/federicobartoli.png?size=120" width="60" height="60" alt="Federico Bartoli"> | **Federico Bartoli** | [@federicobartoli](https://github.com/federicobartoli) | 合作者 |
| <img src="https://github.com/nucliweb.png?size=120" width="60" height="60" alt="Joan León"> | **Joan León** | [@nucliweb](https://github.com/nucliweb) | 合作者 |

---

## 許可證

MIT - 在您的專案、團隊和工具中使用這些技能。
