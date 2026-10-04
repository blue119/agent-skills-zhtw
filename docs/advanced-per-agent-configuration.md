# 進階每個代理的配置

本儲存庫中的技能是故意設計為可移植的。每個 `SKILL.md` 都將其前置元數據限制在 [代理技能規範](https://agentskills.io) 定義的欄位上，因此在 Claude Code、Cursor、Gemini CLI、Antigravity 和任何其他符合規範的客戶端之間都能正常工作。

一些代理還支援額外的運行時控制：模型路由、工具限制、輪次限制、執行隔離。這些控制很有價值，但它們是特定於供應商的，不應包含在共享的、可移植的前置元數據中。本文檔解釋了每個代理支援哪些控制、它們的歸屬以及如何以選擇加入的方式應用它們而不會破壞可移植性。

這是每個代理的配置的規範參考，它取代了將供應商欄位添加到頂層 `SKILL.md` 前置元數據的早期方法。像 `kind`、`model`、`temperature`、`max_turns`、`tools` 和 `context` 這樣的欄位不應出現在技能的頂層。它們應位於 `metadata` 中或在單獨的每個代理的適配器檔案中，如下所述。

## 原則

1. **保持 `SKILL.md` 可移植。** 代理技能規範為頂層前置元數據保留了 `name`、`description`、`license`、`compatibility`、`metadata` 和實驗性的 `allowed-tools`。供應商或客戶端特定的屬性應位於 `metadata` 下，而不是頂層。
2. **未知的頂層欄位不保證會被忽略。** 許多解析器只讀取 `name` 和 `description` 並靜默丟棄其餘部分，但嚴格符合規範的驗證器或客戶端可能會標記或拒絕未識別的頂層鍵。將供應商欄位放在 `metadata` 下可以確保檔案在任何地方都符合規範。
3. **選擇加入，而非全局。** 運行時控制應該是使用者為自己的代理和工作流程添加的內容，而不是內建在儲存庫中每個技能中的預設值。
4. **偏好生成而非手動編輯。** 當配置是重複的和機械性的時候，應從單一的真實來源生成它，而不是手動維護副本。請參閱 [使用腳本自動化](#automating-with-scripts)。

## 供應商欄位的位置

| 欄位類型 | 範例 | 正確位置 |
|---|---|---|
| 規範欄位 | `name`、`description`、`license`、`compatibility` | `SKILL.md` 中的頂層前置元數據 |
| 供應商元數據 | 模型提示、路由標籤 | `SKILL.md` 中的 `metadata` 鍵下 |
| 運行時協調 | 子代理定義、輪次限制、工具允許列表 | 單獨的每個代理的適配器檔案（見下文每個代理） |

## Claude Code

Claude Code 讀取 `name` 和 `description` 以進行發現。自定義命令和技能共享相同的前置元數據，因此下方的 `context` 和 `allowed-tools` 欄位在 `.claude/commands/*.md` 和 `.claude/skills/*/SKILL.md` 中都有效。

- **`context: fork`** 在隔離的子代理上下文中運行命令或技能，而不是在當前的對話中運行。這提供了上下文隔離、從乾淨狀態可重複執行的能力、在分叉中消耗的執行代幣而不是主會話中的執行代幣，以及一個更乾淨的交接，其中只有結果會呈現在前面。一個可選的 **`agent`** 欄位選擇運行分叉的子代理類型（例如 `agent: Plan`）；它默認為 `general-purpose`。
- **`allowed-tools`** 事先批准工具；它不限制工具。列出的工具在調用該命令或技能的輪次中無需進行權限提示即可運行，並且在您的下一個訊息中授予的權限會被清除。工具池中沒有任何內容被移除。
- **`disallowed-tools`** 是實際限制的欄位：它在技能激活期間將列出的工具從可用池中移除，這正是最小權限在審查或審計等被動階段所需的。限制在您的下一個訊息中也會被清除。請注意，這是 Claude Code 的欄位，而不是規範的六個欄位之一，因此它應放在本地副本或適配器檔案中；在已發布的 `SKILL.md` 中，它會因意外的鍵而導致打包失敗。

這些是 Claude Code 的慣例。請將共享的、已發布的 `SKILL.md` 限制在規範欄位上；在您自己的本地命令或技能副本中應用這些運行時控制。

## Gemini CLI 和 Antigravity

Gemini CLI 和 Antigravity 以相同的方式發現技能：它們匹配 `name` 和 `description`，然後按需加載完整的技能。運行時協調存在於單獨的資源中，而不是技能的前置元數據中。

- **子代理定義** 位於 `.gemini/agents/*.md` 中，這與技能是不同的。它們的模式是 `kind`（`local` 或 `remote`）、`model`、`temperature`、`max_turns` 和 `tools`。請注意，這是一個子代理模式，而不是技能模式：這兩者不應合併到一個 `SKILL.md` 中。
- **模型路由** 可以將較輕的任務（格式化、文檔、git 操作）發送給更快的模型，並將更強的模型保留給需要認知負荷較大的工作（除錯、安全審計、介面設計）。
- **工具限制** 對每個子代理實施最小權限，這也會從提示中修剪未使用的工具模式。
- **輪次限制** (`max_turns`) 限制執行以避免失控的修正循環。

### 模型指導

Google 建議省略 `temperature` 以 Gemini 3.x 模型，並改用 `thinking_level`。不要在技能或路由到 3.x 模型的子代理上硬編碼 `temperature`；請偏好 `thinking_level`，並隨著模型的演進重新審視此指導。

```yaml
# .gemini/agents/security-auditor.md
---
kind: local
model: gemini-3-pro
thinking_level: high   # not: temperature: 0.1
tools: [read_file, grep]
max_turns: 10
---
```

## 使用腳本自動化

手動維護每個技能的每個代理的元數據是重複的、會脫離同步，並且容易出錯。更好的方法是保持單一的真實來源並生成代理特定的輸出：

- 一個生成器讀取每個技能的配置映射（每個技能應為給定代理使用哪個模型、工具允許列表、輪次限制和隔離）。
- 對於元數據樣式的欄位，它將它們注入到每個 `SKILL.md` 的 `metadata` 鍵下，保持檔案符合規範。
- 對於協調樣式的欄位，它會發出代理期望的單獨適配器檔案，例如 `.gemini/agents/*.md` 的子代理定義，而 `SKILL.md` 不變。

生成還允許工具驗證**語義，而不僅僅是形狀**：確認模型名稱是否是已知模型、工具名稱是否是真實工具、`max_turns` 是否為正整數，以及 `temperature`（在適用時）是否為範圍內的有效數字。僅基於形狀的檢查會讓無效的值，如 `parseInt("1.5")` 或 `parseFloat("0.2oops")` 溜過。

## 狀態

本文檔整合了 [#272](https://github.com/addyosmani/agent-skills/pull/272)、[#36](https://github.com/addyosmani/agent-skills/pull/36) 和 [#35](https://github.com/addyosmani/agent-skills/issues/35) 的討論，所有這些都已關閉，以支持這種方法。它是每個代理配置的單一參考：它保持了共享技能的可移植性，同時記錄了每個代理提供的進階控制作為選擇加入的選項。上述基於腳本的自動化是一個待商定的提案，在實施之前。
