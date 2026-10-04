# 加固模式

用於 `security-and-hardening` 技能的程式碼模式和決策輔助。該技能陳述每一條規則；此檔案顯示了其一具體的實施方式。當您到達該程式碼時，請打開您需要的區段——它不旨在從上到下閱讀。

共用的檢查清單（威脅建模、預提交檢查、安全標頭、CORS、套件管理器矩陣、安裝腳本閘道和 OWASP 快速參考表）位於 `../../../references/security-checklist.md`。

## OWASP 前 10 避防模式

這些是避防模式，而不是排名。對於 2021 年的排序，請參閱 `../../../references/security-checklist.md#owasp-top-10-quick-reference` 中的快速參考表。

### 注入 (Injection)

```typescript
// 壞：透過字串連接進行 SQL 注入
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// 好：參數化查詢
const user = await db.query('SELECT * FROM users WHERE id = $1', [userId]);

// 好：使用參數化輸入的 ORM
const user = await prisma.user.findUnique({ where: { id: userId } });
```

### 身份驗證失敗 (Broken Authentication)

```typescript
// 密碼雜湊
import { hash, compare } from 'bcrypt';

const SALT_ROUNDS = 12;
const hashedPassword = await hash(plaintext, SALT_ROUNDS);
const isValid = await compare(plaintext, hashedPassword);

// 會話管理
app.use(session({
  secret: process.env.SESSION_SECRET,  // 從環境變數獲取，而非程式碼中硬編碼
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,     // 無法透過 JavaScript 存取
    secure: true,       // 僅 HTTPS
    sameSite: 'lax',    // CSRF 防護
    maxAge: 24 * 60 * 60 * 1000,  // 24 小時
  },
}));
```

### 跨站腳本攻擊 (XSS)

```typescript
// 壞：將使用者輸入渲染為 HTML
element.innerHTML = userInput;

// 好：使用框架的自動轉義（React 預設會這樣做）
return <div>{userInput}</div>;

// 如果您必須渲染 HTML，請先進行清理
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(userInput);
```

### 存取控制失敗 (Broken Access Control)

```typescript
// 始終檢查授權，而不僅僅是身份驗證
app.patch('/api/tasks/:id', authenticate, async (req, res) => {
  const task = await taskService.findById(req.params.id);

  // 檢查已驗證的使用者是否擁有此資源
  if (task.ownerId !== req.user.id) {
    return res.status(403).json({
      error: { code: 'FORBIDDEN', message: '無權修改此任務' }
    });
  }

  // 繼續更新
  const updated = await taskService.update(req.params.id, req.body);
  return res.json(updated);
});
```

### 安全配置錯誤 (Security Misconfiguration)

```typescript
// 安全標頭（使用 helmet 進行 Express）
import helmet from 'helmet';
app.use(helmet());

// 內容安全政策 (Content Security Policy)
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],
    styleSrc: ["'self'", "'unsafe-inline'"],  // 如果可能，請收緊
    imgSrc: ["'self'", 'data:', 'https:'],
    connectSrc: ["'self'"],
  },
}));

// CORS — 限制於已知來源
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || 'http://localhost:3000',
  credentials: true,
}));
```

### 敏感資料外洩 (Sensitive Data Exposure)

```typescript
// 絕不將敏感欄位返回在 API 回應中
function sanitizeUser(user: UserRecord): PublicUser {
  const { passwordHash, resetToken, ...publicFields } = user;
  return publicFields;
}

// 使用環境變數來管理密鑰
const API_KEY = process.env.STRIPE_API_KEY;
if (!API_KEY) throw new Error('STRIPE_API_KEY 未配置');
```

### 伺服器端請求偽造 (SSRF)

任何時候伺服器獲取使用者影響的 URL——Webhook、"從 URL 匯入"、影像代理、連結預覽——攻擊者都可以將其指向內部服務（雲端元數據、`localhost`、私有 IP）。

```typescript
// 壞：獲取使用者給的任何東西
await fetch(req.body.webhookUrl);

// 好：允許列表（allowlist）方案 + 主機，如果任何解析的 IP 是私有的，則拒絕，禁止重定向
import { lookup } from 'node:dns/promises';
import ipaddr from 'ipaddr.js';

const ALLOWED_HOSTS = new Set(['hooks.example.com']);

async function assertSafeUrl(raw: string): Promise<URL> {
  const url = new URL(raw);
  if (url.protocol !== 'https:') throw new Error('僅允許 https');
  if (!ALLOWED_HOSTS.has(url.hostname)) throw new Error('主機不允許');
  // 解析所有記錄；任何單個私有/保留地址都會導致檢查失敗。
  const addrs = await lookup(url.hostname, { all: true });
  if (addrs.some((a) => ipaddr.parse(a.address).range() !== 'unicast')) {
    throw new Error('私有/保留 IP');
  }
  return url;
}

await fetch(await assertSafeUrl(req.body.webhookUrl), { redirect: 'error' });
```

