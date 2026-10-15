# docs/schema.md — 現行 schema 快照

> 由 **dev** live DB 產生(pooler)。schema 版本:**0036**(`select app_schema_version()`)。
> 全 30 張表皆已啟用 **RLS**。欄位型別後 `NN`=NOT NULL。此為快照,異動以 `../hr/supabase/migrations` 為準;過時可重新產生(見末段)。

## 表(依領域)

### 組織 / 權限
- **stores**: id uuid NN, name text NN, address text, is_active bool NN, created_at, updated_at
- **store_members**: id, manager_user_id uuid NN, store_id uuid NN, can_view_salary bool NN, created_at — 店長↔店別授權
- **app_settings**: key text NN, value text NN, updated_by uuid, updated_at NN — owner-only KV(如寄件人設定)
- **audit_logs**: id, actor uuid, action NN, entity NN, entity_id, diff jsonb, created_at NN, store_id — 稽核

### 員工(欄位級隔離三表)
- **employees**: id, employee_no NN, full_name NN, email, job_title, hire_date, resign_date, is_active NN, weekly_hours, store_id NN, termination_type, termination_reason
- **employee_comp**: employee_id NN, store_id NN, pay_type NN(月/時薪), hourly_rate, base_salary, meal_allowance NN, other_allowance NN, attendance_bonus NN, wage_cap_enabled NN, wage_cap_amount — 薪酬
- **employee_private**: employee_id NN, store_id NN, national_id_enc, bank_account_enc, insured_salary NN, labor/health_insured_salary, pension_wage, dependents NN(+health/tax_dependents), labor_pension_self NN, punch_pin, phone — 敏感個資(owner-only)

### 出勤 / 打卡 / 排班
- **time_punches**: id, employee_id NN, punch_at NN, direction NN, source NN, status NN, void*、corrects_id, store_id — 原始打卡(immutable,void/correct 機制)
- **attendance**: id, employee_id NN, period_ym NN, regular/overtime/holiday_hours NN, special_hours jsonb NN, unpaid_special_days NN, overtime_tiers jsonb NN, rest_day_tiers jsonb NN, store_id NN — 月時數彙總
- **attendance_bonus_overrides**: 全勤獎金人工調整(period_ym, granted)
- **shifts**: id, name NN, start/end_time NN, break_minutes NN, color, is_active NN, store_id NN — 班別
- **shift_assignments**: employee_id NN, shift_id NN, work_date NN, status NN, store_id NN — 排班
- **weekly_schedule_templates**: employee_id NN, pattern jsonb NN, store_id — 週班表模板
- **special_days**: work_date NN, start/end_time, type_id int NN, label, store_id NN — 國定/特殊日
- **monthly_rest_quota**: period_ym NN, rest_days NN — 月休息日配額
- **day_off_requests**: employee_id NN, work_date NN, category NN, moved_from_holiday_date, agreed bool NN, source NN — 國定調移/排休
- **overtime_approvals**: employee_id NN, work_date NN, start/end_time NN, status NN, void* — 加班審批

### 休假
- **leave_requests**: id, employee_id NN, leave_type NN, start_at/end_at NN, days NN, period_ym, paid_ratio, status NN, void*, corrects_id, source_holiday_date, store_id NN
- **leave_entitlements**: employee_id NN, period_start/end NN, entitled/carried/used/settled_days NN, settled_amount NN, status NN, payout_period_ym, forfeited_days NN, store_id NN — 特休額度

### 計薪 / 薪資條
- **payroll_runs**: id, period_ym NN, status NN, store_id NN — 每月產薪批次
- **payslips**: id, run_id, employee_id NN, period_ym NN, gross_pay/total_deduction/net_pay NN, status NN, paid NN, paid_at, email_status NN, sent_at, store_id NN
- **payslip_items**: payslip_id NN, item_type NN, code NN, label NN, amount NN, detail, store_id — 薪資條明細列
- **rate_tables**: id, kind NN, effective_ym NN, params jsonb NN — **參數化費率(見下)**

### 獎金 / 獎懲 / 帳目 / 離職
- **group_bonus** / **group_bonus_allocations**: 團體獎金池與分配(period_q, shift_ids jsonb, hours/factor/final_amount)
- **quarterly_personal_bonus**: 季個人獎金(period_q, base_amount NN)
- **merit_demerit_records**: 獎懲(level NN, occurred_on NN)
- **expense_categories** / **ledger_entries**: 每日帳目分類與分錄(direction NN, amount NN, is_recurring NN)
- **terminations**: 離職結算(resign_date NN, termination_type NN, service_months NN, avg_monthly/daily_wage NN, severance_months/pay NN, notice_days/pay NN, leave_payout NN)

## 費率參數(rate_tables.kind → 生效年月)
`annual_leave`(2025-01, 2026-01)、`attendance_bonus`(2025-01)、`health_insurance`(2025-01)、`income_tax`(2025-01)、`labor_compliance`(2025-01)、`labor_insurance`(2025-01)、`labor_pension`(2025-01)、`leave_policy`(2025-01, 2026-01, 2026-10)、`merit_demerit`(2025-01)、`nhi_supplement`(2025-01)、`overtime_multiplier`(2025-01)、`severance`(2025-01)。
> 費率一律「新增帶生效年月的參數」,不改歷史。詳細欄位意義見 `../hr/docs/payroll-rules.md`。

## 函式(public)
`app_schema_version()`、`can_access_store(sid)`、`can_view_salary(sid)`、`is_owner()`/`is_manager()`/`is_kiosk()`、`manager_store_ids()`、`leave_requests_guard()`、`time_punches_guard()`、`purge_before(cutoff date)`、`rls_auto_enable()`、`set_updated_at()`。

## 重新產生此快照
```bash
set -a; . ./.env.dev; set +a
POOL="postgresql://postgres.${SUPABASE_PROJECT_REF}:${SUPABASE_DB_PASSWORD}@aws-0-ap-southeast-2.pooler.supabase.com:5432/postgres"
psql "$POOL" -c "select app_schema_version();"
psql "$POOL" -c "\d public.<table>"   # 看單表完整定義/索引/FK
```
