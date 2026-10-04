# 使用 agent-skills 與 Cursor

如何將 [agent-skills](../README.md) 整合到 **Cursor** 中，以使用當前、受支援的專案上下文 — 而非遺留的單體檔案或 Kaizen 特定的佈局。

---

## Cursor 目前支援的功能

Cursor 結合了 **規則**（簡短政策）和 **技能**（完整工作流程）：

| 層級 | 路徑 | 角色 |
|-------|------|------|
| **專案規則** | `.cursor/rules/*.mdc` | 始終生效或檔案範圍的指示 (`alwaysApply`, `globs`) |
| **專案技能** | `.cursor/skills/<skill-name>/SKILL.md` | 代理發現的工作流程；當任務與技能 `description` 相符時讀取 |
| **使用者規則** | Cursor 設定 → 規則 | 帳戶範圍的政策 |
| **使用者技能** (可選) | `~/.cursor/skills/` | 在每個工作區中可用的全域技能 |

文檔：[規則](https://docs.cursor.com/context/rules) · [技能](https://docs.cursor.com/context/skills) (URL 可能會根據 Cursor 更新文檔而重定向)。

### 規則與技能

- **規則** — 簡潔、穩定（「使用慣例提交」、「為公開 Python API 進行型別註記」）。每個檔案只關注一個問題；避免貼上大型指南。
- **技能** — 來自此儲存庫的逐步流程（`test-driven-development`、`code-review-and-quality` 等）。**不要**將整個 `SKILL.md` 主體複製到規則中；這會重複 `.cursor/skills/` 並浪費上下文。

### 遺留（新設定請避免）

| 遺留 | 優先事項 |
|--------|--------|
| 根目錄 `.cursorrules` | `.cursor/rules/*.mdc` |
| 將 `SKILL.md` 複製到 → `.cursor/rules/` | `.cursor/skills/<name>/SKILL.md` |
| 「將 10 個技能作為始終生效規則」 | 1–2 個精簡的 `alwaysApply` 規則 + 按需使用的技能 |

---

## 推薦的專案佈局

```text
your-project/
├── .cursor/
│   ├── rules/                    # 短的 .mdc 政策（你的）
│   │   └── agent-skills.mdc      # 可選：「使用專案技能」的指標
│   └── skills/                   # Cursor Agent 加載的內容
│       ├── using-agent-skills/
│       ├── test-driven-development/
│       ├── code-review-and-quality/
│       └── …                     # 同步自 agent-skills + 你的技能
└── agent-skills/                 # 可選：git 子模組或供應商複製
    └── skills/                   # 僅上游來源
```

**代理的真實來源：** `.cursor/skills/`。
將 `agent-skills/skills/`（或一個複製的 [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills)）視為**上游** — 同步到 `.cursor/skills/`，不要僅編輯上游而期望 Cursor 能看到它。

---

## 設定（任何儲存庫）

### 1. 將技能安裝到 `.cursor/skills/`

**從 `agent-skills` 的本地複製**（在專案根目錄或其他地方）：

```bash
mkdir -p .cursor/skills
rsync -a /path/to/agent-skills/skills/ .cursor/skills/
```

**首次複製，不覆蓋你的自訂技能：**

```bash
rsync -a --ignore-existing /path/to/agent-skills/skills/ .cursor/skills/
```

**上游更新後：**

```bash
rsync -a /path/to/agent-skills/skills/ .cursor/skills/
```

每個技能資料夾必須包含帶有 YAML 前置標記的 `SKILL.md`，最少如下：

```yaml
---
name: test-driven-development
description: 透過測試驅動開發來推動開發。在實施邏輯、修復錯誤或更改行為時使用。
---
```

Cursor 使用 `description`（及相關的元數據）來決定何時應用該技能。

### 2. 添加最少的專案規則（可選但有用）

建立 `.cursor/rules/agent-skills.mdc`：

```markdown
---
description: 使用 .cursor/skills 中的 agent-skills 工作流程
alwaysApply: true
---

在進行非瑣碎的技術工作之前：

1. 透過 `.cursor/skills/using-agent-skills/SKILL.md` 進行路由。
2. 讀取並遵循 `.cursor/skills/<name>/SKILL.md` 下的匹配技能。
3. 當技能連結到它時，在該資料夾中打開 `reference.md`。
4. 優先使用專案技能而非猜測；使用者不需要每次都說「讀取技能」。
```

為儲存庫特定的標準（風格、語言、堆疊）添加**單獨的** `.mdc` 檔案，保持每個檔案的專注性。

**規則檔案格式：**

```markdown
---
description: 出現在 Cursor 規則 UI 中的內容
alwaysApply: false
globs: "**/*.{ts,tsx}"
---

# 你的規則內容
```

| 欄位 | 用途 |
|-------|------|
| `alwaysApply: true` | 此專案中的每次聊天 |
| `globs` | 當匹配的檔案在上下文中時 |
| `alwaysApply: false` + 無 `globs` | 代理請求 / 手動規則（Cursor UI） |

### 3. 使用者級技能（可選）

將你希望在所有地方使用的技能複製或安裝到 `~/.cursor/skills/` 下。用於堆疊範圍的指南（例如語言模式），這些指南不屬於 agent-skills。

`.cursor/skills/` 中的專案技能對**此**儲存庫的工作流程具有優先權。

### 4. 驗證

1. **設定 → 規則** — 列出專案的 `.mdc` 檔案。
2. **代理聊天** — 如果你的 Cursor 建置暴露了它，技能將出現在技能列表中（來自 `.cursor/skills/`）。
3. 執行一個映射到技能的任務（例如「先用測試添加一個功能」），而無需命名檔案 — 如果路由有效，代理應打開 `test-driven-development`。

---

## 代理應如何使用技能

1. **發現** — `using-agent-skills` 將任務階段映射到技能名稱。
2. **閱讀** — 在 `.cursor/skills/<name>/SKILL.md` 中獲取完整流程。
3. **深入探究** — 當技能指示時，使用 `reference.md`、`references/*.md` 或連結的檢查清單。
4. **組合** — 例如，對於一個 API 片段，使用 `incremental-implementation` + `api-and-interface-design`。

明確的使用者短語（「遵循 TDD」、「使用 code-review-and-quality」）仍然有幫助，以防代理偏離軌道。

### 階段 → 技能（快速對照）

| 你正在做的是… | 技能 |
|----------|--------|
| 明確需求 | `interview-me`, `idea-refine`, `spec-driven-development` |
| 規劃任務 | `planning-and-task-breakdown` |
| 實施 | `incremental-implementation`, `frontend-ui-engineering`, `api-and-interface-design` |
| 測試 | `test-driven-development`, `browser-testing-with-devtools` |
| 除錯 | `debugging-and-error-recovery` |
| 審查 | `code-review-and-quality`, `code-simplification` |
| 安全性/效能 | `security-and-hardening`, `performance-optimization` |
| Git / CI / 發布 | `git-workflow-and-versioning`, `ci-cd-and-automation`, `shipping-and-launch` |

完整樹狀結構：儲存庫中的 `skills/using-agent-skills/SKILL.md`。

---

## 不要做的事

| 避免 | 改為做 |
|-------|------------|
| 將所有技能貼入一個規則 | 同步到 `.cursor/skills/` |
| 維持兩個分歧的副本 | 從上游使用 `rsync`；提交 `.cursor/skills/` |
| 許多 `alwaysApply: true` 規則 | 一個路由規則 + 專注的 `globs` 規則 |
| 只依賴 `.cursorrules` | 遷移到 `.mdc` + 技能 |
| 期望 `agent-skills/agents/*.md` 自動加載 | 在聊天中貼入，或將其提煉成一個簡短的規則 |

---

## 上下文提示

- 將 **始終生效** 規則保持簡短（路由 + 1–2 個不可協商的規則）。
- 讓 **技能** 承載長的檢查清單和合理化表格。
- 僅在需要時添加階段特定的 **globs** 規則（例如 `**/*.py`、`**/components/**`）。
- 如果跳過驗證步驟，則透過技能名稱進行提示。

---

## `agents/` 目錄

`agent-skills/agents/` 下的檔案（例如程式碼審查員角色）**不會**被 Cursor 自動加載。選項：

- 參考技能等效物（`code-review-and-quality`）。
- 將代理 Markdown 貼入聊天中進行一次審查。
- 將一個**簡短**的檢查清單提取到一個 `.mdc` 規則中。

---

## 除錯

| 症狀 | 檢查 |
|---------|--------|
| 技能從未被使用 | `.cursor/skills/<name>/` 下的 `SKILL.md`？有效的 `description` 前置標記？ |
| 規則被忽略 | 擴展名是 `.mdc` 嗎？`alwaysApply` / `globs` 是否正確？ |
| 工作流程過時 | 從 `agent-skills/skills/` 重新 `rsync` |
| 指令重複 | 從規則中移除技能內容；保持一個來源 |
| 選擇了錯誤的技能 | 在自訂技能中縮小 `description`；在聊天中提示 |

---

## 檢查清單（新專案）

- [ ] `mkdir -p .cursor/skills` 並從 `agent-skills/skills/` 同步
- [ ] 可選：帶有路由提示的 `.cursor/rules/agent-skills.mdc`
- [ ] 將專案特定的規則作為單獨的、小的 `.mdc` 檔案添加
- [ ] 提交 `.cursor/skills/` 和 `.cursor/rules/`（團隊共用行為）
- [ ] 除非遺留工具要求，否則跳過巨大的 `.cursorrules`

---

## 參考

- [getting-started.md](getting-started.md)
- [../README.md](../README.md) — Cursor 快速摘要
- 上游：[github.com/addyosmani/agent-skills](https://github.com/addyosmani/agent-skills)
