# RESULT — opvs-ai/vendors V-D (Authoring content)

**Branch:** `content/V-D-authoring`
**Board:** Agent Services (`86a304a8-f951-4403-b099-74895f7ce4e7`)
**Session:** V-D content author (single-shot, no sub-agents)
**Date:** 2026-05-18

## What landed

The 03-authoring/ section started as a landing README plus four TODO-stub pages. It now has real content on all four pages: skill anatomy with a worked example, a contract pointer page, the validation preflight walkthrough, and eight greatest-hits gotchas.

```
03-authoring/
├── README.md                     landing — reading order + nav
├── skill-anatomy.md              SKILL.md + schema.yaml shape, agentboard example
├── the-contract.md               pointer to opvs-ai/marketplace canonical spec
├── validation-preflight.md       4 preflight steps + 7 security checks + LLM completion
└── common-gotchas.md             8 entries — tool-name regex first
```

Plus a new repo-root `CHANGELOG.md` (the repo had none before).

Six commits total — one per page + one combined README+CHANGELOG.

## Decisions

### Contract page is a pointer, not the spec

Followed planner answer #2. `the-contract.md` summarises the four locked rules in one paragraph each and links out to `opvs-ai/marketplace/reference/on-demand-contract-spec.md` as the canonical home. Reasoning: every runtime (OpenClaw, Claude Code, Cursor, Windsurf, langchain) consumes the same contract, so the natural single source of truth is the demand-side repo.

If a reader needs the full schema, version compatibility rules, and example responses, they follow the link out. If they just need a quick reminder of "what are the four rules again," the summary here is enough.

### Tool-name regex is the top of common-gotchas.md

Promoted from the middle of the stub list to position #1 with the longest entry on the page. Reasoning: this is the foundational rule per the memory `feedback-openai-tool-name-regex.md`. One dotted tool name poisons the entire `tools` array on the next OpenAI call and crashes every agent that loaded the skill. The blast radius is bigger than any other authoring mistake. It deserves the prime real-estate.

Cross-referenced from skill-anatomy.md (in the `methods:` section), validation-preflight.md (as the showstopper to look for in step 1 + the reason step 4 exists), and the README's "reading order" guidance.

### "Pre-merge LLM completion" is its own preflight step, not a footnote

Validation-preflight.md elevates the end-to-end LLM check to a numbered step alongside generate / validate / dry-run, and labels it non-negotiable. Reasoning: the May 2026 production incident shipped past a "verification" that consisted of reading a gateway log line. That failure mode is exactly what the easy checks miss, so giving it the same visual weight as the easy checks makes it harder to skip.

### Eight gotchas, not seven

Stub listed seven topics. I split "config.yaml ↔ moltbot.json drift" from "docker compose restart ≠ recreate" into two separate entries because they have different symptoms even though the root cause is related. A reader hitting one of them will search for that specific symptom; keeping them merged would hide the second one.

Final list:
1. Tool-name regex `^[a-zA-Z0-9_-]+$`
2. `requires.env` blocks resolution
3. `config.yaml` ↔ `moltbot.json` drift
4. `docker compose restart` ≠ recreate
5. publishConfig hygiene
6. Stale `dist/` in `validate.yml`
7. `doctor --fix` hangs interactively
8. Env-var injection requires filesystem discovery

The contract-level gotchas (sibling vs wrapper `_guidance`, `intent` vs `query`, telemetry enum) that the stub also listed got moved to `the-contract.md` where they belong, with a one-line nod from `common-gotchas.md`. Otherwise the same rules would have lived in two places, which would have invited drift.

### CHANGELOG.md added at the repo root

The repo had no CHANGELOG. The spawn prompt asked for an entry, so I created the file. Format follows the common "Keep a Changelog" convention with `[Unreleased]` at the top. Added a single bullet for this V-D fill plus a backfilled entry for the redesign-A IA pass (PR #1) so the file isn't artificially empty above this work.

### RESULT file named RESULT-V-D.md to preserve redesign-A's record

The repo root already has a `RESULT.md` from the redesign-A session. Overwriting it would lose that record, so this session's result is `RESULT-V-D.md` instead. Future sessions should follow the same `RESULT-<session-code>.md` pattern.

## Scope I explicitly did NOT touch

Per the spawn prompt's planner answers:

- `06-promotion/payouts.md` — flagged as "not your concern"
- Cert vs key split across 01/02 — "not your concern"
- `CONTRIBUTING.md` — skipped
- Repo visibility — "no action"
- Anything outside `03-authoring/` and `CHANGELOG.md`

## What this PR does NOT include

- No CI workflows (this repo has none)
- No LICENSE changes
- No edits to other sections
- No code, only docs

## Verification

- `git log --oneline content/V-D-authoring` shows six commits in order
- All four content pages have substantive content, no remaining `<!-- TODO V-D: -->` markers, no `> Filled in by Session V-D` footers
- README's reading-order recommendation matches the actual reading flow
- Cross-links between the four pages work (anatomy → contract, anatomy → gotchas, validation → gotchas, contract → validation, all four → 04-publishing/)
- CHANGELOG.md `[Unreleased]` entry is present
- Plain-English style applied throughout — short paragraphs, whitespace, one ASCII tree, headlines first

## Ready for orchestrator

PR opens against `opvs-ai/vendors:main`. No deploy step (docs-only repo). Once merged, the planner can move on to V-B (01-onboarding), V-C (02-authentication), V-E (04-publishing), V-F (05-editing), V-G (06-promotion), and V-H (reference) in any order.
