---
doc_class: metric
status: verified
last_verified: "2026-08-08 (Alper Camli)"
owner: "alper (operator)"
aliases: [purchases, paid conversions, key events]
sources:
  - "customer-verified SQL + API golden: benchmark suite case RB-08 (.contextlayer/benchmark/suite.yaml)"
  - "human doc: entities/conversion.md (routing hub)"
  - "machine doc: ga4.standard.keyEvents:purchase"
depends_on:
  - ga4.standard.keyEvents:purchase
  - ga4.standard.purchaseRevenue
  - supabase.public.subscriptions
contamination: null
---

# conversions

## Definition

**A person starting to pay.** Two systems claim to measure it and they measure
different things, which is why this doc exists rather than one number:

- **Supabase (system of record):** a new `public.subscriptions` row. This is
  what actually happened in the business.
- **GA4 (analytics):** a `purchase` key event fired by the front end. This is
  what the browser reported.

**The Supabase count is authoritative.** GA4's is the marketing-attribution
view of the same intent and is used for *reconciliation*, never as the source
of a revenue number.

## Formula

Supabase: `count(*)` of subscription rows created in the window, grouped by
`status`. GA4: the `keyEvents:purchase` metric over the same window, with
`purchaseRevenue` beside it.

## Implementations

**`supabase`** — verbatim from RB-08's Supabase leg (identical to
[`new-subscriptions`](new-subscriptions.md), which is the same measurement
under its own name):

```sql
SELECT status, count(*) AS new_subscriptions
FROM public.subscriptions
WHERE created_at >= '<window_start>'
  AND created_at <  '<window_end>'
GROUP BY status
ORDER BY status;
```

**`ga4`** — verbatim from RB-08's GA4 leg:

```json
{"operation": "runReport", "property": "properties/542045072",
 "body": {"dateRanges": [{"startDate": "<window_start>", "endDate": "<window_end>"}],
          "dimensions": [],
          "metrics": [{"name": "keyEvents:purchase"}, {"name": "purchaseRevenue"}]}}
```

## Grain & dimensions

Both implementations are property/table-wide totals over a window.
**No row-level join exists** between them — the front end sets no GA4 User-ID,
so a purchase event cannot be traced to a subscription row. Comparison is
aggregate-to-aggregate only, per
[`entities/conversion.md`](../entities/conversion.md).

## Known discrepancies

This is the metric where the two numbers are *expected* to differ, and a report
that shows them as one number is wrong:

- **Trials.** A `trialing` subscription row is a conversion in Supabase and
  usually fires no `purchase` event. Group by `status` before comparing.
- **Blocked analytics.** Ad-blockers and consent refusals suppress GA4 events;
  the subscription row is written regardless. GA4 therefore reads low, by an
  unmeasured amount.
- **Client-side firing.** A purchase completed after the browser closed may
  never report to GA4 at all.
- **`purchaseRevenue` is not billed revenue.** It carries whatever value the
  front end attached to the event; the estate holds no price column, and the
  billed amount lives with the payment provider. The open pricing question in
  the fault ledger (`4c4ecb3d`) is exactly this seam.
- **`subscriptions.status` has no DB CHECK** — the same grounding gap as
  [`active-subscriptions`](active-subscriptions.md).
