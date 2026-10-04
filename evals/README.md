# 技能評估

本儲存庫衡量其技能是否真正有效的方式：它們在應該發生的時候是否會**觸發**，是否**保持區分**，以及是否**改變代理行為**以符合每個技能的承諾。

## 先前工作（以及我們採納的內容）

沒有單一的既定社群標準來評估 `SKILL.md` 技能，但有兩種方法是指導方向的：

- **Anthropic 的 skill-creator v2** 定義了一個每個技能的 `evals.json`（提示 + `expectations[]`，從轉錄中評分）以及對描述的樣本提示的觸發準確性測試。我們採用了它的 [`evals.json` 模式](https://github.com/anthropics/skills/tree/main/skills/skill-creator) 作為我們的行為層級，並添加了一個可選的 `kind` 欄位來選擇正在評分的產物。
- **Superpowers** (obra) 使用 bash + `claude -p` + 提示固定和評分腳本來測試技能。我們的行為運行器遵循相同的無頭 `claude` 模式，評分標準從 `expectations[]` 中提取。

這兩種方法都未提供的是對多技能*目錄*的**確定性、CI 安全**檢查——每個技能的描述是否包含了用戶實際說的詞彙，以及兩個技能的描述是否會衝突？這在下層的層級 2 中，並且是本儲存庫的添加內容。

## 三個層級

| 層級 | 檢查內容 | 運行方式 | 成本 |
|---|---|---|---|
| 1. 結構性 | 前置元數據、命名、必需部分、命令對等性 | CI (`validate-skills.js`, `validate-commands.js`) | 免費 |
| 2. 觸發與路由 | 正向提示將其技能排在前 k 名；負向提示則不會；兩個描述不相近地衝突 | CI (`run-evals.js`) | 免費 |
| 3. 行為性 | 一個遵循技能的代理是否滿足其 `expectations[]` | 按需 (`run-evals.js --behavioral`) | Token |

層級 2 是路由的**詞彙近似**（基於描述的 TF-IDF）。它不能判斷語義——那是層級 3 的工作——但它能捕捉到支配真實觸發錯誤的兩種失敗模式：描述缺少用戶說的詞彙（假陰性），以及過於寬泛的描述排在正確的技能之上（假陽性）。層級 2 的失敗通常意味著*修訂描述*，而不是評估本身。

## 運行

```bash
# 層級 2 — 確定性，在 CI 中運行
node scripts/run-evals.js
node scripts/run-evals.js --min-rank1 95  # 強制當前的路由底線

# 層級 3 — 行為性，通過無頭 claude 對每個評估進行運行，然後進行評分
node scripts/run-evals.js --behavioral test-driven-development            # 花費 Token
node scripts/run-evals.js --behavioral test-driven-development --dry-run  # 只打印計劃
```

層級 3 支持兩種行為產物類型。`execution` 是預設值：每個評估都在一個臨時的 git 儲存庫中運行，來自 `files[]` 的真實項目輸入從 `evals/fixtures/` 中物質化並作為基準提交，評分器判斷完整的 `--output-format stream-json --verbose` 執行軌跡，包括工具調用。`dialogue` 保留給那些交付成果本身就是對話的技能；它不需要固定文件，評分器在不要求文件編輯或命令的情況下判斷助手的對話回合。聲稱 `dialogue` 是人工審查的豁免，而不是執行技能的通用逃生口。

執行器以明確的權限模式運行（`--permission-mode acceptEdits` 加上預先批准的工具列表），以便執行評估可以真正編輯文件、運行命令、檢查差異並進行提交，而不是被拒絕並進行敘述。軌跡被封裝為不可信數據在評分提示中，並通過標準輸入傳輸給評分器（它們可以是兆位元組；`argv` 將會達到操作系統的參數大小限制），執行器和評分器調用帶有超時，並且評分器的輸出在寫入 `evals/results/`（被 git 忽略）中的 `skill-creator` 的 `grading.json` 形狀之前被驗證為 JSON。紀律技能還包括針對時間壓力、沉沒成本和權威壓力的壓力案例；這些驗證在提示要求跳過時工作流程是否仍然成立。

## 外掛評估（Claude Code）

`claude plugin eval`（Claude Code 2.1.269 或更高版本）一次載入整個外掛，所有技能，在 `evals/plugin/` 下運行案例，並對每個案例在有和沒有外掛的情況下進行評分。它補充了層級 2 而不是取代它：層級 2 檢查描述是否包含了用戶說的詞彙；這檢查 Claude Code 自己的路由在自然提示中選擇了該技能，以及該技能為回覆增加了什麼（`Δ` 欄位）。

```bash
claude plugin eval . --no-publish                                          # 每個案例，兩側，每個運行 3 次
claude plugin eval . --case code-review-fires --runs 1 --ablation none     # 對一個案例進行一次便宜的迭代
```

一個案例是一個包含 `prompt.md`（前置元數據：回合和工具限制；內容：逐字複製的提示）以及每個文件下一個評分器的目錄 `graders/`。報告會出現在 `evals/plugin/results/<timestamp>/`，已經被 git 忽略。每次運行和每次 `llm` 評分器都是對您帳戶的真實模型調用，因此這是按需運行的，永遠不會在 CI 中運行，就像層級 3 一樣。

兩條規則保持了其誠實。Manifest 的 `experimental.evals` 鍵將掃描範圍限制在 `evals/plugin/`，因此在 `evals/` 下的其他固定文件永遠不會被視為案例。並且一個 `tool_used: Skill` 評分器只說該技能是否觸發了；在兩側運行中，它被排除在分數之外，因此請與回覆本身的評分器配對。`code-review-fires` 顯示了模式：觸發指示器、一個免費的 `regex` 評分器，它查找技能的嚴重性分類（Critical、Required、Nit、Optional、Consider、FYI）作為標題或標籤，以及一個 `llm` 裁判來判斷偏移一位。`code-review-stays-quiet` 是精確性檢查：審查技能在測試優先請求上不應觸發，並且其 `Δ` 預期為 0；第二個、未評分的指示器記錄了是否觸發了 `test-driven-development`。

第一次運行學到的（Claude Code 2.1.278, claude-opus-5）。技能調用是隨機的：在案例使用的措辭上，審查技能在 7 次運行中有 5 次觸發，而 `test-driven-development` 在 7 次運行中有 2 次觸發，因此請閱讀 `Δ` 和報告，而不是退出代碼，或者像文檔中的 CI 示例一樣傳遞 `--threshold 0.8`。在一次運行中，審查技能沒有觸發，回覆仍然使用其嚴重性分類作為標題，而這在九次基線運行中都沒有發生；載入的技能描述即使在沒有調用下也會形塑回覆。措辭很重要，描述是槓桿。使用原始描述「在合併之前審查此差異」、「你能否對此更改進行適當的代碼審查再合併？」和「在它進入 main 之前為我審查此拉取請求」很少觸發該技能；模型自己審查了差異，能力很強，但沒有判斷，沒有嚴重性層級或五軸通過。描述技能增加的內容沒有幫助，單獨命名差異和拉取請求也沒有幫助；造成差異的因素是說該技能即使差異是內聯貼上的，也適用，這就是模型將其視為「我可以自己做」的情況。在 `claude-opus-5-5` 中，當模型被釘住時，這一條子將 7 次觸發中的 4 個審查措辭變成了 27 次觸發中的 21 次，將 9 個相鄰的禁止觸發提示在兩側都變成了 0/27，並將最簡單的措辭從 9 次中的 0 個變成了 3/9，因此它更好，而不是解決了問題。`code-review-stays-quiet-on-commit-message` 保護了該子句：貼上差異並要求提交信息時，審查技能不應觸發。來自同一天的兩個實用規則：當您報告數字時，請釘住 `--model` 並記錄 Claude Code 版本，因為 CLI 會自動更新，並且模型在上午和晚上之間改變了兩次；並且給一個案例分配 10 次或更多的回合預算，因為模型可能會通過其斜線命令（`agent-skills:review`，然後 `code-review-and-quality`）到達一個技能，這需要兩次回合。

## 案例格式

每個技能一個文件：`evals/cases/<skill-name>.json`。

```json
{
  "skill_name": "test-driven-development",
  "trigger": {
    "positive": [
      { "prompt": "在修復之前為此錯誤編寫一個失敗的測試", "top_k": 3 }
    ],
    "negative": [
      { "prompt": "更新文檔中的架構圖", "owner": "documentation-and-adrs" }
    ]
  },
  "evals": [
    {
      "id": 1,
      "kind": "execution",
      "prompt": "財務部門整理了 BUG.md 中寫的會計錯誤。修復它。",
      "expected_output": "一個用於丟失分數案例的失敗重現測試，一個保留 README 不變性（準確總和、最早分享公平性）的修復，該公平性由其自身的測試覆蓋，完整套件通過",
      "files": [
        "test-driven-development"
      ],
      "expectations": [
        "添加並顯示一個重現 BUG.md 中丟失分數案例的測試，在修改 src/split.js 之前失敗",
        "最終實現滿足完整的 README 公平性不變性（剩餘分數分配給最早的分享者）：splitCents(10000, 3) 返回 [3334, 3333, 3333] 如 BUG.md 期望，而 splitCents(100, 7) 返回 [15, 15, 14, 14, 14, 14, 14] 如 README 示例所示；將所有剩餘部分分配給單個分享者將違反兩者",
        "README 中的公平性不變性從在至少有 2 的餘數輸入（如 splitCents(100, 7)）上的測試中獲得了自己的測試案例，在其中將所有餘數分配給單個分享者將失敗，超出報告的丟失分數案例",
        "修復後使用儲存庫自己的命令運行完整套件"
      ]
    }
  ]
}
```

- `evals[]` 使用 `skill-creator` 的核心模式（`id`、`prompt`、`expected_output`、可選的 `files[]`、`expectations[]`）以及本儲存庫的可選 `kind`。`kind` 必須是 `execution` 或 `dialogue`，並且默認為 `execution` 以保持相容性。執行評估需要非空的 `files[]`；路徑相對於 `evals/fixtures/`，可以命名一個文件或項目目錄。`dialogue` 評估可以省略 `files[]`，因為轉錄就是產物。`expectations` 是評分器檢查相關產物的可驗證陳述——行為，而不是措辭。
- `trigger` 是本儲存庫的擴展。`positive` 提示是應該在此處路由的現實用戶請求（`top_k` 默認為 3；將其收緊到 1 以匹配技能的簽名請求）。`negative` 提示屬於*不同的*技能；該技能在它們身上不得排在第一位。在您能做到時，在 `owner` 中聲明該技能：運行器將斷言該 `owner` **排在**該技能之上，將負向提示變成一個真正的成對路由測試，而不是一個在提示與任何內容不匹配時可以空泛通過的測試。

**撰寫好的觸發提示：** 轉述用戶實際的說法；不要複製描述（那是在操縱評估）。如果一個現實的提示因為描述缺乏其詞彙而無法排序，那就是一個真正的發現——改進描述。

## 添加技能

每個技能都附帶一個評估文件。當您添加 `skills/<name>/` 時，請添加 `evals/cases/<name>.json`，其中至少有 3 個正向觸發、2 個負向觸發和 1 個行為評估。執行評估必須由 `evals/fixtures/<name>/` 支持；僅在技能的交付成果確實是對話本身時使用 `kind: "dialogue"`。缺少案例文件、不完整的案例數量、未知類型、無效的固定路徑和缺失的必需固定文件是 CI 錯誤。

## 應關注的指標

層級 2 運行會打印**觸發排名-1 比例**（排在第一位的正向提示佔比，而不僅僅是前 k 名）。CI 運行使用 `--min-rank1 95`，在檢查的 100% 基線下方留下了有用的餘裕，這樣一個不相關的描述編輯就不會立即使 CI 變為紅色。隨著路由的改善，提高底線；絕不要降低它以使回退通過。數字下降意味著描述在彼此之間漂移。在 $\ge 75\%$ 的成對描述相似度處出現的衝突檢查錯誤，以及在 $\ge 50\%$ 處發出的警告。當這些評估暴露描述-詞彙差距時（參見 [#351](https://github.com/addyosmani/agent-skills/issues/351) 以獲取原始示例），請修訂描述，而不是提示。
