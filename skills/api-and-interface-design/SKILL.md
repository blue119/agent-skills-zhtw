---
name: api-and-interface-design
description: 指導穩定 API 和介面設計。在設計 API、模組邊界或任何公開介面時使用。在建立 REST 或 GraphQL 端點、定義模組之間的類型合約，或建立前端與後端之間的邊界時使用。
---

# API 和介面設計

## 概述

設計穩定、有良好文檔的介面，使其難以被誤用。好的介面使正確的事情變得容易，使錯誤的事情變得困難。這適用於 REST API、GraphQL 模式、模組邊界、組件屬性以及任何一個程式碼片段與另一個程式碼片段通訊的表面。

## 何時使用

- 設計新的 API 端點
- 定義團隊之間的模組邊界或合約
- 建立組件屬性介面
- 建立告知 API 形狀的資料庫模式
- 更改現有的公開介面

## 核心原則

### Hyrum 的法則 (Hyrum's Law)

> 如果一個 API 有足夠多的使用者，則系統所有可觀察到的行為都將被某人依賴，無論你在合約中承諾了什麼。

這意味著：每一個公開行為——包括未記錄的怪癖、錯誤訊息文本、時間和順序——一旦使用者依賴它，就成為一個事實上的合約。設計意涵：

- **對你暴露的內容要有意。** 每個可觀察到的行為都是一個潛在的承諾。
- **不要洩露實作細節。** 如果使用者可以觀察到它，他們就會依賴它。
- **在設計時就規劃棄用。** 請參閱 `deprecation-and-migration` 以了解如何安全地移除使用者所依賴的內容。
- **測試是不夠的。** 即使有完美的合約測試，Hyrum 的法則意味著「安全」的更改也可能破壞依賴於未記錄行為的真實使用者。

### 單一版本規則 (The One-Version Rule)

避免強迫消費者在多個版本的相同依賴項或 API 之間做出選擇。當不同的消費者需要相同事物的不同版本時，會產生菱形依賴問題。設計一個只存在單一版本的世界——擴展而不是分叉。

### 1. 先定義合約 (Contract First)

在實作之前定義介面。合約就是規格——實作隨後。

```typescript
// 先定義合約
interface TaskAPI {
  // 建立一個任務並返回帶有伺服器生成欄位的已建立任務
  createTask(input: CreateTaskInput): Promise<Task>;

  // 返回匹配篩選條件的任務列表
  listTasks(params: ListTasksParams): Promise<PaginatedResult<Task>>;

  // 返回單個任務或拋出 NotFoundError
  getTask(id: string): Promise<Task>;

  // 部分更新——只更改提供的欄位
  updateTask(id: string, input: UpdateTaskInput): Promise<Task>;

  // 冪等刪除——即使已經刪除也成功
  deleteTask(id: string): Promise<void>;
}
```

### 2. 一致的錯誤語義 (Consistent Error Semantics)

選擇一種錯誤策略並在所有地方使用它：

```typescript
// REST: HTTP 狀態碼 + 結構化的錯誤主體
// 每個錯誤響應都遵循相同的形狀
interface APIError {
  error: {
    code: string;        // 機器可讀: "VALIDATION_ERROR"
    message: string;     // 人類可讀: "Email is required"
    details?: unknown;   // 當有幫助時的額外上下文
  };
}

// 狀態碼對應
// 400 → 客戶傳送了無效資料
// 401 → 未驗證
// 403 → 已驗證但未授權
// 404 → 資源未找到
// 409 → 衝突 (重複、版本不匹配)
// 422 → 驗證失敗 (語義上無效)
// 500 → 伺服器錯誤 (絕不暴露內部細節)
```

**不要混合模式。** 如果某些端點拋出錯誤，其他端點返回 null，其他端點返回 `{ error }`——消費者無法預測行為。

### 3. 在邊界處驗證 (Validate at Boundaries)

