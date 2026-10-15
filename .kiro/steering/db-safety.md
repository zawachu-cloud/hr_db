---
inclusion: auto
name: db-safety
description: DB safety rules for the hr DB-experiment repo (highest priority). Use when running SQL writes, schema/migration changes, drop/truncate, RLS/policies, service_role/anon keys, .env/secrets, sensitive fields (身分證/銀行/PIN), bulk update/delete, or any dev/prod DB operation.
---

# DB 安全規範(最高優先)

本 repo 直接連 live Supabase(dev/prod)操作薪資/個資資料,受台灣《個人資料保護法》規範。資安與最小權限第一。

1. **預設 dev、prod 需確認**:破壞性/實驗動作一律先在 dev(`xdznbojweiyvfaeuwjaj`)。對 prod(`vdvcysrbxxfonylodkrk`)任何操作前,先確認 `supabase/.temp/project-ref` 或連線字串的 ref 正確,並向老闆說明。
2. **唯讀 vs 寫入**:`SELECT`/explain 等唯讀查詢可直接跑;任何寫入(含 dev 的 `insert/update/delete`、DDL)先說明「將改什麼、影響哪些列/表」再執行。
3. **additive 原則**:schema 變更一律 `add column/table if not exists`、新增帶生效年月的參數;**不對線上欄位/資料做 `drop`/破壞性 `alter`/覆蓋**。被新機制取代而不再使用的欄位/表:**保留不刪**(可 `comment ... DEPRECATED` 標註),保持 migration 歷史線性、舊 JS 相容。
4. **破壞性動作先取得確認**:`drop`/`truncate`、停用或修改 RLS/權限、大量 `update`/`delete`(無 where 或影響多列)、`git push --force`、`reset --hard` 等——先說明風險、是否可復原,取得老闆確認後才做。遇阻優先選非破壞性替代。
5. **金鑰**:`service_role` 只在後端用,**嚴禁**進前端或版控;真實金鑰只在 `.env.*`(已 gitignore)。讀 `.env.*` 時不要把值回印到回覆或日誌。
6. **RLS 預設拒絕**:含個資的表啟用 Row Level Security;實驗若需測 RLS,用對應角色/JWT 測,勿為方便停用線上 policy。
7. **個資不外流**:身分證/銀行帳號/`punch_pin` 等敏感欄位不輸出到日誌、不傳第三方;需取樣時遮罩或限 dev 假資料。
8. **migration 草案搬回 hr 才正式套用**:本 repo 產出的 migration 僅為草案;正式套用(dev+prod 皆 `supabase db push` + 重載 schema cache)走 `../hr` 的部署流程。
