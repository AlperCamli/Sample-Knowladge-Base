---
doc_class: machine-index
system: supabase
scope: schema
schema: reporting
generated_at: 2026-07-27
source_mode: live
snapshot_version: "1"
status: machine
---

# supabase — schema `reporting`

17 objects · 17 with human docs.

| Object | Kind | Machine doc | Human doc | Status | Purpose |
|---|---|---|---|---|---|
| `v_activation_funnel_monthly` | view | [v_activation_funnel_monthly.schema.md](v_activation_funnel_monthly.schema.md) | [v_activation_funnel_monthly.md](v_activation_funnel_monthly.md) | draft | Monthly signup cohorts with same-month activation counts across four product milestones. |
| `v_ai_runs_by_day` | view | [v_ai_runs_by_day.schema.md](v_ai_runs_by_day.schema.md) | [v_ai_runs_by_day.md](v_ai_runs_by_day.md) | draft | AI runs started per UTC day, split by terminal status — the reliability trend series. |
| `v_ai_runs_by_flow` | view | [v_ai_runs_by_flow.schema.md](v_ai_runs_by_flow.schema.md) | [v_ai_runs_by_flow.md](v_ai_runs_by_flow.md) | draft | AI invocation volume, token consumption and latency, split by flow, provider, model and outcome. |
| `v_ai_tokens_by_month` | view | [v_ai_tokens_by_month.schema.md](v_ai_tokens_by_month.schema.md) | [v_ai_tokens_by_month.md](v_ai_tokens_by_month.md) | draft | Monthly AI token spend by provider, with a count of runs that did not complete. |
| `v_cv_production` | view | [v_cv_production.schema.md](v_cv_production.schema.md) | [v_cv_production.md](v_cv_production.md) | draft | Tailored-CV output per month and language, including how many of that cohort were later soft-deleted. |
| `v_daily_activity` | view | [v_daily_activity.schema.md](v_daily_activity.schema.md) | [v_daily_activity.md](v_daily_activity.md) | draft | Gap-free daily activity across the four main actions: jobs tracked, CVs tailored, exports, and AI runs. |
| `v_exports_by_format` | view | [v_exports_by_format.schema.md](v_exports_by_format.schema.md) | [v_exports_by_format.md](v_exports_by_format.md) | draft | Export volume by output format and outcome. |
| `v_files_by_type` | view | [v_files_by_type.schema.md](v_files_by_type.schema.md) | [v_files_by_type.md](v_files_by_type.md) | draft | Active file inventory and storage footprint by file type and MIME type. |
| `v_imports_by_parser` | view | [v_imports_by_parser.schema.md](v_imports_by_parser.schema.md) | [v_imports_by_parser.md](v_imports_by_parser.md) | draft | CV import volume by parser and outcome — the reliability view for document ingestion. |
| `v_job_status_transitions` | view | [v_job_status_transitions.schema.md](v_job_status_transitions.schema.md) | [v_job_status_transitions.md](v_job_status_transitions.md) | draft | Daily counts of job-application status transitions — the source for a transition matrix. |
| `v_jobs_by_month` | view | [v_jobs_by_month.schema.md](v_jobs_by_month.schema.md) | [v_jobs_by_month.md](v_jobs_by_month.md) | draft | Monthly job-tracking volume: postings created per month, how many users created them, and how many reached the applied stage. |
| `v_jobs_by_status` | view | [v_jobs_by_status.schema.md](v_jobs_by_status.schema.md) | [v_jobs_by_status.md](v_jobs_by_status.md) | draft | Job-application pipeline size by status: how many tracked postings sit in each stage, over how many users, and the date range each stage spans. |
| `v_master_cvs_by_language` | view | [v_master_cvs_by_language.schema.md](v_master_cvs_by_language.schema.md) | [v_master_cvs_by_language.md](v_master_cvs_by_language.md) | draft | Active master-CV inventory by language and how it was created (uploaded, built, imported). |
| `v_subscriptions_by_plan` | view | [v_subscriptions_by_plan.schema.md](v_subscriptions_by_plan.schema.md) | [v_subscriptions_by_plan.md](v_subscriptions_by_plan.md) | draft | Subscription mix by plan and status, with pending cancellations. |
| `v_subscriptions_new_by_month` | view | [v_subscriptions_new_by_month.schema.md](v_subscriptions_new_by_month.schema.md) | [v_subscriptions_new_by_month.md](v_subscriptions_new_by_month.md) | draft | Subscription records created per UTC month and status — counts of records, never revenue. |
| `v_user_cohorts` | view | [v_user_cohorts.schema.md](v_user_cohorts.schema.md) | [v_user_cohorts.md](v_user_cohorts.md) | draft | Signup cohort sizes by month, locale and default CV language, with onboarding completion. |
| `v_user_signups_by_day` | view | [v_user_signups_by_day.schema.md](v_user_signups_by_day.schema.md) | [v_user_signups_by_day.md](v_user_signups_by_day.md) | draft | New user accounts per UTC calendar day — the signup trend series. |
