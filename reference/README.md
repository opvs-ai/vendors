# Reference

Quick-lookup material. Use these pages when you already know what you're after —
an endpoint, a flag, the meaning of a term. Use the numbered sections (01–06)
when you're *learning* the flow end to end.

The difference matters: the numbered sections are **guides** — they walk you from
"I want to publish" to "my skill is live," in order, with the why. This section
is **reference** — flat, alphabetised-where-it-helps, no narrative. You don't
read it front to back; you jump to the row you need.

## Pages

- [api.md](api.md) — the HTTP API surface. Registry public reads, marketplace
  admin writes, ratings/reviews, doctor, credential invites. Method · path ·
  auth, every endpoint traceable to code.
- [cli.md](cli.md) — the `@opvs-ai/skills` CLI. Every command
  (`generate / validate / publish / install / doctor`), its flags, and the env
  vars it reads, with a typical publish session at the end.
- [glossary.md](glossary.md) — ~30 plain-English definitions (cert, partner key,
  entitlement, manifest, bundle, overlay, tier, `skill_layers`, …), each linked
  to the page that covers it in depth.

## When you'd use which

| You want to… | Go to |
|---|---|
| call an endpoint directly / wire CI | [api.md](api.md) |
| script a publish or install | [cli.md](cli.md) |
| look up what a term means | [glossary.md](glossary.md) |
| understand *how* publishing works | [../04-publishing/](../04-publishing/) (guide) |

---

Everything in this section is cross-checked against the OPVS monorepo — the
[registry / marketplace routers](https://github.com/opvs-ai/OPVS/blob/main/apps/api/app/api/v1/),
the [manifest validator](https://github.com/opvs-ai/OPVS/blob/main/apps/spideragent/packages/marketplace/api/app/schemas/package_manifest.py),
and the [CLI source](https://github.com/opvs-ai/OPVS/blob/main/packages/opvs-skills/src/cli.ts).
If a fact here drifts from the code, the code wins — open an issue.
