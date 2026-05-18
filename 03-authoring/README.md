# 03. Authoring

You're authenticated. Now you actually write a skill.

A "skill" on OPVS is a small set of files that tell the platform what your skill does, what tools it exposes, and what it needs to run. The two files that matter most are `SKILL.md` (decision guidance for agents) and `schema.yaml` (the API contract that drives codegen).

This section walks through the anatomy of a skill, the on-demand contract every skill follows, how to validate locally before you submit, and the gotchas that catch first-time authors.

## Pages

- [skill-anatomy.md](skill-anatomy.md) — `SKILL.md` + `schema.yaml` shape, what each field means, a worked example pointing at the agentboard blueprint
- [the-contract.md](the-contract.md) — pointer to the canonical on-demand contract spec in `opvs-ai/marketplace`, with a short summary of the four locked rules
- [validation-preflight.md](validation-preflight.md) — running `opvs-skills generate`, `validate`, and `publish --dry-run` locally, plus the non-negotiable end-to-end LLM completion check
- [common-gotchas.md](common-gotchas.md) — eight greatest-hits mistakes: tool-name regex, `requires.env` blocking resolution, `config.yaml`/`moltbot.json` drift, and more

## Reading order

Read [skill-anatomy.md](skill-anatomy.md) first. It tells you what files to put on disk.

Then [the-contract.md](the-contract.md) for the response shape your skill has to follow if you want to plug into the on-demand discovery system.

Then [validation-preflight.md](validation-preflight.md) before you submit. The end-to-end LLM completion check at the end is non-negotiable.

[common-gotchas.md](common-gotchas.md) is reference material — check it when something breaks.

## A note on the contract

The on-demand contract is the same shape across every runtime. Because of that, its canonical home is [opvs-ai/marketplace](https://github.com/opvs-ai/marketplace) — the demand-side repo. We keep a short pointer page here and link out. If the contract here and the contract in marketplace ever disagree, marketplace wins.

## What's next

Skill files on disk and contract-shaped. Time to package and ship — head to [04-publishing/](../04-publishing/).
