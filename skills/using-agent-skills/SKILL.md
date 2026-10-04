---
name: using-agent-skills
description: 發現和呼叫代理技能。在開始會話時，或當你需要決定適用於手頭工作的技能或工作流程時使用。這是管理所有其他技能被發現和呼叫方式的元技能。
---

# 使用代理技能

## 概述

代理技能是一組按開發階段組織的工程工作流程技能。每項技能都編碼了資深工程師遵循的特定流程。這個元技能幫助你發現並應用適合你當前任務的正確技能。

## 技能發現

當任務到達時，識別開發階段並應用相應的技能：

```
任務到達
    │
    ├── 還不知道你想要什麼？ ──────→ interview-me
    ├── 有一個粗略的概念，需要變體？ → idea-refine
    ├── 新專案/功能/變更？ ──→ spec-driven-development
    ├── 沒有寫下品質標準？ ──→ constraint-driven-development
    ├── 有規格，需要任務？ ──────→ planning-and-task-breakdown
    ├── 實作程式碼？ ────────────→ incremental-implementation
    │   ├── UI 工作？ ─────────────────→ frontend-ui-engineering
    │   ├── API 工作？ ────────────────→ api-and-interface-design
    │   ├── 需要更好的上下文？ ─────→ context-engineering
    │   ├── 需要文件驗證的程式碼？ ───→ source-driven-development
    │   └── 涉及高風險/不熟悉的程式碼？ ──→ doubt-driven-development
    ├── 撰寫/執行測試？ ────────→ test-driven-development
    │   └── 基於瀏覽器？ ───────────→ browser-testing-with-devtools
    ├── 有東西出錯？ ──────────────→ debugging-and-error-recovery
    ├── 審查程式碼？ ───────────────→ code-review-and-quality
    │   ├── 太複雜？ ─────────────→ code-simplification
    │   ├── 安全性問題？ ───────→ security-and-hardening
    │   └── 效能問題？ ────→ performance-optimization
    ├── 提交/分支？ ─────────→ git-workflow-and-versioning
    ├── CI/CD 管線工作？ ──────────→ ci-cd-and-automation
    ├── 棄用/遷移？ ────────→ deprecation-and-migration
    ├── 撰寫文件/ADR？ ───────────→ documentation-and-adrs
    ├── 添加日誌/指標/警報？ ───→ observability-and-instrumentation
    └── 部署/發布？ ─────────→ shipping-and-launch
```

## 核心操作行為

這些行為在所有時間、所有技能上都適用。它們是不可協商的。

### 1. 表達假設

在實作任何非瑣碎的事物之前，明確陳述你的假設：

```
我正在做的假設：
1. [關於需求的假設]
2. [關於架構的假設]
3. [關於範圍的假設]
→ 現在糾正我，否則我將以此進行。
```

不要靜默地填補模糊的需求。最常見的失敗模式是做出錯誤的假設並在未檢查的情況下繼續。及早暴露不確定性——這比重做更便宜。

### 2. 主動管理混亂

當你遇到不一致、相互衝突的需求或不清楚的規格時：

1. **停止。** 不要憑猜測繼續。
2. 命名具體的混亂。
3. 提出權衡或提出澄清問題。
4. 在繼續之前等待解決。

**不好：** 靜默地選擇一種解釋並希望它是正確的。
**好：**「我從規格中看到 X，但從現有程式碼中看到 Y。哪一個具有優先權？」

### 3. 當有理由時提出反對意見

你不是一個是/否機器。當某種方法存在明顯問題時：

- 直接指出問題
- 解釋具體的缺點（如果可能，量化——「這增加了約 200 毫秒延遲」，而不是「這可能更慢」）
- 提出替代方案
- 如果對方提供完整資訊並覆蓋了你的意見，則接受他們的決定。

迎合是失敗模式。「當然！」後面跟著實施一個糟糕的想法對沒有人有幫助。誠實的技術分歧比虛假的同意更有價值。

### 4. 執行簡化

你自然的傾向是過度複雜化。主動抵抗它。

在完成任何實作之前，問自己：
- 能否用更少的行完成？
- 這些抽象是否值得它們的複雜性？
- 一位資深工程師看到後會說「你為什麼不直接...」嗎？

如果你編寫了 1000 行而 100 行就足夠了，你就失敗了。偏好乏味、明顯的解決方案。巧妙是昂貴的。

### 5. 維持範圍紀律

只觸碰你被要求觸碰的部分。

不要：
- 刪除你不理解的註釋
- 「清理」與任務正交的程式碼
- 作為副作用重構相鄰系統
- 在沒有明確批准的情況下刪除看似未使用的程式碼
- 添加規格中沒有的功能，因為它們「看起來有用」

你的工作是外科手術般的精確，而不是非請求的翻新。

### 6. 驗證，而非假設

每項技能都包含一個驗證步驟。在驗證通過之前，任務不被視為完成。「看起來對」永遠是不夠的——必須有證據（通過測試、建置輸出、運行時數據）。

