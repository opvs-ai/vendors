# RESULT — opvs-ai/vendors V-C (Authentication content)

**Branch:** `content/V-C-authentication`
**PR:** https://github.com/opvs-ai/vendors/pull/2
**Task UUID:** `9125ecff-7f96-49e3-a4f8-78ce5196912d` (Agent Services board)
**Session:** redesign-V-C (single-shot content pass, no sub-agents)
**Date:** 2026-05-18

## What landed

3 page fills + 1 landing refresh + 1 new CHANGELOG, 5 commits, no other touched files.

```
02-authentication/README.md                 — landing intro + nav table refresh
02-authentication/partner-keys.md           — full content (~92 added lines)
02-authentication/publishing-tokens.md      — full content (~111 added lines)
02-authentication/credential-rotation.md    — full content (~148 added lines)
CHANGELOG.md                                — new file, [Unreleased]/Added bullet
```

Every page now opens with a one-sentence headline, uses short paragraphs separated by whitespace, and leans on ASCII tables / box diagrams instead of dense prose blocks. Cross-links resolve to the V-B onboarding/cert page and to V-E revocation + reference/cli + reference/api. The cross-link to `01-onboarding/apply-for-a-cert.md` references the V-B stub (V-B hasn't merged yet — link target is the path that will exist once V-B lands).

## Decisions

### Two-layer auth model treated as the spine of the section

Every page restates the same mental model: Verdaccio token gets your *bytes* into the npm registry; partner key gets your *manifest* into the marketplace registry. They are independent, leak independently, and rotate independently. This framing is in the landing, repeated up front on `publishing-tokens.md`, and assumed throughout `credential-rotation.md`. Vendors who skim only one page should still come away with the model.

### Per-vendor Verdaccio user pattern explained at high level, not procedural

Per the planner answers in the spawn prompt + the LEARNINGS.md W2.S.3 entry, the section covers *what* the per-vendor users are (one publisher account per scope) and *why* (credential separation buys bounded blast radius without an ACL plugin swap) — but it does not document how to provision them. Vendor doesn't need that; OPVS staff does, and that procedure lives elsewhere. Provisioning gets one line: "OPVS provisions your user account during onboarding."

### Partner-key prefix + bcrypt explained at vendor level

Per planner brief, internals are abstracted: the page says "8-character prefix, bcrypt-hashed body" and "GitHub does it this way for ghp_*, AWS does it this way for AKIA*", which is enough for a vendor to trust the design. It does not document `partner_key_service.verify_key`'s implementation. Vendors who need internals can read MKT-PARTNER-KEYS-BRIEF.md in the platform repo.

### Three rotation playbooks instead of one

`credential-rotation.md` has separate procedures for partner key, signing cert, and Verdaccio user. They share a 90-day cadence and a leak-emergency timeline at the top, then diverge because each rotation has different downtime characteristics (partner key = zero, cert = dual-trust window, Verdaccio = small downtime). One emergency-leak path at the bottom is unified across all three because the 5-minute → 15-minute response is identical.

### Cross-link discipline

Every page that mentions another concept links it inline. Page revocation vs credential rotation is called out explicitly in `credential-rotation.md` so readers don't conflate "revoke this token" with "pull this published version." That distinction matches the planner brief's hard rules.

### Redaction throughout

Every credential body in example output is `REDACTED_*` or `<your-…>`. No real prefixes, no real keys, no real fingerprints. The `gh*`/`AKIA*` references are illustrative (they're not OPVS-issued), and the `opvs_pk_a1b2c3d4_…` example is a fabricated prefix, not from any real key in the system.

## Inventory — before vs after

Before this PR (state on `redesign/A-information-architecture` after V-A IA merged):

```
02-authentication/README.md             — landing scaffold with "filled by V-C" note
02-authentication/partner-keys.md       — stub with bullet list of topics + TODO V-C
02-authentication/publishing-tokens.md  — stub with bullet list of topics + TODO V-C
02-authentication/credential-rotation.md — stub with bullet list of topics + TODO V-C
```

After: same paths, content filled in, no new files in 02-authentication/.

Total non-section change: 1 new file (`CHANGELOG.md` at repo root, since the repo did not have one).

## Open questions for planner

1. **Signing cert expiry policy.** I wrote `"No fixed expiry today. The cadence is set by OPVS as the trust tier matures — TBD per vendor."` Planner answer on this would let me harden the line in a follow-up if there's an actual policy now.
2. **Self-service partner-key minting.** Today admin-only. I noted it as roadmap in `partner-keys.md`. If planner wants to keep self-service permanently out of scope (per W2.S.3-followup discussion), happy to remove the "roadmap" wording.
3. **`docs/planning/Ops/TOKEN-HYGIENE.md` link.** `credential-rotation.md` references this OPVS-platform-internal doc. If the planner wants vendors to not be aware of internal Ops docs, I should swap the link for a vendor-facing equivalent or remove the path entirely.
4. **Verdaccio rotation second-user mitigation.** I suggested "OPVS can mint a second user account temporarily" as a downtime mitigation. If this is not actually offered, I should soften that line.

## Hard rules compliance

- ✅ Plain English throughout, short paragraphs, headlines first
- ✅ No real keys / tokens in example outputs — all redacted
- ✅ Only 02-authentication/ + CHANGELOG.md touched
- ✅ One commit per page + one for CHANGELOG (5 total)
- ✅ No `--no-verify`, no force pushes
- ✅ Single PR opened to `main`
