---
type: llm
---

如果審查指出 `rows.slice(start, start + size - 1)` 在每一頁上返回的行數比 `size` 少一行（切片的結束索引是排他的），並將其視為合併前必須修復的內容，例如標記為 Critical 或保持未加前綴作為必需的更改。

如果審查沒有提到頁面少了一行，或者文件被標記為 Nit、Optional、Consider 或 FYI。