`range() !== 'unicast'` 檢查涵蓋了迴路、鏈路本地地址 `169.254.169.254`（雲端元數據，第 1 位的 SSRF 目標）、私有和單獨區域的 IPv4 和 IPv6 範圍。

**注意事項 — 這仍然存在 TOCTOU 差距。** `fetch` 在檢查後會再次解析 DNS，因此攻擊者可以使用短 TTL 記錄在驗證和連線之間重新綁定到內部 IP。對於高風險表面，一次解析後連線到釘選的 IP，或在前面放置一個過濾代理（`request-filtering-agent` / `ssrf-req-filter`）。

## 輸入驗證模式 (Input Validation Patterns)

### 邊界上的模式驗證 (Schema Validation at Boundaries)

```typescript
import { z } from 'zod';

const CreateTaskSchema = z.object({
  title: z.string().min(1).max(200).trim(),
  description: z.string().max(2000).optional(),
  priority: z.enum(['low', 'medium', 'high']).default('medium'),
  dueDate: z.string().datetime().optional(),
});

// 在路由處理器中驗證
app.post('/api/tasks', async (req, res) => {
  const result = CreateTaskSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: '輸入無效',
        details: result.error.flatten(),
      },
    });
  }
  // result.data 現在已類型化並經過驗證
  const task = await taskService.create(result.data);
  return res.status(201).json(task);
});
```

### 檔案上傳安全 (File Upload Safety)

```typescript
// 限制檔案類型和大小
const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/webp'];
const MAX_SIZE = 5 * 1024 * 1024; // 5MB

function validateUpload(file: UploadedFile) {
  if (!ALLOWED_TYPES.includes(file.mimetype)) {
    throw new ValidationError('檔案類型不允許');
  }
  if (file.size > MAX_SIZE) {
    throw new ValidationError('檔案太大 (最大 5MB)');
  }
  // 不要信任檔案擴展名 — 如果是關鍵，請檢查魔術位元組
}
```

### 對衍生路徑的破壞性操作 (Destructive Operations on Derived Paths)

刪除、移動或覆寫的安全性取決於命名目標的值。從核心、工作負載或兄弟服務中讀取該值證明了它*來自何處*，而不是誰*寫入*它——另一個程序的命令列與表單欄位一樣可被攻擊者控制。一個形狀檢查（"絕對路徑，至少一層目錄深」）證明了格式的正確性，並被誤認為是授權；這就是清理例程刪除根而不是葉子的方式。

在執行破壞性呼叫之前，要求所有三項：解析出的目標位於一個**允許列表的根**下（解析符連結後比較，絕不比較原始字串）；它**在**該根的**下方**至少一層，因此根本身絕不是目標；並且它攜帶**證明它是您自己的證據**，在操作之前和任何移除它的清理操作之前讀取它——否則「缺失」和「不屬於我」是無法區分的。如果拒絕，則記錄被拒絕的目標並停止：一個回退到更廣泛預設路徑的清理操作是此保護所對抗的失敗。在 `../../../references/security-checklist.md#destructive-path-operations` 中有實例。

兩個限制，因為檢查比它本身更強。樹中的標記是自我證明——任何可以寫入那裡的東西都可以寫入標記——因此預期的擁有者必須來自已驗證的狀態，並且該標記在被視為授權之前需要完整性保護（限制性所有權或 MAC）。而解析路徑然後操作*名稱*是檢查/使用競態條件，在任何不受信任的程序可以替換祖先的環境中：在共享磁碟上，使用描述符並禁止追蹤，進行根以下的運算，或確保層次在整個持續時間內不能改變。

## 依賴項審核分級 (Dependency Audit Triage)

套件管理器審核報告已知建議；它們不能證明一個套件是可信的，或者有漏洞的程式碼是可到達的。使用此決策樹：

