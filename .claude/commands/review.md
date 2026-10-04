---
description: 進行五軸程式碼審查 — 正確性、可讀性、架構、安全性、效能
---

呼叫代理技能：code-review-and-quality。

審查所有五個軸的當前變更（暫存或最近的提交）：

1. **正確性 (Correctness)** — 是否符合規格？是否處理了邊界案例？測試是否足夠？
2. **可讀性 (Readability)** — 名稱是否清晰？邏輯是否直觀？組織是否良好？
3. **架構 (Architecture)** — 是否遵循現有模式？邊界是否乾淨？抽象層級是否正確？
4. **安全性 (Security)** — 輸入是否經過驗證？密鑰是否安全？是否檢查了認證？（使用 security-and-hardening 技能）
5. **效能 (Performance)** — 是否沒有 N+1 查詢？是否沒有無界操作？（使用 performance-optimization 技能）

將發現分類為嚴重 (Critical)、重要 (Important) 或建議 (Suggestion)。
輸出結構化的審查結果，包含具體的檔案：行號引用和修復建議。
