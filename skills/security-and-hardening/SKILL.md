---
name: security-and-hardening
description: Hardens code against vulnerabilities. Use when auditing an input handler for vulnerabilities, when handling user input, authentication, data storage, or external integrations, or when checking a login flow is safe against the OWASP Top Ten. Use when building any feature that accepts untrusted data, manages user sessions, or interacts with third-party services. Use when auditing dependencies for known vulnerabilities, triaging package-manager audit findings, or assessing supply-chain risk in a new package. Use when personal data or privacy compliance (GDPR, CCPA) is involved.
---

# 安全與加固

## 概述

針對網頁應用的安全優先開發實踐。將所有外部輸入視為敵對，所有密碼視為神聖，所有授權檢查視為強制要求。安全不是一個階段——它是對所有接觸使用者資料、身份驗證或外部系統的程式碼的約束。

## 何時使用

- 建構任何接受使用者輸入的內容
- 實施身份驗證或授權
- 儲存或傳輸敏感資料
- 與外部 API 或服務整合
- 新增檔案上傳、Webhook 或回呼
- 處理支付或個人身份資訊 (PII) 資料

## 流程：以威脅模型為先

在沒有威脅模型的控制措施是猜測。在加固之前，花五分鐘像攻擊者一樣思考：

1. **繪製信任邊界。** 不可信資料到您的系統的邊界在哪裡？HTTP 請求、表單欄位、檔案上傳、Webhook、第三方 API、訊息佇列，以及那些看起來是內部值的本地值：另一個程序的命令列或環境、共用磁碟上的檔名、工作負載中的路徑。信任是基於**誰寫了**一個值，而不是哪個通道傳遞了它。每一個邊界都是攻擊面。
2. **命名資產。** 什麼值得被竊取或破壞？憑證、PII、支付資料、管理員操作、金錢流動。
3. **對每個邊界運行 STRIDE**——一個快速的視角，而不是一場儀式：

| 威脅 | 問題 | 常見緩解措施 |
|---|---|---|
| **S**poofing (冒充) | 是否有人可以冒充使用者/服務？ | 身份驗證、簽名驗證 |
| **T**ampering (篡改) | 資料在傳輸或靜止時能否被更改？ | 完整性檢查、參數化查詢、HTTPS |
| **R**epudiation (拒絕) | 後續能否否認某個動作？ | 安全事件的審計日誌記錄 |
| **I**nformation disclosure (資訊洩露) | 資料能否洩露？ | 加密、欄位允許清單、通用錯誤訊息 |
| **D**enial of service (拒絕服務) | 能否被淹沒？ | 速率限制、輸入大小上限、超時 |
| **E**levation of privilege (權限提升) | 使用者能否獲得他們不應擁有的權限？ | 授權檢查、最小權限原則 |

4. **在用例旁邊寫下濫用案例。** 對於每個功能，問「我如何能濫用它？」——然後將其作為你的第一個測試。

如果你無法為一個功能命名信任邊界，你就沒有準備好安全它。這是 OWASP **A04: 不安全設計**——大多數入侵始於設計，而不是程式碼。

## 三層邊界系統

### 務必執行（無例外）

- 在系統邊界（API 路由、表單處理器）**驗證所有外部輸入**
- **參數化所有資料庫查詢**——絕不將使用者輸入直接拼接進 SQL
- **對輸出進行編碼**以防止 XSS（使用框架的自動轉義功能，不要繞過它）
- 對所有外部通訊**使用 HTTPS**
- 使用 bcrypt/scrypt/argon2 **雜湊密碼**（絕不儲存明文）
- **設定安全標頭**（CSP、HSTS、X-Frame-Options、X-Content-Type-Options）
- 對會話使用 `httpOnly`、`secure`、`sameSite` Cookie
- 在每次發布前，對已提交的鎖定檔運行**檢測到的套件管理器原生審計**

### 先詢問（需要人工批准）

- 新增身份驗證流程或更改認證邏輯
- 儲存新的敏感資料類別（PII、支付資訊）
- 新增外部服務整合
- 更改 CORS 設定
- 新增檔案上傳處理器
- 修改速率限制或限制
- 授予提升的權限或角色

### 絕不執行

- **絕不將密碼提交**到版本控制中（API 金鑰、密碼、Token）
- **絕不記錄敏感資料**（密碼、Token、完整信用卡號）
- **絕不信任客戶端驗證**作為安全邊界
- **絕不為便利而禁用安全標頭**
- **絕不使用 `eval()` 或 `innerHTML`** 處理使用者提供的資料
- **絕不將會話儲存在客戶端可存取的儲存中**（用於認證 Token 的 `localStorage`）
- **絕不向使用者暴露堆疊追蹤**或內部錯誤細節

