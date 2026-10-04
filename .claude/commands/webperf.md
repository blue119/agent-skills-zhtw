---
description: 透過 web-performance-auditor 人設執行網頁效能審核
---

`/webperf` 專門針對網頁應用程式。請勿將其用於實用函式庫、CLI 或僅限伺服器端的程式碼，這些程式碼沒有瀏覽器介面輸出。

## 確定模式

**深度模式 (Deep mode)** — 當以下任何一項可用時啟用：
- Lighthouse JSON 報告檔案 (例如 `npx lighthouse <url> --output json --output-path ./report.json`，或從 Chrome DevTools MCP CLI 獲取的 `npx -p chrome-devtools-mcp chrome-devtools lighthouse_audit --output-format=json`)
- PageSpeed Insights JSON 回應 (包含 Lighthouse + CrUX)
- CrUX API 回應 (需要 `CRUX_API_KEY` 或 `GOOGLE_API_KEY`)
- DevTools 效能追蹤 (performance trace)
- 一個即時 URL 以及在框架中配置的 `chrome-devtools` MCP 伺服器 (代理可以透過 `lighthouse_audit` 和 `performance_*` 工具直接擷取指標)
- 在本地呼叫的 Chrome DevTools MCP CLI (透過 `npx -p chrome-devtools-mcp chrome-devtools <tool>` 或在執行 `npm i -g chrome-devtools-mcp` 後) — 使用者執行類似 `chrome-devtools lighthouse_audit --output-format=json` 的指令並將 JSON 輸出傳遞給代理

**快速模式 (Quick mode)** — 當以上任何一項不可用時預設啟用。代理會掃描原始碼以尋找結構性反模式，並將每一次發現標記為 `潛在影響 (potential impact)`。

## 執行審核

啟動 `web-performance-auditor` 子代理。明確傳遞以下資訊：

- 待審查的檔案、元件或差異 (diff)
- 任何工件路徑 (Lighthouse JSON、PSI JSON、CrUX 回應、追蹤) 或貼上的 JSON 內容
- 當已知時的目標 URL 或頁面名稱
- 預期的模式 (快速或深度)，以便代理在深度模式被預期時呈現缺失的輸入

子代理會回傳一個評分表 (僅包含來源值)、一份排名的發現清單、正面觀察結果和主動建議。

## 輸出

將完整的審核報告回傳給使用者。不需要合成或合併步驟 — 這是一個單一人設指令。
