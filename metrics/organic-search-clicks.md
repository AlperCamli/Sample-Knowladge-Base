---
doc_class: metric
status: verified
last_verified: "2026-08-08 (Alper Camli)"
owner: "alper (operator)"
aliases: [search clicks, google clicks, organic clicks]
sources:
  - "customer-verified API golden: benchmark suite cases RB-03 and RB-05 (.contextlayer/benchmark/suite.yaml)"
  - "machine doc: gsc.standard.clicks"
depends_on:
  - gsc.standard.clicks
  - gsc.standard.query
  - gsc.standard.impressions
contamination: null
---

# organic search clicks

## Definition

**Clicks from Google Search results to the property**, as Search Console counts
them. The top of the acquisition funnel and the only stage of it this estate
measures at source rather than by inference.

## Formula

The Search Console `clicks` metric over the window, at the requested dimension
grain (`query`, `page`, or none for the property total).

## Implementations

**`gsc` — Search Analytics API.** Verbatim from RB-03 (top queries):

```json
{"operation": "searchanalytics.query",
 "property": "sc-domain:jobspecificcv.com",
 "body": {"startDate": "<window_start>", "endDate": "<window_end>",
          "dimensions": ["query"], "rowLimit": 20, "dataState": "final"}}
```

And from RB-05 (property total, no dimensions):

```json
{"operation": "searchanalytics.query",
 "property": "sc-domain:jobspecificcv.com",
 "body": {"startDate": "<window_start>", "endDate": "<window_end>",
          "dimensions": [], "dataState": "final"}}
```

**`dataState: "final"` is part of the metric, not a preference.** Search Console
data matures for days after the fact; a `final` request is the difference
between a number that stays put and one that quietly grows after the report is
sent.

## Grain & dimensions

Per query, per page, per country/device, or the property total. `clicks`,
`impressions`, `ctr` and `position` come back together; `ctr` is
`clicks / impressions` **as the API computes it** and must not be recomputed
from rounded values.

## Known discrepancies

- **Rows are omitted, not zeroed.** Queries below Google's privacy threshold
  never appear, so per-query clicks sum to *less* than the property total. A
  report that sums the top-20 and calls it "search traffic" understates it.
- **Not comparable to GA4 sessions.** A click is Google's count of a result
  being clicked; a session is the site's count of a visit. The funnel case
  (RB-05) places them as consecutive stages precisely because they measure
  different events — see [`entities/user.md`](../entities/user.md) on why no
  row-level join exists.
- Position is an average rank and is meaningless when summed.
