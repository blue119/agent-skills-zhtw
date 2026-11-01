# 25 個 Agent Skills：技能總覽、使用順序與流程

本文件整理 `skills/` 目錄中的 25 個技能，說明每個技能的用途、觸發時機，以及它在完整軟體開發生命週期中的位置。

> 並非每項任務都要依序使用全部 25 個技能。先以 `using-agent-skills` 判斷任務階段，再選擇必要技能；條件式技能只在相關風險或工作類型出現時加入。

## 建議使用順序

完整功能的典型順序如下：

```text
任務進入
  ↓
using-agent-skills
  ↓
interview-me → idea-refine
  ↓
spec-driven-development → constraint-driven-development
  ↓
planning-and-task-breakdown
  ↓
context-engineering → source-driven-development
  ↓
incremental-implementation
  ├─ UI：frontend-ui-engineering
  ├─ API：api-and-interface-design
  ├─ 高風險：doubt-driven-development
  └─ 同步建立：observability-and-instrumentation
  ↓
test-driven-development
  ├─ 瀏覽器：browser-testing-with-devtools
  └─ 發生錯誤：debugging-and-error-recovery
  ↓
code-review-and-quality
  ├─ 安全：security-and-hardening
  ├─ 效能：performance-optimization
  └─ 簡化：code-simplification
  ↓
git-workflow-and-versioning → ci-cd-and-automation
  ↓
documentation-and-adrs
  ├─ 淘汰舊系統：deprecation-and-migration
  └─ 正式發布：shipping-and-launch
```

`git-workflow-and-versioning`、`observability-and-instrumentation` 與 `documentation-and-adrs` 不必等到最後才使用；它們通常會貫穿實作過程。上圖只標示最晚應完成的階段。

## 第一階段：技能選擇

### 1. `using-agent-skills`

**技能名稱：** 使用代理技能

**概述：** 所有其他技能的入口。它將任務映射到正確的開發階段，並定義共通操作規則，例如公開假設、管理歧義、控制範圍及以證據驗證結果。

**何時使用：** 每次工作開始時，或不確定應套用哪一個技能時。

**流程位置：** 永遠先使用；工作途中若任務性質改變，也應重新判斷技能組合。

## 第二階段：釐清與定義

### 2. `interview-me`

**技能名稱：** 訪談我

**概述：** 一次詢問一個高價值問題，區分使用者真正想要的結果與表面提出的解法，直到對意圖有足夠信心。

**何時使用：** 需求模糊、缺少目標使用者或成功定義，或使用者明確要求訪談、審問或壓力測試需求時。

**流程位置：** 在規格、計畫與程式碼之前；需求已清楚時可跳過。

### 3. `idea-refine`

**技能名稱：** 想法精煉

**概述：** 以發散、比較、收斂三個階段，把模糊概念轉化為具有問題陳述、建議方向、假設、MVP 範圍與開放問題的可執行提案。

**何時使用：** 已有粗略想法，但尚未決定最佳方向、範圍或取捨時。

**流程位置：** 通常接在 `interview-me` 之後、`spec-driven-development` 之前。

### 4. `spec-driven-development`

**技能名稱：** 基於規格的開發

**概述：** 在編碼前建立活的規格，定義目標、技術棧、命令、專案結構、測試策略、邊界、成功標準與開放問題。

**何時使用：** 新專案、新功能、重大變更，或任何跨越多個可測能力的非瑣碎工作。

**流程位置：** 將已收斂的想法變成正式合約，完成後才進入任務分解。

### 5. `constraint-driven-development`

**技能名稱：** 基於約束的開發

**概述：** 把品質要求寫成可執行的標準，包含底線、量化閾值、只衡量項目與例外，並防止以停用規則、跳過測試或降低門檻來取得假性綠燈。

**何時使用：** 專案尚未明確寫下覆蓋率、效能、無障礙性、安全性或程式碼品質標準時。

**流程位置：** 規格形成後、規劃與實作前；約束會持續約束後續所有階段。

