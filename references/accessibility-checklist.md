# 無障礙性檢查清單

WCAG 2.1 AA 合規性的快速參考。請與 `frontend-ui-engineering` 技能一起使用。

## 目錄

- [基本檢查](#essential-checks)
- [常見 HTML 模式](#common-html-patterns)
- [測試工具](#testing-tools)
- [快速參考：ARIA Live 區域](#quick-reference-aria-live-regions)
- [常見反模式](#common-anti-patterns)

## 基本檢查

### 鍵盤導航
- [ ] 所有互動元素都可透過 Tab 鍵聚焦
- [ ] 焦點順序遵循視覺/邏輯順序
- [ ] 焦點可見（聚焦元素有輪廓/環）
- [ ] 自訂元件具有鍵盤支援（Enter 啟用，Escape 關閉）
- [ ] 沒有鍵盤陷阱（使用者可以隨時 Tab 離開元件）
- [ ] 頁面頂部的跳轉至內容連結 - 在鍵盤聚焦時可見（至少）
- [ ] 模態框在開啟時會鎖定焦點，關閉時將焦點還原

### 螢幕閱讀器
- [ ] 所有圖片都有 `alt` 文字（或用於裝飾圖片的 `alt=""`）
- [ ] 所有表單輸入都有相關的標籤（`<label>` 或 `aria-label`）
- [ ] 按鈕和連結有描述性文字（而不是「點擊這裡」）
- [ ] 僅有圖標的按鈕有 `aria-label`
- [ ] 頁面只有一個 `<h1>` 標題，標題之間不跳級
- [ ] 動態內容會被宣告（`aria-live` 區域）
- [ ] 表格有具有範圍的 `<th>` 標頭

### 視覺
- [ ] 文字對比度 $\ge$ 4.5:1（常規文字）或 $\ge$ 3:1（大文字，18px+）
- [ ] UI 元件與背景的對比度 $\ge$ 3:1
- [ ] 顏色不是傳達資訊的唯一方式
- [ ] 文字可放大至 200% 而不破壞佈局
- [ ] 沒有內容每秒閃爍超過 3 次

### 表單
- [ ] 每個輸入都有可見的標籤
- [ ] 必填欄位有標示（不僅僅是顏色）
- [ ] 錯誤訊息具體且與該欄位相關
- [ ] 錯誤狀態可透過更多資訊（圖標、文字、邊框）顯示
- [ ] 表單提交錯誤被總結並可聚焦
- [ ] 已知的欄位使用自動完成（例如 `type="email" autocomplete="email"`）

### 內容
- [ ] 宣告了語言（`<html lang="en">`）
- [ ] 頁面有描述性的 `<title>`
- [ ] 連結與周圍文字區分開（不僅僅是顏色）
- [ ] 行動區（在行動裝置上）$\ge$ 44x44px
- [ ] 有意義的空狀態（而不是空白畫面）

## 常見 HTML 模式

### 按鈕與連結

```html
<!-- 使用 <button> 進行動作 -->
<button onClick={handleDelete}>刪除任務</button>

<!-- 使用 <a> 進行導航 -->
<a href="/tasks/123">查看任務</a>

<!-- 絕不使用 div/span 作為按鈕 -->
<div onClick={handleDelete}>刪除</div>  <!-- 錯誤 -->
```

### 表單標籤

```html
<!-- 明確的標籤關聯 -->
<label htmlFor="email">電子郵件地址</label>
<input id="email" type="email" required />

<!-- 隱含的包裝 -->
<label>
  電子郵件地址
  <input type="email" required />
</label>

<!-- 隱藏的標籤（首選可見標籤） -->
<input type="search" aria-label="搜尋任務" />
```

### ARIA 角色

```html
<!-- 導航 -->
<nav aria-label="主要導航">...</nav>
<nav aria-label="頁腳連結">...</nav>

<!-- 狀態訊息 -->
<div role="status" aria-live="polite">任務已儲存</div>

<!-- 警報訊息 -->
<div role="alert">錯誤：需要標題</div>

<!-- 模態框對話框 -->
<dialog aria-modal="true" aria-labelledby="dialog-title">
  <h2 id="dialog-title">確認刪除</h2>
  ...
</dialog>

<!-- 載入狀態 -->
<div aria-busy="true" aria-label="載入任務">
  <Spinner />
</div>
```

### 可存取列表

```html
<ul role="list" aria-label="任務">
  <li>
    <input type="checkbox" id="task-1" aria-label="完成：購買雜貨">
    <label htmlFor="task-1">購買雜貨</label>
  </li>
</ul>
```

## 測試工具

```bash
# 自動化審核
npx axe-core          # 程式化無障礙性測試
npx pa11y             # CLI 無障礙性檢查器

# 在瀏覽器中
# Chrome DevTools → Lighthouse → 無障礙性
# Chrome DevTools → Elements → 無障礙性樹

# 螢幕閱讀器測試
# macOS: VoiceOver (Cmd + F5)
# Windows: NVDA (免費) 或 JAWS
# Linux: Orca
```

## 快速參考：ARIA Live 區域

| 值 | 行為 | 用於 |
|---|---|---|
| `aria-live="polite"` | 在下一個暫停時宣告 | 狀態更新、儲存確認 |
| `aria-live="assertive"` | 立即宣告 | 錯誤、時間敏感的警報 |
| `role="status"` | 與 `polite` 相同 | 狀態訊息 |
| `role="alert"` | 與 `assertive` 相同 | 錯誤訊息 |

## 常見反模式

| 反模式 | 問題 | 解決方案 |
|---|---|---|
| 使用 `div` 作為按鈕 | 無法聚焦，沒有鍵盤支援 | 使用 `<button>` |
| 缺少 `alt` 文字 | 螢幕閱讀器無法看到圖片 | 添加描述性的 `alt` |
| 僅使用顏色狀態 | 對色盲使用者不可見 | 添加圖標、文字或圖案 |
| 自動播放媒體 | 令人困惑，無法停止 | 添加控制項，不要自動播放 |
| 沒有 ARIA 的自訂下拉式選單 | 無法透過鍵盤/螢幕閱讀器使用 | 使用原生的 `<select>` 或適當的 ARIA listbox |
| 移除焦點輪廓 | 使用者看不到他們的位置 | 設定輪廓，不要移除它們 |
| 空連結/按鈕 | 宣告為「連結」但沒有描述 | 添加文字或 `aria-label` |
| `tabindex > 0` | 打破自然 Tab 順序 | 只使用 `tabindex="0"` 或 `-1` |
