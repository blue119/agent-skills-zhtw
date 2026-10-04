# sdd-cache hook

跨會話引用快取，用於 [`source-driven-development`](../skills/source-driven-development/SKILL.md)。在不削弱技能的「驗證當前文件」保證下，跳過冗餘的 `WebFetch` 呼叫。

## 原因

`source-driven-development` 會為每一個框架特定的決策獲取官方文件。在不同會話中對同一個專案進行工作意味著重複獲取相同的頁面。將內容作為本地記憶體快取會與該技能相矛盾——文件會改變，過時的快取會掩蓋這一點。

此鉤子將獲取的內容快取在磁碟上，但會透過 HTTP `If-None-Match` / `If-Modified-Since` **在每次重用時與來源伺服器重新驗證**。只有當伺服器回應 `304 Not Modified` 時，內容才會從快取中提供，這是一種最新的驗證——而不是記憶體讀取。

## 設定

1. 將鉤子新增到 `.claude/settings.json`（或個人使用 `.claude/settings.local.json`）：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "WebFetch",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"${CLAUDE_PROJECT_DIR}/hooks/sdd-cache-pre.sh\"",
            "timeout": 10
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "WebFetch",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"${CLAUDE_PROJECT_DIR}/hooks/sdd-cache-post.sh\"",
            "async": true,
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

   `${CLAUDE_PROJECT_DIR}` 解析為您啟動 Claude Code 的目錄。上述片段在鉤子位於同一專案內部時有效。如果您在其他地方安裝了 `agent-skills`（例如作為 `~/agent-skills` 下的共用插件），請將 `${CLAUDE_PROJECT_DIR}/hooks/...` 替換為每個腳本的絕對路徑。

2. 確保 `.claude/sdd-cache/` 包含在您的 `.gitignore` 中（已包含在此儲存庫中）。

3. 如往常使用 `/source-driven-development`（或該技能）。技能或代理的工作流程沒有任何更改——快取是透明的。

## 思維模型

以 URL 為鍵的 HTTP 資源快取。新鮮度透過 `ETag` / `Last-Modified` 委託給來源，沒有 TTL，鍵中不包含提示。

儲存的內容不是原始 HTML——`WebFetch` 會使用呼叫者的提示對每個回應進行模型後處理，因此我們快取的內容是某個代理對頁面的閱讀。鍵僅保留 URL，以便在會話之間重複使用；原始提示作為元數據保留，並在命中訊息中顯示，以便下一個代理可以判斷先前的閱讀是否適用。

## 工作方式

每個 URL 一個快取條目，以 JSON 形式儲存在 `.claude/sdd-cache/<sha>.json` 中：

| 事件 | 行動 |
|---|---|
| `PreToolUse WebFetch` | 如果存在條目，則發送帶有 `If-None-Match` / `If-Modified-Since` 的 `HEAD` 請求。在 `304` 時，會阻止獲取並透過標準錯誤輸出將快取內容返回給代理，並將原始提示作為元數據顯示。否則允許獲取。 |
| `PostToolUse WebFetch` | 捕獲回應，發出 `HEAD` 請求以記錄當前的 `ETag` / `Last-Modified`，並儲存 `{url, prompt, etag, last_modified, content, fetched_at}`。 |

**新鮮度規則：**

- 僅在來源確認 `304 Not Modified` 時才提供條目。
- 沒有 `ETag` 或 `Last-Modified` 標頭的條目永遠不會被快取——沒有驗證器，鉤子無法稍後驗證新鮮度，而快取意味著信任記憶體。
- 快取鍵是 `sha256(url)`。使用不同提示詢問相同的 URL 會命中相同的條目；快取內容反映了第一次獲取時使用的提示，該提示會與命中一起顯示，以便代理可以決定是否手動重新使用或重新獲取。

**代理看到的內容：**

- 快取命中：`WebFetch` 透過退出代碼 2 被阻止。Claude Code 將鉤子的標準錯誤輸出傳遞回代理作為工具錯誤——這是快取命中的預期信號，而不是失敗。該有效載荷會以 `[sdd-cache] Cache hit for <url>` 開頭，並在 `----- BEGIN CACHED CONTENT -----` / `----- END CACHED CONTENT -----` 標記之間包圍快取內容，以便代理可以像 `WebFetch` 剛返回它一樣使用它。
- 快取未命中或過期：`WebFetch` 正常運行；結果將為下次儲存。

技能本身保持不變。它繼續遵循 `DETECT → FETCH → IMPLEMENT → CITE`。鉤子僅改變在 `FETCH` 運行時的底層情況。

## 本地測試

### 1. 直接測試腳本