## 第三階段：規劃與上下文

### 6. `planning-and-task-breakdown`

**技能名稱：** 規劃與任務分解

**概述：** 將規格拆成有依賴順序、驗收標準、檢查點及可平行機會的小型垂直任務。

**何時使用：** 已有清楚規格，但工作太大、跨多個檔案或需要多人／多代理協作時。

**流程位置：** 規格與約束之後，正式實作之前。

### 7. `context-engineering`

**技能名稱：** 上下文工程

**概述：** 管理規則檔、規格、架構、相關原始碼、錯誤輸出與對話摘要，讓代理在有限上下文中保留最重要的資訊。

**何時使用：** 新會話、切換任務、輸出品質下降、上下文衝突，或專案資訊過多時。

**流程位置：** 在實作前建立上下文包；長任務中可重複執行。

### 8. `source-driven-development`

**技能名稱：** 源驅動開發

**概述：** 先確認版本，再查閱官方文件，以官方模式實作並引用來源；對未驗證內容明確標記不確定性。

**何時使用：** 使用框架、函式庫、SDK 或平台 API，而且正確性與版本差異很重要時。

**流程位置：** 在採用技術方案前使用，並在實作過程遇到新 API 決策時再次使用。

## 第四階段：設計與實作

### 9. `incremental-implementation`

**技能名稱：** 逐步實作

**概述：** 用小型、可驗證、可回滾的垂直切片交付功能；每一片都經過實作、測試、驗證及提交，再擴展下一片。

**何時使用：** 任何跨多個檔案的功能或變更，或任務大到無法一次安全完成時。

**流程位置：** 實作階段的主幹，其他設計、測試與驗證技能附著其上。

### 10. `frontend-ui-engineering`

**技能名稱：** 前端 UI 工程

**概述：** 建立生產級元件、狀態管理、設計系統、響應式佈局與 WCAG 2.1 AA 無障礙體驗，並避免通用的 AI 生成視覺風格。

**何時使用：** 建立或修改頁面、元件、版面、互動、視覺系統或無障礙行為時。

**流程位置：** 與 `incremental-implementation` 和 `test-driven-development` 一起使用；不是 UI 任務時跳過。

### 11. `api-and-interface-design`

**技能名稱：** API 和介面設計

**概述：** 以合約優先、穩定錯誤語義、邊界驗證、可預測命名與向後相容策略設計 REST、GraphQL、模組或前後端介面。

**何時使用：** 新增或修改公開 API、模組邊界、資料類型合約或跨系統介面時。

**流程位置：** 在依賴該介面的實作開始前先固定合約。

### 12. `observability-and-instrumentation`

**技能名稱：** 可觀測性與儀器化

**概述：** 先定義「運作正常」的可觀察條件，再加入結構化日誌、指標、追蹤、警報與運行指南，並驗證遙測本身。

**何時使用：** 功能將在生產環境運行、需要診斷行為，或現有資料不足以解釋問題時。

**流程位置：** 與實作平行進行，而不是發布前才補上。

### 13. `doubt-driven-development`

**技能名稱：** 基於疑慮的開發

**概述：** 將每個非平凡決策提取為可審查單元，由新的對抗性上下文檢查假設、失敗模式與證據，再把發現調和回主工作流。

**何時使用：** 生產、安全、認證、不可逆操作、不熟悉程式碼或高風險遷移等正確性優先的工作。

**流程位置：** 在設計與實作期間持續插入，不是單一終點關卡。

## 第五階段：測試、除錯與驗證

### 14. `test-driven-development`

**技能名稱：** 測試驅動開發

**概述：** 使用 RED、GREEN、REFACTOR 循環，先以失敗測試定義行為，再用最小實作通過測試，最後在保持綠燈下重構。

**何時使用：** 實作邏輯、修復錯誤、改變行為，或需要可重現證據證明程式碼有效時。

**流程位置：** 與每個增量切片同步，而不是全部實作完成後才補測試。

### 15. `browser-testing-with-devtools`

