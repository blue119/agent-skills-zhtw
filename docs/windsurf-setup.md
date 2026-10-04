# 使用代理技能與 Windsurf

## 設定

### 專案規則

Windsurf 使用 `.windsurfrules` 來存放專案特定的代理指令：

```bash
# 從你最重要的技能中建立一個組合規則檔案
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .windsurfrules
echo "\n---\n" >> .windsurfrules
cat /path/to/agent-skills/skills/incremental-implementation/SKILL.md >> .windsurfrules
echo "\n---\n" >> .windsurfrules
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> .windsurfrules
```

### 全域規則

對於你希望在所有專案中都使用的技能，請將它們新增到 Windsurf 的全域規則中：

1. 開啟 Windsurf → 設定 (Settings) → AI → 全域規則 (Global Rules)
2. 貼上你最常用的技能內容

## 建議的配置

請將 `.windsurfrules` 保持專注於 2-3 個基本技能，以維持在上下文限制範圍內：

```
# .windsurfrules
# 本專案的必要代理技能

[貼上 test-driven-development SKILL.md]

---

[貼上 incremental-implementation SKILL.md]

---

[貼上 code-review-and-quality SKILL.md]
```

## 使用提示

1. **要選擇性** — Windsurf 的上下文是有限的。請選擇能解決你最大品質差距的技能。
2. **在對話中引用** — 在處理特定階段時，將額外的技能內容貼入聊天中（例如，在建置認證時，貼上 `security-and-hardening`）。
3. **將引用作為檢查清單** — 貼上 `references/security-checklist.md` 並要求 Windsurf 驗證每一項。