```bash
# 模擬 PostToolUse 有效載荷：快取一頁
echo '{
  "tool_input": {
    "url": "https://react.dev/reference/react/useActionState",
    "prompt": "extract the signature"
  },
  "tool_response": "useActionState(action, initialState) returns [state, formAction, isPending]"
}' | bash hooks/sdd-cache-post.sh

# 檢查儲存的條目
ls .claude/sdd-cache/
cat .claude/sdd-cache/*.json | jq .

# 模擬對同一 URL + 提示的下一個 PreToolUse
echo '{
  "tool_input": {
    "url": "https://react.dev/reference/react/useActionState",
    "prompt": "extract the signature"
  }
}' | bash hooks/sdd-cache-pre.sh
echo "exit=$?"
```

預期結果：

- 第一個命令在 `.claude/sdd-cache/` 下創建一個文件（僅在伺服器返回了 `ETag` 或 `Last-Modified` 時）。
- 第二個命令在來源回覆 `304` 時，在標準錯誤輸出中返回快取內容並以退出代碼 `2` 退出，否則靜默退出 `0`。

### 2. 在實際會話中進行端到端測試

1. 如上所示，在 `.claude/settings.local.json` 中註冊鉤子。
2. 在此儲存庫中啟動一個 Claude Code 會話。
3. 要求代理獲取一個文件（例如：「獲取 `https://react.dev/reference/react/useActionState` 並總結」）。
4. 驗證 `.claude/sdd-cache/` 下出現了一個文件。
5. 要求代理再次獲取相同的頁面並使用相同的提示。
6. 驗證第二次 `WebFetch` 被阻止，並且返回了快取內容（在會話記錄中顯示為帶有 `[sdd-cache]` 前綴的工具錯誤）。

### 3. 新鮮度驗證

要確認文件更改時快取會失效，請強制 `ETag` 不匹配。選擇一個特定的條目——一旦快取保存多於一個文件，`*.json` 就不安全：

```bash
# 選擇您要損壞的條目（替換為實際文件名）
ENTRY=.claude/sdd-cache/e49c9f378670cfbb1d7d871b6dee16d9.json

# 將其 ETag 更改為來源不會識別的值
jq '.etag = "W/\"stale-etag-forced\""' "$ENTRY" > "$ENTRY.tmp" && mv "$ENTRY.tmp" "$ENTRY"

# 下一個 PreToolUse 應該會未命中（伺服器返回 200，而不是 304）
echo '{"tool_input":{"url":"...", "prompt":"..."}}' | bash hooks/sdd-cache-pre.sh
echo "exit=$?"   # 預期 0（獲取允許通過）
```

### 4. 除錯

當啟用除錯模式時，兩個鉤子會將時間戳事件寫入 `.claude/sdd-cache/.debug.log`：

```bash
# 選項 A：環境變數（每個會話）
SDD_CACHE_DEBUG=1 claude

# 選項 B：哨兵文件（持久化）
mkdir -p .claude/sdd-cache && touch .claude/sdd-cache/.debug
# …禁用時：rm .claude/sdd-cache/.debug
```

日誌記錄了 URL、檢測到的 `tool_response` 形狀、`HEAD` 狀態以及每次調用的命中或未命中的原因。當快取未命中看起來出乎意料時很有用（通常是：來源停止發出驗證器）。

## 已知限制

- **內容是提示形狀的。** 命中返回的是先前代理對頁面的閱讀，並顯示原始提示，以便當前代理可以決定是否適用。如果不是，請刪除 `.claude/sdd-cache/` 下的文件以強制重新獲取。
- **每次快取寫入都額外花費一個 `HEAD`。** Claude Code 不會暴露 `WebFetch` 已經接收到的回應標頭，因此後鉤子會重新查詢來源以捕獲 `ETag` / `Last-Modified`。每次未命中額外一次往返——這是保持其作為純鉤子而沒有核心更改的代價。
- **沒有 `ETag` 或 `Last-Modified` 的伺服器永遠不會被快取。** 大多數官方文件網站（react.dev, docs.djangoproject.com, developer.mozilla.org）會發出驗證器。那些沒有的網站總是會重新獲取。
- **行為不良的伺服器可能會提供錯誤的 `304`。** 這是一個需要診斷的伺服器錯誤，而不是需要防禦的快取不變性；我們不會用 TTL 來掩蓋它。如果發現過期條目，請刪除該條目。
- **快取是本地的且與專案相關的。** 沒有團隊範圍的共享快取。添加一個需要一個簽名內容可地址的儲存層，這超出了範圍。

## 要求

- `jq`
- `curl`
- `shasum` 或 `sha256sum`（自動檢測）
- Bash 3.2+
