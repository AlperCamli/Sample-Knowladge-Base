---
doc_class: human-object
object: supabase.public.ai_runs
written_against_schema_hash: "sha256:a1d393bf52a05c21597853ebf1a04de62392198342bb2a85f6c736590e258b3c"
status: draft
last_verified: null
purpose: "A single AI model invocation capturing the flow, provider, payloads, token usage, and progress of a generation request."
column_purposes:
  id: "Internal AI-run id."
  user_id: "User on whose behalf the run executed (FK to users.id)."
  master_cv_id: "Master CV the run operated on, if applicable (FK)."
  tailored_cv_id: "Tailored CV the run operated on, if applicable (FK)."
  job_id: "Job context the run operated against, if applicable (FK)."
  flow_type: "Logical AI flow being executed; enum in body."
  provider: "AI provider that executed this run (e.g. gemini)."
  model_name: "Specific model used for this run."
  status: "Terminal status of the run; enum in body."
  input_payload: "Structured input passed to the model as JSONB; structure in body."
  output_payload: "Structured output returned by the model as JSONB; structure in body."
  error_message: "Error description recorded when the run fails."
  started_at: "When the run began."
  completed_at: "When the run reached a terminal state."
  progress_stage: "Granular execution stage; enum in body."
  debug_payload: "Optional diagnostic payload as JSONB; structure in body."
  input_tokens: "Input tokens consumed by the model."
  output_tokens: "Output tokens produced by the model."
  total_tokens: "Total tokens billed for this run."
sources:
  - "customer doc: cv-data-model-kb/CV_data_tool/tables/ai_runs.md"
  - "app DDL: CV_Builder/backend/supabase/migrations (ai_runs_flow_type_check, ai_runs_status_check, ai_runs_progress_stage_check, completion-consistency check)"
  - "app DDL: CV_Builder/backend/supabase/migrations/20260603160000_phase11_parallel_import_improve.sql (flow_type + professional_summary)"
  - "app DDL: CV_Builder/backend/supabase/migrations/20260801120000_phase14_skills_pool_prompt_flow.sql (flow_type + skills_pool; the operative constraint)"
  - "app code: CV_Builder/backend/src (per-flow input_payload builders, typed Record<string, unknown>)"
  - "machine doc: supabase.public.ai_runs"
depends_on:
  - supabase.public.users
  - supabase.public.master_cvs
  - supabase.public.tailored_cvs
  - supabase.public.jobs
  - supabase.public.ai_suggestions
  - supabase.public.ai_prompt_configs
contamination: null
---

# `supabase.public.ai_runs`

## Grain

One row per AI model invocation. Optional `master_cv_id`/`tailored_cv_id`/
`job_id` record the context (any combination may be null for context-free
flows). A consistency CHECK ties termination to timing: `completed_at` is set
**iff** `status` is `completed` or `failed`.

## Column meanings & enum decodings

- `flow_type` — DB-constrained (`ai_runs_flow_type_check`) to the **13-value**
  set shared with `supabase.public.ai_prompt_configs.flow_type`:
  **`job_analysis`, `follow_up_questions`, `tailored_draft`, `block_suggest`,
  `skills_pool`, `block_compare`, `multi_option`, `import_improve`,
  `professional_summary`, `summary`, `improve`, `cv_parse`,
  `cover_letter_generation`**. See Warnings — this doc previously listed 11.
- `status` — DB-constrained to **`pending` | `completed` | `failed`**.
- `progress_stage` — DB-constrained to **`queued`, `building_prompt`,
  `calling_model`, `parsing_output`, `validating_output`, `persisting_result`,
  `completed`, `failed`** (default `queued`); a finer view of a `pending` run.
- `total_tokens` — the token count to use for cost estimation (per customer
  doc); `input_tokens`/`output_tokens` are the split and may be null on older
  rows.
- `input_payload` (non-null), `output_payload`, `debug_payload` (`jsonb`) —
  **open maps** (`Record<string, unknown>` in code) whose shape **varies by
  `flow_type`**: each flow builds its own payload, so there is no single
  schema. `output_payload` is null until the run completes. Documented shape:
  per-flow JSON object; the concrete per-flow keys are app-defined and **not
  grounded** as a fixed structure here.

## Join guidance

- Owner/context: `user_id` → `users.id`; `master_cv_id` → `master_cvs.id`;
  `tailored_cv_id` → `tailored_cvs.id`; `job_id` → `jobs.id` (all
  `supabase.public.*`).
- Output: `supabase.public.ai_suggestions.ai_run_id` → `ai_runs.id` links a
  run to the suggestions it produced.

## Reporting notes

- AI cost/usage reports sum `total_tokens` grouped by `user_id`/`flow_type`;
  join `provider`/`model_name` for per-model breakdowns.
- Success rate = `completed` / all, by `flow_type`; use `progress_stage` to see
  where `pending` runs stall.

## Warnings

- **`flow_type` was documented as an 11-value set; the DB admits 13.** The two
  missing values are `professional_summary`, added by the phase-11 migration on
  **2026-06-03**, and `skills_pool`, added by phase 14 on **2026-08-01**. The
  11-value list was correct when written (it matched the 2026-04-29 constraint)
  and went stale in two steps after it. Reports that enumerate flows from the
  old list — or that treat it as exhaustive in a `CASE`/`IN` filter — silently
  drop both flows rather than erroring.
- Payload structure is flow-dependent — do not assume shared keys across
  `flow_type`s (gap above).
- `input_tokens`/`output_tokens`/`total_tokens` are nullable — treat null as
  unknown, not zero, in cost sums.
