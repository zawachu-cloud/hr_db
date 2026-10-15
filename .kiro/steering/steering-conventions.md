---
inclusion: auto
name: steering-conventions
description: Conventions for writing steering files. Use when creating, editing, or generating steering markdown documents.
---

# Steering 撰寫規範

所有 `.kiro/steering/*.md` 必須以 YAML frontmatter 開頭,依用途選 inclusion mode。
目標:把「每次都要讀」的內容降到最低,細節改用 `auto` 依需求載入,以節省 token。

## always — 每次都載入(僅放最關鍵、最短的規則)
```yaml
---
inclusion: always
---
```

## auto — 依 description 自動判斷是否相關(預設優先使用)
```yaml
---
inclusion: auto
name: <kebab-case-name>
description: <一句話描述用途及觸發關鍵字>
---
```
`name` 與 `description` 為必填。`description` 應含關鍵檔案路徑、功能關鍵字(如 schema、RLS、connect、psql、migration),讓相關問題能精準匹配。

## CLI 注意事項
CLI 確定支援 `always` 與 `auto`;`fileMatch` 與 `manual` 於 CLI 實測不一定生效,優先用前兩種。