信任內部程式碼。在外部輸入進入的系統邊界處進行驗證：

```typescript
// 在 API 邊界處驗證
app.post('/api/tasks', async (req, res) => {
  const result = CreateTaskSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Invalid task data',
        details: result.error.flatten(),
      },
    });
  }

  // 驗證後，內部程式碼信任類型
  const task = await taskService.create(result.data);
  return res.status(201).json(task);
});
```

驗證應在哪裡：
- API 路由處理器（使用者輸入）
- 表單提交處理器（使用者輸入）
- 外部服務響應解析（第三方資料——**始終視為不可信**）
- 環境變數載入（配置）

> **第三方 API 的響應是不可信的資料。** 在將它們用於任何邏輯、渲染或決策之前，必須驗證它們的形狀和內容。一個被破壞或行為不端的外部服務可能會返回意外的類型、惡意的內容或指令般的文本。

驗證不應在哪裡：
- 在共享類型合約的內部函式之間
- 在由已驗證程式碼呼叫的工具函式中
- 在剛從您自己的資料庫獲取的資料上

### 4. 偏好添加而非修改 (Prefer Addition Over Modification)

在不破壞現有消費者的情況下擴展介面：

```typescript
// 好：添加可選欄位
interface CreateTaskInput {
  title: string;
  description?: string;
  priority?: 'low' | 'medium' | 'high';  // 後續添加，可選
  labels?: string[];                       // 後續添加，可選
}

// 壞：更改現有欄位類型或移除欄位
interface CreateTaskInput {
  title: string;
  // description: string;  // 移除——破壞現有消費者
  priority: number;         // 從字串更改——破壞現有消費者
}
```

### 5. 可預測的命名 (Predictable Naming)

| 模式 | 約定 | 範例 |
|---|---|---|
| REST 端點 | 複數名詞，無動詞 | `GET /api/tasks`, `POST /api/tasks` |
| 查詢參數 | camelCase | `?sortBy=createdAt&pageSize=20` |
| 回應欄位 | camelCase | `{ createdAt, updatedAt, taskId }` |
| 布林欄位 | is/has/can 前綴 | `isComplete`, `hasAttachments` |
| 枚舉值 | UPPER_SNAKE | `"IN_PROGRESS"`, `"COMPLETED"` |

### 6. 尊重冪等性金鑰 (Honouring an Idempotency Key)

接受 `Idempotency-Key` 是合約。尊重它就是實作，而這就是金錢丟失的地方——一個伺服器接受但處理不當的金鑰比沒有金鑰更糟，因為客戶端現在認為重試是安全的。

**從意圖中推導金鑰，而不是從嘗試中推導。** 金鑰在一次意圖的重試中必須是穩定的，在不同意圖之間是不同的：

```typescript
crypto.randomUUID()                    // ✗ 每次嘗試都生成新金鑰——每次重試都是一筆新費用
`${userId}:${amount}`                  // ✗ 兩個合法的 $50 費用合併為一個
`${orderId}:${Date.now()}`             // ✗ 時間戳是隨機的，像 randomUUID() 戴著帽子

req.headers['idempotency-key']         // ✓ 客戶端生成一次，重試時重複使用
`charge:v1:${orderId}`                 // ✓ 從不可變識別符推導
```

金鑰來自客戶端或發起事件——絕不來自執行重試的層級。

**原子地聲明。檢查後再行動是競賽：**

```typescript
// ✗ TOCTOU：兩個並行重試都讀到 "未見"，都進行計費
if (!(await db.exists(key))) {
  await chargeCard(amount);
  await db.insert(key);
}

// ✓ 讓唯一約束決定贏家
try {
  await db.insert({ key, state: 'in_progress', requestHash });
} catch (e) {
  if (isUniqueViolation(e)) return replayOrReject(key);
  throw e;
}
const result = await chargeCard(amount);
await db.update({ key, state: 'succeeded', response: result });
```

