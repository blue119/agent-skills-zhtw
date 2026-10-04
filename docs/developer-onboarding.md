# 開發者入門

本指南適用於在 agent-skills 儲存庫本身工作的用戶：貢獻技能、修復文件、改進評估框架。如果你想在自己的專案中使用這些技能，請查看 [getting-started.md](getting-started.md)。

這是一個導覽，而不是規則集。規則存在於 [CONTRIBUTING.md](../CONTRIBUTING.md)（貢獻工作流程）、[skill-anatomy.md](skill-anatomy.md)（技能格式）和 [evals/README.md](../evals/README.md)（評估框架）中；本文告訴你該閱讀哪些文件以及這些部分是如何組合在一起的。

---

## 1. 思維模型

該儲存庫有五個可組合的層次。了解每個層次是「為了什麼」能避免最常見的貢獻錯誤（將參考資料放入技能中、建立導向其他角色的角色、在不同技能間重複內容）。

| 層次 | 位置 | 工作內容 | 一詞概括 |
|---|---|---|---|
| **技能 (Skills)** | `skills/<name>/SKILL.md` | 帶有驗證門的逐步工作流程 | *如何* |
| **角色 (Personas)** | `agents/<role>.md` | 具有視角和輸出格式的角色 | *誰* |
| **指令 (Commands)** | `.claude/commands/`, `.gemini/commands/`, `commands/` | 用戶可見的入口點；協調層 | *何時* |
| **參考資料 (References)** | `references/*.md` | 技能按需引用的檢查清單 | *要檢查什麼* |
| **評估 (Evals)** | `evals/cases/<name>.json` | 證明技能觸發和行為正確 | *是否有效* |

有兩個結構規則值得早期內化：

- **用戶（或斜線指令）是協調者。** 角色絕不呼叫其他角色；唯一被認可的多角色模式是並行分發並帶有合併步驟（參見 [references/orchestration-patterns.md](../references/orchestration-patterns.md)）。
- **不要重複，不要引用。** 技能應連結到其他技能和 `references/` 而不是重複內容。此規則也適用於文件，包括本文。

一個讓大家感到困惑的範圍限制：儲存庫根目錄中的 `AGENTS.md` 和 `CLAUDE.md` 配置了在*本儲存庫*上運行的代理。它們不是可重用的資產，並且設定指南絕不能告訴用戶將它們複製到自己的專案中；可重用的資產是技能。

請注意，指令存在於三個並行的目錄中（Claude Code、Gemini CLI、Antigravity）。接觸其中一個，CI 檢查就能同步所有它們，請參見 §3。

## 2. 本地設定

```bash
git clone https://github.com/addyosmani/agent-skills.git
cd agent-skills
```

沒有建置步驟也沒有 `package.json`；驗證器是純粹的 Node 腳本。你需要：

- **Node 20+**（CI 運行環境）用於 `scripts/` 中的驗證器
- **bash**（推薦 `jq`）用於掛鉤回歸測試
- **`gh` CLI** 用於在提議技能前檢查重複 PR
- **Claude Code** 僅在你想在本地運行第三級行為評估時需要

要嘗試在本地檢查點運行打包內容：

```bash
claude --plugin-dir /path/to/agent-skills
```

## 3. 驗證循環

該儲存庫會為自己烹飪：對於技能來說，驗證是不可協商的，對於對儲存庫的貢獻也是不可協商的。所有 CI 運行的事情，你可以在幾秒鐘內在本地運行：

```bash
# 第一級，結構性：前置元數據、命名、必需部分
node scripts/validate-skills.js

# 跨三個指令目錄同步指令和描述
node scripts/validate-commands.js

# 第二級，觸發與路由：正面提示排名前 k，負面提示不衝突
node scripts/run-evals.js

# 第三級，行為（按需，消耗 Token；--dry-run 打印計劃）
node scripts/run-evals.js --behavioral <skill-name> --dry-run

# 掛鉤回歸測試，如果你修改了 hooks/session-start.sh
# 或 skills/using-agent-skills/SKILL.md
bash hooks/session-start-test.sh
```

