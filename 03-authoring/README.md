# 03. Authoring

You're authenticated. Now you actually write a skill.

A "skill" on OPVS is a small set of files that tell the platform what your skill does, what tools it exposes, and what it needs to run. The two files that matter most are `SKILL.md` (decision guidance for agents) and `schema.yaml` (the API contract that drives codegen).

This section walks through the anatomy of a skill, the on-demand contract every skill follows, how to validate locally before you submit, and the gotchas that catch first-time authors.

## Pages

- [skill-anatomy.md](skill-anatomy.md) — `SKILL.md` + `schema.yaml` shape, what each field means
- [the-contract.md](the-contract.md) — pointer to the canonical on-demand contract spec
- [validation-preflight.md](validation-preflight.md) — running `opvs-skills generate` and `opvs-skills validate` locally
- [common-gotchas.md](common-gotchas.md) — tool-name regex, dotted names, sibling vs wrapper guidance, telemetry enum

## A note on the contract

The on-demand contract is the same shape across every runtime. Because of that, its canonical home is [opvs-ai/marketplace](https://github.com/opvs-ai/marketplace) — the demand-side repo. We keep a short pointer page here and link out. If the contract here and the contract in marketplace ever disagree, marketplace wins.

## What's next

Skill files on disk. Time to package and ship — head to [04-publishing/](../04-publishing/).

---

> Pages in this section are filled in by content session **V-D**.
