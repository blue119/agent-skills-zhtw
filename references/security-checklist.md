# 安全性檢查清單

網頁應用程式安全快速參考。請與 `security-and-hardening` 技能一起使用。

## 目錄

- [威脅建模 (從這裡開始)](#threat-modeling-start-here)
- [提交前檢查](#pre-commit-checks)
- [身份驗證](#authentication)
- [授權](#authorization)
- [輸入驗證](#input-validation)
- [安全標頭](#security-headers)
- [CORS 配置](#cors-configuration)
- [資料保護](#data-protection)
- [依賴項安全](#dependency-security)
- [AI / LLM 安全性](#ai--llm-security)
- [錯誤處理](#error-handling)
- [OWASP Top 10 快速參考](#owasp-top-10-quick-reference)
- [LLM 的 OWASP Top 10 快速參考](#owasp-top-10-for-llms-quick-reference)

## 威脅建模 (從這裡開始)

在尋求控制措施之前，花五分鐘像攻擊者一樣思考：

- [ ] 信任邊界已繪製 (請求、上傳、Webhook、第三方 API、LLM 輸出，以及由您無法控制的程序寫入的本地值)
- [ ] 資產已命名 (憑證、個人身份資訊 (PII)、支付資料、管理操作、金錢流動)
- [ ] 針對每個邊界運行 STRIDE (偽造、篡改、否認、資訊洩露、拒絕服務 (DoS)、提升權限)
- [ ] 在使用案例旁邊寫下濫用案例 ("我如何能濫用這個？")

## 提交前檢查

- [ ] 代碼中沒有密碼 (`git diff --cached | grep -i "password\|secret\|api_key\|token"`)
- [ ] `.gitignore` 涵蓋：`.env`, `.env.local`, `*.pem`, `*.key`
- [ ] `.env.example` 使用佔位符值 (非真實密碼)

## 身份驗證

- [ ] 密碼使用 bcrypt (≥12 輪), scrypt 或 argon2 進行雜湊
- [ ] 會話 Cookie：`httpOnly`, `secure`, `sameSite: 'lax'`
- [ ] 配置會話過期 (合理的 max-age)
- [ ] 登入端點的速率限制 (每 15 分鐘 ≤10 次嘗試)
- [ ] 密碼重設令牌：時間限制 (≤1 小時)，單次使用
- [ ] 重複失敗後帳戶鎖定 (可選，附帶通知)
- [ ] 對敏感操作支援 MFA (可選但建議)

## 授權

- [ ] 每個受保護的端點都檢查身份驗證
- [ ] 每個資源存取都檢查所有權/角色 (防止 IDOR)
- [ ] 管理端點需要管理員角色驗證
- [ ] API 金鑰範圍限制在最必要的權限
- [ ] JWT 令牌已驗證 (簽名、過期時間、發行者)

## 輸入驗證

- [ ] 所有使用者輸入在系統邊界進行驗證 (API 路由、表單處理器)
- [ ] 驗證使用允許清單 (而非拒絕清單)
- [ ] 字串長度受約束 (最小/最大)
- [ ] 數值範圍已驗證
- [ ] 電郵、URL 和日期格式使用適當的函式庫進行驗證
- [ ] 檔案上傳：類型受限制、大小限制、內容驗證
- [ ] SQL 查詢參數化 (無字串連接)
- [ ] HTML 輸出已編碼 (使用框架的自動轉義功能)
- [ ] 在重定向前驗證 URL (防止開放重定向)
- [ ] 伺服器端 URL 獲取允許清單；封鎖私有/保留 IP (防止 SSRF)
- [ ] 破壞性路徑操作 (刪除/移動/覆蓋)：解析符連結，允許清單的根目錄，最小深度，在呼叫前讀取所有權證據

### 破壞性路徑操作

針對由資料命名的目標進行封裝。先解析，然後決定——並將結果視為候選者，而非授權：

```typescript
import { realpath, readFile } from 'node:fs/promises';
import { resolve, relative, isAbsolute, join, sep } from 'node:path';

const ALLOWED_ROOTS = ['/var/lib/myapp/sessions']; // 允許清單，而非模式
const MIN_DEPTH = 1;                               // 這樣根目錄永遠不是目標

async function resolveDeletable(candidate: string, expectedOwner: string) {
  const target = await realpath(resolve(candidate)); // 在檢查之前解析符連結
  const inRoot = ALLOWED_ROOTS.some((root) => {
    const rel = relative(root, target);
    // 僅 '..' / '../' — 單純的 `startsWith('..')` 也會拒絕名為 `..cache` 的合法子項。
    if (rel === '' || rel === '..' || rel.startsWith(`..${sep}`) || isAbsolute(rel)) return false;
    return rel.split(sep).length >= MIN_DEPTH;
  });
  if (!inRoot) throw new Error(`refusing: outside allowed roots (${target})`);

  const owner = await readFile(join(target, '.owner'), 'utf8').catch(() => null);
  if (owner?.trim() !== expectedOwner) throw new Error(`refusing: unproven owner (${target})`);
  return target;
}
```

此程式碼**不**做的事，以及在複製片段時必須說明：

- **標記是自我證明。** 任何可以在根目錄內寫入的內容都可以寫入 `.owner`。`expectedOwner` 必須來自已驗證的狀態，並且標記需要在授權之前獲得完整性保護 (限制性所有權或 MAC)。
- **返回一個路徑會留下檢查/使用競態條件。** 當一個不受信任的程序可以在檢查和呼叫之間交換一個祖先，對一個無追蹤的描述符操作，或對根目錄以下的語義操作，或保證層級在整個期間是不可變的時。

## 安全標頭

```
Content-Security-Policy: default-src 'self'; script-src 'self'
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 0  (禁用，依賴 CSP)
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

## CORS 配置

```typescript
// 限制性 (推薦)
cors({
  origin: ['https://yourdomain.com', 'https://app.yourdomain.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
})

// 絕不應在生產環境中使用：
cors({ origin: '*' })  // 允許任何來源
```

## 資料保護

- [ ] 從 API 回應中排除敏感欄位 (`passwordHash`, `resetToken` 等)
- [ ] 敏感資料不記錄 (密碼、令牌、完整的信用卡號)
- [ ] 個人身份資訊 (PII) 在靜態時加密 (如果法規要求)
- [ ] 所有外部通訊都使用 HTTPS
- [ ] 資料庫備份加密
- [ ] 個人資料被分類，根據既定目的收集，並進行最小化
- [ ] 個人資料有保留期限和工作刪除路徑 (包括備份、快取、索引)
- [ ] 支援在需要時匯出/刪除 (資料主體)；第三方共享需要同意和資料處理協議

## 依賴項安全

首先確定**安裝邊界**。如果套件由父級 `workspaces` 聲明匹配，則使用該工作區根目錄；否則使用擁有其宣告和依賴圖的最近專案根目錄。在該邊界處，核實 `packageManager` (如果存在)、鎖定檔案和 CI 命令。如果它們不一致或存在競爭的管理器鎖定檔案，則停止。嵌套專案僅在它位於父工作區之外時才是獨立的；獨立的子專案可以合法地使用不同的管理器。

| 管理器/版本信號 | 凍結/不可變的 CI 安裝 | 已知審核 |
|---|---|---|
| npm (`package-lock.json` 或 `npm-shrinkwrap.json`) | `npm ci` | `npm audit` |
| pnpm | `pnpm install --frozen-lockfile` | `pnpm audit` |
| Yarn 2+ | `yarn install --immutable` | `yarn npm audit -A -R` |
| Yarn 1 | `yarn install --frozen-lockfile` | `yarn audit` |

對於未列出的管理器或版本，請查閱其官方文件；不要替換其他管理器的命令或較新的預設值。

### 安裝腳本閘道

絕不要在驗證過客戶端的預設值之前，先執行普通安裝來發現依賴項的生命週期腳本。

1. 使用依賴項腳本禁用，或使用有文件記錄的預設拒絕策略加上失敗關閉強制執行。
2. 在批准之前檢查確切的腳本來源和套件版本。
3. 在安裝邊界處記錄最窄的本機允許/拒絕策略並提交它。
4. 使用該策略執行乾淨的凍結/不可變安裝，並驗證所需的套件仍然可以建置。

**時間點快照：** 套件管理器預設值和命令名稱變化很快。在依賴於此之前，請將此矩陣與釘選客戶端的當前官方文件進行核實。

| 管理器版本 | 本機策略 |
|---|---|
| npm 沒有經過驗證的細粒度批准 | 使用 `npm ci --ignore-scripts` 進行初始化，或在需要專案範圍的封鎖時保持 `ignore-scripts=true`。在允許任何審核的依賴項腳本之前，保持腳本禁用或故意升級。 |
| npm 11.18.x (在 11.18.0 上驗證) | 未審核的依賴項腳本預設會發出警告。在正常安裝之前強制執行 `strict-allow-scripts=true`，然後從安裝邊界使用工作區無意識的 `npm install-scripts ls`；保持批准版本釘選並以名稱級別拒絕。 |
| npm 12.x (在 12.0.1 上驗證) | 未審核的依賴項腳本預設會被跳過；`strict-allow-scripts=true` 會在執行前使它們的存在導致安裝失敗。使用相同的 `npm install-scripts` 審核和批准流程。 |
| pnpm 11+ | 使用 `pnpm approve-builds` 並提交 `allowBuilds` 決策；`strictDepBuilds` 預設為 `true`，因此未審核的建置會失敗。 |
| pnpm 10.26–10.x | 明確配置 `allowBuilds`，或使用 `pnpm approve-builds` 配合舊的 `onlyBuiltDependencies` / `ignoredBuiltDependencies` 清單。設置 `strictDepBuilds: true`；其 v10 預設值是 `false`。 |
| pnpm 10.1–10.25 | `pnpm approve-builds` 記錄了舊清單；在受支援的地方啟用 `strictDepBuilds` (10.3+)。 |
| 較舊或未知的 pnpm | 使用 `pnpm install --frozen-lockfile --ignore-scripts` 進行初始化。除非釘選版本記錄了可強制執行的策略，否則保持腳本禁用。 |
| Yarn 4.14+ | 依賴項 postinstalls 預設禁用。僅授予必要的例外，使用頂層 `dependenciesMeta.<package>.built: true`。 |
| Yarn 2–4.13 | 在 `.yarnrc.yml` 中設置 `enableScripts: false`，然後僅授予必要的例外，使用頂層 `dependenciesMeta.<package>.built: true`；不要全局啟用腳本。 |
| Yarn 1 | 使用 `yarn install --ignore-scripts` 進行初始化；除非每個必要的例外都根據釘選客戶端的文檔中記錄的工作流程進行審核，否則保持腳本禁用。 |

權威檢查：[npm install-scripts](https://docs.npmjs.com/cli/v11/commands/npm-install-scripts/)、[安裝策略](https://docs.npmjs.com/cli/v11/commands/npm-install/)，以及 [CLI 版本](https://github.com/npm/cli/releases)；[pnpm approve-builds](https://pnpm.io/cli/approve-builds) 和 [建置設定](https://pnpm.io/settings#allowbuilds)；[Yarn 安全性](https://yarnpkg.com/features/security) 和 [manifest](https://yarnpkg.com/configuration/manifest#dependenciesMeta)。

**供應鏈衛生** (審核不捕獲新惡意套件)：
- [ ] 每個專案/工作區根目錄只提交一個權威鎖定檔案，並且 CI 永遠不會重寫它
- [ ] 關鍵/高風險發現會進行可到達性審查；延期有理由和審查日期
- [ ] 強制審核修復 (`npm audit fix --force` 或等效) 絕不會自動執行；修復差異和變更日誌會進行審查
- [ ] 驗證註冊表簽名/來源，如果管理器支援
- [ ] 依賴項生命週期腳本在首次執行前被阻止，並僅通過釘選管理器的原生策略批准
- [ ] 新依賴項會根據所有權、維護、發布年代、來源、傳遞圖和拼寫錯誤進行審查

## AI / LLM 安全性

對於任何呼叫 LLM 的功能 (聊天機器人、摘要器、代理、RAG)：

- [ ] 模型輸出視為不可信 — 絕不進入 `eval`/SQL/shell/`innerHTML`/檔案路徑
- [ ] 假設提示注入；權限在代碼中強制執行，而不是在系統提示中
- [ ] 密碼、跨租戶資料和完整的系統提示保持在上下文視窗之外
- [ ] 工具/代理權限範圍限制；破壞性或不可逆的操作需要確認
- [ ] 設定令牌、速率和遞迴/迴圈深度限制 (限制消耗)

## 錯誤處理

```typescript
// 生產環境：通用錯誤，無內部細節
res.status(500).json({
  error: { code: 'INTERNAL_ERROR', message: '發生了錯誤' }
});

// 絕不應在生產環境中：
res.status(500).json({
  error: err.message,
  stack: err.stack,         // 暴露內部細節
  query: err.sql,           // 暴露資料庫細節
});
```

## OWASP Top 10 快速參考

| # | 漏洞 | 防範 |
|---|---|---|
| 1 | 存取控制失效 (Broken Access Control) | 每個端點的身份驗證，所有權驗證 |
| 2 | 密碼學失敗 (Cryptographic Failures) | HTTPS，強雜湊，代碼中無密碼 |
| 3 | 注入 (Injection) | 參數化查詢，輸入驗證 |
| 4 | 不安全設計 (Insecure Design) | 威脅建模，基於規範的開發 |
| 5 | 安全配置錯誤 (Security Misconfiguration) | 安全標頭，最小權限，審核依賴項 |
| 6 | 易受攻擊的組件 (Vulnerable Components) | 生態系統的依賴項審核 (`npm audit`, `pip-audit`, ...)，保持依賴項更新，最小依賴項 |
| 7 | 身份驗證失敗 (Auth Failures) | 強密碼，速率限制，會話管理 |
| 8 | 資料完整性失敗 (Data Integrity Failures) | 驗證更新/依賴項，簽名產物 |
| 9 | 日誌失敗 (Logging Failures) | 記錄安全事件，不要記錄密碼 |
| 10 | SSRF (伺服器端請求偽造) | 驗證/允許清單 URL，限制出站請求 |

## LLM 的 OWASP Top 10 快速參考

適用於具有 LLM 功能的應用程式。請參閱 [OWASP GenAI 安全性專案](https://genai.owasp.org/llm-top-10/)。

| ID | 風險 | 防範 |
|---|---|---|
| LLM01 | 提示注入 (Prompt Injection) | 不要信任系統提示作為邊界；在代碼中強制執行權限 |
| LLM02 | 敏感資訊洩露 (Sensitive Information Disclosure) | 將密碼/PII 排除在提示之外；過濾輸出 |
| LLM03 | 供應鏈 (Supply Chain) | 審核模型、資料集和插件，如同審核任何依賴項一樣 |
| LLM04 | 資料和模型中毒 (Data and Model Poisoning) | 使用受信任的模型來源，驗證完整性；審核微調和 RAG 資料 |
| LLM05 | 不當的輸出處理 (Improper Output Handling) | 將模型輸出視為不可信；驗證、參數化、編碼 |
| LLM06 | 過度的代理權 (Excessive Agency) | 範圍限制工具權限；確認破壞性操作 |
| LLM07 | 系統提示洩露 (System Prompt Leakage) | 假設系統提示會洩露；不要在其中放入密碼 |
| LLM08 | 向量和嵌入的弱點 (Vector and Embedding Weaknesses) | 根據租戶分割 RAG 嵌入；在索引前驗證文件 |
| LLM09 | 錯誤資訊 (Misinformation) | 用引文為答案提供依據；驗證關鍵聲明；保持人類在迴路中 |
| LLM10 | 無界消耗 (Unbounded Consumption) | 限制令牌、請求速率和迴圈/遞迴深度 |
