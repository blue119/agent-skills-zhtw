# 門檻守衛：參考實作

`CONSTRAINTS.md` 中的每一個編號維度都對應到一個事實上的工具（步驟 4）。**門檻**並非：它對步驟 6 中的五個移動進行的差異範圍檢查，而且沒有發布的參考資料，每個代理都會創造自己的，因此兩次運行（或一個 Python 倉庫和一個 Go 倉庫）會產生兩種不同的守衛。這正是該技能存在想要消除的非決定性。

這是參考。將模式適應到您的堆疊中；保持合約相同。

## 合約

- **輸入：** 合併基線與工作樹之間的差異（新增*和*移除的行，以及未追蹤的文件）。只讀取 `git diff` 的守衛會遺漏新文件和已暫存但未提交的工作。
- **偵測五個步驟 6 的移動：** `CONSTRAINTS.md` 中較弱的閾值、一個更容易的測試（`.skip`、一個被刪除的測試文件、一個從測試中移除的斷言）、一個被靜音的檢查器（一個新的抑制註釋）、未完成的工作（一個存根或空的 `catch`）、一個新的例外項。
- **退出代碼：** `0` 清潔，`1` 至少有一次門檻違反（阻止更改），`2` 守衛無法運行（沒有合併基線，不是 git 倉庫）。絕不要讓 `2` 被讀為 `0`。
- **報告規則和位置，絕不報告匹配的密碼值。** 遮蔽不是可選的（步驟 4）。
- **收緊是靜默的，放鬆是喧鬧的：** 只顯示降低門檻的移動。

## 參考（節點，~不依賴堆疊的模式）

