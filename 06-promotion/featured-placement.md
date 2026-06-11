# Featured placement

The marketplace catalog has room for editorial slots — a "Featured" row, "New
this week," category hero placements. This page is honest about what's live
today and what's roadmap, because most of the *editorial machinery* is Phase 2.

What ships today: the public registry browse, package detail pages, and the
[marketing overlay](../05-editing-and-versioning/marketing-overlay.md) that makes
a detail page look good. What's planned: self-serve promotion, algorithmic
"similar packages" rows, and AI-agent discovery feeds. Where a feature is
planned, it's labelled.

---

## Section 1 — Editorial vs. algorithmic

Two ways a package surfaces, fed differently:

| Surface | Fed by | Status |
|---|---|---|
| Featured / "New this week" | manual editorial pick by OPVS | editorial today, no self-serve |
| Category browse | the registry's category field | shipped (public read) |
| "More from this vendor" | your vendor scope | planned |
| "Similar packages" | category + entitlement overlap | Phase 2, algorithmic |

There is **no promote-button** — you can't pay or click your way into a featured
slot today. Editorial placement is a human OPVS decision in Phase 1.

---

## Section 2 — What's live: the browse

The public registry list is the shipped discovery surface. Brands (and agents)
find packages through it, filtered by vendor, category, and tier:

```bash
# Everything published
curl -s "https://market.opvs.ai/api/v1/registry/packages"

# Filtered to a category
curl -s "https://market.opvs.ai/api/v1/registry/packages?category=board"

# Full-text search
curl -s "https://market.opvs.ai/api/v1/registry/search?q=leads"
```

Your package shows up here the moment it's published at `beta` tier (an
`internal`-tier version is hidden — see [badges.md](badges.md)). The category
you put in `package.yaml` is what files it under the right browse heading, so
pick it deliberately.

---

## Section 3 — What makes a package feature-worthy

When the editorial picks happen, the inputs are the obvious ones:

- **A complete, polished detail page** — a real hero image, screenshots, a clear
  long description. This is the lever you fully control via the
  [marketing overlay](../05-editing-and-versioning/marketing-overlay.md). A
  package with an empty overlay won't get featured.
- **A clean security score** — `100`, no warnings. See
  [../04-publishing/security-review.md](../04-publishing/security-review.md).
- **Real install velocity** — packages brands actually adopt.
- **Novelty** — a capability the catalog didn't have before.
- **Good ratings** — see [ratings-and-reviews.md](ratings-and-reviews.md).

None of these is a guarantee, but a strong overlay plus a clean review is the
single biggest thing in your control. Ship the package well and the editorial
surface has something to feature.

---

## Section 4 — Planned discovery surfaces

These are designed, not yet shipped — don't build a launch plan around them:

- **"More from this vendor"** — a carousel on every detail page in your scope.
  Once it ships you're auto-eligible; nothing to configure.
- **"Similar packages"** — algorithmic, based on category + entitlement overlap,
  so a well-categorised package with precise entitlements gets matched better.
- **AI-agent discovery feeds** — `llms.txt` and `agent.json` package entries so
  runtimes like Claude Code and Cursor can discover marketplace packages
  directly. (The OPVS public site already ships `llms.txt` / `agent.json` for the
  catalog; per-package marketplace entries are the Phase 2 extension.)

---

## Section 5 — What you can do today

Concretely, to maximise discovery on the shipped surfaces:

1. Publish at `beta` so you're in the browse at all.
2. Fill the marketing overlay completely — it's the detail page that converts.
3. Pick an accurate `category` so the right brands find you.
4. Keep the security score at `100`.
5. Earn install-gated reviews from real users.

Everything else is editorial or Phase 2. Do the five above and you've done the
part that's in your hands.

---

## See also

- [../05-editing-and-versioning/marketing-overlay.md](../05-editing-and-versioning/marketing-overlay.md)
  — the detail-page polish that drives feature-worthiness.
- [badges.md](badges.md) — tier (`internal` hides you, `beta` lists you).
- [ratings-and-reviews.md](ratings-and-reviews.md) — the social proof that feeds
  editorial picks.