**技能名稱：** 使用 DevTools 進行瀏覽器測試

**概述：** 透過真實瀏覽器檢查 DOM、控制台、網路、效能與視覺結果，並把瀏覽器內容視為不可信資料。

**何時使用：** 建構或除錯任何瀏覽器中的 UI、網路流程、效能或視覺行為時。

**流程位置：** 作為 `test-driven-development` 的瀏覽器運行時驗證分支。

### 16. `debugging-and-error-recovery`

**技能名稱：** 除錯與錯誤恢復

**概述：** 依序重現、定位、縮減、修復根本原因、防範重現，再執行端到端驗證；避免猜測式修改。

**何時使用：** 測試失敗、建置錯誤、運行時異常、回歸，或任何行為不符合預期時。

**流程位置：** 發現錯誤時暫停原流程，修復並補上防回歸證據後再返回主流程。

## 第六階段：審查與強化

### 17. `code-review-and-quality`

**技能名稱：** 程式碼審查與品質

**概述：** 從正確性、可讀性、架構、安全性與效能五個軸向審查變更，分類發現並以驗證結果作出合併判斷。

**何時使用：** 合併任何非瑣碎變更前，或被要求審查差異、PR、人類或代理產生的程式碼時。

**流程位置：** 測試通過後、合併前的主要品質門檻。

### 18. `security-and-hardening`

**技能名稱：** 安全與加固

**概述：** 以威脅模型為先，檢查注入、XSS、存取控制、驗證、會話、上傳、SSRF、速率限制、祕密、供應鏈、個資與 AI 功能風險。

**何時使用：** 處理不可信輸入、登入、使用者資料、外部整合、依賴漏洞或隱私合規時。

**流程位置：** 高風險設計時提早使用，並在程式碼審查階段再次執行。

### 19. `performance-optimization`

**技能名稱：** 效能優化

**概述：** 先建立基準與量測，再定位真正瓶頸、做最小修復、比較前後結果，無效則回滾，最後建立防回歸檢查。

**何時使用：** 有明確效能需求、Core Web Vitals 問題、負載時間回歸、N+1 查詢，或 profiler 已顯示瓶頸時。

**流程位置：** 不預先猜測優化；在測量證據出現後，通常接在審查或生產觀測之後。

### 20. `code-simplification`

**技能名稱：** 程式碼簡化

**概述：** 在精確保留外部行為的前提下，移除不必要的抽象、巢狀結構、重複與認知負擔，並遵循專案慣例。

**何時使用：** 程式碼已能運作但過度複雜，或審查發現可讀性與維護成本問題時。

**流程位置：** 實作與測試完成後、最終合併前；每次簡化後重新驗證行為。

## 第七階段：整合、文件與發布

### 21. `git-workflow-and-versioning`

**技能名稱：** Git 工作流程與版本控制

**概述：** 管理分支、worktree、原子提交、描述性提交訊息、衝突處理、語義版本、標籤與變更日誌。

**何時使用：** 任何程式碼變更、提交、分支、PR、版本發布或並行工作場景。

**流程位置：** 從實作開始便持續使用；發布前確認歷史乾淨且版本資訊正確。

### 22. `ci-cd-and-automation`

**技能名稱：** CI/CD 與自動化

**概述：** 建立自動品質門檻、測試管線、預覽環境、功能旗標、分階段部署、回滾與依賴更新自動化。

**何時使用：** 設定或修改建置、測試、部署管線，或手動品質檢查需要自動化時。

**流程位置：** 在團隊依賴該流程前建立；合併與發布前由 CI 執行最終門檻。

### 23. `documentation-and-adrs`

**技能名稱：** 文件與 ADR

**概述：** 記錄架構決策的背景、選項、決策與後果，並維護 API 文件、README、行內說明與變更日誌。

**何時使用：** 產生重要設計決策、修改公開 API、發布功能，或未來維護者需要理解「為什麼」時。

**流程位置：** 關鍵決策產生時立即記錄，發布前確認使用者與維護文件已更新。

