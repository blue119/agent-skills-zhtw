---
name: ci-cd-and-automation
description: 自動化 CI/CD 管線設定。在設定或修改建置和部署管線時使用。當您需要自動化品質門檻、在 CI 中配置測試執行器或建立部署策略時使用。
---

# CI/CD 與自動化

## 概述

自動化品質門檻，確保沒有任何變更在通過測試、程式碼檢查、類型檢查和建置後才能到達生產環境。CI/CD 是所有其他技能的執行機制——它能捕捉到人與代理人遺漏的內容，並在每一次變更上保持一致性地執行。

**左移 (Shift Left)：** 在管線的盡早階段捕捉問題。在程式碼檢查中發現的錯誤可能花費幾分鐘；在生產環境中發現的同一個錯誤可能花費數小時。將檢查移到上游——在測試之前進行靜態分析，在部署之前進行測試，在生產之前進行階段部署。

**更快更安全 (Faster is Safer)：** 較小的批次和更頻繁的發布減少風險，而不是增加風險。包含 3 個變更的部署比包含 30 個變更更容易除錯。頻繁的發布建立了對發布過程本身的信心。

## 何時使用

- 設定新專案的 CI 管線
- 新增或修改自動化檢查
- 配置部署管線
- 當變更應該觸發自動驗證時
- 除錯 CI 失敗

## 品質門檻管線

每一次變更在合併之前都必須通過這些門檻：

```
拉取請求開啟 (Pull Request Opened)
    │
    ▼
┌─────────────────┐
│ 程式碼檢查 (LINT CHECK)     │  eslint, prettier
│ ↓ 通過 (pass)         │
│ 類型檢查 (TYPE CHECK)     │  tsc --noEmit
│ ↓ 通過 (pass)         │
│ 單元測試 (UNIT TESTS)     │  jest/vitest
│ ↓ 通過 (pass)         │
│ 建置 (BUILD)          │  npm run build
│ ↓ 通過 (pass)         │
│ 整合測試 (INTEGRATION)    │  API/DB 測試
│ ↓ 通過 (pass)         │
│ 端到端 (E2E) (可選)     │  Playwright/Cypress
│ ↓ 通過 (pass)         │
│ 安全審核 (SECURITY AUDIT) │  npm audit
│ ↓ 通過 (pass)         │
│ 套件大小 (BUNDLE SIZE)    │  bundlesize check
└─────────────────┘
    │
    ▼
  準備好審查 (Ready for review)
```

**任何門檻都不能被跳過。** 如果程式碼檢查失敗，請修復程式碼——不要禁用規則。如果測試失敗，請修復程式碼——不要跳過測試。

## GitHub Actions 配置

### 基本 CI 管線

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - name: 安裝依賴 (Install dependencies)
        run: npm ci

      - name: 程式碼檢查 (Lint)
        run: npm run lint

      - name: 類型檢查 (Type check)
        run: npx tsc --noEmit

      - name: 測試 (Test)
        run: npm test -- --coverage

      - name: 建置 (Build)
        run: npm run build

      - name: 安全審核 (Security audit)
        run: npm audit --audit-level=high
```

### 整合資料庫測試 (With Database Integration Tests)

```yaml
  integration:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: ci_user
          POSTGRES_PASSWORD: ${{ secrets.CI_DB_PASSWORD }}
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
      - run: npm ci
      - name: 執行遷移 (Run migrations)
        run: npx prisma migrate deploy
        env:
          DATABASE_URL: postgresql://ci_user:${{ secrets.CI_DB_PASSWORD }}@localhost:5432/testdb
      - name: 整合測試 (Integration tests)
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://ci_user:${{ secrets.CI_DB_PASSWORD }}@localhost:5432/testdb
```

> **注意：** 即使對於僅 CI 的測試資料庫，也應使用 GitHub Secrets 來儲存憑證，而不是硬編碼值。這能建立良好的習慣，並防止測試憑證在其他情境中被意外重用。

### 端到端測試 (E2E Tests)

```yaml
  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
      - run: npm ci
      - name: 安裝 Playwright (Install Playwright)
        run: npx playwright install --with-deps chromium
      - name: 建置 (Build)
        run: npm run build
      - name: 執行 E2E 測試 (Run E2E tests)
        run: npx playwright test
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

