# 優化模式

對 `performance-optimization` 技能步驟 3 中提到的反模式進行了修復。該技能說明了需要尋找的規則和簽名；此文件展示了一個具體的實施方式。在步驟 2 識別出瓶頸後，請查看您需要的部分——它不應從上到下閱讀。

檢查清單、測量命令、TTFB 診斷以及快取讀寫策略都位於 `../../../references/performance-checklist.md` 中。

## N+1 查詢 (後端)

```typescript
// 壞：N+1 — 每個任務都進行一次查詢
const tasks = await db.tasks.findMany();
for (const task of tasks) {
  task.owner = await db.users.findUnique({ where: { id: task.ownerId } });
}

// 好：使用 JOIN/include 的單一查詢
const tasks = await db.tasks.findMany({
  include: { owner: true },
});
```

## 無界數據獲取

```typescript
// 壞：獲取所有記錄
const allTasks = await db.tasks.findMany();

// 好：使用限制進行分頁
const tasks = await db.tasks.findMany({
  take: 20,
  skip: (page - 1) * 20,
  orderBy: { createdAt: 'desc' },
});
```

## 無視索引的查詢

「添加索引」只是猜測。查詢計劃是測量：

```sql
EXPLAIN ANALYZE
SELECT id, title FROM tasks
WHERE owner_id = 42 ORDER BY created_at DESC LIMIT 20;
```

輸出中的三件事決定了修復方式：

| 您看到的是 | 它的含義 |
|---|---|
| 在您預期有索引的大表上進行 `Seq Scan` | 此謂詞沒有可用的索引 |
| `rows=` 的估計值與實際值相差一個數量級 | 統計數據過時；規劃器是根據錯誤信息進行選擇 |
| 掃描之上有一個 `Sort` 節點 | 索引覆蓋了過濾條件但沒有 `ORDER BY` |

索引應針對**查詢的形狀**，而不是孤立的欄位。在複合索引中，等值欄位應放在最前面，然後是範圍或排序欄位：

```sql
CREATE INDEX idx_tasks_owner_created ON tasks (owner_id, created_at DESC);
```

**索引不會有幫助的情況：**

| 情境 | 原因 |
|---|---|
| 低選擇性，查詢主導值（一個 `status` 欄位有 95% 是 `active`，過濾 `active`） | 順序掃描實際上更便宜；規劃器會忽略索引。過濾於罕見值是相反的情況，部分索引能很好地服務於此 |
| 前導通配符 (`LIKE '%term'`) | B-tree 無法在沒有前綴的情況下查找；需要三元組或全文檢索 |
| 欄位上的函數 (`WHERE lower(email) = ?`) | 簡單的欄位索引不可用；對表達式進行索引 |
| 寫入量大的表 | 每個索引都是對每個 `INSERT`/`UPDATE` 的稅；測量寫入成本，而不僅僅是讀取增益 |

之後重新運行 `EXPLAIN ANALYZE`。一個沒有改變計劃的索引是回退（步驟 4），它並不是免費的：它在每次寫入操作中仍然產生成本。

## 連接池耗盡

簽名是獨特的：**所有**端點同時變慢，慢時間花在等待連接上而不是執行，並且資料庫報告的大多是空閒會話。

```typescript
// 壞：每個請求或每個模組都有一個連接池 — 在無伺服器環境下，這會乘以實例數量並耗盡資料庫的連接限制
// 好：每個進程一個連接池，根據資料庫的上限進行調整大小
const pool = new Pool({
  max: 10,                        // 實例 × max 必須保持在 max_connections 以下
  idleTimeoutMillis: 30_000,
  connectionTimeoutMillis: 5_000, // 快速失敗而不是永遠排隊
});
```

**越大不一定越快。** 大於資料庫可以同時執行的連接的連接池只是將隊列從您的應用程序重新定位到資料庫，在那裡更難看到。當實例數量是無界的（無伺服器、自動擴展）時，一個多路復用連接的代理（pgbouncer、RDS Proxy）是解決方案，而不是更高的 `max`。

## 缺少圖像優化 (前端)

```html
<!-- 壞：沒有尺寸，沒有格式優化 -->
<img src="/hero.jpg" />

<!-- 好：首屏/LCP 圖像 — 藝術指導 + 解析度切換，高優先級 -->
<!--
  兩種技術的組合：
  - 藝術指導 (媒體)：每個斷點不同的裁剪/構圖
  - 解析度切換 (srcset + sizes)：根據螢幕密度設置正確的文件大小
-->
<picture>
  <!-- 行動裝置：肖像裁剪 (8:10) -->
  <source
    media="(max-width: 767px)"
    srcset="/hero-mobile-400.avif 400w, /hero-mobile-800.avif 800w"
    sizes="100vw"
    width="800"
    height="1000"
    type="image/avif"
  />
  <source
    media="(max-width: 767px)"
    srcset="/hero-mobile-400.webp 400w, /hero-mobile-800.webp 800w"
    sizes="100vw"
    width="800"
    height="1000"
    type="image/webp"
  />
  <!-- 電腦版：橫向裁剪 (2:1) -->
  <source
    srcset="/hero-800.avif 800w, /hero-1200.avif 1200w, /hero-1600.avif 1600w"
    sizes="(max-width: 1200px) 100vw, 1200px"
    width="1200"
    height="600"
    type="image/avif"
  />
  <source
    srcset="/hero-800.webp 800w, /hero-1200.webp 1200w, /hero-1600.webp 1600w"
    sizes="(max-width: 1200px) 100vw, 1200px"
    width="1200"
    height="600"
    type="image/webp"
  />
  <img
    src="/hero-desktop.jpg"
    width="1200"
    height="600"
    fetchpriority="high"
    alt="首屏圖像描述"
  />
</picture>

<!-- 好：次要內容圖像 — 惰性加載 + 異步解碼 -->
<img
  src="/content.webp"
  width="800"
  height="400"
  loading="lazy"
  decoding="async"
  alt="內容圖像描述"
/>
```

