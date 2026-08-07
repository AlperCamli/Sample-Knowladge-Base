---
doc_class: human-object
object: supabase.reporting.v_subscriptions_by_plan
written_against_schema_hash: "sha256:1abbef8441881fd2ac21600b36be45426643c2b38c1609fe6d31fb9da753b853"
status: draft
last_verified: null
purpose: "Subscription mix by plan and status, with pending cancellations."
column_purposes:
  plan_code: "Plan the subscription was bought on; see body for the catalogue and its list prices."
  status: "Subscription status."
  subscription_count: "Subscriptions matching this plan/status."
  cancelling_count: "How many are flagged to cancel at period end."
sources:
  - "platform: deploy/reporting-views.sql (view definition, CP-6/M2)"
  - "machine doc: supabase.reporting.v_subscriptions_by_plan"
  - "human doc: supabase.public.subscriptions"
  - "customer-confirmed, Alper, 2026-08-06 (list prices: weekly 4.99, monthly 14.99, annual 99.99 USD)"
  - "app code: CV_Builder/backend/src/modules/entitlements/plan-definitions.ts @4265702 (plan catalogue: the six codes, checkout_allowed, trial_supported, legacy flags)"
  - "app code: CV_Builder/frontend/src/content/pricing.ts @4265702, 2026-06-28 (displayed price constants and plan cards)"
  - "app code: CV_Builder/frontend/src/content/pricing.test.ts @4265702 (test-pinned: plan set = free/weekly/monthly/annual, PLAN_VALUE_USD, pro+lifetime legacy-only)"
  - "app code: CV_Builder/backend/src/shared/config/env.ts @4265702 (STRIPE_*_PRICE_ID wiring; BILLING_TRIAL_PERIOD_DAYS default 3)"
  - "app code: CV_Builder/backend/src/modules/billing/subscriptions.repository.ts @4265702 (free/inactive linkage stub insert; trial-capable plans pro+weekly)"
  - "customer doc: cv-data-model-kb/CV_data_tool/queries/subscription_conversion_revenue_report.md (no price column; multiply plan counts by price externally for MRR)"
depends_on:
  - supabase.public.subscriptions
contamination: null
---

# `supabase.reporting.v_subscriptions_by_plan`

## Grain

One row per (plan_code, status) pair observed.

## Column meanings & enum decodings

### `plan_code` — the catalogue and its list prices

The application's plan catalogue defines six codes in two groups. Prices are
**list prices in USD**; none of them is stored in the database (see Warnings).

| `plan_code` | Name | Sold today | List price | Billed |
|---|---|---|---|---|
| `free` | Free | no — assigned, not bought | $0 | — |
| `weekly` | Weekly | yes | $4.99 | per week |
| `monthly` | Monthly | yes | $14.99 | per month |
| `annual` | Annual | yes | **$99.99 — sources disagree, see Warnings** | per year |
| `pro` | Legacy Pro | no — closed | not grounded | — |
| `lifetime` | Legacy Lifetime Pro | no — closed | not grounded | — |

- `weekly`, `monthly` and `annual` are the only codes checkout will accept.
  `pro` and `lifetime` are carried by pre-existing subscriptions only and are
  marked `legacy` in the catalogue, so their counts can shrink but never grow.
- `free` is not a purchase. It is written on signup and again as a billing
  **linkage stub** (`plan_code = 'free'`, `status = 'inactive'`) when a
  provider customer record is attached to a user. Those rows represent no
  money and no active entitlement — exclude them from any revenue figure.
- Free-plan entitlements (the monthly caps that make `free` different from a
  paid plan) are documented on `supabase.public.usage_counters`, not here.
- The `weekly` plan is the only one sold with a trial — **3 free days**, the
  configured default. A row in `status = 'trialing'` is therefore billing
  nothing yet. Legacy `pro` also granted trials historically.

## Reporting notes

- The revenue-mix view. `cancelling_count` against `subscription_count` on
  active rows is forward-looking churn — these have not cancelled yet, so it is
  the number worth acting on.
- **Computing recurring revenue.** The estate stores no amount, so revenue is
  `subscription_count × list price`, multiplied outside the database from the
  table above. Three things make that arithmetic wrong if ignored: the plans
  bill on **different periods** (a weekly and an annual row are not comparable
  until both are normalised to the same interval), `trialing` rows are billing
  **nothing yet**, and `free`/`inactive` rows are not revenue at all. Any MRR
  or ARR figure built here is an **estimate from list prices** — it sees no
  discount, coupon, proration, refund, tax or failed payment.

## Warnings

- **These are aggregate views over row-level-security-protected tables.** The
  base tables (`public.*`) enforce per-user RLS, so the execution role reads
  **zero rows** from them directly. These views answer anyway because a view
  evaluates RLS as its *owner*, and they are owned by a role that bypasses it.
  That is a deliberate, narrow opening — and it is why every column here is an
  aggregate or a non-identifying dimension. Do not expect to drill from these
  numbers to a person: no view exposes `user_id`, an email, a name, or any free
  text a user wrote.
- **Small counts can still identify.** The estate is ~24 users. A group of size
  1–2 is potentially personal even though no column names anyone; treat such
  cells as sensitive rather than reportable.
- **No provider identifiers are exposed here** (no customer or subscription
  ids) — deliberately, since they are personal. This view cannot be joined back
  to an individual account.
- Counts subscriptions, not distinct payers; one user with two subscriptions
  counts twice.
- Carries no monetary amount. The prices above are documented here because the
  estate has nowhere else to hold them — there is **no price, amount, currency
  or billing-interval column anywhere in `supabase`**. They are prose, not data:
  nothing validates them against a schema and no drift check will notice when
  they go out of date.
- **The annual price is not settled. Sources disagree: $99.99 (customer-
  confirmed) against $99.90 (the app's own displayed constant,
  `ANNUAL_TOTAL_PRICE`, and its analytics value `PLAN_VALUE_USD.annual = 99.9`,
  last changed 2026-06-28).** The table above records the customer-confirmed
  figure. A nine-cent gap is immaterial to a plan-mix report and material to a
  reconciliation against the payment provider — do not use this number to tie
  out to Stripe until someone resolves which is right.
- **The provider, not this estate, is the system of record for what a customer
  was actually charged.** The application never holds an amount: it sends
  Stripe a `stripe_price_id` taken from environment configuration
  (`STRIPE_WEEKLY_PRICE_ID` and siblings) and Stripe prices the line item. A
  price changed in the Stripe dashboard changes the charge with no code change,
  no schema change, and no signal to this document. Treat the table above as
  the *intended list price*, and the provider's own records as the truth.
- **These are list prices as of the citation date, not per-row historical
  prices.** The base table stores only the current `plan_code`, with no price
  and no plan-change history, so if a price has ever changed, a subscriber
  still billing the older one is indistinguishable here. Whether existing
  subscriptions were repriced or left on the old price is a question only the
  provider's records answer — nothing in the estate does.
- `pro` and `lifetime` have **no grounded price** — a named gap, not a zero.
- **`lifetime` is a one-off payment and never expires.** It is recorded from a
  Stripe checkout session in `payment` mode, not `subscription` mode, and lands
  as `status = 'active'` with `current_period_end = null` and a synthetic
  provider id (`lifetime_<session_id>`). Nothing ever moves that row off
  `active`. So a `lifetime` row inflates every "currently paying" count
  forever, and contributes **no recurring revenue** — exclude it from MRR and
  ARR rather than pricing it.