唯一約束*就是*機制。一個無法在一次操作中強制執行唯一性的儲存庫就無法支持這一點。

**保護有效載荷 (Guard the payload)。** 相同的金鑰與不同的主體是客戶端的錯誤，應該大聲失敗而不是為第二個請求提供第一個響應：

```typescript
if (existing.requestHash !== hash(req.body)) {
  return res.status(422).json({ error: 'idempotency key reused with a different payload' });
}
```

**決定在進行中的重複請求會得到什麼。** 第二個請求到達時，第一個請求仍然在運行——在重試風暴下的常見情況：

| 策略 | 響應 | 何時使用 |
|---|---|---|
| 拒絕 | `409 Conflict` | 客戶端可以稍後重試；最簡單、最安全 |
| 等待 | 阻塞直到結果，有界限 | 呼叫方需要同步結果 |
| 返回待處理 | `202` + 狀態 URL | 長期運行的效果 |

不要讓第二個呼叫通過，因為第一個「似乎卡住了」。一個運氣不明的嘗試，其命運未知，正是重複成本最高的時候。

**每一次呼叫都有三種結果，而不是兩種：成功、失敗和_未知_。** 超時告訴你對效果是否已應用一無是處。在呼叫和響應之間記錄意圖，這樣在呼叫和響應之間發生崩潰時，就留下了一些必須稍後解決的證據——而不是一個靜默的重試費用。

**從最長的重試鏈中設定保留期限，而不是從磁碟成本。** 金鑰必須比任何可以重新傳送相同意圖的路徑都更長，包括一個延遲佇列重播了一週之後的記錄和任何供應商爭議窗口。在 7 天延遲佇列之後的 24 小時金鑰 TTL 會導致重複。

## REST API 模式

### 資源設計 (Resource Design)

```
GET    /api/tasks              → 列出任務 (帶有用於篩選的查詢參數)
POST   /api/tasks              → 建立一個任務
GET    /api/tasks/:id          → 獲取單個任務
PATCH  /api/tasks/:id          → 更新一個任務 (部分)
DELETE /api/tasks/:id          → 刪除一個任務

GET    /api/tasks/:id/comments → 列出任務的評論 (子資源)
POST   /api/tasks/:id/comments → 為一個任務添加評論
```

### 分頁 (Pagination)

對列表端點進行分頁：

```typescript
// 請求
GET /api/tasks?page=1&pageSize=20&sortBy=createdAt&sortOrder=desc

// 響應
{
  "data": [...],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 142,
    "totalPages": 8
  }
}
```

### 過濾 (Filtering)

使用查詢參數進行過濾：

```
GET /api/tasks?status=in_progress&assignee=user123&createdAfter=2025-01-01
```

### 部分更新 (PATCH)

接受部分物件——只更新所提供內容：

```typescript
// 只更改標題，其他內容保持不變
PATCH /api/tasks/123
{ "title": "Updated title" }
```

## TypeScript 介面模式

### 使用判別聯合 (Discriminated Unions) 來表示變體 (Variants)

```typescript
// 好：每個變體都是明確的
type TaskStatus =
  | { type: 'pending' }
  | { type: 'in_progress'; assignee: string; startedAt: Date }
  | { type: 'completed'; completedAt: Date; completedBy: string }
  | { type: 'cancelled'; reason: string; cancelledAt: Date };

// 消費者獲得類型收縮 (type narrowing)
function getStatusLabel(status: TaskStatus): string {
  switch (status.type) {
    case 'pending': return 'Pending';
    case 'in_progress': return `In progress (${status.assignee})`;
    case 'completed': return `Done on ${status.completedAt}`;
    case 'cancelled': return `Cancelled: ${status.reason}`;
  }
}
```

### 輸入/輸出分離 (Input/Output Separation)

