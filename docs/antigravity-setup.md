# 使用 agent-skills 與 Antigravity CLI (agy)

`agent-skills` 套件可以作為 Antigravity CLI (`agy`) 的原生外掛程式安裝，讓代理程式可以存取結構化的工作流程和角色。

## 設定

### 選項 1：原生外掛程式安裝（推薦）

Antigravity CLI 具有一個一等公民的[外掛程式系統](https://www.agy.dev/docs/plugins/)，用於註冊技能和代理程式。該儲存庫也包含舊的命令 TOML 檔案，但受影響的 `agy` 版本不暴露它們轉換後的包裝器；請參閱[生命週期工作流程與命令相容性](#lifecycle-workflows-and-command-compatibility)。

**從遠端儲存庫安裝：**

```bash
agy plugin install https://github.com/addyosmani/agent-skills.git
```

**從本地克隆安裝：**

1. 克隆儲存庫：
   ```bash
   git clone https://github.com/addyosmani/agent-skills.git
   ```
2. 使用 `agy` 安裝外掛程式：
   ```bash
   agy plugin install /path/to/agent-skills
   ```

這將驗證外掛程式並將其安裝到您的全域 Antigravity 設定目錄 (`~/.gemini/config/plugins/agent-skills/`)。

> **注意：** 在目前的 `agy` 版本中，外掛程式會位於 `~/.gemini/config/plugins/` 下，而不是舊版本使用的 `~/.gemini/antigravity-cli/plugins/` 路徑。如果您在舊路徑中看不到外掛程式，請先檢查 `~/.gemini/config/plugins/agent-skills/`。

### 選項 2：從 Gemini CLI 匯入

如果您已在舊的 Gemini CLI 安裝中安裝了 `agent-skills`，您可以直接匯入它：
```bash
agy plugin import gemini
```

安裝後，驗證活躍的外掛程式：
```bash
agy plugin list
```

---

## 生命週期工作流程與命令相容性

Antigravity 的[遷移工具](https://www.agy.dev/docs/cli/gcli-migration/) 將 `commands/*.toml` 中的 9 個舊定義報告為「已轉換為技能」。在受影響的 `agy` 1.1.x 版本中，驗證成功，但轉換後的包裝器並不會出現在斜線命令或技能目錄中。因此，成功的 `agy plugin validate` 確認檔案格式正確，而不是確認 `/build` 和其他簡短包裝器是否可用。這在[agent-skills #445](https://github.com/addyosmani/agent-skills/issues/445)以及上游的[antigravity-cli #788](https://github.com/google-antigravity/antigravity-cli/issues/788)中有所追蹤。

在該匯入限制適用時，直接使用原生外掛程式技能：

| 預期包裝器 | 直接 Antigravity 呼叫 | 備註 |
| :--- | :--- | :--- |
| `/spec` | `/agent-skills:spec-driven-development` | 在程式碼前寫出結構化的規格 |
| `/constraints` | `/agent-skills:constraint-driven-development` | 定義並強制執行專案的品質標準 |
| `/planning` | `/agent-skills:planning-and-task-breakdown` | Antigravity 的內建 `/planning` 命令是單獨的規劃模式控制 |
| `/build` | `/agent-skills:incremental-implementation` | 也呼叫 `/agent-skills:test-driven-development`；無法使用僅包裝器的 `/build auto` 調度 |
| `/test` | `/agent-skills:test-driven-development` | 執行紅綠重構工作流程 |
| `/review` | `/agent-skills:code-review-and-quality` | 執行五軸審查工作流程 |
| `/code-simplify` | `/agent-skills:code-simplification` | 在不改變行為的情況下簡化 |
| `/ship` | `/agent-skills:shipping-and-launch` | 包裝器的自動角色擴散不可用；請單獨呼叫專門代理程式 |
| `/webperf` | 從 `/agents` 中選取 `web-performance-auditor` | 此工作流程是一個角色，而不是一個技能 |

不要在 TOML 檔案中添加 YAML 前置內容作為變通方法。Gemini CLI 使用嚴格的解析器讀取並行的 TOML 命令格式，而 `---` 前置內容在不改變 Antigravity 的轉換行為的情況下會使這些檔案無效。

---

## 技能與發現

Antigravity 會自動在外掛程式的 `skills/` 目錄中發現技能。
* Antigravity 會即時對使用者任務和意圖進行匹配，以找到相關技能。
* 如果任務與某個技能匹配，代理程式將載入該技能並在執行前提示您進行許可。

---

## 驗證與確認

要驗證您的本地外掛程式是否結構正確並包含所有技能，請執行：
```bash
agy plugin validate /path/to/agent-skills
```

然後開始一個新的會話，輸入 `/agent-skills:` 以檢查命名空間技能目錄。如果驗證器報告命令已被轉換但 `/build` 返回 `No matches`，請使用上述直接呼叫；重新安裝或在 TOML 中添加 `name` 欄位並不能解決已知的匯入限制。

---

## 工作原理

### 1. 即時技能啟用
Antigravity CLI 會自動發現安裝的外掛程式 `skills/` 目錄中的 `SKILL.md` 檔案。使用每個技能前置內容中的觸發描述，當代理程式偵測到匹配的開發者意圖時，它將動態啟用適當的工作流程。

例如，當您要求代理程式：
- **設計一個新系統** $\rightarrow$ 它將建議/啟用 `spec-driven-development`。
- **實作一個功能** $\rightarrow$ 它將啟用 `incremental-implementation` 和 `test-driven-development`。
- **修復一個錯誤** $\rightarrow$ 它將啟用 `debugging-and-error-recovery`。

### 2. 專門代理角色
該外掛程式從 `agents/` 目錄中註冊可重複使用的子代理定義：
- `code-reviewer.md`
- `security-auditor.md`
- `test-engineer.md`

您可以在會話中直接呼叫這些角色，或在委派任務時使用子代理。

---

## 設定與自訂

### 專案特定強制執行 (`AGENTS.md`)
要強制執行嚴格的技能合規性（例如，在編寫程式碼前要求提供規格或計畫），請將 `AGENTS.md` 複製或連結到工作區的根目錄。Antigravity CLI 會讀取此檔案，以使代理程式的行為和規劃階段與您的團隊慣例保持一致。

### 沙盒模式
如果您想以有限的終端機權限運行技能或腳本（在運行第三方驗證測試時的安全性），請使用以下方式啟動 CLI：

```bash
agy --sandbox
```

---

## 使用提示

1. **保持外掛程式更新：** 您可以使用以下命令更新 CLI 或檢查較新的外掛程式版本：
   ```bash
   agy update
   ```
2. **執行前審查：** 當代理程式使用這些技能執行複雜的重構任務時，請使用 `Ctrl+r` 進入**工件審查**畫面，在提交程式碼前審查、編輯或批准程式碼。
3. **控制權限：** 您只能在信任的本地專案中使用 `--dangerously-skip-permissions` 標誌，以繞過手動工具批准提示。
