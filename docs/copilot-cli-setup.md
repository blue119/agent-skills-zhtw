# 使用 agent-skills 與 GitHub Copilot CLI

獨立的 `copilot` 命令列工具會將此儲存庫安裝為一個外掛程式並在 `skills/` 中發現所有技能。對於 VS Code 中的 Copilot，請參閱 [copilot-setup.md](copilot-setup.md) — 設定和呼叫模型是不同的。

## 安裝

**從此儲存庫的市場** — 註冊它，然後從中安裝。`addy-agent-skills` 是此儲存庫宣告的市場名稱，而不是 GitHub 範圍的註冊表，該名稱僅在執行 `marketplace add` 後才會解析：

```bash
copilot plugin marketplace add addyosmani/agent-skills
copilot plugin install agent-skills@addy-agent-skills
```

**直接從儲存庫**，無需註冊市場：

```bash
copilot plugin install addyosmani/agent-skills
```

**從本地克隆**，用於會話範圍的開發安裝：

```bash
git clone https://github.com/addyosmani/agent-skills.git
copilot --plugin-dir /path/to/agent-skills
```

`--plugin-dir` 只為該會話載入外掛程式，並且不會永久安裝任何內容 — 在編輯技能時使用它。

## 驗證

```bash
copilot plugin list   # agent-skills@addy-agent-skills
copilot skill list    # 外掛程式的技能，以及內建的技能
```

在互動會話中，`/skills list` 會顯示相同的目錄。

## 您獲得的和沒有獲得的內容

根目錄的 `plugin.json` 是 Copilot CLI 讀取的宣告文件 — 它優先於屬於 Claude Code 的 `.claude-plugin/plugin.json`。該根宣告檔只宣告了名稱、版本和描述，因此組件路徑會回退到其預設值：

- **技能 — 可用。** 沒有明確的路徑，CLI 使用慣例的 `skills/` 目錄，技能在其中被發現。
- **生命週期指令 — 不可用。** 根宣告檔沒有 `commands` 欄位，因此沒有註冊 `/spec`、`/plan`、`/build`、`/test`、`/review` 或 `/ship`。這些檔案位於 `.claude/commands/` 中，是 Claude Code 的指令。

有關宣告檔的優先順序和每個組件的路徑預設值，請參閱 [CLI 外掛程式參考](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-plugin-reference) 和 [建立外掛程式](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/plugins-creating)。

## 使用方法

命名您想要的技能，或描述任務讓代理導向它：

> 使用 spec-driven-development 技能來為 [該功能] 編寫規格。

> 使用 test-driven-development 技能：先為這個錯誤編寫一個失敗的測試，然後修復它。

> 使用 code-review-and-quality 技能來審查我的暫存變更。

## 疑難排解

| 症狀 | 應檢查內容 |
|---------|---------------|
| `plugin install` 無法解析 `agent-skills@addy-agent-skills` | 首先執行 `copilot plugin marketplace add addyosmani/agent-skills` — 該名稱只有在市場被註冊後才會解析。或者直接安裝儲存庫。 |
| 外掛程式已安裝但沒有技能 | 執行 `copilot plugin list` 確認外掛程式，然後執行 `copilot skill list`（或會話中的 `/skills list`）以查看被發現的內容。 |
| 找不到 `/spec`、`/build` 等 | 這是預期的，而不是安裝錯誤：根宣告檔沒有註冊任何指令。請改用名稱來請求該技能。 |
| 本地技能已更改但 CLI 顯示舊的副本 | 開始一個新的會話，或使用 `--plugin-dir` 指向您的克隆。 |