### 24. `deprecation-and-migration`

**技能名稱：** 棄用與遷移

**概述：** 建立替代方案、公告棄用、逐步遷移呼叫者與資料，最後在有證據確認無使用者後移除舊系統；包含 strangler、adapter、feature flag 與 expand/contract 模式。

**何時使用：** 移除舊 API、功能、程式碼或資料庫欄位，或把使用者遷移到新實作時。

**流程位置：** 只有涉及替換舊系統時使用；通常橫跨多次發布，而非單一 PR 完成。

### 25. `shipping-and-launch`

**技能名稱：** 發布與部署

**概述：** 使用發布前檢查表、功能旗標、分階段發布、監控、錯誤預算與可操作的回滾計畫安全上線。

**何時使用：** 準備部署到生產環境、正式發布功能，或需要 go/no-go 判斷與回滾策略時。

**流程位置：** 生命週期最後的發布門檻；發布後仍須執行監控與驗證。

## 常見任務的技能流程

### 新功能

```text
using-agent-skills
→ interview-me（需求不清楚時）
→ idea-refine（方向未收斂時）
→ spec-driven-development
→ constraint-driven-development
→ planning-and-task-breakdown
→ context-engineering
→ source-driven-development
→ incremental-implementation
→ test-driven-development
→ code-review-and-quality
→ code-simplification
→ git-workflow-and-versioning
→ documentation-and-adrs
→ shipping-and-launch
```

依功能類型插入 `frontend-ui-engineering`、`api-and-interface-design`、`security-and-hardening`、`performance-optimization` 與 `observability-and-instrumentation`。

### 錯誤修復

```text
using-agent-skills
→ debugging-and-error-recovery
→ test-driven-development
→ code-review-and-quality
→ git-workflow-and-versioning
```

### UI 功能

```text
spec-driven-development
→ frontend-ui-engineering
→ incremental-implementation
→ test-driven-development
→ browser-testing-with-devtools
→ code-review-and-quality
→ shipping-and-launch
```

### API 變更

```text
spec-driven-development
→ api-and-interface-design
→ source-driven-development
→ test-driven-development
→ security-and-hardening
→ code-review-and-quality
→ documentation-and-adrs
→ shipping-and-launch
```

若會破壞既有呼叫者，在發布前加入 `deprecation-and-migration`。

### 安全性工作

```text
using-agent-skills
→ context-engineering
→ security-and-hardening
→ doubt-driven-development
→ test-driven-development
→ code-review-and-quality
→ observability-and-instrumentation
→ shipping-and-launch
```

### 效能問題

```text
using-agent-skills
→ observability-and-instrumentation
→ performance-optimization
→ incremental-implementation
→ test-driven-development
→ code-review-and-quality
→ shipping-and-launch
```

### 棄用與遷移

```text
spec-driven-development
→ api-and-interface-design
→ deprecation-and-migration
→ incremental-implementation
→ test-driven-development
→ observability-and-instrumentation
→ documentation-and-adrs
→ shipping-and-launch
```

## 選擇原則

1. 每次先用 `using-agent-skills` 判斷階段與風險。
2. 需求不清楚時，不要直接進入規格或實作。
3. 非瑣碎工作沒有規格時，從 `spec-driven-development` 開始。
4. 實作以 `incremental-implementation` 為主幹，測試與驗證跟著每個切片進行。
5. UI、API、安全、效能、瀏覽器與遷移技能都是條件式分支。
6. `doubt-driven-development` 用於高風險決策，不必套用到每個微小選擇。
7. 驗證未通過就不能進入發布階段。
8. 文件、遙測、Git 與 CI 是生命週期的一部分，不是最後補上的附加工作。

## 完成條件

- 已選擇與任務階段相符的技能。
- 每個被選用技能的步驟與驗證均已完成。
- 測試、建置、運行時或量測證據支持完成宣告。
- 安全、效能、無障礙性與專案約束沒有被靜默降低。
- 文件、提交歷史、部署與回滾計畫符合變更風險。
