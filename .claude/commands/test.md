---
description: 執行 TDD 工作流程 — 編寫失敗的測試，實作，驗證。對於錯誤，使用 Prove-It 模式。
---

呼叫代理技能：test-driven-development。

對於新功能：
1. 編寫描述預期行為的測試（它們應該會失敗）
2. 實作代碼使它們通過
3. 在保持測試通過的同時重構

對於錯誤修復（Prove-It 模式）：
1. 編寫一個重現錯誤的測試（必須失敗）
2. 確認測試失敗
3. 實作修復
4. 確認測試通過
5. 運行完整的測試套件以檢查迴歸

對於瀏覽器相關問題，也呼叫代理技能：browser-testing-with-devtools 以使用 Chrome DevTools MCP 進行驗證。
