# hr_db — hr 專案資料庫實驗場

AI 直接連 Supabase(PostgreSQL)做查詢、驗證、schema/計薪實驗的工作區。正式應用在 `../hr`。

- **先讀** `AGENTS.md`(規範、邊界、DB 安全、close session)。
- **連線 / schema**:`.env.dev`/`.env.prod`(已 gitignore);細節見 `.kiro/steering/db-connect.md`、現行 schema 見 `docs/schema.md`。
- **安全**:預設 dev;破壞性動作先確認;金鑰/個資不外流。見 `.kiro/steering/db-safety.md`。

> 設定:`cp .env.example .env.dev`(或由 `../hr` 複製真實憑證)。實驗產出放 `tmp/`、`dumps/`(不進版控)。
