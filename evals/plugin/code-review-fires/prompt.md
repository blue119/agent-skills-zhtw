---
description: 一個指明審查維度且包含內聯小型 diff 的審查請求。該技能應被觸發（透過 review 指令或直接觸發），且回覆應包含其嚴重性標籤。
expected_outcome: 具有結論的結構化審查、使用技能的嚴重性分類標記的發現，並將 pageOrders 中的差一錯誤（off-by-one）視為必須修改的變更。
max_turns: 12
allowed_tools: [Read, Glob, Grep, Skill]
---

在合併之前，我希望針對正確性、可讀性、安全性和效能對此 diff 進行程式碼審查。它為 orders 端點新增了分頁功能。

```diff
diff --git a/src/orders.js b/src/orders.js
--- a/src/orders.js
+++ b/src/orders.js
@@ -12,3 +12,15 @@ function listOrders(db) {
   return db.query('SELECT * FROM orders ORDER BY created_at DESC');
 }
-module.exports = { listOrders };
+function pageOrders(db, page, size) {
+  const rows = db.query('SELECT * FROM orders ORDER BY created_at DESC');
+  const start = (page - 1) * size;
+  return rows.slice(start, start + size - 1);
+}
+
+app.get('/orders', (req, res) => {
+  const page = Number(req.query.page) || 1;
+  const size = Number(req.query.size) || 20;
+  res.json(pageOrders(db, page, size));
+});
+
+module.exports = { listOrders, pageOrders };
```
