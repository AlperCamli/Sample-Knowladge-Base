---
doc_class: metric
status: verified
last_verified: "2026-08-08 (Alper Camli)"
owner: "alper (operator)"
aliases: [activation funnel, onboarding funnel, cohort activation]
sources:
  - "customer-verified SQL: benchmark suite case RB-07 (.contextlayer/benchmark/suite.yaml)"
  - "machine doc: supabase.reporting.v_activation_funnel_monthly (view definition)"
  - "human doc: supabase.public.users"
depends_on:
  - supabase.public.users
  - supabase.public.master_cvs
  - supabase.public.tailored_cvs
  - supabase.public.exports
  - supabase.public.subscriptions
  - supabase.reporting.v_activation_funnel_monthly
contamination: null
---

# activation rate

## Definition

**The share of a signup cohort that reached each product step, within the same
period they signed up in.** Five steps, in order: signed up → created a master
CV → created a tailored CV → completed an export → started a subscription. Each
step counts *distinct users from the cohort*, so a user with three CVs counts
once.

The same-period rule is what makes this deterministic: a step only counts if it
happened before the cohort period ended, which is why a June cohort's numbers
stop moving once June does.

## Formula

Per step, `count(DISTINCT user_id)` among the cohort where the step's event
occurred before the cohort window closed; the rate is that count over the
cohort size (step 1).

## Implementations

**`supabase` — base tables**, verbatim from RB-07 (window as authored):

```sql
WITH cohort AS (
  SELECT id FROM public.users
  WHERE created_at >= '<window_start>' AND created_at < '<window_end>'
)
SELECT
  (SELECT count(*) FROM cohort)                                     AS s1_signed_up,
  (SELECT count(DISTINCT m.user_id)
     FROM public.master_cvs m JOIN cohort c ON c.id = m.user_id
     WHERE m.is_deleted = false AND m.created_at < '<window_end>')  AS s2_created_master_cv,
  (SELECT count(DISTINCT t.user_id)
     FROM public.tailored_cvs t JOIN cohort c ON c.id = t.user_id
     WHERE t.is_deleted = false AND t.created_at < '<window_end>')  AS s3_created_tailored_cv,
  (SELECT count(DISTINCT e.user_id)
     FROM public.exports e JOIN cohort c ON c.id = e.user_id
     WHERE e.status = 'completed' AND e.created_at < '<window_end>') AS s4_exported,
  (SELECT count(DISTINCT s.user_id)
     FROM public.subscriptions s JOIN cohort c ON c.id = s.user_id
     WHERE s.created_at < '<window_end>')                            AS s5_subscribed;
```

**`supabase` — reporting view** (`reporting.v_activation_funnel_monthly`, the
RLS route), the same five steps on **calendar-month cohorts**:

```sql
SELECT cohort_month, signed_up, created_master_cv, created_tailored_cv, exported, subscribed
FROM reporting.v_activation_funnel_monthly
ORDER BY cohort_month;
```

## Grain & dimensions

One row per cohort. The view fixes the cohort to a calendar month and the
"within the period" bound to that month's end; the base-table version takes any
window. **They are not interchangeable** — a two-week cohort cannot be read off
the monthly view.

## Known discrepancies

- **Soft deletes are respected for CVs and not for subscriptions.** Steps 2 and
  3 exclude `is_deleted = true` rows; steps 4 and 5 have no such column to
  exclude. A deleted CV removes a user from step 2; a cancelled subscription
  does not remove them from step 5.
- **Step 5 counts any subscription row, at any status**, including a trial that
  never converted. It is "started a subscription", not "paid".
- Steps are **not** strictly nested: a user can export without a tailored CV
  (master-CV exports exist), so a later step can exceed an earlier one. Do not
  render this as a monotonic funnel without checking.