按技能進行的驗證是本地檢查。適用於*每一個*變更的專案級標準是完成的定義：測試通過，沒有回歸，行為在運行時得到驗證，文件已更新。請參閱 `../../references/definition-of-done.md`。它補充了每個任務的接受標準，而不是取代它們。

## 應避免的失敗模式

這些是看起來像生產力但會造成問題的細微錯誤：

1. 在不檢查的情況下做出錯誤的假設
2. 不管理自己的混亂——在迷失時繼續前進
3. 不暴露你注意到的不一致
4. 對非明顯決策不提出權衡
5. 對有明顯問題的方法表現出迎合（「當然！」）
6. 過度複雜化程式碼和 API
7. 修改與任務正交的程式碼或註釋
8. 刪除你不完全理解的東西
9. 因為「顯而易見」而編寫而沒有規格
10. 因為「看起來對」而跳過驗證

## 技能規則

1. **在開始工作前，檢查是否有適用的技能。** 技能編碼了可以防止常見錯誤的流程。

2. **技能是工作流程，而不是建議。** 按照順序執行步驟。不要跳過驗證步驟。

3. **多個技能可以適用。** 一個功能實作可能涉及 `idea-refine` → `spec-driven-development` → `planning-and-task-breakdown` → `incremental-implementation` → `test-driven-development` → `code-review-and-quality` → `code-simplification` → `shipping-and-launch` 的順序。

4. **有疑問時，從規格開始。** 如果任務是非瑣碎的且沒有規格，則從 `spec-driven-development` 開始。

## 生命週期順序

對於一個完整的特性，典型的技能順序是：

```
1.  interview-me                → 提取用戶實際想要的東西
2.  idea-refine                 → 完善模糊的想法
3.  spec-driven-development     → 定義我們正在構建的內容
4.  planning-and-task-breakdown → 分解為可驗證的塊
5.  context-engineering         → 載入正確的上下文
6.  source-driven-development   → 對官方文檔進行驗證
7.  incremental-implementation  → 一塊一塊構建
8.  observability-and-instrumentation → 在構建時進行儀表化（與 7-9 並行，而不是在之後）
9.  doubt-driven-development    → 審查進行中的非瑣碎決策
10. test-driven-development     → 證明每一塊都有效
11. code-review-and-quality     → 合併前審查
12. code-simplification         → 在保持行為的同時減少不必要的複雜性
13. git-workflow-and-versioning → 清理提交歷史
14. documentation-and-adrs      → 文件化決策
15. deprecation-and-migration   → 在需要時安全地淘汰舊系統並遷移用戶
16. shipping-and-launch         → 安全部署
```

不是每個任務都需要所有技能。一個錯誤修復可能只需要：`debugging-and-error-recovery` → `test-driven-development` → `code-review-and-quality`。

## 快速參考

| 階段 | 技能 | 單行摘要 |
|-------|-------|-----------------|
| 定義 | interview-me | 在任何計劃、規格或程式碼存在之前，先確定用戶實際想要什麼 |
| 定義 | idea-refine | 透過結構化的發散和收斂思考來完善想法 |
| 定義 | spec-driven-development | 在程式碼之前定義需求和接受標準 |
| 計劃 | planning-and-task-breakdown | 分解為小而可驗證的任務 |
| 建置 | incremental-implementation | 進行垂直切片，在擴展前測試每一塊 |
| 建置 | source-driven-development | 在實作之前，對官方文檔進行驗證 |
| 建置 | doubt-driven-development | 對每一個非瑣碎的決策進行對立性的新上下文審查 |
| 建置 | context-engineering | 在正確的時間獲得正確的上下文 |
| 建置 | frontend-ui-engineering | 具有無障礙性的生產級 UI |
| 建置 | api-and-interface-design | 具有清晰合約的穩定介面 |
| 驗證 | test-driven-development | 先讓測試失敗，然後讓它通過 |
| 驗證 | browser-testing-with-devtools | Chrome DevTools 用於運行時驗證的 MCP |
| 驗證 | debugging-and-error-recovery | 重現 → 定位 → 修復 → 保護 |
| 審查 | code-review-and-quality | 帶有品質門檻的五軸審查 |
| 審查 | code-simplification | 在減少不必要的複雜性的同時保持行為 |
| 審查 | security-and-hardening | OWASP 防護、輸入驗證、最小權限 |
| 審查 | performance-optimization | 先衡量，只優化真正重要的事情 |
| 發布 | git-workflow-and-versioning | 原子提交，乾淨的歷史記錄 |
| 發布 | ci-cd-and-automation | 對每一次變更進行自動化的品質門檻 |
| 發布 | deprecation-and-migration | 安全地移除舊系統並遷移用戶 |
| 發布 | documentation-and-adrs | 文件化「為什麼」，而不僅僅是「什麼」 |
| 發布 | observability-and-instrumentation | 結構化的日誌、RED 指標、追蹤、症狀導向的警報 |
| 發布 | shipping-and-launch | 發布前的檢查清單、監控、回滾計劃 |
