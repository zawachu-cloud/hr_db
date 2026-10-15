---
inclusion: auto
name: db-connect
description: How to connect to the dev/prod Supabase Postgres for the hr DB-experiment repo. Use when running psql, SUPABASE_DB_URL, pooler, supabase link/db push, loading .env.dev/.env.prod, or any DB connection task.
---

# 連線方式(dev / prod)

憑證在 `.env.dev` / `.env.prod`(已授權、勿印值/勿提交)。**預設只對 dev**。

## 載入環境變數(不印出值)
```bash
set -a; . ./.env.dev; set +a      # prod 換成 .env.prod,並先確認是刻意對 prod
```

## psql(走 pooler)
- **dev 直連 host(`db.<ref>.supabase.co`)是 IPv6,本機連不到**;dev 一律用 pooler 組連線字串:
```bash
# dev(ap-southeast-2)— .env.dev 的 SUPABASE_DB_URL 是直連,psql 要改用 pooler:
POOL="postgresql://postgres.${SUPABASE_PROJECT_REF}:${SUPABASE_DB_PASSWORD}@aws-0-ap-southeast-2.pooler.supabase.com:5432/postgres"
psql "$POOL" -c "select app_schema_version();"
```
- **prod**:`.env.prod` 的 `SUPABASE_DB_URL` 本身即 pooler(`aws-0-ap-southeast-1.pooler.supabase.com`),可直接 `psql "$SUPABASE_DB_URL"`。
- host 規則:`aws-0-<region>.pooler.supabase.com`、user `postgres.<ref>`;dev=`ap-southeast-2`、prod=`ap-southeast-1`。

## Supabase CLI(migration 草案)
```bash
export SUPABASE_ACCESS_TOKEN="$(grep '^SUPABASE_ACCESS_TOKEN=' .env.dev | cut -d= -f2-)"
PW="$(grep '^SUPABASE_DB_PASSWORD=' .env.dev | cut -d= -f2-)"
supabase link --project-ref "$SUPABASE_PROJECT_REF" --password "$PW"   # 確認 supabase/.temp/project-ref 是對的環境
```
> 每個 shell 獨立,CLI 指令都要重新 export token。對 prod 操作前務必確認 ref = `vdvcysrbxxfonylodkrk`。

## 重載 PostgREST schema cache(新增/更名 表‧欄位‧函式後)
```bash
psql "$POOL" -c "notify pgrst, 'reload schema';"   # 否則前端經 anon 報 PGRST205
```
