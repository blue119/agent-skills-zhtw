# 貢獻代理技能

感謝您的興趣貢獻！這個專案是為 AI 編碼代理收集的生產級工程技能的集合。

新來的？[docs/developer-onboarding.md](docs/developer-onboarding.md) 是關於倉庫如何組織的指導，介紹了五個層次、驗證迴圈和貢獻路徑，並告訴您何時需要閱讀這份文件，[skill-anatomy.md](docs/skill-anatomy.md) 和 [evals/README.md](evals/README.md)。此文件是權威的規則書；入職指南是地圖。

## 新增新技能

### 在提議新技能之前

這個套件已經涵蓋了大部分的開發生命週期，許多提議與現有的技能或其他開放 PR 有重疊。在開啟 PR 之前，請執行以下檢查，以免審閱者需要處理重複內容：

1. **搜尋目錄。** 瀏覽 [README.md](README.md) 中的技能列表，並在 `skills/` 中掃描是否存在涵蓋您想法的現有技能，無論是全部還是部分。
2. **檢查開放的 PR。** 執行 `gh pr list --state open`（或瀏覽 PR 標籤）並查找同一主題的提議。已經存在大量幾乎重複的技能組合；不要新增到它們上面。
3. **檢查被拒絕的提議。** 在 [evals/skill-impact.md](evals/skill-impact.md) 中搜尋與您的想法有重疊的早期提議，並在重複工作之前審查它們的評估證據。
4. **閱讀解剖圖。** 確認您的想法符合 [docs/skill-anatomy.md](docs/skill-anatomy.md) 中的格式，這是一個帶有驗證的可操作工作流程，而不是模糊的建議。
5. **在 PR 描述中說明差距。** 明確說明為什麼這沒有被現有技能、開放 PR 或先前被拒絕的提議所涵蓋。如果存在重疊，請提議擴展現有技能而不是新增一個。
6. **移除模型特定的變通方法。** 如果在不提及模型、模型版本或某個代理的私有工具名稱的情況下無法證明某個步驟，它就不屬於一個技能——請描述該能力（參見 [Write the Procedure, Not the Workaround](docs/skill-anatomy.md#write-the-procedure-not-the-workaround)）。

如果您的想法是對現有技能的改進，請優先對該技能進行聚焦的編輯，而不是新增一個目錄。

### 建立技能

1. 在 `skills/` 下建立一個使用連字符命名法的目錄。
2. 添加一個遵循 [docs/skill-anatomy.md](docs/skill-anatomy.md) 格式的 `SKILL.md`。
3. 包含具有 `name` 和 `description` 欄位的 YAML 前置內容，頂層不包含供應商或執行時的鍵（這些應放在 `metadata` 中或在每個代理適配器文件中；參見 [docs/advanced-per-agent-configuration.md](docs/advanced-per-agent-configuration.md)）。
4. 確保 `description` 以技能的描述（第三人稱）開頭，然後包含一個或多個 `Use when` 觸發條件。

### 技能品質標準

技能應該是：

- **具體** — 可操作的步驟，而不是模糊的建議
- **可驗證** — 具有證據要求的明確退出標準
- **經過實戰檢驗** — 基於真實的工程工作流程，而不是理論上的理想
- **最小化** — 僅包含引導代理正確操作所需的內容

### 結構

每個新技能必須包含：

- 技能目錄中的 `SKILL.md`
- 具有有效 `name` 和 `description` 的 YAML 前置內容
- 在 `evals/cases/<skill-name>.json` 中的評估案例文件——至少 3 個正面觸發條件、2 個負面觸發條件（在可能的情況下包含 `owner`），以及 1 個行為評估。執行評估必須由 `evals/fixtures/` 下的真實文件支持；對話型技能可以使用審閱門控的 `kind: "dialogue"` 評估（參見 [evals/README.md](evals/README.md)）代替（CI 執行這些要求）。

新技能通常應遵循標準的解剖結構：

- **概述** — 此技能是做什麼以及為什麼它很重要
- **使用時機** — 觸發條件
- **流程** — 逐步工作流程
- **常見的合理化** — 代理用來跳過步驟的藉口，以及反駁
- **危險信號** — 表明技能被錯誤應用警告的信號
- **驗證** — 如何確認該技能被正確應用

上述前置內容欄位是必需的。結構解剖是推薦的模式：如 `How It Works`、`Workflow` 或 `Core Process` 等等等效標題都可以，只要它們保持相同的意圖並使技能易於遵循即可。

### 不要做的事

- 不要在技能之間重複內容——請引用其他技能。
- 不要新增那些是模糊建議而不是可操作流程的技能。
- 除非內容超過 100 行，否則不要建立支持文件。
- 不要僅為了匹配另一個技能而建立空的 `scripts/` 目錄——只有當技能包含可執行的輔助程序時才添加 `scripts/`。
- 不要將參考資料放在技能目錄中——請使用 `references/`。

## 修改現有技能

在提議更改之前，請在 [evals/skill-impact.md](evals/skill-impact.md) 中搜尋先前影響相同技能的嘗試，並審查它們的評估證據。

- 保持更改聚焦且最小化
- 保留現有的結構和語氣
- 測試編輯後 YAML 前置內容是否仍然有效

如果基於評估結果而拒絕了技能或描述的更改，請在記錄中添加一行，包含日期、受影響的技能、簡潔的嘗試更改、前後的排名第 1 分數、被拒絕的 PR 連結和結果。將該記錄單獨放在預設分支上；不要將其僅留在被拒絕的提議分支上，因為關閉或強制推送提議可能會丟棄該記錄。

## 新增主機指南

README 中的安裝部分僅列出了維護者已安裝並運行當前版本的主機。其他所有主機在 [docs/other-hosts.md](docs/other-hosts.md) 中獲得一行：主機名稱及其安裝命令。沒有專用的設定頁面、供應商連結、標誌、表情符號或螢幕截圖。

- 如果您為主機背後的公司工作，請在 PR 中說明。
- 當維護者運行並願意保持其條目最新的後，主機才會將其內容移動到 README 中。
- 主機指南會按照它們被開啟的順序進行審查。

## 倉庫範圍文件

倉庫根目錄中的 `AGENTS.md` 和 `CLAUDE.md` 配置在 [`addyosmani/agent-skills`](https://github.com/addyosmani/agent-skills) 倉庫本身上運作的代理。在撰寫設定指南或文件時，不要指示使用者將這些文件複製到他們自己的專案或全局代理配置中；可重用的資產是 `skills/` 中的技能。

## 翻譯

我們不接受文件（README、`docs/`）或技能及其內容的翻譯。隨著技能和文件的演進，翻譯副本會脫離同步，我們沒有長期維護它們的方法，除非依賴代理翻譯加上社群修正，這會增加有限價值的維護成本。請將所有技能、文件和貢獻都保持在英文。

## 測試掛鉤

會話開始腳本 (`hooks/session-start.sh`) 在連接到主機的 `SessionStart` 掛鉤時注入 `using-agent-skills` 元技能。Claude Code 外掛並未註冊它——Claude Code 原生路由技能，並且全時注入會為同一任務創建兩個路由器（參見 [docs/getting-started.md](docs/getting-started.md)）；該腳本仍然存在於沒有原生技能路由的主機上。在 `hooks/session-start-test.sh` 中的回歸測試驗證了腳本的 JSON 有效載荷——無論 `jq` 是否可用還是不可用。

在開啟任何涉及以下文件的 PR 之前運行它：

- `hooks/session-start.sh`
- `skills/using-agent-skills/SKILL.md`（掛鉤嵌入的元技能內容）

```bash
bash hooks/session-start-test.sh
```

預期輸出：`session-start JSON payload OK`。如果任何斷言失敗，腳本將以非零退出。

### 重現無 jq 後備方案

當 `jq` 不在 `PATH` 中時，掛鉤仍然會發出相同的 `hookSpecificOutput` 包，其中 `additionalContext` 解釋了需要 `jq`。要在本地執行該分支，請從測試調用中移除 `jq` 的目錄：

```bash
JQ_DIR=$(dirname "$(command -v jq)")
PATH=$(echo "$PATH" | tr ':' '\n' | grep -v "^${JQ_DIR}$" | tr '\n' ':' | sed 's/:$//') \
  bash hooks/session-start-test.sh
```

當 `jq` 位於其自己的目錄中時（例如 Homebrew 中的 `/opt/homebrew/bin`，或手動安裝中的 `/usr/local/bin`），此方法可以乾淨地工作。如果您的 `jq` 與測試依賴的其他工具（例如 `/usr/bin` 中的 `mktemp`）共享系統二進制文件，更簡單的方法是通過單獨的套件管理器安裝 `jq`，使其擁有自己的二進制目錄，然後重新運行。

在被移除的 `PATH` 下，掛鉤的 `command -v jq` 檢查會失敗，`jq` 缺失的後備方案會運行，並且測試會斷言 `additionalContext` 中的 `jq is required` 指導，而不是元技能的主體。

## 報告問題

如果您發現：

- 給出不正確或過時指導的技能
- 對常見工程工作流程的覆蓋不足
- 技能之間的不一致性

請開啟一個問題。

如果某個技能的指導有誤、過時或在您的專案中不適用（例如，它假設在 Maven 或 Gradle 倉庫中是 `npm test`），請使用 [Skill gap](https://github.com/addyosmani/agent-skills/issues/new?template=skill-gap.yml) 問題表單。它要求您提供受影響的技能、相關摘錄、您的專案上下文以及您採取的替代方案——這足以讓維護者在無需自由書寫的情況下進行分類。

## 授權

通過貢獻，您同意您的貢獻將根據 MIT 許可證進行授權。
