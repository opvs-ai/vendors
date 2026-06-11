# Payouts

> **There is no monetization model on the platform today.** Every package is
> free to install. Nothing meters, bills, or pays out. This page documents that
> honestly and sketches the Phase 3 plan so the docs tree is ready when it
> lands.

If you're a vendor deciding whether to publish, decide on the value of
distribution and adoption — not on revenue, because there isn't any yet.

---

## Section 1 — What's shipped: nothing billable

The only monetization-adjacent thing in the system today is a single manifest
field, `pricing`, which defaults to `none`:

```yaml
# package.yaml
pricing: none      # the only shipped value; Phase 3 metadata placeholder
```

It's declared in the manifest validator
([`package_manifest.py`](https://github.com/opvs-ai/OPVS/blob/main/apps/spideragent/packages/marketplace/api/app/schemas/package_manifest.py))
as forward-looking metadata. It does **not** gate installs, meter usage, or
trigger billing. There is no payout endpoint, no metering job, no wallet, no
invoice. Installs are free and unmetered.

---

## Section 2 — Why it's here anyway

Two reasons this page exists despite there being nothing to pay:

1. **The information architecture is ready.** When monetization ships, the page
   is already in the tree at a stable URL — no reshuffle, no broken links.
2. **It sets expectations.** A vendor reading the promotion section shouldn't
   have to guess whether they'll get paid. The answer is: not yet, here's the
   plan.

The redesign planner flagged removing this page entirely until there's a real
model — that open question lives in `RESULT.md` on the
`redesign/A-information-architecture` branch. For now it stays as an honest
placeholder.

---

## Section 3 — The Phase 3 sketch (not committed)

The shape under discussion — **subject to change, nothing wired**:

- **Revenue share** — a target split (the figure discussed is 70/30
  vendor/OPVS), settled per billing period.
- **Per-call metering** — counting what a skill actually does at runtime, via the
  existing gateway telemetry rather than a new meter.
- **Free vs. paid tiers** — a free allowance per package with paid unlocks above
  it.
- **Billing cadence** — monthly metering, monthly payout.
- **Reseller relationships** — one vendor's skills billed through another
  vendor's contract.

Treat all of the above as a design sketch. None of it is implemented, and the
numbers are illustrative. Don't build a business model around a 70/30 split that
isn't in code.

---

## Section 4 — What to do today

If monetization matters to your roadmap:

- **Publish anyway if distribution is the goal.** Free installs still build
  adoption, reviews, and install velocity — the things that'll matter when
  monetization arrives.
- **Set `pricing: none`** (or omit it — it defaults). There's no other valid
  value yet.
- **Watch the changelog.** When the metering + payout work ships, this page gets
  the real model and the `pricing` field gets real values.

---

## See also

- [../04-publishing/packaging.md](../04-publishing/packaging.md#the-rest-optional-with-defaults)
  — the `pricing` manifest field among the other optional fields.
- [analytics.md](analytics.md) — revenue is explicitly out of the planned
  analytics surface until Phase 3.
