---
name: frontend-ui-engineering
description: 建構生產級、無障礙、響應式的使用者介面。在建立或修改介面和頁面、建立元件、實作佈局、滿足 WCAG 無障礙要求、管理狀態，或當輸出需要看起來和感覺像生產級而非 AI 生成時使用。
---

# 前端 UI 工程

## 概述

建立可存取、高效且視覺上精美的使用者介面。目標是產生看起來像是由具備設計意識的工程師在頂尖公司所建立的 UI — 而非由 AI 生成的。這意味著真正的設計系統遵循、適當的無障礙性、深思熟慮的互動模式，以及沒有通用的「AI 美學」。

## 何時使用

- 建構新的 UI 元件或頁面
- 修改現有的使用者介面
- 實作響應式佈局
- 添加互動性或狀態管理
- 修正視覺或使用者體驗問題

## 元件架構

### 文件結構

將所有與元件相關的內容放在一起：

```
src/components/
  TaskList/
    TaskList.tsx          # 元件實作
    TaskList.test.tsx     # 測試
    TaskList.stories.tsx  # Storybook 故事 (如果使用)
    use-task-list.ts      # 自訂 Hook (如果狀態複雜)
    types.ts              # 元件特定的類型 (如果需要)
```

### 元件模式

**偏好組合而非配置：**

```tsx
// 好：可組合
<Card>
  <CardHeader>
    <CardTitle>任務</CardTitle>
  </CardHeader>
  <CardBody>
    <TaskList tasks={tasks} />
  </CardBody>
</Card>

// 避免：配置過多
<Card
  title="任務"
  headerVariant="large"
  bodyPadding="md"
  content={<TaskList tasks={tasks} />}
/>
```

**保持元件的專注性：**

```tsx
// 好：只做一件事
export function TaskItem({ task, onToggle, onDelete }: TaskItemProps) {
  return (
    <li className="flex items-center gap-3 p-3">
      <Checkbox checked={task.done} onChange={() => onToggle(task.id)} />
      <span className={task.done ? 'line-through text-muted' : ''}>{task.title}</span>
      <Button variant="ghost" size="sm" onClick={() => onDelete(task.id)}>
        <TrashIcon />
      </Button>
    </li>
  );
}
```

**將資料獲取與呈現分開：**

```tsx
// 容器：處理資料
export function TaskListContainer() {
  const { tasks, isLoading, error } = useTasks();

  if (isLoading) return <TaskListSkeleton />;
  if (error) return <ErrorState message="載入任務失敗" retry={refetch} />;
  if (tasks.length === 0) return <EmptyState message="尚未有任務" />;

  return <TaskList tasks={tasks} />;
}

// 呈現：處理渲染
export function TaskList({ tasks }: { tasks: Task[] }) {
  return (
    <ul role="list" className="divide-y">
      {tasks.map(task => <TaskItem key={task.id} task={task} />)}
    </ul>
  );
}
```

## 狀態管理

**選擇最簡單可行的方法：**

```
本地狀態 (useState)           → 元件特定的 UI 狀態
提升狀態 (Lifted state)         → 在 2-3 個兄弟元件之間共享
Context                          → 主題、認證、地區 (讀取多，寫入少)
URL 狀態 (searchParams)         → 過濾、分頁、可分享的 UI 狀態
伺服器狀態 (React Query, SWR)  → 帶有快取的遠端資料
全域儲存 (Zustand, Redux)    → 跨應用程式共享的複雜客戶端狀態
```

**避免深度超過 3 層的屬性傳遞 (prop drilling)。** 如果您透過元件傳遞屬性，而這些元件不使用它們，請引入 Context 或重新組織元件樹。

## 設計系統遵循

### 基於參考的 UI 品質

當產品需要明確的視覺方向時，在選擇佈局之前收集證據：

1. 在受信任的參考目錄中搜尋，或使用產品團隊提供的參考。
2. 研究兩個或三個相關的螢幕。記錄關於層次、密度、導航、控制、響應式行為和互動狀態的決策。
3. 在實作之前，將這些決策轉化為一份簡短的設計合約。命名螢幕的職責、主要操作、所需狀態、響應式規則和需要拒絕的模式。
4. 在產品自身的元件、主題、內容和視覺語言中重建有用的結構。絕不複製其他產品的品牌標誌、專有文本、圖像或確切佈局。

