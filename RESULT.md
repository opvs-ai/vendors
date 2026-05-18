# RESULT — opvs-ai/vendors content(V-B) onboarding

**Branch:** `content/V-B-onboarding`
**Base:** `main@c945de7`
**Session:** V-B (single-shot content fill, no sub-agents)
**Date:** 2026-05-18

## What landed

The three TODO stubs in `01-onboarding/` and the section landing README now have real content. Plus a new repo-level `CHANGELOG.md`.

```
01-onboarding/
  README.md                  10 → 23 lines    landing — intro + 3-row nav table
  become-a-vendor.md          7 → 129 lines    vendor journey + ASCII flow
  vendor-types.md             9 → 118 lines    3 types + comparison table
  apply-for-a-cert.md        12 → 189 lines    ed25519 mint + one-shot reveal
CHANGELOG.md                  -  →  11 lines    new file (Keep-a-Changelog format)
```

Total: 5 files touched, 5 commits (one per file).

## Page-by-page

### become-a-vendor.md (129 lines)

Walks the full vendor journey end-to-end. Opens with an ASCII flow diagram from "I'm interested" to "my first skill is live." Sections: who can be a vendor (with link to vendor-types.md), what OPVS needs from you as intake info, approval timeline (hours not days in Phase 1), what you get on day one (vendor scope + cert + partner key — three named credentials clearly distinguished), what your first week looks like, what's intentionally manual in Phase 1, and the escape hatches if a credential is compromised.

Lead message: cert and partner key are two different things and you need both. This was the planner's "Cert vs key → cert in onboarding, partner-key in auth" decision, and the page is written to make that distinction stick.

### vendor-types.md (118 lines)

Three types in clear named sections: internal teams (`@opvs-ai`), partner teams (`@spideriq`, `@di-atomic` — Phase 1 live), third-party developers (Phase 2). Each section answers: who, scope, cert flow, default trust tier, support model, what's expected. Closes with a side-by-side comparison table and a short trust-tier breakdown so readers don't have to bounce out to MARKETPLACE-CONCEPT.md just to understand `internal` vs `beta`.

Calls out plainly: third-party sign-up is not open today. Wait until Phase 2.

### apply-for-a-cert.md (189 lines)

The longest of the three because the cert flow has the most moving parts. Covers what the cert is for (signing at three verification points: publish, install, rolling re-check), the full mint flow with an ASCII sequence diagram (vendor → OPVS ops → CA → reveal URL), what cert fingerprints are and where they appear, where the private key actually lives and where it must not live, scheduled and incident-driven rotation, the option to hold multiple active certs (zero-downtime rotation), what's intentionally not in the Phase 1 CA (no public CRL, no OCSP, no chain, no auto-renewal), a quick reference table of the actual API endpoints, and a "common mistakes" closer.

Cross-links to `02-authentication/partner-keys.md` and `02-authentication/credential-rotation.md`.

### 01-onboarding/README.md (23 lines)

Two-paragraph intro that answers "what is this section for" and "what is it not for." Three-row nav table to the pages above. Closes with a one-paragraph pointer to 02-authentication for the partner key.

### CHANGELOG.md (11 lines)

New file. Keep-a-Changelog format. One entry under `[Unreleased] → Added` per the spawn-prompt requirement.

## Style notes

Wrote against `feedback-writing-style-plain-english.md`. Short paragraphs, whitespace between them, one idea per sentence. No dense multi-clause prose. Two ASCII diagrams (the journey in become-a-vendor.md and the mint sequence in apply-for-a-cert.md) plus one comparison table in vendor-types.md. Bullet lists used where the content is naturally a list (e.g. "what OPVS needs from you"); prose used everywhere else.

## Decisions

### Where to draw the cert vs partner-key line

The spawn prompt was explicit: cert in onboarding, partner-key in auth. I held that line and made it the load-bearing point in the apply-for-a-cert page. The page actively warns the reader against confusing the two and points to `02-authentication/partner-keys.md` for the other half. The become-a-vendor page also enumerates both as separate day-one deliverables.

### Quick reference table on the cert page

Added a small endpoint table at the end of apply-for-a-cert. The page is meant for non-engineering readers (vendor leads, ops contacts) who will not call the endpoints directly, but the table makes the doc a usable reference for engineers who eventually do.

### Did not invent monetization or third-party flows

Third-party developers section is explicit that the surface is not open in Phase 1. Did not speculate about pricing, marketplace fees, or revenue share — those are Phase 3 per `MARKETPLACE-CONCEPT.md` and out of scope for V-B.

### CHANGELOG.md was missing

`CHANGELOG.md` did not exist at the root of `opvs-ai/vendors`. Created with Keep-a-Changelog format. Single `[Unreleased] → Added` entry per the spawn-prompt requirement.

## Open follow-ups for the planner

1. **Quick reference endpoint paths** on apply-for-a-cert.md assume `/api/v1/marketplace/ca/*` shape. That matches MARKETPLACE-CONCEPT.md but the live marketplace service may have shifted some paths since the concept was written. V-H (reference content) should cross-check.
2. The become-a-vendor page tells new vendors to run `opvs-skills doctor` to verify both credentials end-to-end. That command exists in `@opvs-ai/skills@0.4.2` on Verdaccio (per MARKETPLACE-CONCEPT.md), but the exact output shape is not documented here. V-E (publishing) is the place for that.
3. The "what your first week looks like" section in become-a-vendor.md mentions seven security checks. The actual seven are spec'd in `VENDOR-GUIDE.md`. V-E should be the canonical list; this page just teases the count.
4. Third-party developer onboarding is described in the future tense everywhere. When Phase 2 opens, vendor-types.md will need an update — most of the "not open today" hedging in that section will need to come out.

## What this PR does NOT include

Per spawn-prompt hard rules:

- No touches to `02-authentication/`, `03-authoring/`, `04-publishing/`, `05-editing-and-versioning/`, `06-promotion/`, or `reference/`.
- No CI, no LICENSE, no `.github/workflows/`.
- No root-level CONTRIBUTING.md (planner decision: skip).
- No CHANGELOG entries under any heading other than `[Unreleased] → Added`.

## Verification

- `git log --oneline content/V-B-onboarding ^main` shows 5 clean commits, one per file.
- `wc -l 01-onboarding/*.md CHANGELOG.md` total 470 lines.
- Every page ends with the "Filled in by content session V-B" footer.
- All cross-links resolve to files that exist in the repo.
- Three pages range 118–189 lines (target was 200–400; rejected padding for the sake of word count — content is the right length to make its point).

## Ready for orchestrator

PR opens against `opvs-ai/vendors:main` with title:
`content(V-B): onboarding — become a vendor + types + cert apply`

No deploy step. Orchestrator merges. Planner reads the PR and writes the V-C spawn prompt for `02-authentication/`.
