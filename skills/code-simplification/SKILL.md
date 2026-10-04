---
name: code-simplification
description: 簡化程式碼以提高清晰度。在不改變行為的情況下使用。當程式碼雖然能運作，但比它應該的更難閱讀、維護或擴展時使用。當審查累積了不必要複雜度的程式碼時使用。
---

# 程式碼簡化

> 受到 [Claude Code Simplifier 插件](https://github.com/anthropics/claude-plugins-official/blob/main/plugins/code-simplifier/agents/code-simplifier.md) 的啟發。在此處被改編為任何 AI 編程代理的無模型、流程驅動的技能。

## 概述

在保持完全相同的行為的同時減少複雜度。目標不是減少行數，而是讓程式碼更容易閱讀、理解、修改和除錯。每一次簡化都必須通過一個簡單的測試：「新團隊成員會比原始版本更快地理解它嗎？」

## 何時使用

- 功能已經運作且測試通過，但實作感覺比需要的多
- 在程式碼審查中，當標記出可讀性或複雜度問題時
- 當遇到深度嵌套的邏輯、冗長的函式或不清晰的名稱時
- 在時間壓力下重構程式碼時
- 當整合分散在各檔案中的相關邏輯時
- 在合併引入重複或不一致性的變更後

**不應使用時：**

- 程式碼已經乾淨且易於閱讀——不要為了簡化而簡化
- 你還不理解程式碼的運作方式——先理解再簡化
- 程式碼對效能至關重要，而「更簡單」的版本會明顯變慢
- 你正準備完全重寫模組——簡化暫時性的程式碼會浪費精力

## 五大原則

### 1. 精確保留行為

不要改變程式碼的運作方式——只改變它表達的方式。所有輸入、輸出、副作用、錯誤行為和邊界情況都必須保持一致。如果你不確定簡化是否保留了行為，就不要這麼做。

```
在每次更改前提問：
→ 這對所有輸入都產生相同的輸出嗎？
→ 這是否保持相同的錯誤行為？
→ 這是否保持相同的副作用和順序？
→ 所有現有的測試在不修改下是否仍然通過？
```

### 2. 遵循專案慣例

簡化意味著讓程式碼與程式碼庫保持一致，而不是強加外部偏好。在簡化之前：

```
1. 閱讀 CLAUDE.md / 專案慣例
2. 研究相鄰程式碼如何處理類似的模式
3. 遵循專案的風格，針對：
   - 導入順序和模組系統
   - 函式宣告風格
   - 名稱慣例
   - 錯誤處理模式
   - 型別註釋的深度
```

破壞專案一致性的簡化不是簡化——而是變動（churn）。

### 3. 偏好清晰而非巧妙

當簡潔的版本需要停頓思考才能解析時，明確的程式碼勝於緊湊的程式碼。

```typescript
// 不清晰：密集的三元運算符鏈
const label = isNew ? 'New' : isUpdated ? 'Updated' : isArchived ? 'Archived' : 'Active';

// 清晰：可讀的映射
function getStatusLabel(item: Item): string {
  if (item.isNew) return 'New';
  if (item.isUpdated) return 'Updated';
  if (item.isArchived) return 'Archived';
  return 'Active';
}
```

```typescript
// 不清晰：鏈式操作與內聯邏輯
const result = items.reduce((acc, item) => ({
  ...acc,
  [item.id]: { ...acc[item.id], count: (acc[item.id]?.count ?? 0) + 1 }
}), {});

// 清晰：命名中間步驟
const countById = new Map<string, number>();
for (const item of items) {
  countById.set(item.id, (countById.get(item.id) ?? 0) + 1);
}
```

### 4. 維持平衡

簡化有一個失敗模式：過度簡化。注意這些陷阱：

- **過度內聯**——移除提供概念名稱的輔助函式會使呼叫點更難閱讀
- **合併不相關的邏輯**——將兩個簡單的函式合併到一個複雜的函式中並不是更簡單
- **移除「不必要的」抽象**——有些抽象是為了可擴展性或可測試性而存在的，而不是複雜性
- **以行數優化**——減少行數不是目標；更容易理解才是目標

### 5. 僅限於變更的部分

預設是簡化最近修改的程式碼。除非明確要求擴大範圍，否則避免對不相關的程式碼進行「隨意重構」。無範圍的簡化會在差異中產生雜訊，並有潛在的意外回退。

## 簡化流程

### 步驟 1：在開始之前理解（切斯特頓的圍欄）

在更改或移除任何內容之前，理解它的存在原因。這是切斯特頓的圍欄：如果你在路上看到一堵圍欄，但不知道它在哪裡，就不要把它拆掉。首先理解原因，然後決定該原因是否仍然適用。

```
在簡化之前回答：
- 這段程式碼的職責是什麼？
- 哪些地方呼叫它？它呼叫了什麼？
- 邊界情況和錯誤路徑是什麼？
- 有沒有定義預期行為的測試？
- 為什麼它可能被這樣寫？（效能？平台限制？歷史原因？）
- 檢查 git blame：這段程式碼的原始上下文是什麼？
```

如果你不能回答這些問題，你就準備好簡化了。先閱讀更多上下文。

### 步驟 2：識別簡化的機會

掃描這些模式——每一個都是一個具體的信號，而不是模糊的氣味：

**結構複雜度：**

| 模式 | 信號 | 簡化 |
|---|---|---|
| 深度嵌套（3+層） | 控制流程難以跟蹤 | 將條件提取到保護子句或輔助函式中 |
| 長函式（50+行） | 多重職責 | 分割成具有描述性名稱的專注函式 |
| 嵌套的三元運算符 | 需要心智堆疊來解析 | 用 if/else 鏈、switch 或查找物件替換 |
| 布林參數標誌 | `doThing(true, false, true)` | 用選項物件或單獨的函式替換 |
| 重複的條件判斷 | 多個地方相同的 `if` 檢查 | 提取為一個命名良好的判斷函式 |

**命名和可讀性：**

| 模式 | 信號 | 簡化 |
|---|---|---|
| 通用名稱 | `data`, `result`, `temp`, `val`, `item` | 改名以描述內容：`userProfile`, `validationErrors` |
| 縮寫名稱 | `usr`, `cfg`, `btn`, `evt` | 除非縮寫是普遍的（`id`, `url`, `api`），否則使用全稱 |
| 誤導性名稱 | 名為 `get` 的函式也修改了狀態 | 改名以反映實際行為 |
| 用註釋解釋「什麼」（what） | 在 `count++` 上方的 `// increment counter` | 刪除註釋——程式碼本身就足夠清晰 |
| 用註釋解釋「為什麼」（why） | `// Retry because the API is flaky under load` | 保留這些註釋——它們承載了程式碼無法表達的意圖 |

**冗餘：**

| 模式 | 信號 | 簡化 |
|---|---|---|
| 重複的邏輯 | 多個地方相同的 5+ 行程式碼 | 提取到共用函式 |
| 無效程式碼 | 無法到達的分支、未使用的變數、註釋掉的區塊 | 移除（在確認它確實是無用的之後） |
| 不必要的抽象 | 增加無價值的包裝器 | 將包裝器內聯，直接呼叫底層函式 |
| 過度設計的模式 | 為工廠而工廠、策略與單一策略 | 用簡單的直接方法替換 |
| 多餘的型別斷言 | 轉換為已經推斷的型別 | 移除斷言 |

### 步驟 3：逐步應用更改

一次只進行一個簡化。在每次更改後運行測試。**將重構更改與功能或錯誤修復更改分開提交。** 包含重構和新增功能的 PR 是兩個 PR——將它們分開。

```
對於每一次簡化：
1. 進行更改
2. 運行測試套件
3. 如果測試通過 → 提交（或繼續進行下一個簡化）
4. 如果測試失敗 → 撤銷並重新考慮
```

避免將多個簡化批次化為一次未經測試的更改。如果某個東西出錯了，你需要知道是哪個簡化導致了錯誤。

**500 法則：** 如果重構會影響超過 500 行程式碼，請投資自動化（程式碼模式、sed 腳本、AST 轉換），而不是手動進行更改。在該規模上進行手動編輯是容易出錯且令人筋疲力盡的審查過程。

### 步驟 4：驗證結果

完成所有簡化後，退一步並評估整體：

```
比較前後：
- 簡化後的版本是否真正更容易理解？
- 你是否引入了與程式碼庫不一致的新模式？
- 差異是否乾淨且可審查？
- 同事是否會批准此更改？
```

如果「簡化」後的版本更難理解或審查，就撤銷。並非每一次簡化嘗試都能成功。

## 特定語言指導

### TypeScript / JavaScript

```typescript
// 簡化：不必要的 async 包裝器
// 之前
async function getUser(id: string): Promise<User> {
  return await userService.findById(id);
}
// 後
function getUser(id: string): Promise<User> {
  return userService.findById(id);
}

// 簡化：冗長的條件賦值
// 之前
let displayName: string;
if (user.nickname) {
  displayName = user.nickname;
} else {
  displayName = user.fullName;
}
// 後
const displayName = user.nickname || user.fullName;

// 簡化：手動陣列建構
// 之前
const activeUsers: User[] = [];
for (const user of users) {
  if (user.isActive) {
    activeUsers.push(user);
  }
}
// 後
const activeUsers = users.filter((user) => user.isActive);

// 簡化：多餘的布林返回值
// 之前
function isValid(input: string): boolean {
  if (input.length > 0 && input.length < 100) {
    return true;
  }
  return false;
}
// 後
function isValid(input: string): boolean {
  return input.length > 0 && input.length < 100;
}
```

### Python

```python
# 簡化：冗長的字典建構
# 之前
result = {}
for item in items:
    result[item.id] = item.name
# 後
result = {item.id: item.name for item in items}

# 簡化：帶有提前返回的嵌套條件判斷
# 之前
def process(data):
    if data is not None:
        if data.is_valid():
            if data.has_permission():
                return do_work(data)
            else:
                raise PermissionError("No permission")
        else:
            raise ValueError("Invalid data")
    else:
        raise TypeError("Data is None")
# 後
def process(data):
    if data is None:
        raise TypeError("Data is None")
    if not data.is_valid():
        raise ValueError("Invalid data")
    if not data.has_permission():
        raise PermissionError("No permission")
    return do_work(data)
```

### React / JSX

```tsx
// 簡化：冗長的條件渲染
// 之前
function UserBadge({ user }: Props) {
  if (user.isAdmin) {
    return <Badge variant="admin">Admin</Badge>;
  } else {
    return <Badge variant="default">User</Badge>;
  }
}
// 後
function UserBadge({ user }: Props) {
  const variant = user.isAdmin ? 'admin' : 'default';
  const label = user.isAdmin ? 'Admin' : 'User';
  return <Badge variant={variant}>{label}</Badge>;
}

// 簡化：通過中間組件傳遞屬性（Prop Drilling）
// 之前 — 考慮上下文或組合是否能更好地解決這個問題。
// 這是一個判斷問題——標記它，不要自動重構。
```

## 常見合理化

| 合理化 | 現實 |
|---|---|
|「它正在運作，無需動它」 | 難以閱讀的運作中的程式碼在出錯時會更難修復。現在簡化可以節省未來每次更改的時間。 |
|「更少的行總是更簡單」 | 一行嵌套的三元運算符比五行 if/else 不更簡單。簡潔性是關於理解速度，而不是行數。 |
|「我會快速簡化這不相關的程式碼」 | 無範圍的簡化會產生雜訊的差異，並在你不打算更改的程式碼中帶來潛在的回退風險。保持專注。 |
|「型別讓它自解釋」 | 型別描述結構，而不是意圖。一個命名良好的函式比型別簽名更能解釋*為什麼*更好。 |
|「這個抽象可能稍後有用」 | 不要保留推測性的抽象。如果現在沒被使用，它就是沒有價值的複雜性。移除它，在需要時再添加。 |
|「原始作者一定有理由」 | 可能。檢查 git blame——應用切斯特頓的圍欄。但累積的複雜度通常沒有理由；它只是在壓力下迭代的殘留物。 |
|「我會在我添加這個功能時重構」 | 將重構與功能工作分開。混合的更改更難審查、撤銷和在歷史中理解。 |

## 警示標誌

- 需要修改測試才能通過的簡化（你可能改變了行為）
- 比原始版本更長、更難跟蹤的「簡化」程式碼
- 為了符合你的偏好而重新命名而不是遵循專案慣例
- 因為「讓程式碼更乾淨」而移除錯誤處理
- 簡化你不完全理解的程式碼
- 將許多簡化批次化為一個大型、難以審查的提交
- 在未經要求的情況下，在當前任務範圍之外重構程式碼

## 驗證

完成簡化過程後：

- [ ] 所有現有的測試在不修改下都通過
- [ ] 建置成功，沒有新的警告
- [ ] Linter/格式化通過（沒有風格回退）
- [ ] 每次簡化都是可審查的、增量的更改
- [ ] 差異乾淨——沒有不相關的更改混入
- [ ] 簡化後的程式碼遵循專案慣例（與 CLAUDE.md 或等效內容進行比較）
- [ ] 沒有移除或削弱錯誤處理
- [ ] 沒有留下無效程式碼（未使用的導入、不可到達的分支）
- [ ] 同事或審查代理會批准此更改，認為它是一個淨進步
