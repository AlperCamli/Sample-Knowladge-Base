---
doc_class: metric
status: verified
last_verified: "2026-08-08 (Alper Camli)"
owner: "alper (operator)"
aliases: [paying base, active base, live subscriptions]
sources:
  - "customer-verified SQL: benchmark suite case RB-02 (.contextlayer/benchmark/suite.yaml)"
  - "machine doc: supabase.reporting.v_subscriptions_by_plan (view definition)"
  - "human doc: supabase.public.subscriptions"
depends_on:
  - supabase.public.subscriptions
  - supabase.reporting.v_subscriptions_by_plan
contamination: null
---

# active subscriptions

## Definition

**Subscriptions in a live state right now**, where live means
`status IN ('active','trialing')`. A *stock* measure: it describes the base at
the moment of the query and has no window. The customer resolved "who's
currently paying" to exactly these two states (RB-02) because a partial unique
index (`subscriptions_one_active_per_user_idx`) admits at most one such row per
user — so counting rows counts people, with no de-duplication needed.

## Formula

`count(*)` over `public.subscriptions` where `status IN ('active','trialing')`,
optionally grouped by `plan_code`.

## Implementations

**`supabase` — base table.** Verbatim from RB-02:

```sql
SELECT plan_code, status, count(*) AS subscribers
FROM public.subscriptions
WHERE status IN ('active','trialing')
GROUP BY plan_code, status
ORDER BY plan_code, status;
```

**`supabase` — reporting view** (`reporting.v_subscriptions_by_plan`, the RLS
route), filtered to the same two states:

```sql
SELECT plan_code, status, subscription_count
FROM reporting.v_subscriptions_by_plan
WHERE status IN ('active','trialing');
```

## Grain & dimensions

One row per `(plan_code, status)`. `trialing` is **not** paying money and is
included by the customer's own definition of the live base; split by `status`
before quoting a revenue-adjacent number.

## Known discrepancies

- **`status` has no database CHECK constraint.** `active`, `trialing` and
  `canceled` are grounded (index + customer statement); the provider's full
  vocabulary (`past_due`, `incomplete`, `unpaid`, …) is not. A row in an
  ungrounded state is silently outside this metric. Named as a gap in
  [`subscriptions.md`](../systems/supabase/public/subscriptions.md) and unclosed
  until values are observed in data.
- **The table held 0 rows when last queried (2026-08-07).** Nothing here has
  been validated against real values.
- `plan_code` does not encode billing cadence — the same `pro` plan is sold
  weekly, monthly and annually. Do not derive cadence from this metric.
