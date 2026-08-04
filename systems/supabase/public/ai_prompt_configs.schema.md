---
doc_class: machine-object
object: supabase.public.ai_prompt_configs
kind: table
schema_hash: "sha256:9346b889e28062380fae34e5c838e89b2abfbb138452f56de47327ed04bde6ee"
generated_at: 2026-08-04
source_mode: live
snapshot_version: "1"
status: machine
---

# `supabase.public.ai_prompt_configs`

## Identity

| Fact | Value |
|---|---|
| Object | `supabase.public.ai_prompt_configs` |
| Kind | table |
| Schema hash | `sha256:9346b889e28062380fae34e5c838e89b2abfbb138452f56de47327ed04bde6ee` |

## Columns

| # | Column | Type | Nullable | Default | Description | Purpose |
|---|---|---|---|---|---|---|
| 1 | `id` | `uuid` | false | `gen_random_uuid()` | — | Internal prompt-config id. |
| 2 | `profile` | `text` | false | — | — | Named prompt profile this config belongs to (e.g. phase3-v1). |
| 3 | `flow_type` | `text` | false | — | — | AI flow this prompt targets; enum in body. |
| 4 | `action_type` | `text` | true | — | — | Optional block action this prompt targets; enum in body. |
| 5 | `provider` | `text` | false | `'gemini'::text` | — | Provider this prompt is registered for; 'any' is the fallback. |
| 6 | `model_name` | `text` | true | — | — | Specific model this prompt is pinned to, or null for provider default. |
| 7 | `prompt_key` | `text` | false | — | — | Stable lookup key identifying the prompt content (e.g. cv-parse). |
| 8 | `prompt_version` | `text` | false | — | — | Semantic version label of the prompt content (e.g. phase5-v3). |
| 9 | `system_prompt` | `text` | false | — | — | System prompt body sent to the model. |
| 10 | `user_prompt_template` | `text` | true | — | — | Optional user prompt template body. |
| 11 | `is_active` | `boolean` | false | `true` | — | Whether this row is eligible for runtime resolution. |
| 12 | `created_at` | `timestamp with time zone` | false | `now()` | — | When the prompt config was created. |
| 13 | `updated_at` | `timestamp with time zone` | false | `now()` | — | When the prompt config was last updated. |

## Keys & indexes

Primary key: `id`

Foreign keys: —

Unique constraints: —

Indexes:

- `CREATE INDEX ai_prompt_configs_profile_lookup_idx ON public.ai_prompt_configs USING btree (profile, is_active, flow_type, action_type, provider)`
- `CREATE UNIQUE INDEX ai_prompt_configs_active_unique_idx ON public.ai_prompt_configs USING btree (profile, flow_type, COALESCE(action_type, ''::text), provider) WHERE (is_active = true)`

Check constraints:

- `CHECK (action_type IS NULL OR (action_type = ANY (ARRAY['rewrite'::text, 'summarize'::text, 'improve'::text, 'ats_optimize'::text, 'options'::text, 'expand'::text, 'shorten'::text])))`
- `CHECK (flow_type = ANY (ARRAY['job_analysis'::text, 'follow_up_questions'::text, 'tailored_draft'::text, 'block_suggest'::text, 'skills_pool'::text, 'block_compare'::text, 'multi_option'::text, 'import_improve'::text, 'professional_summary'::text, 'summary'::text, 'improve'::text, 'cv_parse'::text, 'cover_letter_generation'::text]))`

## Row estimate & freshness

Row estimate: 16

Freshness: facts reflect the snapshot recorded in `generated_at` (front-matter).

## Referenced-by

—
