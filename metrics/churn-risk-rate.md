---
doc_class: metric
status: draft
last_verified: null
owner: "alper (operator) — pending"
aliases: [scheduled cancels, forward churn, cancel-at-period-end rate]
sources:
  - "customer-verified SQL: benchmark suite case RB-10 (.contextlayer/benchmark/suite.yaml)"
  - "machine doc: supabase.reporting.v_subscriptions_by_plan (view definition)"
  - "human doc: supabase.public.subscriptions"
depends_on:
  - supabase.public.subscriptions
  - supabase.reporting.v_subscriptions_by_plan
contamination: null
---

# churn risk rate

## Definition

**The share of the live base that is flagged to cancel at period end.**
Forward-looking: these subscriptions have *not* cancelled — they are still
active and still paying through the current period — so this is the number
worth acting on, not a record of churn that already happened.

## Formula

`scheduled_to_cancel / active_base`, where the base is
[`active-subscriptions`](active-subscriptions.md) and the numerator is that
same base with `cancel_at_period_end = true`. Expressed as a percentage to one
decimal place in the customer's own golden.

## Implementations

**`supabase` — base table.** Verbatim from RB-10:

```sql
SELECT
  count(*) FILTER (WHERE status IN ('active','trialing'))                          AS active_base,
  count(*) FILTER (WHERE status IN ('active','trialing') AND cancel_at_period_end)  AS scheduled_to_cancel,
  round(100.0 * count(*) FILTER (WHERE status IN ('active','trialing') AND cancel_at_period_end)
        / nullif(count(*) FILTER (WHERE status IN ('active','trialing')), 0), 1)    AS churn_risk_pct
FROM public.subscriptions;
```

**`supabase` — reporting view** (`reporting.v_subscriptions_by_plan`, the RLS
route), which carries both counts per plan and status:

```sql
SELECT round(100.0 * sum(cancelling_count) / nullif(sum(subscription_count), 0), 1) AS churn_risk_pct
FROM reporting.v_subscriptions_by_plan
WHERE status IN ('active','trialing');
```

`nullif` is load-bearing in both: an empty base returns null, never a division
error and never a fabricated `0.0%`.

## Grain & dimensions

A single scorecard number, or one per `plan_code` through the view. No time
dimension — the flag is current state, and the estate keeps no subscription
status history, so **this metric cannot be trended**. Anyone asking for churn
over time is asking for a table that does not exist.

## Known discrepancies

- **Not the same as realised churn.** A cancellation that already completed
  leaves the live base entirely and appears in neither term of this fraction.
- `trialing` rows count in the base, so a trial ending without conversion reads
  as churn risk. That is the customer's stated definition; split by `status`
  if the distinction matters.
- The `status` grounding gap from `active-subscriptions` applies unchanged.
