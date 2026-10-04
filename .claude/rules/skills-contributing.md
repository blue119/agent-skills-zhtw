---
description: 添加或更改技能的反重複保護欄
paths:
  - "skills/**"
---

# 添加或更改技能

此儲存庫已經涵蓋了大部分的開發生命週期，因此大多數新的技能想法會與現有的技能或已開啟的 PR 重疊。在建立新的 `skills/<name>/` 目錄或大幅修改現有的目錄之前：

- 執行 [CONTRIBUTING.md](../../CONTRIBUTING.md#before-proposing-a-new-skill) 中的預檢檢查：搜尋目錄，檢查已開啟的 PR (`gh pr list --state open`)，並證明該差距。
- 傾向於擴展現有的技能而不是添加幾乎重複的技能。如果該想法與現有技能重疊，請編輯該技能而不是添加一個新目錄。
- 將 `SKILL.md` 保留在 [docs/skill-anatomy.md](../../docs/skill-anatomy.md) 中，並且絕不在技能之間重複內容，而是引用其他技能。

`CONTRIBUTING.md` 是完整工作流程的唯一真實來源；此規則指向它，而不是重複其檢查清單。
