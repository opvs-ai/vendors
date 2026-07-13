# Profession anatomy

A **profession** is the marketplace's second package type. Where a *skill* is a single tool — static, you install a copy — a **profession is a job**: it **links** a toolkit of skills, carries a **library** of declarative workflows, and once installed onto an already-hired agent (**install = equip**, exactly like a skill) it gives that agent a role with method and judgment. Every agent running the same profession is a member of its **guild**: a shared address where anonymized craft compounds over time.

You author, sign, publish, version, and revoke a profession with the **same CLI and the same security review** as a skill. Only the manifest shape differs, and one extra coherence check runs. If you can publish a skill ([skill-anatomy.md](skill-anatomy.md)), you can publish a profession.

## Skill or profession — which am I publishing?

| | **Skill** | **Profession** |
|---|---|---|
| Mental model | a tool | a guild / a job |
| Manifest | `type` omitted (defaults to `skill`) | `type: profession` |
| Runtime surface | its own `skills[]` → typed tools | `links` (a toolkit it references) + `workflow_refs` (procedures it runs) |
| Owns tools? | yes — `skills/<id>/schema.yaml` | no — it **links** skills by id + range; they stay independently owned and versioned |
| `provider_config` | required (it calls an API) | **optional** — its linked skills carry their own |
| Install | adds a tool | **equips a hired agent** with the whole role |

Wrapping one API surface → publish a **skill**. Packaging a *role* (a toolkit + proven workflows + judgment) → publish a **profession**.

## The manifest — `type: profession` + the `profession` block

A profession manifest is a normal `package.yaml` (see [../04-publishing/packaging.md](../04-publishing/packaging.md)) with two additions: the `type` discriminator and a `profession` block.

```yaml
name: "@yourvendor/email-outreach"
version: "1.0.0"
description: "Email Marketing Specialist — outbound + retention email as a role."
vendor: yourvendor
category: outreach
license: UNLICENSED
type: profession                      # the discriminator (omit → defaults to `skill`)

# provider_config is OPTIONAL for a profession — its linked skills carry theirs.

profession:                           # required iff type: profession
  address: email-outreach@profession.md      # <slug>@profession.md — the guild address
  role_policy:                        # the ONLY required sub-field
    title: Email Marketing Specialist
    default_trust_state: probationary
    checklists:                       # definition-of-done gates the role must satisfy
      - client-intake complete before sequencing
      - deliverability checks pass before the first send
    escalation: "@ops::your-domain"   # where the role escalates
  negative_scope:                     # the rules-of-engagement floor — what the role must NOT do
    - sourcing leads (this profession sends; it does not find them)
    - starting a campaign on an unauthenticated domain
  capabilities:                       # locked = always installed; optional = installable extras
    locked:   [agentboard, mail-skills]
    optional: [copy-engine, voice-builder]
  workflow_refs:                      # the workflow library (path-shaped → files in workflows/)
    - workflows/client-intake.yaml
    - workflows/cold-sequence.yaml
  links:                              # the toolkit — reference skills by id + range (NOT bundled)
    - { skill: "@opvs-ai/agentboard", range: "^1" }
    - { skill: "@opvs-ai/agentmemory", range: "^1" }
    - { skill: "@yourvendor/smartlead", range: "^1.8" }

# Mirror your `profession.links` here so the toolkit auto-installs on equip:
peer_dependencies:
  - { name: "@opvs-ai/agentboard", range: "^1" }
  - { name: "@opvs-ai/agentmemory", range: "^1" }
  - { name: "@yourvendor/smartlead", range: "^1.8" }
```

**`role_policy` is the only required sub-field.** `links`, `workflow_refs`, `negative_scope`, `address`, and `capabilities` all default to empty — but a profession that links nothing *and* carries no workflow has no substance and fails the coherence check (below).

## Links, not bundles

A profession **references** skills by `{skill, range}`; it never copies their content. This is the core rule:

- The linked skill stays **independently owned and versioned** by its vendor. When that vendor ships a new version, every profession that links it sharpens — you don't republish.
- Many professions can link one skill.
- Mirror `profession.links` into top-level `peer_dependencies` — that's the rail that actually installs the toolkit onto the agent when a brand equips your profession.
- Linked skills can be yours **or** another vendor's, as long as they're published on the registry.

## Workflows — a library, not one procedure

`workflow_refs` point at declarative workflow files in your `workflows/` directory. A profession carries a **library** (cold-sequence, warm-up, reply-triage, re-engage, …), not a single flow. Workflows are **declarative data, never code** — authored YAML that compiles into a task DAG on the platform; the agent runs it as a stateless per-heartbeat worker, never as the orchestrator. Each path-shaped `workflow_refs` entry (`workflows/x.yaml`) must exist in the bundle and parse as YAML, or the publish is rejected.

## `role_policy` + `negative_scope` — the judgment layer

- `role_policy.checklists` — definition-of-done gates the role must satisfy.
- `role_policy.escalation` — an address the role escalates to.
- `negative_scope` — the rules-of-engagement floor: an explicit list of what the role must NOT do. It's monotonic — a brand can tighten it, never loosen it.

## The address — `<slug>@profession.md`

`profession.address` is the guild's network identity, e.g. `email-outreach@profession.md`. The local part is your profession slug; the domain is the fixed `profession.md` namespace. It must match `^[a-z0-9][a-z0-9\-]*@profession\.md$`. This is where members pool anonymized craft. You may publish before you use the address.

## `provider_config` is optional

Unlike a skill, a profession has no direct upstream of its own — its linked skills each carry their own `provider_config`. Omit it.

## The coherence check — what gets rejected

On top of the standard security review ([../04-publishing/security-review.md](../04-publishing/security-review.md)), a `type: profession` package runs one extra check. Blocking findings:

| Finding code | Meaning | Fix |
|---|---|---|
| `profession.substanceless` | `links` and `workflow_refs` are both empty | link at least one skill or add a workflow |
| `profession.dangling_link` | a `links[].skill` isn't a valid `@vendor/slug`, or doesn't resolve to a published package | publish the linked skill first, or fix the name |
| `profession.workflow_missing` | a path-shaped `workflow_refs` entry has no matching file in the bundle | add the file, or drop the ref |
| `profession.workflow_unparseable` | a workflow file isn't valid YAML | fix the YAML |
| `profession.md_missing` | `type: profession` but no `profession.md` in the bundle | include your `profession.md` source |

`dangling_link` is the common one: links resolve against the live registry, so **publish the linked skills first**, then the profession.

## Entitlements

A profession itself usually declares no entitlements — its capabilities come from the skills it links, each of which declares its own (see [../reference/glossary.md](../reference/glossary.md) and the entitlement pages). The guild-hall skill agents use to read and contribute craft (`@opvs-ai/profession`) uses the platform `profession.craft.read` / `profession.craft.write` / `profession.workflow.write` codes; you do not need those to author your own profession.

## What's next

Manifest shaped and files on disk? Head to [04-publishing/](../04-publishing/) to package, sign, and submit — the flow is identical to a skill.