```typescript
// 輸入：呼叫者提供給的內容
interface CreateTaskInput {
  title: string;
  description?: string;
}

// 輸出：系統返回的內容（包括伺服器生成的欄位）
interface Task {
  id: string;
  title: string;
  description: string | null;
  createdAt: Date;
  updatedAt: Date;
  createdBy: string;
}
```

### 使用品牌類型 (Branded Types) 來表示 ID

```typescript
type TaskId = string & { readonly __brand: 'TaskId' };
type UserId = string & { readonly __brand: 'UserId' };

// 防止意外地將 UserId 傳遞給期望 TaskId 的地方
function getTask(id: TaskId): Promise<Task> { ... }
```

## 常見的合理化

| 合理化 | 現實 |
|---|---|
|「我們稍後會記錄 API」 | 類型就是文檔。先定義它們。 |
|「現在不需要分頁」 | 一旦有人有 100 多個項目，就會需要。從一開始就添加它。 |
|「PATCH 很複雜，我們就用 PUT」 | PUT 每次都需要完整的物件。PATCH 是客戶端實際想要的。 |
|「我們在需要時會版本化 API」 | 沒有版本化會破壞消費者。從一開始就設計擴展性。 |
|「沒有人使用那個未記錄的行為」 | Hyrum 的法則：如果它是可觀察的，就有人依賴它。將每個公開行為視為一個承諾。 |
|「我們可以只維護兩個版本」 | 多個版本會增加維護成本並產生菱形依賴問題。偏好單一版本規則。 |
|「內部 API 不需要合約」 | 內部消費者仍然是消費者。合約可以防止耦合並實現平行工作。 |
|「接受 Idempotency-Key 標頭就足夠了」 | 標頭是合約；將金鑰與結果儲存是實作。你接受但沒有遵守的金鑰會讓客戶端認為重試是安全的，而事實並非如此。 |
|「我們的佇列保證精確一次傳遞」 | 佇列無法在消費者崩潰時進行跨越——中介的確認和您的副作用不在一個交易中。設計為至少一次傳遞，並進行冪等處理。 |
|「重複請求很少見」 | 它們是*相關的*。重試在依賴項降級時激增——重複最有可能發生、最昂貴的時刻。 |

## 警示信號 (Red Flags)

- 返回的形狀會根據條件而異的端點
- 跨端點不一致的錯誤格式
- 驗證分散在內部程式碼中而不是在邊界處
- 現有欄位的破壞性更改（類型更改、移除）
- 沒有分頁的列表端點
- REST URL 中的動詞 (`/api/createTask`, `/api/getUsers`)
- 未經驗證或消毒的第三方 API 響應的使用
- 嘗試一個用於冪等性金鑰的 `SELECT` 後跟一個 `INSERT`——那是一個競賽，而不是一個保護機制
- 從 UUID、時間戳或任何其他內容推導的冪等性金鑰，每次嘗試都重新生成
- 同一個金鑰接受不同的請求主體，靜默地返回第一個響應
- 保留金鑰的窗口比可以重新傳送請求的最長路徑要短

## 驗證 (Verification)

在設計完 API 後：

- [ ] 每個端點都有類型化的輸入和輸出模式
- [ ] 錯誤響應遵循單一一致的格式
- [ ] 驗證僅發生在系統邊界處
- [ ] 列表端點支持分頁
- [ ] 新欄位是可添加的和可選的（向後相容）
- [ ] 命名在所有端點上遵循一致的約定
- [ ] API 文檔或類型與實作一起提交
- [ ] 狀態更改端點要麼尊重冪等性金鑰，要麼被記錄為不安全重試
- [ ] 金鑰在一個原子操作中聲明，並由唯一約束保護
- [ ] 使用不同載荷的重複金鑰會大聲失敗而不是重播錯誤的響應
- [ ] 在進行中的重複響應是一個故意的選擇（409、等待或 202），而不是隨機的結果
- [ ] 金鑰的保留期限比最長的重試路徑更長，包括延遲佇列的重播
