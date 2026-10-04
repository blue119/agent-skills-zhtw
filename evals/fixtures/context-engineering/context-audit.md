# 會話情境審核

該儲存庫是一個 TypeScript 服務。目前的代理會話載入整個 `docs/archive/` 目錄（1,800 個檔案）、生成的 API 輸出、六個舊的事件記錄，以及啟動時的所有 ADR。它不會載入活躍的 `CONTRIBUTING.md` 或 `docs/current-architecture.md`。

觀察到的失敗情況：

- 回應建議使用 JavaScript，儘管新來源必須是 TypeScript。
- 測試是使用 Jest 提出的，但此專案使用 Vitest。
- 代理會反覆忘記資料庫存取應該在儲存庫中。
- 經過長時間的工具追蹤後，答案變得籠統。

當前任務：向現有的 HTTP 處理器添加驗證。
