# Skill anatomy

A skill is a directory with two load-bearing files. That's it. Everything else is optional.

This page shows you the shape, walks one real example end-to-end, and tells you what each piece is for.

## The minimum viable skill

```
my-skill/
├── SKILL.md              ← decision guidance for the agent (Tier 2)
└── client/
    └── schema.yaml       ← API contract that drives codegen (Tier 3)
```

Two files. That gives you a working skill on the platform.

Authors who need to ship extra reference material can add optional sidekicks (see [Optional files](#optional-files) below). They are not required and most skills do not have them.

## A real example

The clearest way to learn the format is to read a skill that already ships in production. Use [`agentboard`](https://github.com/opvs-ai/opvs/tree/main/apps/spideragent/packages/shared/skills/agentboard) as your model.

It has 60+ methods, a guidance block, the standard `OPVS_PAT` auth, and brand-scoping. If your skill looks structurally like agentboard, you are on the right track.

For a smaller second example see [`vayapin`](https://github.com/opvs-ai/opvs/tree/main/apps/spideragent/packages/shared/skills/vayapin) — three methods, GraphQL-backed, same shape.

## SKILL.md — the agent-facing doc

This is the file the agent loads into its context when the skill is in scope. It is short on purpose. Think of it as a one-page cheat sheet, not a reference manual.

### Frontmatter

```yaml
---
name: my-skill
version: "1.0.0"
description: >
  One-sentence summary of what this skill does.
category: utility           # one of 12 platform categories
requires_auth: true
requires_brand: true
triggers:
  - my-skill
  - keyword the agent might think of
client: my-skill            # links to client/ directory
client_version: "1.0.0"
metadata:
  openclaw:
    primaryEnv: OPVS_PAT
---
```

A few field notes:

- **`name`** — kebab-case. Matches the directory name. Cannot contain dots, `@`, or `/`. Same rule as tool names (see [common-gotchas.md](common-gotchas.md)).
- **`category`** — one of: `foundation`, `admin`, `board`, `memory`, `planner`, `data-collection`, `communication`, `media`, `integration`, `utility`, `auth`, `onboarding`. Used by the dashboard and by `CONTEXT.md` generation.
- **`triggers`** — short list of words an agent might use when this skill is needed. Helps with discovery, not enforcement.
- **`primaryEnv`** — documentation only. Lists the main env var the skill reads. **Do not** add `requires.env` here unless your skill owns the env var — using it for vars provided by another skill (like `opvs-foundation`) blocks resolution. This is a real production bug and is explained in [common-gotchas.md](common-gotchas.md).

### Body

Below the frontmatter is plain markdown. The convention is:

```
# skill-name #category

one-line tagline

/#related-skill → what that other skill does (and why this one isn't it)
/#another-related → same

## Methods

- `methodName(arg1, arg2?)` — one-line description.
- `anotherMethod(arg)` — one-line description.
```

Keep method descriptions to one line each. The full type info comes from the generated plugin — you do not need to repeat parameter shapes in markdown.

## schema.yaml — the API contract

This is the file that drives codegen. From one `schema.yaml` the platform generates:

- a Tier 3 gateway plugin (typed tools the agent calls natively)
- an MCP tool set
- a TypeScript SDK module
- CLI subcommands
- an OpenAPI 3.1 document

So your `schema.yaml` is doing a lot of heavy lifting. Get it right and the rest is free.

### Top-level shape

```yaml
name: my-skill
version: "1.0.0"
description: >
  One-sentence summary (same as SKILL.md description).
category: utility
base_url: /api/v1/my-skill
auth:
  type: pat
  header: Authorization
  prefix: Bearer
  env_var: OPVS_PAT
defaults:
  format: yaml
  timeout_ms: 30000
requires_brand: true
triggers:
  - my-skill
guidance:
  use: "one line on when to use this skill"
  not:
    - "other-skill → what other-skill does instead"
  chains:
    - "methodA → methodB (poll 5s) → methodC"
methods:
  ...
```

A few field notes:

- **`base_url`** — the URL prefix the platform's gateway proxies for you. For OPVS-hosted skills this is `/api/v1/<skill>`. For external-vendor skills it is still the OPVS proxy path, not your origin (the gateway routes through a proxy so credentials stay server-side).
- **`auth.env_var`** — usually `OPVS_PAT`. The agent's Personal Access Token, scoped to the brand it is acting on behalf of.
- **`defaults.format`** — `yaml` saves 40-76% tokens vs JSON on agent-facing responses. Use it unless you have a strong reason not to.
- **`guidance`** — short prose that the generator inlines into `SKILL.md`. Tells the agent *when* to use the skill, *when not to*, and what the typical method chain looks like.

### methods: block

Each method declares its HTTP shape, parameters, and per-method guidance.

```yaml
methods:
  doSomething:
    description: >
      One sentence describing what this method does.
    guidance:
      use: "when the agent has X and needs Y"
      next: "doSomethingElse (to follow up)"
      warn: "max_results >50 takes 5-10min"
    http: POST /things
    params:
      thing_id:
        type: uuid
        required: true
        description: The thing to operate on
      filter:
        type: enum
        values: [open, closed, all]
        required: false
        description: Optional filter
    response:
      format: yaml
```

Method names follow the same regex as skill names — `^[a-zA-Z0-9_-]+$`. No dots. No `@`. This matters: OpenAI's chat-completions API rejects any tool name with a dot in it, which would crash every agent that loads your skill on the next LLM call. Foundational rule, see [common-gotchas.md](common-gotchas.md).

The `guidance.next` and `guidance.warn` strings get appended to the tool's description when the agent sees it. That is your chance to nudge the agent toward the right call sequence without writing a giant SKILL.md.

## Optional files

You do not need any of these to ship. Add them only if you have a reason.

| File | What it is for |
|------|----------------|
| `CHANGELOG.md` | Human-readable history of skill versions. Picked up by the marketplace UI. |
| `LEARNINGS.md` | Internal gotchas you want future maintainers to see. Not shown to agents. |
| `client/SKILL_HELPERS.md` | Extra reference material a generator pass might use. Rarely needed. |
| `tests/` | Unit tests for any custom logic. Not required for codegen-only skills. |

## What gets generated

When you run `opvs-skills generate my-skill`, the generator reads `client/schema.yaml` and emits artefacts into multiple output trees:

```
my-skill/client/schema.yaml
        │
        ├── → gateway plugin     (api.registerTool() bundle)
        ├── → MCP tool definitions
        ├── → TypeScript SDK module
        ├── → CLI subcommand tree
        └── → OpenAPI 3.1 document (.yaml + .json)
```

You do not commit the generated output to your skill directory. Generation happens at publish time, and the platform routes each artefact to the right consumer.

## What's next

Once your skill files are on disk, the next step is the locked response contract every skill follows. That is [the-contract.md](the-contract.md).

Then validate locally before you submit — see [validation-preflight.md](validation-preflight.md).
