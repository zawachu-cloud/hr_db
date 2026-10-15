# AGENTS.md

本 repo 是 **hr 專案的資料庫實驗場**:AI 直接連線操作 Supabase(PostgreSQL)做查詢、驗證、schema/計薪實驗。
正式應用程式碼在 `../hr`;本 repo **只碰 DB、不放前端/應用**。開工前先讀本檔;細節規則放 `.kiro/steering/*`(`auto` 載入,省 token)。

## 用途與邊界

- **做什麼**:連 dev/prod DB 跑 SQL、檢視 schema、驗證資料/計薪假設、起草 migration 草案。
- **不做什麼**:不放前端、不重建 hr 應用;正式 migration 仍以 `../hr/supabase/migrations` 為單一真實來源,本 repo 只產「草案」,確認後再搬回 hr 走正式部署流程。
- **現行 schema 概覽**:見 `docs/schema.md`(由 live DB 產生的快照;可重新產生)。

## 連線(dev / prod)

- 憑證在 `.env.dev` / `.env.prod`(已由 hr 複製真實值、已 gitignore,**已授權使用、勿印出值/勿提交**)。
- **預設只對 dev 操作**;dev=`xdznbojweiyvfaeuwjaj`(ap-southeast-2)、prod=`vdvcysrbxxfonylodkrk`(ap-southeast-1)。
- psql 走 pooler(dev 直連為 IPv6 本機連不到)。**連線指令、載入 env 的寫法見 `.kiro/steering/db-connect.md`**。

## DB 安全(最高優先)

- **預設 dev**:破壞性/實驗動作一律先在 dev;對 prod 操作前務必確認 link 的 ref 正確。
- **additive 原則**:schema 變更一律 `add column/table if not exists`、新增帶生效年月的參數;**不對線上欄位/資料 drop / 破壞性 alter / 覆蓋**。被取代的欄位/表保留(可 `comment ... DEPRECATED`)。
- **破壞性動作先確認**:`drop`/`truncate`、改 RLS/權限、大量 update/delete、`git push --force`、`reset --hard` 等,先說明風險並取得老闆確認。
- **唯讀查詢可直接跑**;寫入(含 dev)先說明將改什麼。
- **金鑰**:`service_role` 僅後端用,不進前端/版控;真實值只在 `.env.*`。個資(身分證/銀行/PIN)不外流至日誌或第三方。
- 完整規範見 `.kiro/steering/db-safety.md`(`auto`)。

## 慣例

- 資料表/欄位 `snake_case`;金額以**整數(新台幣元)**;時間存 UTC、顯示 Asia/Taipei。
- 文件/說明用繁體中文,SQL 識別符用英文。
- 實驗性 SQL/產出放 `tmp/` 或 `dumps/`(已 gitignore,不進版控、不讀入 context)。
- 文件力求精簡:每次都要讀的放本檔,其餘細節放 `auto` steering,避免重複、省 token。

## 收尾檢查(close session)

當老闆說「close session / 收工 / 結束這個 session」時,依序:
1. **結清手邊**:列出本 session 未辦事項(未跑完的查詢、暫緩/TODO、未回覆問題),先處理或向老闆確認,再往下。
2. **檢查文件結構可否更省 token**:評估去重、分層(細節移到 `auto` steering);有可優化處提建議讓老闆決定,**不擅自大改**。
3. **文件與現況一致性**:核對 `AGENTS.md`、`docs/schema.md`、`.kiro/steering/*` 是否與 live DB / `../hr` migrations 相符,有落差就補正或回報。
4. **純文件變更直接 commit & push**:本 session 若只動不影響資料的文件(`AGENTS.md`、`docs/`、`.kiro/steering/*`、`README.md`),收尾時直接 `git add -A && git commit && git push`(已授權,不需再問)。**若含實際 DB 寫入/migration 則不適用**,照 DB 安全與部署流程處理。
