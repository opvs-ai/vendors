# Badges

The trust badge on your package detail page is the first signal a brand reads.
It's driven by your package's **tier** — a per-version manifest field with four
values. This page explains what each tier means, what's enforced today versus
planned, and how a tier changes.

The tier is real and shipped: it's the `TrustTier` enum in the manifest
validator
([`package_manifest.py`](https://github.com/opvs-ai/OPVS/blob/main/apps/spideragent/packages/marketplace/api/app/schemas/package_manifest.py)),
and it gates marketplace visibility today. The *promotion process* around it
(human review, telemetry-driven transitions) is partially shipped — flagged
inline below.

---

## Section 1 — The four tiers

| Tier | Visibility | What it signals |
|---|---|---|
| `internal` | hidden from the customer browse | OPVS-owned or in private testing |
| `beta` | visible + installable | published and opted-in; where every live `@opvs-ai` / `@spideriq` / `@di-atomic` package sits today |
| `community` | community-contributed (Phase 2) | auto-checks only, no human review |
| `verified` | formal OPVS vetting (Phase 2) | human-reviewed, full docs, clean security, stable telemetry |

What's enforced **today**: `internal` hides a package from the public catalog
browse; `beta` makes it visible and installable. That's the live behaviour — see
[../04-publishing/versioning.md § Tier transitions](../04-publishing/versioning.md#section-4--tier-transitions-across-versions).

`community` and `verified` are defined in the enum and reserved for the Phase 2
promotion policy. The security review already applies the verified-tier *checks*
(it warns on a missing `LEARNINGS.md` / `tests/` for verified candidates — see
[../04-publishing/security-review.md](../04-publishing/security-review.md)), but
the human-review workflow that grants the badge isn't a self-serve flow yet.

---

## Section 2 — Tier is per version

Each published version carries its own tier, declared in its manifest. They can
differ across one package's history.

```
@spideriq/spiderflows
  0.1.0  tier: internal   (private testing)
  0.1.1  tier: beta       (made visible — a NEW version, not an edit)
  0.2.0+ tier: beta
```

A package doesn't "switch" tier in place — you can't edit a published version.
Promoting from `internal` to `beta` means **publishing a new version** that
declares `beta`. There's no automatic promotion; each version inherits exactly
the tier you write in its manifest.

---

## Section 3 — Requesting a higher tier

Moving a package to `verified` is the one transition that *isn't* just a
republish — it goes through the promotion flow, not a manifest edit. In Phase 2
the expected evidence is: the verified-tier required docs present (`README.md`,
`SKILL.md`, `CHANGELOG.md`, `LEARNINGS.md`, a non-empty `tests/`), a clean
security review (score `100`), and a period of stable runtime telemetry.

Until that flow is self-serve, the practical path is: ship a package that *would*
pass verified-tier checks (clean docs, clean review, real tests), publish it as
`beta`, and coordinate with OPVS for the review. The
[security-review](../04-publishing/security-review.md) warnings tell you exactly
what verified would additionally require.

---

## Section 4 — What feeds the badge

Two inputs back the badge a brand sees:

- **Tier** — the manifest field, described above.
- **Security score** — the deterministic `0–100` from the
  [security review](../04-publishing/security-review.md#section-3--the-score),
  stored on the published version and shown on the detail page. A clean publish
  is `100`; warnings dock `5` each.

Auto-downgrade — telemetry pushing a `verified` package back to `beta` when its
runtime health degrades — is part of the Phase 2 design, not shipped today. When
it lands, you'll see the signal first via [analytics](analytics.md). For now the
tier is exactly what each version's manifest declares.

---

## Section 5 — What a brand sees per tier

- **`internal`** — not in the public browse at all. A brand only reaches it via
  a direct install (platform-curated).
- **`beta`** — listed in the catalog, installable, badge reads "Beta." This is
  the normal state for a live third-party package.
- **`verified` / `community`** — Phase 2 badges with their own copy and, for
  `community`, some entitlement gating.

The honest summary: today, your real lever is `internal` vs `beta`, set per
version in the manifest. The richer badge ladder is built into the schema and
the security checks, waiting on the Phase 2 promotion workflow.

---

## See also

- [../04-publishing/versioning.md](../04-publishing/versioning.md#section-4--tier-transitions-across-versions)
  — tier as a per-version field, with a real history example.
- [../04-publishing/security-review.md](../04-publishing/security-review.md) —
  the checks and the score that back the badge.
- [analytics.md](analytics.md) — where the telemetry for future tier transitions
  surfaces.
