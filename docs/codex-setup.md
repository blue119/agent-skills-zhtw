# 使用 agent-skills 與 Codex

此儲存庫也是一個 [Codex 外掛](https://developers.openai.com/codex/plugins/build)。與 Claude Code 使用的相同根層級 `skills/` 目錄被 Codex 消耗，因此沒有檔案被複製或重複。

## 安裝

```bash
codex plugin marketplace add addyosmani/agent-skills
codex plugin add agent-skills@agent-skills
```

> 需要 Codex CLI v0.122 或更高版本。在舊版本中，該指令是 `codex marketplace add`。請參閱 [Codex CLI 文件](https://developers.openai.com/codex/cli)。

第一個指令將此儲存庫註冊為 `agent-skills` 市場。第二個指令從該市場安裝並啟用 `agent-skills` 外掛。安裝後請開始一個新的 Codex 會話，以便發現這些技能。

本地克隆也可以：

```bash
codex plugin marketplace add /path/to/your/clone
codex plugin add agent-skills@agent-skills
```

## 使用方法

安裝後，在 Codex 聊天中用 `@` 呼叫一個技能（例如 `@spec-driven-development`），或者直接描述任務讓 Codex 選擇正確的技能。`skills/` 下的所有 25 個技能都可用。

[Codex 採用漸進式揭露](https://developers.openai.com/codex/skills)：它從每個技能的 `name` 和 `description` 開始，按需選擇技能，然後僅在選擇時才載入完整的 `SKILL.md`。不要將 `using-agent-skills/SKILL.md` 也貼到 `AGENTS.md`、系統提示或其他常駐上下文：這會將該套件的元路由堆疊在 Codex 的原生路由上，並增加不必要的路由工作。該元技能可以與套件一起保持安裝；警告是針對預先載入其完整指令的。

## 工作原理

- `.codex-plugin/plugin.json` — 儲存庫根目錄下的 Codex 外掛標記。將 `skills` 指向 `./skills/` 並提供 Codex 所需的元數據。
- `.agents/plugins/marketplace.json` — 市場入口，宣告儲存庫根目錄 (`./`) 為外掛來源。
- `skills/<name>/SKILL.md` — 不變。Codex 和 Claude Code 共享相同的 `name` + `description` 前置元數據格式，因此一個檔案同時服務於這兩個平台。

`.claude/commands/` 中的斜線指令和 `agents/` 中的角色保持特定於 Claude Code。`hooks/` 下的 `SessionStart` 腳本是針對沒有原生技能路由的主機的獨立輔助工具，並且未被任何外掛連接。在 Codex 中，請直接呼叫底層技能而不是斜線指令（例如 `@spec-driven-development` 而不是 `/spec`）。
