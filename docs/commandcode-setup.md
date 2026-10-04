# 使用代理技能與指令代碼

[Command Code](https://commandcode.ai) 具有一個原生的技能系統。內建的 `cmd skills` 命令會複製一個 GitHub 儲存庫，遞迴發現所有的 `SKILL.md`，並安裝你選擇的那些。

Command Code 二進制文件可用於 `cmd`（在 Windows 上別名為 `cmdc` 和 `command-code`）。以下範例使用 `cmd`。

## 安裝

**專案範圍**（安裝在當前 git 根目錄下的 `.commandcode/skills/` — 預設）：

```bash
cmd skills add addyosmani/agent-skills
```

在互動終端機中，這會顯示多選，讓你選擇要安裝的 25 個技能。管道/非互動式調用會安裝所有發現的技能。

**安裝特定技能：**

```bash
cmd skills add addyosmani/agent-skills -s spec-driven-development
```

**使用者範圍**（安裝在 `~/.commandcode/skills/`，在每個專案中可用）：

```bash
cmd skills add addyosmani/agent-skills --global
```

**其他支援的格式：**

```bash
cmd skills add addyosmani/agent-skills@main            # 一個特定的分支
cmd skills add addyosmani/agent-skills/skills/interview-me   # 儲存庫中的特定路徑
cmd skills add addyosmani/agent-skills --force         # 如果已安裝則覆蓋/更新
```

## 管理

```bash
cmd skills list                       # 列出已安裝的技能（專案 + 使用者 + 捆綁）
cmd skills remove spec-driven-development           # 移除專案範圍的技能
cmd skills remove spec-driven-development --global  # 移除使用者範圍的技能
```

`add` 上的 `--force` 會重新獲取並覆蓋現有的技能，這是更新到最新版本的做法。

## 使用方法

已安裝的技能會自動被發現，並出現在 TUI 斜線選單中，標記為 `[skill]`：

```
/spec-driven-development   [skill] 在編寫代碼前先寫個規範…
```

輸入 `/` 即可瀏覽，或開始輸入技能名稱以過濾。使用 `/skills` 來啟用/禁用技能。

## 技能的存放位置

Command Code 從這些位置發現技能（專案條目會對應到最近的 git 根目錄）：

| 範圍 | 路徑 |
|-------|------|
| 專案 | `.commandcode/skills/<name>/SKILL.md` |
| 專案 (agents-compat) | `.agents/skills/<name>/SKILL.md` |
| 使用者 | `~/.commandcode/skills/<name>/SKILL.md` |
| 使用者 (agents-compat) | `~/.agents/skills/<name>/SKILL.md` |

`cmd skills add` 會寫入 `.commandcode/skills/`（專案）或 `~/.commandcode/skills/`（`--global`）。