將參考作為證據，而非模板。如果參考不可用，請記錄假設並根據產品現有的設計系統驗證結果。

### 避免 AI 美學

AI 生成的 UI 具有可識別的模式。避免所有這些模式：

| AI 預設 | 為什麼是問題 | 生產品質 |
|---|---|---|
| 所有紫色/靛藍色 | 模型預設視覺上「安全」的調色板，使每個應用程式看起來都一樣 | 使用專案的實際色板 |
| 過多的漸層 | 漸層會增加視覺雜訊，並與大多數設計系統產生衝突 | 與設計系統匹配的扁平或細微漸層 |
| 所有圓角 (rounded-2xl) | 最大化圓角表示「友善」，但忽略了真實設計中角落半徑的層次關係 | 來自設計系統的一致邊框半徑 |
| 通用的英雄區塊 | 沒有與實際內容或使用者需求關聯的模板化佈局 | 以內容為先的佈局 |
| Lorem ipsum 風的文字 | 佔位符文本會隱藏真實內容揭示的佈局問題（長度、換行、溢出） | 逼真的佔位符內容 |
| 每個地方過大的內邊距 | 相等的寬裕內邊距破壞了視覺層次結構並浪費螢幕空間 | 一致的間距尺度 |
| 庫存卡片網格 | 單一的網格是忽略資訊優先順序和掃描模式的佈局捷徑 | 以目的為導向的佈局 |
| 陰影過多的設計 | 層疊的陰影增加了深度，與內容競爭，並在低階設備上減慢渲染速度 | 除非設計系統指定，否則不使用陰影 |

### 間距與佈局

使用一致的間距尺度。不要憑空發明數值：

```css
/* 使用尺度：0.25rem 的增量 (或專案使用的任何值) */
/* 好 */  padding: 1rem;      /* 16px */
/* 好 */  gap: 0.75rem;       /* 12px */
/* 壞 */   padding: 13px;      /* 不在任何尺度上 */
/* 壞 */   margin-top: 2.3rem; /* 不在任何尺度上 */
```

### 字體

尊重字體層次結構：

```
h1 → 頁面標題 (每頁一個)
h2 → 區段標題
h3 → 子區段標題
body → 預設文本
small → 次要/輔助文本
```

不要跳過標題層級。不要將標題樣式用於非標題內容。

### 顏色

- 使用語義顏色標記：`text-primary`、`bg-surface`、`border-default` — 而非原始十六進制值
- 確保足夠的對比度 (普通文本 4.5:1，大文本 3:1)
- 不要僅依賴顏色來傳達資訊 (也使用圖示、文本或模式)

## 無障礙性 (WCAG 2.1 AA)

每個元件都必須滿足這些標準：

### 鍵盤導航

```tsx
// 每個互動元素都必須可由鍵盤存取
<button onClick={handleClick}>點擊我</button>        // ✓ 預設可聚焦
<div onClick={handleClick}>點擊我</div>               // ✗ 不可聚焦
<div role="button" tabIndex={0} onClick={handleClick}    // ✓ 但請使用 <button>
     onKeyDown={e => {
       if (e.key === 'Enter') handleClick();
       if (e.key === ' ') e.preventDefault();
     }}
     onKeyUp={e => {
       if (e.key === ' ') handleClick();
     }}>
  點擊我
</div>
```

### ARIA 標籤

```tsx
// 為缺乏可見文本的互動元素添加標籤
<button aria-label="關閉對話框"><XIcon /></button>

// 為表單輸入添加標籤
<label htmlFor="email">電子郵件</label>
<input id="email" type="email" />

// 或者在沒有可見標籤時使用 aria-label
<input aria-label="搜尋任務" type="search" />
```

### 焦點管理