## 將 CI 失敗回饋給代理人 (Feeding CI Failures Back to Agents)

結合 AI 代理人的 CI 的威力在於回饋迴路。當 CI 失敗時：

```
CI 失敗 (CI fails)
    │
    ▼
複製失敗輸出 (Copy the failure output)
    │
    ▼
將其提供給代理人 (Feed it to the agent):
"CI 管線因以下錯誤而失敗：
[貼上具體錯誤]
請修復問題並在推送之前進行本地驗證。"
    │
    ▼
代理人修復 → 推送 → CI 再次運行 (Agent fixes → pushes → CI runs again)
```

**關鍵模式 (Key patterns)：**

```
程式碼檢查失敗 (Lint failure) → 代理人執行 `npm run lint --fix` 並提交 (Agent runs `npm run lint --fix` and commits)
類型錯誤 (Type error) → 代理人讀取錯誤位置並修復類型 (Agent reads the error location and fixes the type)
測試失敗 (Test failure) → 代理人遵循除錯和錯誤恢復技能 (Agent follows debugging-and-error-recovery skill)
建置錯誤 (Build error) → 代理人檢查配置和依賴項 (Agent checks config and dependencies)
```

## 部署策略 (Deployment Strategies)

### 預覽部署 (Preview Deployments)

每個 PR 都會獲得一個預覽部署以供手動測試：

```yaml
# 在 PR 上部署預覽 (Vercel/Netlify/等)
deploy-preview:
  runs-on: ubuntu-latest
  if: github.event_name == 'pull_request'
  steps:
    - uses: actions/checkout@v4
    - name: 部署預覽 (Deploy preview)
      run: npx vercel --token=${{ secrets.VERCEL_TOKEN }}
```

### 功能旗標 (Feature Flags)

功能旗標將部署與發布分開。在旗標後部署不完整或有風險的功能，以便您可以：

- **在不啟用時發布程式碼。** 儘早合併到 main，準備好時啟用。
- **在不重新部署的情況下回滾。** 禁用旗標而不是撤銷程式碼。
- **進行金絲雀發布 (Canary)。** 對 1% 的使用者啟用，然後是 10%，再是 100%。
- **執行 A/B 測試。** 比較有和沒有該功能下的行為。

```typescript
// 簡單的功能旗標模式 (Simple feature flag pattern)
if (featureFlags.isEnabled('new-checkout-flow', { userId })) {
  return renderNewCheckout();
}
return renderLegacyCheckout();
```

**旗標生命週期 (Flag lifecycle)：** 建立 → 進行測試啟用 → 金絲雀發布 → 完全推出 → 移除旗標和死程式碼。永遠存在的旗標會成為技術債——在建立它們時設定清理日期。

### 分階段部署 (Staged Rollouts)

```
PR 合併到 main (PR merged to main)
    │
    ▼
  階段部署 (自動) (Staging deployment (auto))
    │ 手動驗證 (Manual verification)
    ▼
  生產部署 (手動觸發或階段後自動) (Production deployment (manual trigger or auto after staging))
    │
    ▼
  監控錯誤 (15 分鐘視窗) (Monitor for errors (15-minute window))
    │
    ├── 偵測到錯誤 → 回滾 (Errors detected → Rollback)
    └── 乾淨 → 完成 (Clean → Done)
```

### 回滾計畫 (Rollback Plan)

每一次部署都應該是可逆的：

```yaml
# 手動回滾工作流程 (Manual rollback workflow)
name: Rollback
on:
  workflow_dispatch:
    inputs:
      version:
        description: '要回滾到的版本'
        required: true

jobs:
  rollback:
    runs-on: ubuntu-latest
    steps:
      - name: 回滾部署 (Rollback deployment)
        run: |
          # 部署指定的先前版本 (Deploy the specified previous version)
          npx vercel rollback ${{ inputs.version }}
```

## 環境管理 (Environment Management)

```
.env.example       → 已提交 (開發者的範本)
.env                → 未提交 (本地開發)
.env.test           → 已提交 (測試環境，無真實密鑰)
CI secrets          → 儲存在 GitHub Secrets / vault
生產密鑰 (Production secrets)  → 儲存在部署平台 / vault
```

