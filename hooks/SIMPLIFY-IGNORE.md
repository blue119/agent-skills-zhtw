# simplify-ignore hook

針對 `/code-simplify` 的區塊級保護。標記不應被簡化的程式碼——模型將看不到它。

## 設定

1. 註記你想要保護的區塊：

```js
/* simplify-ignore-start: perf-critical */
// 手動展開的 XOR — 比迴圈快 3 倍
result[0] = buf[0] ^ key[0];
result[1] = buf[1] ^ key[1];
result[2] = buf[2] ^ key[2];
result[3] = buf[3] ^ key[3];
/* simplify-ignore-end */
```

2. 將掛鉤新增到 `.claude/settings.json`：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Read",
        "hooks": [{ "type": "command", "command": "bash \"${CLAUDE_PROJECT_DIR}/hooks/simplify-ignore.sh\"" }]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": "bash \"${CLAUDE_PROJECT_DIR}/hooks/simplify-ignore.sh\"" }]
      }
    ],
    "Stop": [
      {
        "hooks": [{ "type": "command", "command": "bash \"${CLAUDE_PROJECT_DIR}/hooks/simplify-ignore.sh\"" }]
      }
    ]
  }
}
```

3. 執行 `/code-simplify` — 受保護的區塊將變成 `/* BLOCK_de115a1d: perf-critical */` 的佔位符。模型在不看到受保護的實作的情況下，對周圍程式碼進行推理。

> **注意：** 掛鉤將臨時備份儲存在 `.claude/.simplify-ignore-cache/` 中。請確保此路徑在你的 `.gitignore` 中。

## 工作原理

一個腳本，三個掛鉤事件：

| 事件 | 行動 |
|---|---|
| `PreToolUse Read` | 備份檔案，將區塊替換為 `BLOCK_<hash>` 佔位符（原地操作） |
| `PostToolUse Edit\|Write` | 將佔位符展開回實際程式碼，儲存模型的更改，重新過濾 |
| `Stop` | 展開每個檔案上留下的佔位符，以便在 `Edit`/`Write` 之外所做的更改得以保留；如果沒有留下佔位符則回退到備份 |

每個區塊都會進行內容雜湊（透過 `shasum`/`sha1sum` 產生 8 個十六進位字元），因此即使模型複製或重新排序佔位符，往返過程也是明確的。快取是專案範圍的，以防止跨會話干擾。

## 註記語法

```js
/* simplify-ignore-start */           // 基本 — 隱藏區塊
/* simplify-ignore-start: reason */   // 附帶理由 — 出現在佔位符中
/* simplify-ignore-end */
```

任何註解風格都適用（`//`、`/*`、`#`、`<!--`）。支援檔案中的多個區塊和單行區塊。佔位符會保留原始的註解語法（例如，Python 中的 `# BLOCK_xxx`，HTML 中的 `<!-- BLOCK_xxx -->`）。

## 崩潰恢復

如果 Claude Code 在沒有觸發 `Stop` 掛鉤的情況下崩潰，磁碟上的檔案可能仍然有 `BLOCK_<hash>` 佔位符。手動還原方法如下：

```bash
echo '{}' | bash hooks/simplify-ignore.sh
```

備份儲存在你的專案目錄下的 `.claude/.simplify-ignore-cache/` 中。

## 已知限制

- **單行區塊會隱藏整行。** 如果 `simplify-ignore-start` 和 `simplify-ignore-end` 出現在與其他程式碼相同的行上，則整行會對模型隱藏，而不僅僅是註記的部分。請使用專門的行進行註記。
- **註解後綴偵測僅涵蓋 `*/` 和 `-->`。** 使用非標準註解結尾的模板引擎（如 ERB 的 `%>`，Blade 的 `--}}`）可能會產生不平衡的佔位符。請改用 `#` 或 `//` 風格的註解。
- **回退展開是漸進的，而非精確的。** 如果模型更改了佔位符的格式（例如更改了理由文字），掛鉤會嘗試更簡單的匹配：完整佔位符 → 前綴+雜湊+後綴 → 僅雜湊。僅雜湊的回退可能會留下外觀殘留物（例如多餘的 `:` 或理由文字）。發生此情況時會向標準錯誤輸出列印警告。
- **完全重寫會回退到備份。** 如果某個內容替換了過濾後的檔案的全部內容並移除了所有 `BLOCK_<hash>` 佔位符，掛鉤就無法知道受保護區塊的位置。在 `Stop` 時，它會還原備份並列印警告；重寫不會丟失，而是保留在快取中作為 `.claude/.simplify-ignore-cache/<id>.recovered`，警告會列印該路徑，你需要手動合併它。那些保持佔位符的更改——`Bash` 編輯、格式化器、外部編輯器——會保留在檔案本身中。
- **重新命名檔案會留下佔位符。** 如果模型透過 shell 命令重新命名或移動檔案，新檔案將保留 `BLOCK_<hash>` 佔位符。在會話停止時，原始程式碼會儲存為 `<old-filename>.recovered`。你必須手動將還原的程式碼還原到新檔案中。

## 要求

- `jq`、`shasum` 或 `sha1sum`（自動偵測），Bash 3.2+
