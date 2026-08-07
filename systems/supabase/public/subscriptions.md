---
doc_class: human-object
object: supabase.public.subscriptions
written_against_schema_hash: "sha256:aa2bf4a4a88ec655bbb5e0ff78af326ac6c5f20da62f6803cfaa4940fb8c5972"
status: draft
last_verified: null
purpose: "Billing subscription records linking a user to an external payment-provider plan and its lifecycle state."
column_purposes:
  id: "Internal subscription id."
  user_id: "Owning user (FK to users.id)."
  provider: "External billing provider name (e.g. Stripe)."
  provider_customer_id: "Customer id on the provider side."
  provider_subscription_id: "Subscription id on the provider side."
  plan_code: "Internal product plan code purchased; see body for the grounded value set."
  status: "Current lifecycle status; see body for grounding limits."
  current_period_start: "Start of the current billing period."
  current_period_end: "End of the current billing period."
  cancel_at_period_end: "Whether the subscription will cancel at period end."
  created_at: "When the subscription record was created."
  updated_at: "When the subscription record was last updated."
sources:
  - "customer doc: cv-data-model-kb/CV_data_tool/tables/subscriptions.md"
  - "app DDL: CV_Builder/backend/supabase/migrations/20260417121000_phase1_tables.sql (subscriptions_one_active_per_user_idx)"
  - "machine doc: supabase.public.subscriptions"
  - "customer-provided, alper, 2026-08-07 (plan_code value set: free, pro)"
  - "observed, 2026-08-07: SELECT plan_code, status, count(*) FROM public.subscriptions GROUP BY 1,2 returned 0 rows"
  - "customer-provided, eda, 2026-08-07 (pro plan pricing by cadence: 4.99 weekly, 14.99 monthly, 99.00 annually; all three the same pro plan)"
  - "customer-provided, alper, 2026-08-07 (annual price 99.00 confirmed against a conflicting 99.99 figure in a second, still-open request)"
depends_on:
  - supabase.public.users
---

# `supabase.public.subscriptions`

## Grain

One row per subscription record for a user. A user can accumulate several over
time (renewals, plan changes, re-subscribes), but a partial unique index
enforces **at most one *active* subscription per user**
(`subscriptions_one_active_per_user_idx`: unique on `user_id` where
`status in ('active','trialing')`).

## Column meanings & enum decodings

- `status` — lifecycle state. **The DB does not CHECK-constrain this column**,
  so there is no closed enum to publish. Grounded values: **`active`** and
  **`trialing`** are the "active" states (from the partial unique index
  above); the customer model additionally lists **`canceled`** as an example.
  The complete provider-driven vocabulary (e.g. `past_due`, `incomplete`,
  `unpaid`) is **not grounded here** — a named gap; do not assume a fixed set.
  Note there is no `inactive` value in any grounded source, and `trialing` is
  a distinct third state — this column does not reduce to an active/inactive
  binary.
- `plan_code` — internal product plan identifier. Stated value set:
  **`free`** and **`pro`**. This set is **customer-stated, not observed and
  not DB-enforced** — there is no CHECK constraint on the column, and the
  table held **0 rows** when queried on 2026-08-07, so no value has ever been
  seen in data. Authoritative for intent; verify against rows before pinning
  a certified metric to it.
  The paid tier is sold on three billing cadences at one price each —
  **4.99 weekly**, **14.99 monthly**, **99.00 annually** — and all three are
  the *same* `pro` plan. Cadence is therefore **not** encoded in `plan_code`:
  a `pro` row may be any of the three. These amounts are customer-stated (see
  `sources`) and appear nowhere in the data — this table has no price, amount,
  or currency column, and no plan or price object exists elsewhere in the
  snapshot. The currency is stated only as "dollars"; no ISO code is grounded
  (gap).
- `provider` — free text; the customer model gives Stripe as the example. Not
  DB-enumerated.
- `provider_customer_id` / `provider_subscription_id` — the provider-side ids;
  both nullable and indexed *per provider* (`(provider, provider_*_id)` where
  not null), so uniqueness is scoped within a provider.
- `current_period_start` / `current_period_end` — nullable; absent for records
  with no established billing period yet.

## Join guidance

- Owner: `subscriptions.user_id` → `supabase.public.users.id`. There are no
  inbound FKs (nothing references `subscriptions`).
- To reconcile against the billing provider, match on
  `(provider, provider_subscription_id)`.

## Reporting notes

- "Currently paying / trialing users" = rows with `status in ('active',
  'trialing')` (one per user by the unique index). Do not `count(*)` all
  rows per user — historical rows inflate it.
- Churn signals: `cancel_at_period_end = true` flags a scheduled cancel that
  has not yet ended; a terminal `status` is a separate, already-ended state.
- Plan mix is `group by plan_code`, but see the empty-table warning below
  before publishing any such number.
- **Revenue cannot be computed from this table.** There is no price, amount,
  or currency column here and no plan/price object elsewhere in the snapshot;
  the plan prices recorded above are customer-stated reference values, not
  data. Take revenue from the billing provider and reconcile via
  `(provider, provider_subscription_id)`.
- **Billing cadence is not available.** Every paid row carries the same `pro`
  `plan_code`, so weekly, monthly, and annual subscribers are
  indistinguishable by plan. `current_period_start`/`current_period_end` do
  span a period, but no grounded source says that span is the cadence — do
  not derive one from it.

## Warnings

- `status` is **not** a DB enum — treat any set as open and provider-defined
  (gap above). Certified revenue/subscription metrics must pin the exact
  status values they count and cite the provider's own status vocabulary.
- `plan_code` is **not** a DB enum either — no CHECK constraint exists. The
  `free`/`pro` set is customer-stated (see `sources`), not enforced and not
  observed. A new plan code can enter the data with no schema change and no
  change to this doc's schema hash, so nothing will flag this doc stale when
  it happens. Do not treat it as closed in a certified metric.
- The table was **empty (0 rows) as of 2026-08-07**. Any plan-mix or
  subscription-count report built on it returns nothing today, which makes a
  zero result indistinguishable from a broken query. Confirm the table is
  non-empty before interpreting a zero.
- The plan prices above are **customer-stated on 2026-08-07 and absent from
  the data**, so nothing in the estate can confirm or contradict them and no
  drift check will ever re-examine them. A price change ships no schema
  change, so this doc will keep asserting these figures until a human edits
  it. Re-confirm before quoting them externally.
- The stated amounts carry **no explicit currency** — "dollars", with no ISO
  code grounded and no currency column on the table. A named gap; do not
  assume USD in a certified metric.
- No `is_deleted`/soft-delete here; lifecycle is expressed through `status`.
