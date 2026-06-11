# Analytics

OPVS collects telemetry on every package — install state, agent usage, upstream
health, ratings — and surfaces a slice of it back to vendors. This page is
honest about which slice is queryable **today** versus which dashboard is
**planned**, and points you at the real endpoints behind each number.

The short version: per-package quality signals (rating, score, version count)
and per-skill health (doctor) are queryable now. A dedicated vendor analytics
dashboard with install counts, per-tool call volume, and latency percentiles is
the planned richer surface.

---

## Section 1 — What's queryable today

Three live sources, no special analytics endpoint needed.

**1. Package detail metrics** — the registry detail read carries the quality
signals shown on your package page:

```bash
curl -s "https://market.opvs.ai/api/v1/registry/packages/@your-vendor/your-skill"
```

Fields you get back (from `PackageDetail` in
[`registry.py` schemas](https://github.com/opvs-ai/OPVS/blob/main/apps/spideragent/packages/marketplace/api/app/schemas/registry.py)):

| Field | Meaning |
|---|---|
| `security_score` | the `0–100` from the [security review](../04-publishing/security-review.md) |
| `avg_rating` | mean star rating (`null` until rated) |
| `rating_count` | number of ratings |
| `version_count` | how many versions you've published |

**2. Usage telemetry via search** — the marketplace search ranks packages using
a `agent_skill_usage_30d` rollup, and exposes the signals per result:
`cohort_call_count` (30-day agent call volume) and `success_rate`. When
telemetry is absent the result carries `telemetry_signal=absent` so you know the
number is "no data," not "zero."

**3. Upstream health via doctor** — the per-skill health check returns an
`upstream` block: `status`, latency, and `last_health_check`, with a 60-second
Redis cache and a HEAD-ping fallback. It **fails open** — no telemetry means
`upstream.status="unknown"`, not a false "down."

```bash
curl -s "https://app.opvs.ai/api/v1/skills/{skill_id}/doctor" \
  -H "X-Admin-Key: $OPVS_MARKETPLACE_ADMIN_KEY" -H "X-Brand-ID: {brand_id}"
```

---

## Section 2 — Platform stats (admin)

The marketplace exposes an aggregate `stats` roll-up — vendor and package
counts, the 24-hour publish rate, and revocation counts:

```bash
curl -s "https://app.opvs.ai/api/v1/marketplace/stats" \
  -H "X-Admin-Key: $OPVS_MARKETPLACE_ADMIN_KEY"
```

This is platform-wide, not per-vendor, and admin-gated. It's useful for "how big
is the marketplace right now," not for "how is *my* package doing" — for that,
use the per-package detail read in Section 1.

---

## Section 3 — Reading the numbers

A few interpretation notes so you don't over-read a metric:

- **`avg_rating` is `null`, not `0`, before the first rating.** Zero ratings ≠
  bad rating. Same for `cohort_call_count` with `telemetry_signal=absent`.
- **`success_rate` is the agent-cohort view**, not your upstream's own SLA. A
  low rate can mean agents are calling it wrong, not that your API is down — pair
  it with the doctor `upstream.status` to tell the two apart.
- **`security_score` is per version.** If you've published several versions,
  check you're reading the one brands are actually installing.

---

## Section 4 — The planned vendor dashboard

A first-class vendor analytics surface is on the roadmap. The intended shape:

- **Install counts** — total, active (last 30 days), new this week.
- **Per-tool call volume** — for each method on each skill, not just the package
  aggregate.
- **Error rates split by HTTP class** — 4xx (agent calling it wrong) vs 5xx
  (your upstream failing).
- **Latency percentiles** — p50 / p95 / p99 on outbound calls.
- **Tier-transition signals** — when runtime health would push a `verified`
  package back to `beta`, you'd see it here first (see [badges.md](badges.md)).

Two things are explicitly **out** of the planned surface: per-brand identifying
analytics (a privacy boundary — you see aggregates, not which brand did what),
and revenue (Phase 3, see [payouts.md](payouts.md)).

---

## See also

- [../04-publishing/security-review.md](../04-publishing/security-review.md) —
  where `security_score` comes from.
- [ratings-and-reviews.md](ratings-and-reviews.md) — the source of `avg_rating`
  / `rating_count`.
- [../reference/api.md](../reference/api.md) — the doctor and stats endpoints in
  the full table.
