# 測試模式參考 (JavaScript/TypeScript)

JavaScript/TypeScript 測試模式的快速參考 — 介紹 Jest、React Testing Library、Supertest 和 Playwright — 說明了從「測試驅動開發」中普遍的原則。這些原則（Arrange-Act-Assert、命名、模擬紀律、反模式）適用於任何生態系統；這裡展示的語法和工具是特定於 JS/TS 的。在其他堆疊中，請使用該儲存庫自己的測試框架和命令遵循相同的原則。

## 目錄

- [測試結構 (Arrange-Act-Assert)](#test-structure-arrange-act-assert)
- [測試命名約定](#test-naming-conventions)
- [常用斷言](#common-assertions)
- [模擬模式](#mocking-patterns)
- [React/元件測試](#reactcomponent-testing)
- [API / 整合測試](#api--integration-testing)
- [端到端測試 (Playwright)](#e2e-testing-playwright)
- [測試反模式](#test-anti-patterns)

## 測試結構 (Arrange-Act-Assert)

```typescript
it('描述預期的行為', () => {
  // Arrange: 設定測試資料和先決條件
  const input = { title: '測試任務', priority: 'high' };

  // Act: 執行要測試的操作
  const result = createTask(input);

  // Assert: 驗證結果
  expect(result.title).toBe('Test Task');
  expect(result.priority).toBe('high');
  expect(result.status).toBe('pending');
});
```

## 測試命名約定

```typescript
// 模式: [單元] [預期行為] [條件]
describe('TaskService.createTask', () => {
  it('以預設的待定狀態建立任務', () => {});
  it('當標題為空時拋出驗證錯誤', () => {});
  it('從標題中移除空白字元', () => {});
  it('為每個任務生成一個唯一的 ID', () => {});
});
```

## 常用斷言

```typescript
// 等值性
expect(result).toBe(expected);           // 嚴格等值性 (===)
expect(result).toEqual(expected);        // 深層等值性 (物件/陣列)
expect(result).toStrictEqual(expected);  // 深層等值性 + 型別匹配

// 真實性
expect(result).toBeTruthy();
expect(result).toBeFalsy();
expect(result).toBeNull();
expect(result).toBeDefined();
expect(result).toBeUndefined();

// 數字
expect(result).toBeGreaterThan(5);
expect(result).toBeLessThanOrEqual(10);
expect(result).toBeCloseTo(0.3, 5);      // 浮點數

// 字串
expect(result).toMatch(/pattern/);
expect(result).toContain('substring');

// 陣列 / 物件
expect(array).toContain(item);
expect(array).toHaveLength(3);
expect(object).toHaveProperty('key', 'value');

// 錯誤
expect(() => fn()).toThrow();
expect(() => fn()).toThrow(ValidationError);
expect(() => fn()).toThrow('specific message');

// 非同步
await expect(asyncFn()).resolves.toBe(value);
await expect(asyncFn()).rejects.toThrow(Error);
```

## 模擬模式

### 模擬函式

```typescript
const mockFn = jest.fn();
mockFn.mockReturnValue(42);
mockFn.mockResolvedValue({ data: 'test' });
mockFn.mockImplementation((x) => x * 2);

expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2');
expect(mockFn).toHaveBeenCalledTimes(3);
```

### 模擬模組

```typescript
// 模擬整個模組
jest.mock('./database', () => ({
  query: jest.fn().mockResolvedValue([{ id: 1, title: 'Test' }]),
}));

// 模擬特定匯出
jest.mock('./utils', () => ({
  ...jest.requireActual('./utils'),
  generateId: jest.fn().mockReturnValue('test-id'),
}));
```

### 僅模擬邊界

```
模擬這些：                    不要模擬這些：
├── 資料庫呼叫             ├── 內部工具函式
├── HTTP 請求              ├── 業務邏輯
├── 檔案系統操作         ├── 資料轉換
├── 外部 API 呼叫         ├── 驗證函式
└── 時間/日期 (在需要時)    └── 純函式
```

## React/元件測試

```tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';

describe('TaskForm', () => {
  it('使用輸入資料提交表單', async () => {
    const onSubmit = jest.fn();
    render(<TaskForm onSubmit={onSubmit} />);

    // 透過可存取角色/標籤查找元素 (而非測試 ID)
    await screen.findByRole('textbox', { name: /title/i });
    fireEvent.change(screen.getByRole('textbox', { name: /title/i }), {
      target: { value: 'New Task' },
    });
    fireEvent.click(screen.getByRole('button', { name: /create/i }));

    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith({ title: 'New Task' });
    });
  });

  it('顯示空標題的驗證錯誤', async () => {
    render(<TaskForm onSubmit={jest.fn()} />);

    fireEvent.click(screen.getByRole('button', { name: /create/i }));

    expect(await screen.findByText(/title is required/i)).toBeInTheDocument();
  });
});
```

## API / 整合測試

```typescript
import request from 'supertest';
import { app } from '../src/app';

describe('POST /api/tasks', () => {
  it('建立任務並返回 201', async () => {
    const response = await request(app)
      .post('/api/tasks')
      .send({ title: 'Test Task' })
      .set('Authorization', `Bearer ${testToken}`)
      .expect(201);

    expect(response.body).toMatchObject({
      id: expect.any(String),
      title: 'Test Task',
      status: 'pending',
    });
  });

  it('對無效輸入返回 422', async () => {
    const response = await request(app)
      .post('/api/tasks')
      .send({ title: '' })
      .set('Authorization', `Bearer ${testToken}`)
      .expect(422);

    expect(response.body.error.code).toBe('VALIDATION_ERROR');
  });

  it('在沒有驗證的情況下返回 401', async () => {
    await request(app)
      .post('/api/tasks')
      .send({ title: 'Test' })
      .expect(401);
  });
});
```

## 端到端測試 (Playwright)

```typescript
import { test, expect } from '@playwright/test';

test('使用者可以建立並完成一個任務', async ({ page }) => {
  // 導航並驗證身份
  await page.goto('/');
  await page.getByRole('textbox', { name: /email/i }).fill('test@example.com');
  await page.getByLabel(/password/i).fill('testpass123');
  await page.getByRole('button', { name: /log in/i }).click();

  // 建立任務
  await page.getByRole('button', { name: /new task/i }).click();
  await page.getByRole('textbox', { name: /title/i }).fill('買雜貨');
  await page.getByRole('button', { name: /create/i }).click();

  // 驗證任務出現
  const task = page.getByRole('listitem', { name: /buy groceries/i });
  await expect(task).toBeVisible();

  // 完成任務
  await task.getByRole('checkbox', { name: /complete buy groceries/i }).check();
  await expect(task).toHaveCSS('text-decoration-line', 'line-through');
});
```

## 測試反模式

| 反模式 | 問題 | 更好的方法 |
|---|---|---|
| 測試實作細節 | 重構