## 不必要的重新渲染 (React)

```tsx
// 壞：每次渲染都創建新對象，導致子組件重新渲染
function TaskList() {
  return <TaskFilters options={{ sortBy: 'date', order: 'desc' }} />;
}

// 好：穩定引用
const DEFAULT_OPTIONS = { sortBy: 'date', order: 'desc' } as const;
function TaskList() {
  return <TaskFilters options={DEFAULT_OPTIONS} />;
}

// 對昂貴組件使用 React.memo
const TaskItem = React.memo(function TaskItem({ task }: Props) {
  return <div>{/* 昂貴的渲染 */}</div>;
});

// 對昂貴計算使用 useMemo
function TaskStats({ tasks }: Props) {
  const stats = useMemo(() => calculateStats(tasks), [tasks]);
  return <div>{stats.completed} / {stats.total}</div>;
}
```

## 大型捆綁包大小

```typescript
// 現代打包器 (Vite, webpack 5+) 會自動處理命名導入的樹狀割 (tree-shaking)，
// 前提是依賴項發布為 ESM 並且在 package.json 中標記為 `sideEffects: false`。
// 在更改導入樣式之前進行分析 — 真正的增益來自於拆分和惰性加載。

// 好：對於大型、很少使用的功能使用動態導入
const ChartLibrary = lazy(() => import('./ChartLibrary'));

// 好：在 Suspense 中包裝的路由級代碼拆分
const SettingsPage = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <SettingsPage />
    </Suspense>
  );
}
```

## 缺少快取 (後端)

對那些產生和讀取成本高、變化頻率低於其變化次數的數據進行快取。對已經快速的查詢進行快取會增加一次網絡跳躍、一個陳舊性錯誤，以及一個需要維護的淘汰策略，而沒有任何回報。

**有意識地選擇層級：**

| 層級 | 可見性 | 何時使用 | 成本 |
|---|---|---|---|
| 內程 (`Map`, LRU) | 單一實例 | 較小、熱的，可以接受每個實例的陳舊性 | 每個實例獨立漂移；失效只傳達給一個 |
| 共享 (Redis, Memcached) | 所有實例 | 實例必須達成一致，否則重新計算該值成本高昂 | 一次網絡跳躍，以及另一個服務來運行和監控 |
| CDN / 邊緣 | 每個人，每個 URL | 回應是公開的，對於給定鍵是相同的 | 失效是困難的部分；假設您無法快速召回錯誤的響應 |

```typescript
// 快取經常讀取、很少變化的數據
const CACHE_TTL = 5 * 60 * 1000; // 5 分鐘
let cachedConfig: AppConfig | null = null;
let cacheExpiry = 0;

async function getAppConfig(): Promise<AppConfig> {
  if (cachedConfig && Date.now() < cacheExpiry) {
    return cachedConfig;
  }
  cachedConfig = await db.config.findFirst();
  cacheExpiry = Date.now() + CACHE_TTL;
  return cachedConfig;
}

// 靜態資產的 HTTP 快取標頭
app.use('/static', express.static('public', {
  maxAge: '1y',           // 快取 1 年
  immutable: true,        // 永不重新驗證（在文件名中使用內容雜湊）
}));

// API 回應的 Cache-Control
res.set('Cache-Control', 'public, max-age=300'); // 5 分鐘
```

**關鍵設計決定正確性。** 每個改變響應的輸入都應該在鍵中：租戶、地區、權限、功能標誌。一個省略了查看者的鍵就是如何將某用戶的數據提供給另一個用戶，這會以性能優化的方式傳達。

**選擇一個失效策略，而不是三個：**

| 策略 | 權衡 |
|---|---|
| TTL | 最簡單。您接受在 TTL 內存在陳舊性，因此明確說明可接受的時間窗口 |
| 基於事件或標籤 | 寫入時保持最新，但寫入者現在必須知道快取拓撲 |
| 版本化鍵 (`user:42:profile:v7`) | 從不失效，只需停止讀取舊的鍵。直到淘汰，它會消耗內存 |

**防範雪崩。** 一個熱鍵過期，所有並行請求一起錯過，源頭一次性承擔全部負載，這就是快取變成中斷而不是防止單個事件發生的方式。在單個請求重新計算期間提供陳舊數據（`stale-while-revalidate`），或者將所有並行的錯過合併到一個進行中的 Promise 後，讓 N 個等待者只導致一次重新計算。

**不要快取：** 任何其陳舊性是正確性錯誤（餘額、權限、結賬庫存）或在不識別用戶的鍵下的每用戶數據。請參閱 `../../../references/performance-checklist.md#caching-strategies` 以獲取請求合併、寫入策略、負向快取和快取檢查清單。
