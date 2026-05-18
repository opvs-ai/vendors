# RESULT — opvs-ai/vendors redesign-A (Information Architecture)

**Branch:** `redesign/A-information-architecture`
**Task UUID:** `fada756d-fec5-4d34-a165-6321106ec3fa` (Agent Services board)
**Session:** redesign-A (single-shot IA pass, no sub-agents)
**Date:** 2026-05-18

## What landed

The repo started as a single `README.md` (one line: "OPVS Marketplace — vendor portal."). It now has a complete directory skeleton with seven section landings and 25 page stubs ready to be filled in by content sessions V-B through V-H.

```
README.md                            top-level audience + nav tree
01-onboarding/                       3 pages (V-B)
02-authentication/                   3 pages (V-C)
03-authoring/                        4 pages (V-D)
04-publishing/                       5 pages (V-E)
05-editing-and-versioning/           3 pages (V-F)
06-promotion/                        5 pages (V-G)
reference/                           3 pages (V-H)
```

Total: 33 markdown files, 8 commits (one per section + one for top-level), zero real content (per scope — structure-only pass).

## Decisions

### Kept the draft nav tree mostly as-is

The spawn prompt's draft tree was sound. The six numbered sections walk a vendor end-to-end (onboarding → auth → authoring → publishing → editing → promotion) with `reference/` as the quick-lookup catch-all. I didn't see a stronger cut.

### Kept `apply-for-a-cert.md` in 01-onboarding

The draft tree placed it there; I confirmed that's right rather than moving it to 02-authentication. Reason: you literally cannot publish until you have a cert, so cert minting is part of *becoming* a vendor, not part of *authenticating* a vendor. The two pages cross-link.

### Contract pointer in 03-authoring, canonical home in opvs-ai/marketplace

Followed the spawn-prompt recommendation. The on-demand contract is consumed by every runtime, so its single source of truth lives in opvs-ai/marketplace. `03-authoring/the-contract.md` is a short pointer page that summarises the four locked rules and links out.

### `06-promotion/payouts.md` flagged for planner decision

No monetization model exists today (Phase 3 in MARKETPLACE-CONCEPT.md). I included the page as a placeholder so the IA tree is ready when payouts ship, but the page header explicitly says "Placeholder — no monetization model on the platform today" and the V-G TODO note tells the content session to fill it in *only* if planner keeps the page. See Open Questions below.

## Inventory — what was in the repo

Before this PR: 1 file, 2 lines.

```
README.md   "# vendors\nOPVS Marketplace — vendor portal. How to become a vendor, ..."
```

After this PR: 33 files across 7 directories. No CI, no LICENSE, no `.github/` workflows — content-first per spawn-prompt hard rules.

## Open questions for planner

1. **Keep or strike `06-promotion/payouts.md`?** No payout model exists. Page is gated behind a clear "placeholder" header. If the planner thinks it's premature even as a stub, V-G can delete the file when it runs.
2. **Where does the canonical on-demand contract live?** I assumed `opvs-ai/marketplace`. If the planner decides it lives here in `opvs-ai/vendors` instead, `03-authoring/the-contract.md` would become a content page rather than a pointer, and the spec would move with it.
3. **Cert vs partner-key split across sections.** Cert minting is in 01-onboarding (because onboarding-blocking); partner key is in 02-authentication (because HTTP-API-auth). They cross-link, but a reader could reasonably expect both in one place. If V-B/V-C content sessions find the split awkward, the structure can absorb a merge later.
4. **Should there be a CONTRIBUTING.md at the root?** Skipped for the structure pass — adding it now would be content. V-H or a later sweep can add one if planner wants vendor-facing change-request flow documented.
5. **Visibility of this repo.** Currently private. Memory notes flag "flip to public at launch." Out of scope for this PR; just flagging that the README's external links (`market.opvs.ai`, sibling repos) assume public visibility eventually.

## What this PR does NOT include

Per spawn-prompt hard rules:

- No CI — `.github/workflows/` does not exist
- No LICENSE — Martin sets license policy
- No real content — every page is a TODO stub with a "Filled in by Session V-X" footer

## Verification

- `git log --oneline` shows 8 clean commits, one per section + top-level
- `find . -type f -name '*.md' | wc -l` returns 33
- Every TODO stub has a "Filled in by Session V-X" footer
- Top-level README plus all 7 section READMEs use plain-English style (short paragraphs, whitespace, ASCII tree for nav)

## Ready for orchestrator

PR opens against `opvs-ai/vendors:main`. No deploy step (docs-only repo). Orchestrator merges; planner reads PR and writes content fill-in spawn prompts for V-B through V-H.