```
原生套件管理器審核報告一個漏洞
├── 嚴重性：critical 或 high
│   ├── 有漏洞的程式碼在運行時、建置、測試或部署路徑中可到達嗎？
│   │   ├── 是 --> 立即修復（更新、修補或替換依賴項）
│   │   └── 否（在這些路徑中確認未使用） --> 盡快修復，但不是阻礙
│   └── 是否有修復可用？
│       ├── 是 --> 更新到修補版本
│       └── 否 --> 尋找變通方法，考慮替換依賴項，或添加至允許列表並設定審查日期
├── 嚴重性：moderate
│   ├── 在生產環境中可到達嗎？ --> 在下一個發布週期中修復
│   └── 僅限開發？ --> 方便時修復，追蹤到待辦事項中
└── 嚴重性：low
    └── 在定期依賴項更新期間追蹤並修復
```

**關鍵問題：**
- 有漏洞的函數是否在您的程式碼路徑中實際被呼叫？
- 該依賴項是運行時依賴還是僅限開發？
- 考慮到您的部署環境，該漏洞是否可被利用（例如，在僅限客戶端的應用程式中的伺服器端漏洞）？

當您推遲修復時，請記錄原因並設定審查日期。

## 速率限制 (Rate Limiting)

```typescript
import rateLimit from 'express-rate-limit';

// 一般 API 速率限制
app.use('/api/', rateLimit({
  windowMs: 15 * 60 * 1000, // 15 分鐘
  max: 100,                   // 每窗口 100 個請求
  standardHeaders: true,
  legacyHeaders: false,
}));

// 對身份驗證端點的更嚴格限制
app.use('/api/auth/', rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 10,  // 15 分鐘內 10 次嘗試
}));
```

**在共享儲存中，一次計數發生在多個程序之間。** `express-rate-limit` 預設將計數保存在程序記憶體中。在負載平衡器後面，每個實例都持有自己的計數，因此有效限制是 `max × instances`；在無伺服器或邊緣執行環境中，每次新的呼叫都從零開始，因此上述身份驗證限制可能永遠不會觸發。傳遞一個共享的 `store`（透過 `rate-limit-redis` 進行 Redis），或者使用一種在長期 TCP 連線不存在時也能運作的 HTTP 基礎限制器（例如 `@upstash/ratelimit`）：

```typescript
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const authLimiter = new Ratelimit({
  redis: Redis.fromEnv(),                       // UPSTASH_REDIS_REST_URL + _TOKEN
  limiter: Ratelimit.slidingWindow(10, '15 m'), // 15 分鐘內 10 次嘗試，跨所有實例
});
const { success } = await authLimiter.limit(`login:${req.ip}`);
if (!success) return res.status(429).end();
```

## 密鑰管理 (Secrets Management)

```
.env 檔案：
  ├── .env.example  → 已提交（帶有佔位符值的範本）
  ├── .env          → 未提交（包含真實密鑰）
  └── .env.local    → 未提交（本地覆蓋）

.gitignore 必須包含：
  .env
  .env.local
  .env.*.local
  *.pem
  *.key
```

**始終在提交前檢查：**
```bash
# 檢查是否意外暫存了密鑰
git diff --cached | grep -i "password\|secret\|api_key\|token"
```

**如果密鑰被提交，請輪換它。** 刪除該行或重寫歷史是不夠的——一旦它到達遠端，就應假設它已被洩露。首先撤銷並重新發行密鑰，然後從歷史中清除它。

## 資料分類 (Data Classification)

隨著您添加欄位，將個人資料分類，以便匯出和刪除請求可以稍後找到它：

| 類別 | 範例 | 處理方式 |
|---|---|---|
| **非個人** | 彙總、匿名計數 | 常規處理 |
| **個人 (PII)** | 姓名、電子郵件、IP、設備/使用者 ID | 最小化、存取控制、包含在匯出/刪除中 |
| **敏感** | 健康、財務、位置、生物測量、政府 ID、任何關於未成年人的資訊 | 收集的額外依據、更嚴格的存取、通常是加密 + 審計日誌 |

## LLM 輸出處理 (LLM Output Handling)

```typescript
// 壞：信任模型輸出作為命令或標記
const sql = await llm.generate(`為：${userQuestion} 寫 SQL`);
await db.query(sql);                                   // 任意查詢執行
container.innerHTML = await llm.reply(userMessage);   // 儲存的 XSS，透過模型

// 好：模型輸出是資料 — 防禦性地解析，然後驗證，然後編碼
let intent;
try {
  intent = CommandSchema.parse(JSON.parse(await llm.replyJson(userMessage)));
} catch {
  throw new ValidationError('意外的模型輸出'); // JSON.parse 或模式失敗
}
await runAllowlistedAction(intent.action, intent.params);
container.textContent = await llm.reply(userMessage);
```
