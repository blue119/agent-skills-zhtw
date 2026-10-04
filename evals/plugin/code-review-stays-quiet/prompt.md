---
description: 一個屬於測試驅動開發的測試優先請求。審查技能不得觸發。
expected_outcome: 回覆首先提出一個失敗的測試；代碼審查和質量從未被調用。
max_turns: 12
allowed_tools: [Read, Glob, Grep, Skill]
---

為我們修復它之前為此錯誤編寫一個失敗的測試：splitCents(100, 3) 返回 [33, 33, 33] 並且缺少一個分。