這三個評估層次值得理解，即使你從不接觸框架，因為紅色的第二級通常意味著*修復你的技能描述*，而不是評估：第二級是路由的詞彙近似（基於描述的 TF-IDF），它的兩個目標失敗模式是描述缺少用戶實際說的詞彙，以及過於寬泛的描述排在正確的技能之上。完整的設計、模式和信任級別規則在 [evals/README.md](../evals/README.md) 中。

在每次 PR 之前運行相關子集。通過第一級 + 第二級 + 指令一致性綠燈的 PR 是可審查的；不綠燈的 PR 會在任何人閱讀內容之前在機制上被彈回。

## 4. 貢獻路徑

### 路徑 1：修復或改進現有技能（最常見，首選 PR）

1. 保持更改集中且最小化；保留技能的結構和語氣。
2. 如果你更改了前置元數據 `description`，請預期第二級的影響；運行 `node scripts/run-evals.js` 並檢查技能的觸發提示是否仍然排名。
3. 運行第一級以確認前置元數據仍然有效。

### 路徑 2：提議新技能（門檻更高，進行預先檢查）

目錄已經涵蓋了大部分生命週期，因此證明負擔在於差距。在編寫任何內容之前，請在 [CONTRIBUTING.md](../CONTRIBUTING.md#before-proposing-a-new-skill) 中運行預先檢查：搜索目錄，檢查已打開的 PR（`gh pr list --state open`；已存在近重複的集群），確認想法符合 [skill-anatomy.md](skill-anatomy.md)，並在你的 PR 描述中明確說明差距。如果它與現有技能重疊，則專注編輯該技能勝於建立新目錄。

新技能以一套的形式發布，而不是單個文件：`skills/<kebab-case-name>/SKILL.md`、匹配的 `evals/cases/<name>.json`，以及 `scripts/` 目錄僅在發布可運行的輔助程序時才存在（參考資料放在 `references/` 中，絕不放在技能內部）。確切的前置元數據規則、部分結構和評估案例的最小值存在於 [CONTRIBUTING.md](../CONTRIBUTING.md#structure) 和 [skill-anatomy.md](skill-anatomy.md) 中；請從那裡獲取，而不是從這次導覽中獲取，這樣兩者就不會產生偏差。

一個值得內化的觀點：在撰寫觸發提示時，請轉述用戶實際的說法；將描述複製到提示中會影響評估，但不會告訴你任何信息。

### 路徑 3：文件、參考資料、框架

- 文件和技能**僅限英文**；不接受翻譯，因為它們會產生偏差（[CONTRIBUTING.md](../CONTRIBUTING.md#translations) 說明了原因）。
- 對 `scripts/run-evals.js` 或評估模式的更改應與技能創建者的 `evals.json` 模式保持相容性（行為層級採用該模式的兼容性是特性，而不是意外）。
- 任何觸及 session-start 掛鉤或其嵌入的元技能的內容都需要進行掛鉤回歸測試（§3）。

## 5. PR 前檢查清單

- [ ] 第一級綠燈：`node scripts/validate-skills.js`
- [ ] 第二級綠燈：`node scripts/run-evals.js`
- [ ] 如果你更改了任何指令目錄：指令一致性綠燈：`node scripts/validate-commands.js`
- [ ] 如果你更改了 `hooks/` 或 `using-agent-skills`：掛鉤測試綠燈
- [ ] 新技能 → 存在評估案例文件，並有最小的觸發/行為計數
- [ ] 新技能 → PR 描述中說明差距；已檢查目錄和打開的 PR
- [ ] 沒有重複內容；使用交叉引用
- [ ] 更改很小且集中（儲存庫自身的 `code
