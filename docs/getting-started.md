# 使用 agent-skills 入門

agent-skills 適用於任何接受 Markdown 指令的 AI 編碼代理。本指南涵蓋了通用方法。有關工具特定的設定，請參閱專門的指南。

在設定您自己的專案之前，需要一個實例嗎？
[互動式教學](https://skills.addy.ie/tutorials/) 介紹了綠地建置、棕地功能以及一個安全的自動化迴圈，並提供可複製的提示，適用於 Claude Code、Codex 或任何其他代理。

## 技能如何運作

每個技能都是一個 Markdown 檔案（`SKILL.md`），描述一個特定的工程工作流程。當其載入到代理的上下文時，代理會遵循該工作流程——包括驗證步驟、應避免的反模式和退出標準。

**技能不是參考文件。** 它們是代理遵循的循序漸進的過程。

## 快速入門（任何代理）

### 1. 克隆儲存庫

```bash
git clone https://github.com/addyosmani/agent-skills.git
```

### 2. 選擇一個技能

瀏覽 `skills/` 目錄。每個子目錄都包含一個 `SKILL.md`，其中包含：
- **何時使用** — 表示此技能適用的觸發條件
- **流程** — 循序漸進的工作流程
- **驗證** — 如何確認工作已完成
- **常見的合理化** — 代理可能用來跳過步驟的藉口
- **危險信號** — 技能被違反的跡象

### 3. 將技能載入到您的代理中

將相關的 `SKILL.md` 內容複製到您代理的系統提示、規則檔案或對話中。最常見的方法：

**系統提示：** 在會話開始時貼上技能內容。

**規則檔案：** 將技能內容添加到您專案的規則檔案中（CLAUDE.md、.cursorrules 等）。

**對話：** 在給出指令時引用該技能：「遵循測試驅動開發流程來處理此更改。」

### 4. 在需要時使用元技能進行發現

如果您的代理無法原生路由技能，請從載入 `using-agent-skills` 技能開始。它包含一個流程圖，將任務類型映射到適當的技能。

如果您的主機已經從其描述中發現並啟用技能，請不要將 `using-agent-skills` 貼入始終啟用的系統提示或規則檔案中。這會為相同任務創建兩個路由。安裝單獨的技能，讓主機按需啟用它們即可。

### 現有專案無需遷移

使用正常的設定路徑從現有專案的根目錄安裝套件，然後在該專案中繼續工作。技能會針對匹配的任務啟用；它們不需要新的儲存庫佈局或現有程式碼的一次性轉換。

不要將此儲存庫的根目錄 `AGENTS.md` 或 `CLAUDE.md` 複製到專案中。這些檔案配置貢獻者使用 agent-skills 本身。只添加技能以及您的代理通常讀取的任何專案特定指令。在既有程式碼庫中進行漸進式推出，請遵循[採用指南](adoption-guide.md)。

## 推薦的設定

要推出到實際專案？**[採用指南](adoption-guide.md)** 涵蓋了兩種端到端路徑：對於綠地專案，從第一天開始的完整生命週期，以及對於既有程式碼庫，增量式、以驗證為先的推出。下面的設定是快速版本。

### 最簡（從這裡開始）

將三個基本技能載入到您的規則檔案中：

1. **spec-driven-development** — 用於定義要建立的內容
2. **test-driven-development** — 用於證明它有效
3. **code-review-and-quality** — 用於合併前驗證品質

這三者涵蓋了 AI 輔助開發中最關鍵的品質差距。

### 完整生命週期

為了全面覆蓋，按階段載入技能：

```
開始專案：  spec-driven-development → planning-and-task-breakdown
開發期間：  incremental-implementation + test-driven-development
合併前：        code-review-and-quality + security-and-hardening
部署前：       shipping-and-launch
```

### 上下文感知載入

不要一次載入所有技能——這會浪費上下文。載入與當前任務相關的技能：

- 在處理 UI？載入 `frontend-ui-engineering`
- 除錯？載入 `debugging-and-error-recovery`
- 設定 CI？載入 `ci-cd-and-automation`

## 技能解剖

每個技能都遵循相同的結構：

```
YAML 前置信息 (name, description)
├── Overview — 此技能做什麼
├── When to Use — 觸發條件和條件
├── Core Process — 循序漸進的工作流程
├── Examples — 程式碼範例和模式
├── Common Rationalizations — 藉口和反駁
├── Red Flags — 技能被違反的跡象
└── Verification — 退出標準檢查表
```

請參閱 [skill-anatomy.md](skill-anatomy.md) 以獲取完整的規格。

## 使用代理

`agents/` 目錄包含預先配置的代理人形象：

| 代理人 | 用途 |
|-------|---------|
| `code-reviewer.md` | 五軸程式碼審查 |
| `test-engineer.md` | 測試策略和編寫 |
| `security-auditor.md` | 漏洞偵測 |
| `web-performance-auditor.md` | 核心網路指標和效能審查（通過 `/webperf`） |

當您需要專業審查時，載入代理定義。例如，請您的編碼代理「使用 code-reviewer 代理人形象審查此更改」，並提供代理定義。

## 使用指令

`.claude/commands/` 目錄包含 Claude Code 的斜線指令：

| 指令 | 啟動的技能 |
|---------|---------------|
| `/spec` | spec-driven-development |
| `/constraints` | constraint-driven-development |
| `/plan` | planning-and-task-breakdown |
| `/build` | incremental-implementation + test-driven-development |
| `/build auto` | planning-and-task-breakdown → incremental-implementation + test-driven-development（整個計畫，一次批准） |
| `/test` | test-driven-development |
| `/review` | code-review-and-quality |
| `/code-simplify` | code-simplification |
| `/ship` | shipping-and-launch |
| `/webperf` | web-performance-auditor（專業代理，僅限網頁應用） |

> **注意：** 當作為 Claude Code 外掛安裝時，您可能會看到類似於
> _"Default commands/ 資料夾被忽略，因為 manifest 設定了 'commands'"_ 的警告。
> 這是可以預期的。根目錄 `commands/` 資料夾屬於 Antigravity CLI，並且故意與 `.claude/commands/` 分開。所有 Claude Code 的斜線指令都從 `.claude/commands/` 正確載入；該警告只是美觀上的。

## 使用參考資料

`references/` 目錄包含補充檢查表：

| 參考資料 | 與什麼一起使用 |
|-----------|----------------|
| `testing-patterns.md` | test-driven-development |
| `performance-checklist.md` | performance-optimization |
| `security-checklist.md` | security-and-hardening |
| `accessibility-checklist.md` | frontend-ui-engineering |
| `definition-of-done.md` | 所有技能 / 每次更改 |
| `observability-checklist.md` | observability-and-instrumentation |
| `orchestration-patterns.md` | doubt-driven-development |

當您需要超出技能覆蓋範圍的詳細模式時，載入參考資料。

如果您使用 `npx skills add ... --skill <name>` 安裝一個技能，只會複製選定的 `skills/<name>/` 目錄。該技能仍然有效，但無法存取儲存庫級別 `references/` 目錄中的補充檢查表的路徑。請使用整個儲存庫整合、克隆儲存庫，或將所需的檢查表複製到已安裝技能中的 `references/` 目錄中。此可攜性差距在 [addyosmani/agent-skills#361](https://github.com/addyosmani/agent-skills/issues/361) 中被追蹤。

## 規格和任務產物

`/spec` 和 `/plan` 指令會創建工作產物（`SPEC.md`、`tasks/plan.md`、`tasks/todo.md`）。在工作進行期間，請將它們視為**活的文件**：

- 在開發期間將它們保留在版本控制中，以便人與代理擁有共同的真實來源。
- 當範圍或決策改變時進行更新。
- 如果您的儲存庫不希望這些檔案長期存在，請在合併前刪除它們或將資料夾添加到 `.gitignore` 中——該工作流程不需要它們是永久的。

### 在會話之間工作

相同的產物是會話之間的交接。對於一個小任務，在一個會話中運行整個生命週期。對於任何非平凡的任務，每個階段（spec → plan → build → review）都進行一個新的會話可以保持上下文集中——傳遞工作的不是對話，而是已批准的檔案：

- 規格 — `SPEC.md`，或您的規格實際存在的地方
- `tasks/plan.md` 和 `tasks/todo.md` — 或如果您使用其中，則是由計畫確定的外部追蹤器

**在切換之前**，請確保這些檔案反映了仍然適用的決策、您批准的範圍、仍然懸而未決的問題、下一個任務以及當前的驗證狀態（運行了哪些測試，針對什麼）。

**在新會話中**，閱讀實際的檔案並在做任何事情之前查看 `git status`。不要假設您在產物中看不到的批准。將記錄的「測試通過」視為對特定基準的聲明：如果程式碼自那時以來已移動，請重新運行它覆蓋的檢查，如果它沒有說明運行了什麼對什麼，或者您即將觸碰它覆蓋的區域。如果基準仍然成立，就採取它並繼續下一個任務——重點是與變動成比例的檢查，而不是在每次交接時都進行完整的套件。

#### 任務邊界重啟和 Ralph 迴圈

`/build auto` 可以在一會話中運行整個已批准的計畫。它不需要或執行每個任務的全新過程。其每個任務的狀態更新、驗證結果和提交使每個完成的任務成為一個可重啟的邊界，因此一個有能力的外部機制可以退出並在那裡恢復，而無需依賴聊天記錄。

一個層級的「Ralph 迴圈」是機制行為，而不是單獨的技能工作流程。如果您使用它，請在當前任務達到記錄的邊界後才重啟；重新進入時，在選擇下一個待處理任務之前，請閱讀持久的產物和儲存庫狀態。過程退出不是任務通過的證據，重啟不應繞過批准閘門。請參閱 `context-engineering` 技能的**可重啟會話邊界**部分以獲取交接檢查表。

這不需要 `/spec` 和 `/plan` 的包裝——純粹的請求在任何代理中都有效，包括只安裝了技能的 `npx skills add` 安裝：

> 閱讀 SPEC.md，然後將其分解為具有接受標準和依賴順序的小型可驗證任務。將它們保存到 tasks/plan.md 和 tasks/todo.md。還沒有產品程式碼——先給我計畫。

> 閱讀 SPEC.md、tasks/plan.md 和 tasks/todo.md，然後檢查事情實際的狀態——`git status`，以及重新運行任何記錄的驗證狀態不再覆蓋的檢查。告訴我下一個未檢查的任務以及任何仍然懸而未決的事項，然後停止：我會在您開始之前確認範圍。如果計畫看起來不完整，請說明缺少什麼而不是重寫它。

## 提示

1. 對於任何非平凡的工作，**從 spec-driven-development 開始**
2. 在編寫程式碼時，**始終載入 test-driven-development**
3. **不要跳過驗證步驟**——這就是全部重點
4. **選擇性地載入技能**——更多的上下文不一定更好
5. **使用代理進行審查**——不同的視角可以發現不同的問題
