# opvs-ai/vendors

> **These docs now live at [opvs.ai/docs › Vendors](https://opvs.ai/docs/guides/vendors)**, checked and updated on 8 Nov 2026. Read and link to the live pages; this repository is kept as-is for history and is no longer updated.

Vendor portal docs for the OPVS Marketplace. How to become a vendor, authenticate, author a skill (or a **profession**), publish it, version it, and promote it.

> **Two package types.** A **skill** is a tool. A **profession** is a job — it links a toolkit of skills, carries a library of workflows, and equips a hired agent with a whole role (install = equip). Both ship on the same rails. New: [03-authoring/profession-anatomy.md](03-authoring/profession-anatomy.md).

## Who this is for

You want to publish a skill on [market.opvs.ai](https://market.opvs.ai). You might be:

- An OPVS-internal team building `@opvs-ai/*` packages
- A partner team like SpiderIQ or Di-Atomic with your own scope
- A third-party developer building on top of OPVS

If you are an **agent runtime team** trying to *consume* marketplace skills — OpenClaw, Claude Code, Cursor, Windsurf, langchain, langgraph, and the rest — you want [opvs-ai/marketplace](https://github.com/opvs-ai/marketplace) instead. This repo is the supply side; that one is the demand side.

## How this repo is organized

Six numbered sections walk you end-to-end from "I'm interested" to "my skill is live."

```
01-onboarding/                Become a vendor. Pick a vendor type. Get a signing cert.
02-authentication/            Partner keys, publishing tokens, rotation.
03-authoring/                 The SKILL.md + schema.yaml contract.
04-publishing/                Package, sign, submit, security review, version.
05-editing-and-versioning/    What's mutable after publish — and what isn't.
06-promotion/                 Badges, ratings, featured placement, analytics.

reference/                    API surface, CLI commands, glossary.
```

Start at [01-onboarding/](01-onboarding/) and walk forward. Each section landing README points at the next.

## Quick links

- [01-onboarding/](01-onboarding/) — first stop for new vendors
- [02-authentication/](02-authentication/) — keys and tokens
- [03-authoring/](03-authoring/) — what a skill looks like
- [04-publishing/](04-publishing/) — how to ship
- [05-editing-and-versioning/](05-editing-and-versioning/) — life after publish
- [06-promotion/](06-promotion/) — make people find your work
- [07-harvest/](07-harvest/) — learn from real usage and improve your skill
- [reference/](reference/) — API, CLI, glossary

## Status

This repo is currently in **structure-only** state. The directory tree and section landing READMEs are in place, but the individual content pages are TODO stubs.

Content is filled in by follow-on sessions V-B through V-H, one per section. See `RESULT.md` on the `redesign/A-information-architecture` branch for the redesign log.

## Related repos

- [opvs-ai/marketplace](https://github.com/opvs-ai/marketplace) — demand side. One section per runtime (OpenClaw, Claude Code, Cursor, Windsurf, langchain, etc.) covering install + sample code + auth.
- [martinshein/opvsHUB](https://github.com/martinshein/opvsHUB) — legacy file-based skill source for pre-marketplace OpenClaw deployments. Frozen.
