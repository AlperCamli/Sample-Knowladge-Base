---
doc_class: metric
status: verified
last_verified: "2026-08-08 (Alper Camli)"
owner: "alper (operator)"
aliases: [subscription starts, new paid conversions, subscription flow]
sources:
  - "customer-verified SQL: benchmark suite case RB-08 (.contextlayer/benchmark/suite.yaml)"
  - "machine doc: supabase.reporting.v_subscriptions_new_by_month (view definition)"
  - "human doc: supabase.public.subscriptions"
depends_on:
  - supabase.public.subscriptions
  - supabase.reporting.v_subscriptions_new_by_month
contamination: null
---

# new subscriptions

## Definition

**Subscription rows created in the window, by `created_at`.** A *flow* measure,
and the counterpart to [`active-subscriptions.md`](active-subscriptions.md)'s
stock: this one counts events (a subscription began), that one counts state
(a subscription is live). Grouped by `status` so a trial start sits beside a
paid start rather than hiding inside one total.

## Formula

`count(*)` over `public.subscriptions` where `created_at` falls in the window,
grouped by `status`.

## Implementations

**`supabase` — base table.** Verbatim from RB-08's Supabase leg:

```sql
SELECT status, count(*) AS new_subscriptions
FROM public.subscriptions
WHERE created_at >= '<window_start>'
  AND created_at <  '<window_end>'
GROUP BY status
ORDER BY status;
```

**`supabase` — reporting view** (`reporting.v_subscriptions_new_by_month`,
monthly buckets, the RLS route):

```sql
SELECT month, status, new_subscriptions
FROM reporting.v_subscriptions_new_by_month
WHERE month >= '<window_start_month>' AND month < '<window_end_month>';
```

## Grain & dimensions

Base-table implementation: one row per `status` over an arbitrary window. View
implementation: one row per `(month, status)`. **The two grains do not mix** —
a monthly view row cannot answer a mid-month window.

## Known discrepancies

- A subscription that started and was cancelled inside the window still counts
  here and does not appear in `active-subscriptions`. That is the intended
  difference between a flow and a stock, and it is the usual cause of the two
  numbers "disagreeing".
- The `status` grounding gap from `active-subscriptions` applies unchanged.
- Reconciliation against GA4 purchases is documented in
  [`conversions.md`](conversions.md), which is where the two sources' numbers
  are expected to differ.
