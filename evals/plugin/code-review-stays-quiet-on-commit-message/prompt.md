---
description: 貼上的差異內容，以及屬於 git-workflow-and-versioning 的請求。這會驗證審查技能描述中的「即使差異內容是內嵌貼上」條款，因為單獨貼上的差異不應觸發審查技能。
expected_outcome: 一則提交訊息；絕不呼叫 code-review-and-quality。
max_turns: 12
allowed_tools: [Read, Glob, Grep, Skill]
---

請為這份差異內容撰寫提交訊息。

```diff
diff --git a/src/orders.js b/src/orders.js
--- a/src/orders.js
+++ b/src/orders.js
@@ -12,3 +12,9 @@ function listOrders(db) {
   return db.query('SELECT * FROM orders ORDER BY created_at DESC');
 }
-module.exports = { listOrders };
+function pageOrders(db, page, size) {
+  const rows = db.query('SELECT * FROM orders ORDER BY created_at DESC');
+  const start = (page - 1) * size;
+  return rows.slice(start, start + size);
+}
+
+module.exports = { listOrders, pageOrders };
```
