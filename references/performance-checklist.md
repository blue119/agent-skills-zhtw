# 效能檢查清單

網頁應用程式效能的快速參考清單。請與 `performance-optimization` 技能一起使用。

## 目錄

- [核心網頁指標目標](#core-web-vitals-targets)
- [TTFB 診斷](#ttfb-diagnosis)
- [前端檢查清單](#frontend-checklist)
- [後端檢查清單](#backend-checklist)
- [快取策略](#caching-strategies)
- [測量指令](#measurement-commands)
- [常見反模式](#common-anti-patterns)

## 核心網頁指標目標

| 指標 | 良好 | 需要改進 | 差 |
|---|---|---|---|
| LCP (最大內容繪製時間) | ≤ 2.5s | ≤ 4.0s | > 4.0s |
| INP (互動至下一個繪製時間) | ≤ 200ms | ≤ 500ms | > 500ms |
| CLS (累積佈局偏移) | ≤ 0.1 | ≤ 0.25 | > 0.25 |

## TTFB 診斷

當 TTFB 緩慢 (> 800ms) 時，請檢查 DevTools 中的網路瀑布圖中的每個組件：

- [ ] **DNS 解析** 緩慢 → 對已知的來源添加 `<link rel="dns-prefetch">` 或 `<link rel="preconnect">`
- [ ] **TCP/TLS 握手** 緩慢 → 啟用 HTTP/2，考慮邊緣部署，驗證 keep-alive
- [ ] **伺服器處理** 緩慢 → 分析後端，檢查慢查詢，添加快取

## 前端檢查清單

### 影像
- [ ] 影像使用現代格式 (WebP, AVIF)
- [ ] 影像具有響應式尺寸 (`srcset` 和 `sizes`)
- [ ] 影像和 `<source>` 元素具有明確的 `width` 和 `height` (防止藝術方向中的 CLS)
- [ ] 頁面下方的影像使用 `loading="lazy"` 和 `decoding="async"`
- [ ] 主視覺/LCP 影像使用 `fetchpriority="high"` 且不使用延遲載入

### JavaScript
- [ ] 壓縮後編譯大小在 200KB 以下 (初始載入)
- [ ] 使用動態 `import()` 進行代碼分割，以處理路由和重型功能
- [ ] 啟用樹搖動 (Tree shaking) (驗證依賴項發送 ESM 並標記 `sideEffects: false`)
- [ ] 不在 `<head>` 中有阻塞式 JavaScript (使用 `defer` 或 `async`)
- [ ] 重計算卸載到 Web Worker (如果適用)
- [ ] 對於重新渲染的昂貴組件使用 `React.memo()`
- [ ] 僅在分析顯示有益時使用 `useMemo()` / `useCallback()`
- [ ] 將長時間任務 (> 50ms) 分割開，以保持主線程可用 — 這是 INP 的主要槓桿
- [ ] 在長時間運行的迴圈內部使用 `yieldToMain` 模式，以便在區塊之間運行輸入事件
- [ ] 在可用時使用現代排程 API：`scheduler.yield()` (首選)，`scheduler.postTask()` 帶有優先級，`isInputPending()` 僅在需要時才讓出
- [ ] 對可延遲、非緊急的工作使用 `requestIdleCallback` (分析緩衝、預取、預熱)
- [ ] 將非關鍵工作延遲到事件處理器之外 (例如分析、日誌)，以確保對互動的響應不會延遲
- [ ] 第三方腳本使用 `async` / `defer` 載入，檢查大小，並在重型情況下由一個外殼 (facade) 進行前置 (聊天小工具、嵌入)

### CSS
- [ ] 關鍵 CSS 內聯或預載入
- [ ] 對非關鍵樣式無渲染阻塞 CSS
- [ ] 在生產環境中無 CSS-in-JS 運行時成本 (使用提取)

### 字體
- [ ] 限制在 2-3 種字體族，每種族 2-3 種字重 (額外的字重是另一個請求)
- [ ] 僅使用 WOFF2 格式 (最小、通用支援 — 跳過 WOFF/TTF/EOT)
- [ ] 盡可能自託管 (第三方字體 CDN 會增加 DNS + TCP + TLS 的往返時間)
- [ ] LCP 關鍵字體預載入：`<link rel="preload" as="font" type="font/woff2" crossorigin>`
- [ ] 使用 `font-display: swap` (或非關鍵的 `optional`) 以避免 FOIT 阻塞渲染
- [ ] 透過 `unicode-range` 進行子集化，僅發送每頁所需的字形
- [ ] 當需要多種字重/樣式時考慮變體字體 (一個檔案取代多個)
- [ ] 使用 `size-adjust`、`ascent-override`、`descent-override` 調整備用字體指標，以減少字體切換的 CLS
- [ ] 在任何自定義字體之前考慮系統字體堆疊

### 網路
- [ ] 靜態資產使用長 `max-age` + 內容雜湊進行快取
- [ ] 在適當的地方快取 API 回應 (`Cache-Control`)
- [ ] 啟用 HTTP/2 或 HTTP/3
- [ ] 對已知的來源預連接 (`<link rel="preconnect">`)
- [ ] 對關鍵的非影像資源使用 `fetchpriority` (例如，關鍵的 `<link rel="preload">`、首屏的 `<script>`) — 不僅僅是 `<img>`
- [ ] 無不必要的重定向

### 渲染
- [ ] 無佈局抖動 (強制同步佈局)
- [ ] 動畫使用 `transform` 和 `opacity` (GPU 加速)
- [ ] 長列表使用虛擬化 (例如，`react-window`)
- [ ] 無不必要的全頁重新渲染
- [ ] 螢幕外區域使用 `content-visibility: auto` 配合 `contain-intrinsic-size` 以跳過非可見區域的佈局/繪製
- [ ] 不在 HTML 回應上使用 `unload` 事件處理器，也不使用 `Cache-Control: no-store` — 保持後/前快取 (bfcache) 的資格

## 後端檢查清單

### 資料庫
- [ ] 無 N+1 查詢模式 (使用預載入 / 連接)
- [ ] 查詢具有適當的索引
- [ ] 列表端點進行分頁 (絕不使用 `SELECT * FROM table`)
- [ ] 配置連接池
- [ ] 啟用慢查詢日誌

#### 查詢計畫
- [ ] `EXPLAIN ANALYZE` 在修復**之前**捕獲，而不僅僅是在之後 — 它是基準線
- [ ] 對大型表的 `Seq Scan` 理解：索引缺失、不可用或實際上不值得
- [ ] 估計的 `rows=` 與實際的差異在一個數量級內 (如果不是，請在修改索引之前刷新統計資訊)
- [ ] 沒有複合索引可以吸收的 `Sort` 節點
- [ ] 更改後重新檢查計畫 — 如果索引沒有更改計畫，則會被撤銷

#### 索引策略
- [ ] 複合索引的欄位順序是等值優先，然後是範圍/排序
- [ ] 索引覆蓋查詢形狀 (過濾 + 排序)，而不僅僅是單個欄位
- [ ] 考慮覆蓋索引以用於熱讀取路徑 (索引僅掃描避免堆疊獲取)
- [ ] 不對低選擇性欄位建立索引 *針對主值*；部分索引仍然服務於罕見值的查詢 (`WHERE status = 'failed'`)
- [ ] 當查詢應用函數時使用表達式索引 (`lower(email)`)
- [ ] 對於前導通配符搜索使用全文或三元組索引，而不是 B-樹
- [ ] 對於寫入量大的表測量寫入成本 (每個索引都會對每個 `INSERT`/`UPDATE` 產生負擔)
- [ ] 刪除未使用的和重複的索引 (它們會增加寫入成本且沒有任何價值)

#### 連接池
- [ ] 每一個進程一個連接池，而不是每個請求或每個模組
- [ ] `instances × pool max` 保持在資料庫的 `max_connections` 以下
- [ ] 設定 `connectionTimeoutMillis` 以便在耗盡時快速失敗而不是無限期排隊
- [ ] 在調整大小之前診斷耗盡：找出持有連接的對象 (長時間交易、缺少 `await`、洩漏的客戶端)
- [ ] 透過多路復用代理 (pgbouncer, RDS Proxy) 而不是更大的連接池來為無伺服器/自動擴展提供支持

### API
- [ ] 回應時間 < 200ms (p95)
- [ ] 請求處理器中無同步的重計算
- [ ] 使用批量操作而不是單個調用的迴圈
- [ ] 回應壓縮 (gzip/brotli)
- [ ] 適當的快取 (內存、Redis、CDN)

### 基礎設施
- [ ] 靜態資產的 CDN
- [ ] 伺服器位於用戶附近 (或邊緣部署)
- [ ] 配置水平擴展 (如果需要)
- [ ] 負載平衡器的健康檢查端點

## 快取策略

決策材料 (哪個層級、哪個失效策略、什麼永不快取) 存在於 `performance-optimization` 技能中。本節涵蓋讀取/寫模式和檢查清單。

### 讀取和寫入模式

| 模式 | 工作方式 | 何時使用 | 需要注意的地方 |
|---|---|---|---|
| **Cache-aside** (惰性) | 應用程式檢查快取，在未命中時讀取來源並填充 | 預設選擇；讀取密集型，容忍冷啟動 | 每次未命中都會訪問來源，因此需要防範湧入 |
| **Read-through** | 快取層在未命中時自行載入 | 您希望載入路徑集中在一個地方，而不是在每個調用點 | 隱藏來源延遲；慢的來源看起來就像慢的快取 |
| **Write-through** | 寫入同時發送到快取和來源，同步進行 | 寫入後讀取絕不能看到過時的值 | 為每次寫入增加快取延遲 |
| **Write-behind** (寫回) | 寫入命中快取，來源異步更新 | 寫入密集型，且來源是瓶頸 | 如果快取在刷新之前損壞，則會丟失資料。需要你能防禦的持久性 |

### 負面快取

也快取結果的*缺失*。一個在每次查找中都未命中的鍵 (在迴圈中探測的不存在的使用者 ID，一個 404 資產) 會將每個請求發送到來源，該來源只保護了「快樂路徑」。

- 使用比正向條目**更短**的 TTL 儲存明確的「未找到」哨兵
- 保持負向 TTL 足夠短，以便新創建的記錄能及時出現
- 絕不要讓來源的*錯誤*成為負向快取條目，或一個失敗的分鐘變成許多

### 請求合併 (湧入防護)

一次重新計算，N 個等待者。防止熱鍵的過期導致完整的並行負載傳遞給來源：

```typescript
const inFlight = new Map<string, Promise<unknown>>();

function loadOnce<T>(key: string, fetcher: () => Promise<T>): Promise<T> {
  const existing = inFlight.get(key) as Promise<T> | undefined;
  if (existing) return existing;
  const p = fetcher().finally(() => inFlight.delete(key));
  inFlight.set(key, p);
  return p;
}
```

對於共享快取，同樣的想法需要分散式鎖，或者 `stale-while-revalidate`，這樣等待者可以提供過時的值而不是阻塞。

### 快取檢查清單
- [ ] 快取的調用首先被測量為昂貴 (快取一個快速的調用會增加一次跳躍，但沒有帶來任何益處)
- [ ] 讀寫比證明了快取 (讀取比寫入多得多)
- [ ] 快取鍵包含所有影響響應的輸入：租戶、檢視者、地區、權限、功能標誌
- [ ] 不在一個不識別使用者的鍵下快取單用戶資料
- [ ] 選擇一個失效策略 (TTL、事件/標籤或版本化鍵)，而不是意外的混合
- [ ] 寫下可接受的陳舊窗口，而不是由所輸入的 TTL 所暗示的
- [ ] 對熱鍵進行湧入防護 (合併、鎖定或 `stale-while-revalidate`)
- [ ] 使用較短的 TTL 快取負結果；絕不快取來源錯誤
- [ ] 設定淘汰策略和記憶體上限 (無上限的快取是記憶體洩漏)
- [ ] 監控命中率 — 沒有人測量的快取是假設，低命中率純粹是額外開銷
- [ ] 不快取任何其陳舊性是正確性錯誤的內容 (結算、權限、結帳庫存)

## 測量指令

### INP 欄位資料和 DevTools 工作流程

1. **首先是欄位資料** — 在優化之前，請檢查 [CrUX Vis](https://developer.chrome.com/docs/crux/vis) 或您的 RUM 工具以獲取真實用戶的 INP
2. **識別慢互動** — 打開 DevTools → 效能面板 → 在互動時記錄；查找由點擊/鍵盤輸入觸發的長時間任務
3. **在中等 Android 上測試** — INP 問題通常只在較慢的硬體上才會出現；使用真實設備或 DevTools CPU 節流 (4×–6× 減速)

```bash
# Lighthouse CLI
npx lighthouse https://localhost:3000 --output json --output-path ./report.json

# 捆綁分析
npx webpack-bundle-analyzer stats.json
# 或對於 Vite:
npx vite-bundle-visualizer

# 檢查捆綁大小
npx bundlesize

# 代碼中的 Web Vitals
import { onLCP, onINP, onCLS } from 'web-vitals';
onLCP(console.log);
onINP(console.log);
onCLS(console.log);

# 帶有互動級別細節的 INP (歸因建構)
import { onINP } from 'web-vitals/attribution';
onINP(({ value, attribution }) => {
  const { interactionTarget, inputDelay, processingDuration, presentationDelay } = attribution;
  console.log({ value, interactionTarget, inputDelay, processingDuration, presentationDelay });
});
```

## 常見反模式

| 反模式 | 影響 | 解決方案 |
|---|---|---|
| N+1 查詢 | 線性地增加資料庫負載 | 使用連接、包含或批次載入 |
| 無上限查詢 | 記憶體耗盡、超時 | 始終分頁，添加 `LIMIT` |
| 缺少索引 | 隨著資料增長，讀取變慢 | 為過濾/排序的欄位添加索引 |
| 在不讀取計畫的情況下建立索引 | 支付了寫入成本，但讀取收益未被證明 | 在前後使用 `EXPLAIN ANALYZE`；如果計畫沒有更改，則撤銷 |
| 多餘/未使用的索引 | 每次寫入都為它們付費 | 審核使用統計，刪除無用索引 |
| 每請求一個連接池 | 在負載下耗盡 `max_connections` | 每一個進程一個連接池；無伺服器使用代理 |
| 快取鍵缺少檢視者 | 一個使用者的資料被服務給另一個用戶 | 鍵應基於租戶、檢視者、地區、權限 |
| 無上限快取 | 穿著優化的記憶體洩漏 | 設定淘汰策略和記憶體上限 |
| 熱鍵的快取湧入 | 來源在過期時承受全部並行負載 | 合併未命中，或使用 `stale-while-revalidate` |
| 佈局抖動 | 卡頓，丟失幀 | 批處理 DOM 讀取，然後批處理寫入 |
| 未優化的影像 | 慢的 LCP，浪費頻寬 | 使用 WebP、響應式尺寸、延遲載入 |
| 大型捆綁包 | 緩慢的互動時間 | 代碼分割、樹搖動、審核依賴項 |
| 阻塞主線程 | 差的 INP，無響應的 UI | 使用 `scheduler.yield()` / `yieldToMain` 分割長時間任務，卸載到 Web Worker |
| 記憶體洩漏 | 記憶體不斷增長，最終崩潰 | 清理監聽器、間隔、引用 |