## 加固控制措施

以下規則是工作流程；每個規則的具體實施都存在於 [references/hardening-patterns.md](references/hardening-patterns.md)。在你到達該程式碼時，才打開你需要的區段，而不是在那之前。

### 注入、XSS 和存取控制

- **參數化每個查詢。** 絕不從輸入字串構建 SQL、NoSQL 或 shell 命令。
- 透過框架的自動轉義功能**編碼輸出**。如果無法避免原始 HTML，請先使用允許清單的清理器進行清理。
- 在**每個請求上檢查授權**，而不僅僅是身份驗證：已驗證的使用者必須擁有，或被允許存取，特定的資源（A01, IDOR）。

模式：[Injection](references/hardening-patterns.md#injection), [XSS](references/hardening-patterns.md#cross-site-scripting-xss), [Access control](references/hardening-patterns.md#broken-access-control)。

### 身份驗證和會話

- 使用 bcrypt（≥12 輪）、scrypt 或 argon2 **雜湊密碼**。會話密鑰來自環境，絕不來自程式碼。
- 會話 Cookie 應是 `httpOnly`、`secure`，並且 `sameSite` 為 `'lax'` 或 `'strict'`（CSRF 防禦；`'none'` 在跨站請求中傳送 Cookie），並設定一個有界 `maxAge`。

模式：[Authentication](references/hardening-patterns.md#broken-authentication)。

### 標頭、CORS 和回應

- 在每個回應上設定安全標頭（使用 helmet 或框架等效功能）；CSP 從 `default-src 'self'` 開始，並且是收緊而不是放寬。
- CORS 僅限於從配置中明確的來源列表。絕不使用憑證的 `*`。
- 在任何回應之前，移除敏感欄位（`passwordHash`、重設 Token）。錯誤主體應是通用的；內部細節只記錄在伺服器日誌中。

模式：[Misconfiguration](references/hardening-patterns.md#security-misconfiguration), [Sensitive data exposure](references/hardening-patterns.md#sensitive-data-exposure)。

### 輸入驗證和上傳

- 在邊界處使用**模式**進行驗證：允許的形狀、長度、枚舉值、格式。以 422 和結構化細節拒絕；下游程式碼僅使用解析和類型化的值。
- 上傳：允許 MIME 類型，限制大小，在需要時驗證內容（魔術字元）。副檔名無法證明任何事。

模式：[Schema validation](references/hardening-patterns.md#schema-validation-at-boundaries), [File upload](references/hardening-patterns.md#file-upload-safety)。

### 伺服器端獲取（SSRF）

任何使用者影響的 URL——Webhook、從 URL 匯入、影像代理、連結預覽——都可能指向內部服務。允許來源和主機，解析**所有** DNS 記錄，並拒絕任何私有或保留的地址（迴路、鏈路本地 `169.254.169.254`，私有、唯一本地，對於 IPv4 和 IPv6），並禁止重定向。該檢查仍然存在 DNS 重新綁定 TOCTOU 差距：對於高風險表面，請釘選解析的 IP 或在前面放置一個過濾代理。

模式：[SSRF](references/hardening-patterns.md#server-side-request-forgery-ssrf)。

### 對衍生路徑的破壞性操作

刪除、移動或覆寫只在目標值的命名上是安全的，而信任是基於**誰寫了**該值，而不是哪個通道傳遞了它：另一個程序的命令列與表單欄位一樣可被攻擊者控制。形狀檢查證明了格式的正確性，而不是授權。在呼叫之前，要求這三種條件：解析的目標（符號連結已解析）位於一個**允許清單的根**之下；它至少在該根**下方**；並且它攜帶了**操作前讀取的擁有權證據**。如果拒絕，則記錄被拒絕的目標並停止；絕不退回到更廣泛的預設路徑。

為什麼這個檢查比它讀取的內容更弱（標記自我證明、檢查/使用競態條件）：[Destructive paths](references/hardening-patterns.md#destructive-operations-on-derived-paths)。實際程式碼：`../../references/security-checklist.md`。

### 速率限制

對 API 和認證端點進行嚴格限制（每 15 分鐘約 10 次嘗試）。一旦多個程序提供流量，內存計數器會靜默地變成 `最大值 × 實例數`，或者在無伺服器環境中從不觸發：使用共享儲存來後退限制器。

模式：[Rate limiting](references/hardening-patterns.md#rate-limiting)。

### 密碼

密碼來自環境。`.env.example` 包含佔位符並提交；實際的 `.env*` 檔案和金鑰資料被 gitignore；提交前使用 `grep` 檢查暫存差異。**任何到達遠端的密碼在到達的那一刻就已洩露：先輪換它，然後清除歷史記錄。**

模式：[Secrets management](references/hardening-patterns.md#secrets-management)。

### 依賴項和供應鏈

1. **找到安裝邊界和管理器。** 使用擁有鎖定檔的工作區根目錄，或者僅在它在該工作區之外時才使用獨立的嵌套專案。核實 `packageManager`（如果存在）、鎖定檔和 CI；在意見不一致或競爭的鎖定檔時停止。釘選管理器版本。
2. **在首次執行前阻止依賴腳本。** 使用禁用腳本或記錄失敗策略進行啟動，檢查待處理腳本的來源，只批准最少的內容，提交策略，然後使用乾淨的凍結/不可變安裝進行驗證。絕不自動批准。
3. **在每次發布前，對已提交的鎖定檔運行原生審計。** 根據**可到達性**（運行時、建置、測試、部署路徑）對關鍵/高風險進行分類並修復可用性。絕不自動應用強制修復（`npm audit fix --force` 或等效的），因為強制修復可能跨越聲明的依賴範圍；預覽、閱讀變更日誌，測試每次升級。用原因和審查日期記錄每一次延遲。
4. **審計僅與已知建議相符。** 它們無法發現新惡意或拼錯的套件（`cross-env` vs `crossenv`）。一起審查新的依賴項、鎖定檔差異和腳本策略更改：所有權、維護、發布年代、來源、傳遞圖。在受支援的地方驗證註冊表簽名（`npm audit signatures`，`pnpm audit signatures`），並將其缺失視為需要調查的信號，而不是自動的入侵證明（A06, LLM03）。

決策樹：[Dependency audit triage](references/hardening-patterns.md#dependency-audit-triage)。管理器矩陣和安裝腳本閘門：`../../references/security-checklist.md`。

### 個人資料和隱私權

加固詢問「攻擊者能否讀取它？」隱私權詢問「我們是否應該持有它，以及持有多久？」保護、洩露和遵守的成本最低的資料是那些你從未收集的資料；將個人資料視為需要最小化的負擔。

- **在新增欄位時進行分類**（非個人、PII、敏感），並相應處理每一類別。你無法保護或回應刪除請求的資料，如果你找不到它。
- **僅針對已聲明的目的收集資料。**「稍後可能有用」是潛在的洩露範圍，而不是目的。將 PII 排除在遙測之外（`observability-and-instrumentation` 技能從營運角度也提出了同樣的觀點）。
- **事先設定保留期限，然後實際刪除。** 每個個人資料儲存都需要 TTL 和一個可運作的刪除路徑，包括備份、快取、搜尋索引和分析副本。
- **支持你司法管轄區要求的資料主權**（GDPR、CCPA，以及其他）：匯出、更正、刪除。設計模式，使使用者的資料可以被找到和刪除，而不是在系統中不可逆地分散。
- **同意閘門收集和第三方共享，並且是可審計的。** 將 PII 發送到分析、廣告或 LLM 供應商是共享；供應商需要數據處理協議。將地區設為可配置的政策，而不是硬編碼的假設。

分類表：[Data classification](references/hardening-patterns.md#data-classification)。隱私事件啟動了洩露通知時鐘；使用 `debugging-and-error-recovery` 技能進行事後分析。

### AI / LLM 功能

呼叫 LLM——聊天機器人、摘要器、代理、RAG——增加了新的攻擊面；將其映射到 [OWASP LLM 應用程式 (2025)](https://genai.owasp.org/llm-top-10/)：

- **模型輸出是不可信的輸入**（LLM05）。絕不放入 `eval`、SQL、Shell、`innerHTML` 或檔案路徑；進行防禦性解析，根據模式進行驗證，然後編碼。
- **提示可以被劫持**（LLM01）。上下文中的不可信文本——使用者訊息、獲取的頁面、PDF——可以攜帶指令。系統提示不是安全邊界；在程式碼中強制執行權限。
- **將密碼、其他租戶的資料和完整的系統提示排除在上下文視窗之外**（LLM02, LLM07）；限制工具權限，驗證每個工具參數，並確認破壞性操作（LLM06）；限制 Token、請求速率和遞迴深度（LLM10）；按租戶分割 RAG 嵌入並在索引前驗證文件（LLM08）。

模式：[LLM output handling](references/hardening-patterns.md#llm-output-handling)。

## 審查清單

在簽署之前，瀏覽 `../../references/security-checklist.md`：它涵蓋了身份驗證、授權、輸入、資料保護和隱私權、標頭和 CORS、依賴項和供應鏈、AI/LLM 以及錯誤處理，以及 OWASP 的快速參考表。

## 常見合理化

| 合理化 | 現實 |
|---|---|
|「這是一個內部工具，安全不重要」 | 內部工具會被入侵。攻擊者會針對最薄弱的環節。 |
|「我們稍後會加上安全措施」 | 後期補救比一開始就建構要困難十倍。現在就加上它。 |
|「沒人會試圖利用它」 | 自動掃描器會找到它。安全模糊性不是安全。 |
|「框架處理安全」 | 框架提供工具，而不是保證。你仍然需要正確地使用它們。 |
|「這只是一個原型」 | 原型會變成生產環境。從第一天開始就養成安全習慣。 |
|「這裡不需要威脅建模」 | 五分鐘的「我如何能攻擊這個？」可以防止設計缺陷在之後無法修補。 |
|「它只是 LLM 輸出，它只有文字」 | 那個「文字」可能是一個 SQL 陳述式、一個腳本標籤或一個 Shell 命令。把它視為任何不可信的輸入。 |
|「審計通過了，所以依賴項是安全的」 | 審計與已知建議相符。它們無法檢測到新惡意套件或使未審查的安裝腳本安全執行。 |
|「現在收集，我們可能稍後需要它」 | 你不持有、不能洩露、傳票或錯誤刪除的資料。你「可能需要它」是洩露範圍，而不是目的。 |
|「我們將手動處理刪除請求」 | 手動擦除會遺漏備份、快取和分析副本。如果模式找不到使用者的資料，你就無法履行請求——需要為此設計。 |
|「合規是法律的問題，不是我們的問題」 | 匯出、刪除、保留和同意是模式和程式碼。法律無法在你將 PII 分散到十個系統後才補上。 |

## 警報信號

- 使用者輸入直接傳遞給資料庫查詢、Shell 命令或 HTML 渲染
- 刪除、移動或覆寫的目標來自於負載、配置值或另一個程序的命令列，僅由路徑上的形狀檢查保護
- 程式碼或提交歷史中的密碼
- 沒有身份驗證或授權檢查的 API 端點
- 缺少 CORS 配置或通配符 (`*`) 來源
- 認證端點上沒有速率限制，或在多於一個實例前有內存限制
- 堆疊追蹤或內部錯誤暴露給使用者
- 依賴項存在已知關鍵漏洞，在一個安裝邊界處存在競爭的鎖定檔，不可重現的安裝，或自動批准的腳本
- 進行伺服器獲取使用者提供的 URL 而沒有允許清單（SSRF）
- LLM/模型輸出傳遞到查詢、DOM、Shell 或 `eval`
- 將密碼、PII 或完整的系統提示放在 LLM 上下文視窗中
- 收集的個人資料沒有聲明的目的、保留限制或刪除路徑
- 將 PII 發送到分析/廣告/LLM 供應商而沒有同意或數據處理協議
- 「刪除我的帳戶」只會切換一個標誌，而個人資料仍然保留在儲存和備份中

## 驗證

在實施安全相關程式碼後：

- [ ] 原生審計沒有未緩解的可到達的關鍵/高風險發現；CI 保留權威的鎖定檔並阻止未審查的依賴腳本
- [ ] 程式碼或 git 歷史中沒有密碼
- [ ] 所有使用者輸入在系統邊界處都經過驗證
- [ ] 破壞性的檔案系統操作會解析符號連結，然後在運行前驗證允許清單的根、最小深度和所有權
- [ ] 所有受保護端點都進行了身份驗證和授權檢查
- [ ] 回應中存在安全標頭（使用瀏覽器開發工具檢查）
- [ ] 錯誤回應不暴露內部細節
- [ ] 認證端點上啟用了速率限制，當多於一個實例提供流量時，由共享儲存提供後備支持
- [ ] 伺服器端的 URL 獲取經過允許清單驗證（無 SSRF）
- [ ] 如果存在 AI 功能，LLM/模型輸出在使用前經過驗證和編碼
- [ ] 個人資料被分類、最小化到聲明的目的，並有保留限制
- [ ] 刪除和匯出請求是端到端運作的（包括備份、快取和分析副本）
