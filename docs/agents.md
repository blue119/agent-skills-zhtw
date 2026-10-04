# 代理人角色

專門扮演單一角色、單一視角的角色。每個角色都是作為系統提示被您的工具（Claude Code、Cursor、Copilot 等）消費的 Markdown 文件。

| 角色 | 職責 | 最適合 |
|---------|------|----------|
| [code-reviewer](../agents/code-reviewer.md) | 資深工程師 | 合併前五軸審查 |
| [security-auditor](../agents/security-auditor.md) | 安全工程師 | 漏洞偵測、OWASP 風格審查 |
| [test-engineer](../agents/test-engineer.md) | QA 工程師 | 測試策略、覆蓋率分析、Prove-It 模式 |
| [web-performance-auditor](../agents/web-performance-auditor.md) | 網頁效能工程師 | 核心網頁指標審查、載入/渲染/網路分析 |

## 角色與技能及指令的關係

三層結構，每一層都有不同的職責：

| 層級 | 是什麼 | 範例 | 組合成角色 |
|-------|-----------|---------|------------------|
| **技能 (Skill)** | 一個具有步驟和退出標準的工作流程 | `code-review-and-quality` | 執行方式 — 在角色或指令內部調用 |
| **角色 (Persona)** | 一個具有視角和輸出格式的角色 | `code-reviewer` | 執行者 — 採納一個觀點，產生一份報告 |
| **指令 (Command)** | 用戶可見的入口點 | `/review`, `/ship` | 時間點 — 組合成角色和技能 |

用戶（或斜線指令）是協調者。**角色不會呼叫其他角色。**技能是角色工作流程內部的必要步驟。

## 使用時機

### 直接角色呼叫
當您希望對當前的變更獲得單一視角，且用戶在循環中時，請選擇此選項。

- "審查這個 PR" → 直接呼叫 `code-reviewer`
- " `auth.ts` 中是否有安全問題？" → 直接呼叫 `security-auditor`
- "結帳流程缺少哪些測試？" → 直接呼叫 `test-engineer`
- "審查產品頁面的核心網頁指標" → 直接呼叫 `web-performance-auditor`

### 斜線指令（背後單一角色）
當存在一個可以重複的流程，否則您每次都需要重新解釋時，請選擇此選項。

- `/review` → 將 `code-reviewer` 與專案的審查技能包裝起來
- `/test` → 將 `test-engineer` 與 TDD 技能包裝起來
- `/webperf` → 將 `web-performance-auditor` 包裝起來，用於網頁應用程式的效能相關審查

### 斜線指令（協調者 — 擴散）
僅在可以**獨立**進行並行調查並產生單一代理人後進行合併的報告時，請選擇此選項。

- `/ship` → 並行擴散到 `code-reviewer` + `security-auditor` + `test-engineer`，然後將它們的報告綜合為一個通過/不通過的決定

這是本儲存庫所支持的唯一協調模式。請參閱 [references/orchestration-patterns.md](../references/orchestration-patterns.md) 以獲取完整的模式目錄和反模式。

## 決策矩陣

```
工作是單一實體上的單一視角嗎？
├── 是 → 直接角色呼叫
└── 否 → 子任務是否獨立（沒有共享的可變狀態，沒有順序）？
         ├── 是 → 帶有並行擴散的斜線指令（例如 `/ship`）
         └── 否 → 由用戶執行的連續斜線指令（`/spec` → `/plan` → `/build` → `/test` → `/review`）
```

## 實例：有效的協調

`/ship` 是本儲存庫中的標準擴散協調器：

```
/ship
  ├── (並行) code-reviewer    → 審查報告
  ├── (並行) security-auditor → 審查報告
  └── (並行) test-engineer    → 覆蓋率報告
                  ↓
        合併階段（主代理人）
                  ↓
        通過/不通過決定 + 回滾計劃
```

這樣有效的原因：
- 每個子代理人都在相同的差異上操作，但產生**不同的視角**
- 它們之間沒有相互依賴 → 真正的並行，實際的時鐘節省
- 每個代理人在一個新的上下文視窗中運行 → 主要會話保持不混亂
- 合併步驟很小，受益於完整的上下文，因此保留在主代理人中

## 實例：無效的協調（請勿建立此類）

一個職責是「決定呼叫哪個其他角色的」`meta-orchestrator` 角色：

```
/work-on-pr → meta-orchestrator
                  ↓（決定「這需要審查」）
              code-reviewer
                  ↓（回傳）
              meta-orchestrator（轉述結果）
                  ↓
              用戶
```

這樣失敗的原因：
- 純粹的路由層，沒有領域價值
- 增加了兩個轉述步驟 → 資訊丟失 + 2 倍的 Token 成本
- 用戶已經知道他們想要審查；讓他們直接呼叫 `/review`
- 重複了斜線指令和 `AGENTS.md` 中的意圖映射已經完成的工作

## 角色規則

1. 一個角色是一個單一的角色和單一的輸出格式。如果您發現自己正在添加第二個角色，請建立第二個角色。
2. **角色不會呼叫其他角色。** 組合成是斜線指令或用戶的工作。在 Claude Code 中這也是一個硬平台約束——「子代理人不能生成其他子代理人」——因此此規則對您來說是強制執行的。
3. 一個角色可以呼叫技能（執行方式）。
4. 每個角色文件結尾都包含一個「Composition」區塊，說明其適當的位置。（當使用 `/ Invoke` 時直接呼叫；否則不要從其他角色呼叫）。

## Claude Code 互操作性

本儲存庫中的角色被設計為可以無修改地作為 Claude Code 的子代理人和代理團隊的隊友：

- **作為子代理人：** 啟用此插件時自動發現（無需路徑配置）。使用 Agent 工具並設定 `subagent_type: code-reviewer`（或 `security-auditor`、`test-engineer`）。`/ship` 是標準範例。
- **作為代理團隊隊友**（實驗性，需要 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`）：在生成隊友時引用相同的角色名稱。該角色的內容會**附加到**隊友的系統提示中作為額外的指令（而不是替換），因此您的角色文本會疊加在領導者安裝的團隊協調指令之上（SendMessage、task-list 工具等）。

子代理人只向主代理人報告結果。代理團隊允許隊友直接互相發訊息。當報告足夠時，請使用子代理人；當子代理人需要互相質疑彼此的發現時（例如競爭假設除錯），請使用代理團隊。請參閱 [references/orchestration-patterns.md](../references/orchestration-patterns.md) 以獲取完整的映射。

插件代理人不支持 `hooks`、`mcpServers` 或 `permissionMode` 的前置元數據——這些欄位會被靜默忽略。在在此處編寫新角色時，請避免依賴它們。

## 添加新角色

1. 在 `agents/<role>.md` 中創建文件，使用現有角色的相同前置元數據格式。
2. 定義角色、範圍、輸出格式和規則。
3. 在底部添加一個 **Composition** 區塊（當使用 `/ Invoke` 時直接呼叫；否則不要從其他角色呼叫）。
4. 將該角色添加到此文件的頂部表格中。
5. 如果該角色啟用新的協調模式，請在 `references/orchestration-patterns.md` 中記錄它，而不是在角色文件中發明該模式。