CI 絕不應該包含生產密鑰。請為 CI 測試使用單獨的密鑰。

## 超越 CI 的自動化 (Automation Beyond CI)

### Dependabot / Renovate

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: npm
    directory: /
    schedule:
      interval: weekly
    open-pull-requests-limit: 5
```

### 建置 Cop 角色 (Build Cop Role)

指定某人負責保持 CI 的綠色狀態。當建置失敗時，Build Cop 的工作是修復或回滾——而不是造成破壞的變更者。這可以防止有破壞性的建置累積，而所有人都假設別人會修復它。

### PR 檢查 (PR Checks)

- **必需的審查 (Required reviews)：** 合併前至少需要 1 個批准
- **必需的狀態檢查 (Required status checks)：** 合併前 CI 必須通過
- **分支保護 (Branch protection)：** 禁止對 main 進行強制推送 (force-pushes)
- **自動合併 (Auto-merge)：** 如果所有檢查都通過且獲得批准，則自動合併

## CI 優化 (CI Optimization)

當管線超過 10 分鐘時，請按照影響程度的順序應用這些策略：

```
CI 管線緩慢？ (Slow CI pipeline?)
├── 快取依賴項 (Cache dependencies)
│   └── 使用 actions/cache 或 setup-node cache 選項來快取 node_modules
├── 並行運行工作 (Run jobs in parallel)
│   └── 將程式碼檢查、類型檢查、測試、建置分割成單獨的並行工作
├── 只運行變更的部分 (Only run what changed)
│   └── 使用路徑過濾器跳過不相關的工作 (例如，對於僅有文檔的 PR，跳過 E2E)
├── 使用矩陣建置 (Use matrix builds)
│   └── 在多個執行器上分片測試套件
├── 優化測試套件 (Optimize the test suite)
│   └── 從關鍵路徑中移除慢速測試，在排程中運行它們
└── 使用更大的執行器 (Use larger runners)
    └── 使用 GitHub 主託的更大執行器或自託的執行器以進行 CPU 密集型建置
```

**範例：快取與並行 (Example: caching and parallelism)**
```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'npm' }
      - run: npm ci
      - run: npm run lint

  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'npm' }
      - run: npm ci
      - run: npx tsc --noEmit

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'npm' }
      - run: npm ci
      - run: npm test -- --coverage
```

## 常見的合理化 (Common Rationalizations)

| 合理化 (Rationalization) | 現實 (Reality) |
|---|---|
|「CI 太慢」("CI is too slow") | 優化管線（參見 CI 優化部分），不要跳過它。一個 5 分鐘的管線可以避免數小時的除錯。 |
|「這個變更很簡單，跳過 CI」("This change is trivial, skip CI") | 簡單的變更會破壞建置。CI 對於簡單的變更來說本來就很快。 |
|「測試很不可靠，只是重新運行」("The test is flaky, just re-run") | 不可靠的測試會掩蓋真正的錯誤並浪費所有人的時間。修復這種不穩定性。 |
|「我們稍後再添加 CI」("We'll add CI later") | 沒有 CI 的專案會累積破壞的狀態。在第一天就設定它。 |
|「手動測試就足夠了」("Manual testing is enough") | 手動測試無法擴展且無法重複。自動化你能自動化的部分。 |

## 警示信號 (Red Flags)

- 專案中沒有 CI 管線
- CI 失敗被忽略或靜音
- 在 CI 中禁用測試以使管線通過
- 在沒有階段驗證的情況下進行生產部署
- 沒有回滾機制
- 密鑰儲存在程式碼或 CI 配置檔案中（而不是密鑰管理系統）
- CI 時間過長但沒有優化工作

## 驗證 (Verification)

在設定或修改 CI 後：

- [ ] 所有品質門檻都存在（程式碼檢查、類型、測試、建置、審核）
- [ ] 管線在每次 PR 和 push 到 main 上運行
- [ ] 失敗會阻止合併（分支保護已配置）
- [ ] CI 結果回饋到開發迴路中
- [ ] 密鑰儲存在密鑰管理系統中，而不是在程式碼中
- [ ] 部署有回滾機制
- [ ] 測試套件的管線運行時間在 10 分鐘以內
