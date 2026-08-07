---
doc_class: human-object
object: supabase.reporting.v_mart_dim_breakouts
written_against_schema_hash: "sha256:3c9c774ef9a3f94a4aa89215020fe4ee1e2ce3a5d0e41dab50ef1f2ce83104ed"
status: draft
last_verified: null
purpose: "Six unrelated category breakdowns stacked into one long table, discriminated by `dim`; rows from different `dim` values are not comparable."
column_purposes:
  dim: "Which breakdown the row belongs to; closed set of six values — see body. Always filter on this first."
  key1: "Primary category of the breakdown; means a different thing per `dim` — see body."
  key2: "Secondary category, or null where the breakdown has only one — see body."
  item_count: "Count of the thing being broken out; the unit differs per `dim` — see body."
  user_count: "Distinct users behind the row; null for `subscriptions_by_plan`, and never additive — see Warnings."
  total_bytes: "Stored bytes; populated only for `dim = 'files_by_type'`, null everywhere else."
  cancelling_count: "Subscriptions set to cancel at period end; populated only for `dim = 'subscriptions_by_plan'`."
sources:
  - "machine doc: supabase.reporting.v_mart_dim_breakouts (view definition, pg_get_viewdef canonical)"
  - "machine doc: the six source views' definitions — v_exports_by_format, v_imports_by_parser, v_files_by_type, v_master_cvs_by_language, v_jobs_by_status, v_subscriptions_by_plan"
  - "human doc: supabase.public.subscriptions (0 rows observed 2026-08-07)"
  - "observed 2026-08-07: `count(*)` as `contextlayer_exec` returns `permission denied` on the mart views, while the same role reads their source views normally — the grant is missing on the mart views specifically"
depends_on:
  - supabase.reporting.v_exports_by_format
  - supabase.reporting.v_imports_by_parser
  - supabase.reporting.v_files_by_type
  - supabase.reporting.v_master_cvs_by_language
  - supabase.reporting.v_jobs_by_status
  - supabase.reporting.v_subscriptions_by_plan
  - supabase.public.subscriptions
---

# `supabase.reporting.v_mart_dim_breakouts`

## Grain

One row per `(dim, key1, key2)`. The view is a six-way `UNION ALL` over six
existing reporting views, each contributing its own rows under a literal
`dim` tag. Rows from different `dim` values are **not comparable** — they
count different objects in different units.

## Column meanings & enum decodings

`dim` is a closed set, fixed in the view text as six SQL literals. It can
only change by a migration that rewrites the view, so it is safe to treat as
an enum:

| `dim` | `key1` | `key2` | `item_count` counts |
|---|---|---|---|
| `exports_by_format` | export format | export status | export jobs |
| `imports_by_parser` | parser name | import status | import jobs |
| `files_by_type` | file type | MIME type | non-deleted files |
| `master_cvs_by_language` | CV language | source type | master CVs |
| `jobs_by_status` | job status | *(null)* | job postings |
| `subscriptions_by_plan` | plan code | subscription status | subscription records |

Measure columns are populated per `dim` — everything not listed is a literal
`NULL` cast in the view, not missing data:

| `dim` | `user_count` | `total_bytes` | `cancelling_count` |
|---|---|---|---|
| `exports_by_format` | yes | — | — |
| `imports_by_parser` | yes | — | — |
| `files_by_type` | yes | **yes** | — |
| `master_cvs_by_language` | yes | — | — |
| `jobs_by_status` | yes | — | — |
| `subscriptions_by_plan` | **null** | — | **yes** |

## Reporting notes

- The shape — a `dim` discriminator over generic `key1`/`key2` columns —
  suits a chart whose breakdown is selected at query time by parameterising
  `dim`. That is an inference from the view's structure; **no source states
  who consumes this view or what it was built for** (named gap).
- `key2` is null for `jobs_by_status` because that breakdown has a single
  category. Grouping on `(key1, key2)` within that `dim` is still correct.

## Warnings

- **The reporting role cannot read this view.** A `count(*)` as
  `contextlayer_exec` fails with `permission denied` (observed 2026-08-07),
  while the same role reads all six source views normally. No `GRANT SELECT`
  reaches the execution role, so everything below is derived from the view
  text rather than from data — including the claim that the `dim` set has
  exactly six values, which is read off the `UNION ALL` and has not been
  confirmed against rows. Until a DBA applies the grant, query the six
  source views directly.
- **Always filter to one `dim` before aggregating.** `SUM(item_count)`
  across the whole view adds exports to files to job postings to
  subscriptions and returns a number that means nothing. Nothing in the
  schema prevents this — the column is a plain `bigint` — so the filter is
  the consumer's responsibility on every query.
- **`user_count` is not additive within a `dim` either.** Each source view
  computes `count(DISTINCT user_id)` inside its own group, so a user active
  in three export formats is counted once per format. Summing rows
  double-counts; there is no correct total-user figure derivable from this
  view.
- **`user_count` is null — not zero — for `subscriptions_by_plan`.** The
  source view does not compute it, and the union casts a literal null. An
  aggregate that coalesces it to zero silently asserts "no users had
  subscriptions", which is a different claim from "not measured".
- **`total_bytes` excludes soft-deleted files.** `v_files_by_type` filters
  `WHERE NOT is_deleted`, so this is live storage, not everything ever
  uploaded. It will not reconcile against a raw `sum(size_bytes)` on
  `public.files`.
- **The `subscriptions_by_plan` rows are empty in practice.**
  `public.subscriptions` held 0 rows when queried on 2026-08-07, so that
  `dim` currently contributes no rows at all. A breakdown that renders blank
  is expected today and is not a broken query — see the human doc for
  `supabase.public.subscriptions`, including the caveat that `plan_code`'s
  value set is customer-stated rather than DB-enforced.
- **Columns are dropped in the union.** `v_jobs_by_status.first_created` /
  `last_created` and `v_files_by_type.avg_bytes` exist upstream but have no
  slot here. Go to the source view when you need them; do not reconstruct
  `avg_bytes` as `total_bytes / item_count`, which would differ because the
  source rounds.