```js
#!/usr/bin/env node
// floor-guard.mjs — 根據 CONSTRAINTS.md 的門檻進行的差異範圍強制執行。
// 用法: node floor-guard.mjs [--base <ref>]   (預設基線: origin/main)
import { execFileSync } from 'node:child_process';

const base = (() => {
  const i = process.argv.indexOf('--base');
  return i > -1 ? process.argv[i + 1] : 'origin/main';
})();

// `git diff --no-index` 當兩側不同時退出 1，這是新文件的正常情況，因此輸出被保留。任何其他失敗都是 null，null 絕不會被讀為乾淨。
const git = (args, { diffExit = false } = {}) => {
  try { return execFileSync('git', args, { encoding: 'utf8', stdio: ['ignore', 'pipe', 'pipe'] }); }
  catch (e) { return diffExit && e.status === 1 && typeof e.stdout === 'string' ? e.stdout : null; }
};
const bail = (msg) => { console.error('floor-guard: ' + msg); process.exit(2); };

// 從工作樹的頂部運行。`git ls-files` 只列出當前目錄的子樹，相對於它，因此在子資料夾中啟動的守衛會遺漏其他地方的未追蹤文件，並且名稱與 `git diff` 不同，而後者總是涵蓋整個樹。
const top = git(['rev-parse', '--show-toplevel'])?.trim();
if (!top) bail('不在 git 工作樹內');
process.chdir(top);

// 合併基線；退出 2 而不是假裝淺層/無根克隆是乾淨。
const mergeBase = git(['merge-base', base, 'HEAD'])?.trim();
if (!mergeBase) bail('沒有對 ' + base + ' 的合併基線');

// 統一差異加上未追蹤文件（`git diff` 本身無法看到新文件）。
const tracked = git(['diff', '--unified=0', mergeBase, '--']);
if (tracked === null) bail('無法與 ' + mergeBase + ' 進行差異比較');
const untrackedFiles = git(['ls-files', '--others', '--exclude-standard']);
if (untrackedFiles === null) bail('無法列出未追蹤文件');
const untracked = untrackedFiles.split('\n').filter(Boolean).map((f) => {
  const d = git(['diff', '--no-index', '--unified=0', '/dev/null', f], { diffExit: true });
  if (d === null) bail('無法比較未追蹤文件 ' + f);
  return d;
}).join('\n');
const diff = tracked + '\n' + untracked;

// 遍歷差異。`---` 和 `+++` 僅在文件的 `diff` 行和其第一個 `@@` 區塊之間存在；在一個區塊內，每一行都是內容，因此新增的 `++i`（顯示為 `+++i`）或移除的 `-- comment` 是一個更改，而不是標頭。兩個標頭都命名了文件，所以刪除（`+++ /dev/null`）保留了其名稱。
const added = [], removed = [], deleted = [];
const pathOf = (s) => s.replace(/^[ab]\//, '');
let file = '', oldFile = '', inHeader = false;
for (const line of diff.split('\n')) {
  if (line.startsWith('diff ')) inHeader = true;
  else if (line.startsWith('@@')) inHeader = false;
  else if (inHeader) {
    if (line.startsWith('--- ')) oldFile = pathOf(line.slice(4));
    else if (line.startsWith('+++ ')) {
      const newFile = pathOf(line.slice(4));
      file = newFile === '/dev/null' ? oldFile : newFile;
      if (newFile === '/dev/null') deleted.push(file);
    }
  }
  else if (line.startsWith('+')) added.push({ file, text: line.slice(1) });
  else if (line.startsWith('-')) removed.push({ file, text: line.slice(1) });
}

const findings = [];
const flag = (rule, f, text) => findings.push({ rule, file: f, text: text.trim().slice(0, 120) });
const isTest = (f) => /\.(test|spec)\.|_test\.|test_/.test(f);
const isConstraints = (f) => /CONSTRAINTS\.md$/.test(f);

// 1. 被靜音的檢查器 — 為您的生態系統擴展此列表。
const SUPPRESSIONS = /@ts-ignore|@ts-nocheck|eslint-disable|biome-ignore|# *noqa|# *type: *ignore|istanbul ignore|nosemgrep|gitleaks:allow|Stryker disable/;
// 4. 未完成的工作。
const STUBS = /throw new (Error|NotImplemented).*[Nn]ot implemented|catch\s*\(\w*\)\s*\{\s*\}|catch\s*\{\s*\}|\bTODO\b|\bpass\s*# *stub/;
// 2. 測試變得更容易（添加跳過）。
const SKIPS = /\.(skip|todo)\b|\bxit\(|\bxdescribe\(|@pytest\.mark\.skip|t\.Skip\(/;

for (const { file, text } of added) {
  if (SUPPRESSIONS.test(text)) flag('silenced-checker', file, text);
  if (STUBS.test(text)) flag('unfinished-work', file, text);
  if (SKIPS.test(text)) flag('test-made-easier', file, text);
  if (isConstraints(file) && /^\| *(W|E)\d+ *\|/.test(text)) flag('new-exception', file, text);
}

// 2b. 刪除了一個測試文件，或從一個仍然存在的測試文件中移除了一個斷言。
for (const f of deleted) if (isTest(f)) flag('test-deleted', f, '文件已刪除');
for (const { file, text } of removed) {
  if (isTest(file) && !deleted.includes(file) && /\b(expect|assert|should)\b/.test(text)) {
    flag('assertion-removed', file, text);
  }
}

// 1b/2c. `CONSTRAINTS.md` 中的規則被削弱或移除。一個規則是門檻的項目符號或表格行，由其第一個冒號之前的項目文本或行中的第一個單元格識別。每個數字都帶有從其周圍詞語中讀取的方向：最小（`>=`、至少、不得低於）通過向下放鬆，最大（`<=`、最多、之下、不得增長）通過向上放鬆。無法讀取方向的數字在每次更改時都會被報告，因為守衛無法區分收緊和放鬆，靜默是錯誤的預設值。數字在它們的方向內配對（第一個最小與第一個最小，依此類推），因此在文本的其他地方添加的數字不會改變配對；一個在編輯後沒有對應的閾值，以及一個添加的閾值會收緊。
const ruleKey = (t) => {
  const s = t.trim();
  if (s.startsWith('|')) return s.split('|').map((c) => c.trim()).filter(Boolean)[0] ?? '';
  if (/^[-*] /.test(s)) return s.slice(2).split(':')[0].trim();
  return null; // 文字、標題、日期：不是規則
};
const isException = (t) => /^\| *(W|E)\d+ *\|/.test(t.trim());
const MIN_BEFORE = /(>=|>|≥|at least|minimum|\bmin\b|no less than|not fall|not drop)\s*$/;
const MAX_BEFORE = /(<=|<|≤|at most|maximum|\bmax\b|no more than|under|below|not grow|not exceed)\s*$/;
const MIN_AFTER = /^\s*\S*\s*(or more|or higher|must not fall|must not drop)/;
const MAX_AFTER = /^\s*\S*\s*(or less|or lower|must not grow|must not exceed)/;
const thresholds = (t) => {
  const out = [], re = /\d+(?:\.\d+)?/g;
  let m;
  while ((m = re.exec(t))) {
    const before = t.slice(Math.max(0, m.index - 24), m.index).toLowerCase();
    const after = t.slice(m.index + m[0].length, m.index + m[0].length + 40).toLowerCase();
    const dir = MIN_BEFORE.test(before) || MIN_AFTER.test(after) ? 'min'
      : MAX_BEFORE.test(before) || MAX_AFTER.test(after) ? 'max' : null;
    out.push({ n: Number(m[0]), dir });
  }
  return out;
};
const removedRules = removed.filter((l) => isConstraints(l.file) && ruleKey(l.text) !== null);
const addedRules = added.filter((l) => isConstraints(l.file) && ruleKey(l.text) !== null);
for (const r of removedRules) {
  const a = addedRules.find((x) => ruleKey(x.text) === ruleKey(r.text));
  if (!a) {
    if (!isException(r.text)) flag('rule-removed', r.file, r.text); // 移除例外會收緊：靜默
    continue;
  }
  const before = thresholds(r.text), after = thresholds(a.text);
  let verdict = null;
  for (const dir of ['min', 'max', null]) {
    const was = before.filter((x) => x.dir === dir), now = after.filter((x) => x.dir === dir);
    was.forEach((b, i) => {
      const n = now[i];
      if (verdict) return;
      if (!n) verdict = 'threshold-removed';
      else if (n.n === b.n) return;
      else if (dir === 'min' ? n.n < b.n : dir === 'max' ? n.n > b.n : true) {
        verdict = dir ? 'threshold-loosened' : 'threshold-changed';
      }
    });
  }
  if (verdict) flag(verdict, r.file, r.text + '  ->  ' + a.text);
}

if (findings.length === 0) { console.log('floor-guard: clean'); process.exit(0); }
console.error('floor-guard: ' + findings.length + ' 個門檻違反(s):');
for (const f of findings) console.error(`  [${f.rule}] ${f.file}: ${f.text}`);
if (findings.some((f) => f.rule === 'rule-removed')) {
  console.error('\n移除規則的發現也可能意味著規則的標籤發生了變化：在一個提交中更改規則，在另一個提交中更改其閾值。');
}
if (findings.some((f) => f.rule === 'threshold-removed')) {
  console.error('\n移除閾值的發現也可能意味著一個數字獲得或失去了其方向詞（">= 80%" 變成 "80%"，或反之）：比較前後兩行，然後才假設閾值已被刪除。');
}
console.error('\n每一個都是降低門檻的移動。修復代碼，或通過一個已追蹤的例外路由它。');
process.exit(1);
```

## 適應它

- **模式是唯一特定於堆疊的部分。** 將您語言的抑制和存根形式添加到三個正則表達式中；差異管道、`CONSTRAINTS.md` 檢查和退出代碼保持不變。
- **一個 `.constraintsignore`**（每行一個萬用字元）讓您可以豁免守衛會標記的路徑；在標記之前檢查每一行新增文件的文件是否與之匹配，這樣一個真正的例外就成為一個被追蹤的文件，而不是一個放鬆的規則。
- **這是一個起點，而不是一個完成的工具。** 它故意是正則表達式淺層的：它捕捉了代理實際進行的、通往綠色的廉價路徑移動，而不是一個隱藏更改的決定性人類。這對於每次差異都運行檢查來說是正確的權衡。當您超越它時，請轉向真正的運行器（升級路徑級別 3）。