```tsx
// 當內容改變時移動焦點
function Dialog({ isOpen, onClose }: DialogProps) {
  const closeRef = useRef<HTMLButtonElement>(null);

  useEffect(() => {
    if (isOpen) closeRef.current?.focus();
  }, [isOpen]);

  // 當打開時鎖定焦點在對話框內
  return (
    <dialog open={isOpen}>
      <button ref={closeRef} onClick={onClose}>關閉</button>
      {/* 對話框內容 */}
    </dialog>
  );
}
```

### 有意義的空狀態和錯誤狀態

```tsx
// 不要顯示空白螢幕
function TaskList({ tasks }: { tasks: Task[] }) {
  if (tasks.length === 0) {
    return (
      <div role="status" className="text-center py-12">
        <TasksEmptyIcon className="mx-auto h-12 w-12 text-muted" />
        <h3 className="mt-2 text-sm font-medium">沒有任務</h3>
        <p className="mt-1 text-sm text-muted">開始建立新任務。</p>
        <Button className="mt-4" onClick={onCreateTask}>建立任務</Button>
      </div>
    );
  }

  return <ul role="list">...</ul>;
}
```

## 響應式設計

以行動裝置優先設計，然後擴展：

```tsx
// Tailwind：行動裝置優先響應式
<div className="
  grid grid-cols-1      /* 行動裝置：單欄 */
  sm:grid-cols-2        /* 小螢幕：2欄 */
  lg:grid-cols-3        /* 大螢幕：3欄 */
  gap-4
">
```

在這些斷點上進行測試：320px、768px、1024px、1440px。

## 載入與過渡

```tsx
// 骨架載入 (非內容的旋轉圖示)
function TaskListSkeleton() {
  return (
    <div className="space-y-3" aria-busy="true" aria-label="載入任務">
      {Array.from({ length: 3 }).map((_, i) => (
        <div key={i} className="h-12 bg-muted animate-pulse rounded" />
      ))}
    </div>
  );
}

// 為了感知速度而進行樂觀更新
function useToggleTask() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: toggleTask,
    onMutate: async (taskId) => {
      await queryClient.cancelQueries({ queryKey: ['tasks'] });
      const previous = queryClient.getQueryData(['tasks']);

      queryClient.setQueryData(['tasks'], (old: Task[]) =>
        old.map(t => t.id === taskId ? { ...t, done: !t.done } : t)
      );

      return { previous };
    },
    onError: (_err, _taskId, context) => {
      queryClient.setQueryData(['tasks'], context?.previous);
    },
  });
}
```

## 參考資料

有關詳細的無障礙要求和測試工具，請參閱 `../../references/accessibility-checklist.md`。

## 常見合理化

| 合理化 | 現實 |
|---|---|
|「無障礙性是可有可無的」 | 在許多司法管轄區，它是一項法律要求和工程品質標準。 |
|「我們稍後會處理響應式設計」 | 後期添加響應式設計比從一開始建構要困難 3 倍。 |
|「設計尚未定稿，所以我會跳過樣式」 | 使用設計系統的預設值。無樣式 UI 會給審閱者留下糟糕的第一印象。 |
|「這只是一個原型」 | 原型會變成生產代碼。從一開始就建立基礎。 |
|「目前的 AI 美學暫時可以」 | 它暗示著低品質。從一開始就使用專案的實際設計系統。 |

## 警示標誌

- 擁有超過 200 行的元件（請拆分它們）
- 行內樣式或任意像素值
- 缺少錯誤狀態、載入狀態或空狀態
- 沒有鍵盤導航測試
- 僅依賴顏色作為狀態的唯一指示器（無文本或圖示的紅/綠色）
- 通用的「AI 外觀」（紫色漸層、過大的卡片、庫存佈局）

## 驗證

在建構 UI 後：

- [ ] 元件在控制台中沒有錯誤渲染
- [ ] 所有互動元素都可由鍵盤存取（按 Tab 遍歷整個頁面）
- [ ] 螢幕閱讀器可以傳達頁面的內容和結構
- [ ] 響應式：在 320px、768px、1024px、1440px 上運行正常
- [ ] 在適用情況下處理載入、空、錯誤、成功和權限狀態
- [ ] 遵循專案的設計系統（間距、顏色、字體）
- [ ] 渲染的結果通過最終的 UI 特定完成審查
- [ ] 開發工具或 axe-core 中沒有無障礙警告
