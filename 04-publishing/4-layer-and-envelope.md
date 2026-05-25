# 4-Layer Format + Envelope Contract — Publish Guide

How to publish a skill that ships **both** the on-demand envelope contract
(declared in `client/schema.yaml`) **and** the 4-layer format (skill files on
disk). Together they make a skill agents can route to, drill into, and learn
from — without you writing custom runtime code per skill.

> If you only ship one of the two, your skill is half-built. Read on.

---

## TL;DR

- The 4-layer format puts your skill content into four directories: `SKILL.md`
  (the router), `references/`, `learnings/`, `registry/`. Layer 1 is required;
  the rest are recommended.
- The envelope contract is a `guidance:` block you add to `client/schema.yaml`.
  The gateway proxy reads it and decorates every response with `_guidance`,
  `telemetry_signal`, and (with the right header) `_findings`.
- Both ship together in **one signed marketplace package**. Versioning is per
  skill (frontmatter) and per registry item (its own `version` field).
- Three decisions are locked: declare `intent_aliases` explicitly,
  `_findings` defaults to `[{audit:"ok"}]` and fires when
  `X-OPVS-Audit-Level: all`, and `telemetry_signal_default` defaults to
  `"absent"`.
- Worked example: the in-house `agentboard` skill is the lighthouse. Browse it
  in the public OPVS repo and copy its shape.

---

## Why both initiatives matter

Two problems, one package.

The **envelope contract** answers "what should the agent do next, and is this
call worth caching?" When your skill responds to a method, the gateway proxy
synthesises a `_guidance` sibling next to the payload, plus a
`telemetry_signal` (hot/warm/cold/absent) and — under audit mode — a
`_findings` block. The agent reads those siblings cheaply on every call. No
extra round-trips, no skill-side code to write the envelope.

The **4-layer format** answers "where do I drill in when I need more than the
router?" Agents start by reading `SKILL.md` — a small router pointing at
procedures. When the user's intent matches a row, the agent loads the matching
`references/` file. When something looks gnarly, it scans `learnings/`
summaries for a relevant gotcha. When the user wants to install a board
template or starter doc, it pulls the right `registry/` item.

Envelopes give agents the right guidance at the right token cost. Layers give
them deep procedures + institutional memory + reusable assets when they need
to drill in.

Ship both and the agent has a clean decision flow:

```
            ┌──────────────────────────────────────────────────────────┐
            │  Agent calls your skill's method                          │
            └──────────────────────────────────────────────────────────┘
                                       │
                                       ▼
        ┌────────────────────────────────────────────────────────────┐
        │  Gateway proxy decorates response with _guidance +           │
        │  telemetry_signal (+ _findings if audit=all).               │
        │  ◄── from `guidance:` block in your schema.yaml             │
        └────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
            ┌──────────────────────────────────────────────────────────┐
            │  Agent reads _guidance.next — decides next step           │
            └──────────────────────────────────────────────────────────┘
                                       │
                          ┌────────────┴────────────┐
                          ▼                         ▼
            ┌──────────────────────────┐  ┌──────────────────────────┐
            │  Drill-in needed?         │  │  Or call the next method  │
            │  → references/<recipe>.md │  │  → repeat                 │
            │  → learnings/<ki>/...     │  └──────────────────────────┘
            │  → registry/<item>/...    │
            │  ◄── from your 4 layers   │
            └──────────────────────────┘
```

A skill that ships only one side either has no drill-in surface (envelope-
only — agents bounce off when they need a procedure) or no runtime guidance
(layers-only — agents have to read your whole `SKILL.md` before every call to
figure out what's next). You want both.

---

## The 4-layer format

The canonical spec is
[`SKILLS-4-LAYER-FORMAT.md`](https://github.com/opvs-ai/OPVS/blob/main/docs/planning/opvsHUB/architecture/SKILLS-4-LAYER-FORMAT.md)
in the OPVS repo. Read it whole before you publish. The summary below is the
shortest version that still tells you what to put where.

### Directory layout

```
skills/<your-skill>/
├── SKILL.md                              ← Layer 1 — router (required)
├── client/
│   └── schema.yaml                       ← API surface + envelope contract block
├── references/                           ← Layer 2 — procedures (recommended)
│   ├── <task-name>.md
│   └── ...
├── learnings/                            ← Layer 3 — institutional memory (recommended)
│   ├── <yyyy-mm-dd-slug>/
│   │   ├── metadata.json
│   │   └── artifacts/what-we-learned.md
│   └── ...
├── registry/                             ← Layer 4 — typed installable assets (recommended)
│   ├── registry.json
│   └── <item-name>/
│       ├── registry-item.json
│       └── <asset-files>
└── scripts/                              ← Optional helper layer
    └── verify-<thing>.mjs
```

### Layer 1 — `SKILL.md` (the router)

A short, ≤500-line markdown file with frontmatter. Body is a decision tree
pointing at `references/` files, not a manual. The `description:` line is the
runtime trigger — list every verb/noun pattern an agent might encounter, then
add delegation pointers to sister skills.

What goes here:

- Frontmatter — `name`, `description`, `version`, optional `category`,
  `triggers`, `requires_auth`, `vendor`, `entitlements`.
- One `<HARD-GATE>` block, max. Reserve it for the single highest-risk failure
  mode. Sprinkling them defeats them.
- A decision tree table: "if the user wants X → read `references/X.md`."
- A short anti-patterns list. The long list lives in
  `references/anti-patterns.md`.
- Pointers to `learnings/`, `registry/`, `scripts/`, upstream service docs.

Size budget: ≤500 lines. Hyperframes' production reference hits 491. Treat it
as a real ceiling.

### Layer 2 — `references/` (procedures)

One markdown file per task or per anti-pattern. Two shapes:

- **WRONG/RIGHT** when you're teaching a trap. Show the broken code, then the
  fix. Real values, no `foo`/`bar`.
- **Steps / Gotchas / Verify** when you're teaching a multi-step workflow
  (publish a campaign, enrich a lead, set up a board).

Size budget: 50–200 lines per file. Anything longer should split.

Plain English. Imperative voice ("MUST", "always", "never"). Every "MUST" is
followed by **why** — so an agent can judge edge cases instead of blindly
following.

### Layer 3 — `learnings/` (institutional memory)

One folder per atomic learning. Each folder has:

- `metadata.json` — title, summary, `created_at`, `updated_at`, `references[]`
  provenance, optional `tags[]` and `stale_after`.
- `artifacts/what-we-learned.md` — the actual content. Plus any supporting
  files (diagrams, code samples, transcripts).

The summary is the trigger. Agents read every learning's summary at session
start and load the artifacts only when a summary matches the current task.

Provenance via `references[]` is required for marketplace-published learnings.
Link back to the PR, incident report, customer ticket, or memory file that
produced the learning. The agent should be able to verify the learning is
still valid by following the reference.

Tell readers (in your `SKILL.md`) that learnings are **starting points, not
ground truth** — verify against current code before trusting.

### Layer 4 — `registry/` (typed installable assets)

The agent installs these instead of writing them. Board templates, doc
starters, prompt packs, HTML components. Each item lives in its own folder:

- `registry-item.json` — validates against
  [`registry-item.schema.json`](https://github.com/opvs-ai/OPVS/blob/main/docs/planning/opvsHUB/architecture/registry-item.schema.json).
- One or more asset files referenced by the manifest.

Type vocabulary: `opvs:block` (drop-in), `opvs:component` (parameterised),
`opvs:template` (multi-file starting point), `opvs:blueprint` (multi-skill
workspace setup).

Items version themselves with their own semver — independent of the parent
skill's version. A fix to one template doesn't bump the whole skill.

Required fields on `registry-item.json`: `$schema`, `name`, `type`, `title`,
`description`, `version`, `vendor`, `skill`, `files`. `preview` is required
for items with a visible surface. `references[]` provenance is required for
marketplace-published items.

### `scripts/` — the optional fifth folder

Deterministic verification scripts the agent runs to check its own work. A
20-line script saves ~500 tokens of context per use. Not a layer — a helper.
Document each script in `SKILL.md` so the agent knows when to run it.

---

## The envelope contract block

Add this to your skill's `client/schema.yaml`. It lives next to the existing
method definitions and tells the gateway proxy what to put in the envelope
when responses come back.

### Skill-level guidance

```yaml
# skills/<your-skill>/client/schema.yaml

name: your-skill
version: "1.0.0"
description: >
  One line about what the skill does.
base_url: /api/v1/your-skill
auth:
  type: pat
  header: Authorization
  prefix: Bearer
  env_var: OPVS_PAT

guidance:
  use: "what this skill is for, one line"
  not:
    - "<sibling-skill> → what to do instead in that case"
  intent_aliases:
    - "query"
    - "search"
    - "<any-textual-search-param-name>"
  pitfalls:
    - "<top-1 gotcha — one sentence>"
    - "<top-2 gotcha>"
```

### Per-method guidance

```yaml
methods:
  yourMethod:
    description: >
      Plain-English summary of the method.
    guidance:
      use: "when an agent should call this"
      next: "<methodName>, <methodName>"
      warn: "<the one thing that bites first-timers>"
      telemetry_signal_default: "warm"
    http: GET /things/{id}
    params:
      id:
        type: uuid
        required: true
        description: The thing's ID.
    response:
      format: yaml
```

### What the gateway does with it

When a method returns, the gateway proxy synthesises three siblings on the
response payload:

- **`_guidance`** — copied from your method's `guidance:` block. Agent reads
  this on every call to decide what's next.
- **`telemetry_signal`** — one of `hot`, `warm`, `cold`, `absent`. Picked from
  `telemetry_signal_default` unless the runtime overrides.
- **`_findings`** — appears only when the caller sets `X-OPVS-Audit-Level: all`.
  Default is `[{audit:"ok"}]`. Override per-skill by setting
  `audit_findings_template` under top-level `guidance:`.

You don't write any code to emit envelopes. The proxy does it from your
schema.

### Locked decisions (don't re-litigate)

These three were decided up-front and apply to every vendor. Treat them as
contract, not opinion.

| ID | Decision | What it means for you |
|---|---|---|
| **Q1** | `intent_aliases` is declared explicitly per skill | If any method on your skill takes a textual search/query/intent parameter, list every alias under top-level `guidance.intent_aliases`. The runtime never guesses. Omit only if NO method takes a textual search input. |
| **Q3** | `_findings` default = `[{audit:"ok"}]`, fires from the proxy when `X-OPVS-Audit-Level: all` is set | You do not need to emit `_findings` yourself. The proxy injects the default. Override with a custom `audit_findings_template` only if your skill has genuine audit semantics. |
| **Q4** | `telemetry_signal_default = "absent"` if unset | Pick a richer value (`cold` / `warm` / `hot`) only when the method has natural usage-signal semantics. The generator warns you if a method is missing this field. |

Reference: `§3 Locked decisions` in the OPVS planner threads
[`skills-envelopes-injection-opvs-thread.md`](https://github.com/opvs-ai/OPVS/blob/main/.claude/plans/skills-envelopes-injection-opvs-thread.md)
and [`skills-4-layer-rollout-opvs-thread.md`](https://github.com/opvs-ai/OPVS/blob/main/.claude/plans/skills-4-layer-rollout-opvs-thread.md).

---

## Worked example — `agentboard`

The in-house `agentboard` skill is the lighthouse. Both initiatives shipped on
it first, in production, before any external vendor was asked to copy the
shape.

Browse it live in the OPVS repo:
[`apps/spideragent/packages/shared/skills/agentboard/`](https://github.com/opvs-ai/OPVS/tree/main/apps/spideragent/packages/shared/skills/agentboard).

### Layer 1 — `SKILL.md` router (excerpt)

```markdown
---
name: agentboard
version: "3.4.0"
description: >
  Task management with AgentBoard — get/assign/progress tasks, move them
  across columns, run multi-stage handoffs, post threaded comments, manage
  boards and columns, attach files. Use whenever the user wants to track
  work, assign tasks, organize a kanban, build a CRM pipeline, or move a
  card. For document/page management see agentdocs; for enrichment-specific
  board flows see lead-enrichment.
category: board
requires_auth: true
requires_brand: true
triggers: [task, board, kanban, assign, column, stage, comment, heartbeat]
---

# agentboard

Task management for AI agents and humans. Boards, columns, tasks, stages,
comments, files, metrics.

<HARD-GATE>
**Status is bound to the column, not to your `updateTask` call.**
... (one block, the highest-risk failure mode)
</HARD-GATE>

## Decision tree → references

| The user wants to… | Read |
|---|---|
| Create a single task on a board | `references/create-task.md` |
| Create many tasks at once | `references/bulk-create-tasks.md` |
| Move a task between columns | `references/move-task-between-columns.md` |
| Find an existing task before creating | `references/find-before-creating.md` |
| Hand off work between agents | `references/multi-stage-handoff.md` |
| Have a structured conversation on a task | `references/comment-threading.md` |
```

Whole file: 200 lines. Sits well under the 500-line ceiling.

### Layer 2 — one `references/` file

`references/create-task.md` is a **Steps / Gotchas / Verify** recipe:

```markdown
# Recipe: create a task on a board

**Goal:** create a single, well-formed task on the right board, in the
right column, with the right context for whoever picks it up.

**Use when:** the user asks to create / open / file / add a task, ticket,
card, item, work-item, todo, or work request.

**Prerequisites:**
- You know the `board_id`. If not — `listBoards()` first, then pick.
- You know the `column_id`. If not — `listColumns(board_id)` and pick.

## Steps
1. `searchTasks(query="<task title>", board_id=<id>)` — confirm no duplicate.
2. `listColumns(board_id)` — find the column with `mapped_status="pending"`.
3. `createTask` with title, board_id, column_id, description, priority.
4. For agent tasks, also pass `agent_instructions` (STEP 0 = worktree setup).

## Gotchas
| Gotcha | What happens | Fix |
|---|---|---|
| Wrong column_id for the board | Task stuck in frozen column | Always derive from listColumns |
| priority not in enum | 422 | Use low/medium/high/critical |

## Verify
- `getTask(task_id)` — confirm column.mapped_status is what you expected.

## See also
- `move-task-between-columns.md` — column binding rules
- `find-before-creating.md` — duplicate prevention
```

### Layer 3 — one `learnings/` KI

`learnings/2026-04-22-ab-users-table-rename/metadata.json`:

```json
{
  "$schema": "https://market.opvs.ai/schemas/learning-item.json",
  "title": "AgentBoard's users table is named ab_users — collides with core OPVS users",
  "summary": "Both AgentBoard and core OPVS (Better Auth) live in the same `starter` PostgreSQL database. To avoid a table-name collision, AgentBoard's user table is renamed to `ab_users`. Any code, SQL, or query that says `users` is wrong in AgentBoard context.",
  "created_at": "2026-04-22T00:00:00Z",
  "updated_at": "2026-05-24T13:54:00Z",
  "references": [
    "file:///root/OPVS/docs/planning/AgentBoard/LEARNINGS.md#5-ab_users-table-rename",
    "file:///root/OPVS/apps/spideragent/packages/agentboard/api/app/models/user.py"
  ],
  "tags": ["schema", "postgres", "naming-collision", "agentboard-specific"]
}
```

The matching `artifacts/what-we-learned.md` explains the problem, the fix,
what it means for sister skills, and the pattern to apply elsewhere. Short.
Plain English. Provenance points at the PR + the model file. If the learning
goes stale, you can verify against current code by following the references.

### Layer 4 — one `registry/` item

`registry/sales-pipeline-deals/registry-item.json`:

```json
{
  "$schema": "https://market.opvs.ai/schemas/registry-item.json",
  "name": "sales-pipeline-deals",
  "type": "opvs:template",
  "title": "Sales Pipeline (Deals)",
  "description": "6-column deal-stage progression. Discovery → Proposal → Negotiation → Contract → Closed Won (terminal) with Closed Lost as the failed terminal. Deals board type — pairs with a Contacts board for CRM workflows.",
  "version": "1.0.0",
  "vendor": "opvs-ai",
  "skill": "agentboard",
  "tags": ["deals", "sales", "crm", "pipeline"],
  "files": [
    {
      "path": "board.json",
      "target": "boards/sales-pipeline-deals.json",
      "type": "opvs:board-definition"
    }
  ],
  "references": [
    "file:///root/OPVS/apps/spideragent/packages/agentboard/api/app/services/board_type_schemas.py"
  ],
  "compatibility": { "min_skill_version": "3.0.0" }
}
```

Sibling `board.json` carries the actual column + field schema. Five templates
ship in the agentboard registry: `planner-orchestration-v2`, `bug-fix-pool`,
`factory-stages`, `company-enrichment`, `sales-pipeline-deals`.

### The envelope contract block in `client/schema.yaml`

```yaml
name: agentboard
version: "3.4.0"
base_url: /api/v1/board

guidance:
  use: "task management — boards, columns, tasks, stages, links, members, files, metrics"
  not:
    - "agentdocs → document/page management"
    - "lead-enrichment → enrichment-specific board workflows"
  intent_aliases:
    - "query"
    - "search"
  pitfalls:
    - "column.mapped_status is the binding column→status contract — read column config before moving tasks"
    - "comments support threading via parent_comment_id — use QUESTION:/ANSWER:/DECISION: prefixes"

methods:
  getSession:
    description: >
      Get full board context + assigned tasks in one call.
    guidance:
      use: "always call first — returns board context + assigned tasks"
      warn: "required before any other board operation"
      telemetry_signal_default: "hot"
    http: GET /views/boards/{board_id}/session
    params:
      board_id: { type: uuid, required: true }

  createTask:
    description: >
      Create a new task on a board.
    guidance:
      next: "moveTask to assign to column"
      telemetry_signal_default: "cold"
    http: POST /boards/{board_id}/tasks
```

`getSession` is `hot` (an agent that just started always needs it).
`createTask` is `cold` (used once per task, no repeat signal). The proxy reads
those values straight off the schema and tags every response with
`telemetry_signal: hot|cold`.

### Compare to Hyperframes (third-party reference)

HeyGen's [Hyperframes](https://github.com/heygen-com/hyperframes) is the
public production reference the OPVS spec is derived from. It uses the same
shape — `SKILL.md` router (491 lines), `references/` with WRONG/RIGHT pairs,
`registry/` of typed assets with per-item `registry-item.json`, helper
`scripts/`. Their domain is video templates; the structure is the same.

Two practical differences for OPVS publishers:

- Hyperframes has no `learnings/` directory yet — that pattern comes from
  Google Antigravity's Knowledge Items spec. OPVS adopts both.
- Hyperframes doesn't have an envelope contract — they don't run inside a
  guidance-injecting proxy. OPVS does, so you also fill the `guidance:` block
  in `schema.yaml`.

Read their `skills/hyperframes/SKILL.md` for a real, large router. Browse
their `skills/hyperframes/registry/` for the typed-asset pattern at scale.

---

## Validate before you publish

Run these locally before `opvs-skills publish`. Each catches one class of
mistake the registry's security review would also catch — better to find them
on your laptop.

### 1. `registry-item.json` schema validation

```bash
SCHEMA="https://raw.githubusercontent.com/opvs-ai/OPVS/main/docs/planning/opvsHUB/architecture/registry-item.schema.json"

# Validate every item in your registry
for item in skills/<your-skill>/registry/*/; do
  npx ajv validate -s "$SCHEMA" \
    -d "$item/registry-item.json" --spec=draft7
done
```

A failure exits non-zero with the offending field. Fix the item before
shipping.

### 2. `SKILL.md` size budget

```bash
wc -l skills/*/SKILL.md
# Every line must read ≤500
```

If a router crept over 500 lines, refactor: move long sections into a new
`references/<topic>.md` and replace them with a decision-tree row.

### 3. Envelope contract present

```bash
grep -E "intent_aliases:|guidance:|telemetry_signal_default:" \
  skills/<your-skill>/client/schema.yaml
```

You should see at least one `intent_aliases:` at the skill level (if any
method takes textual search input), one top-level `guidance:`, and one
`telemetry_signal_default:` per method. Methods missing
`telemetry_signal_default` get the `"absent"` default per Q4, but the
generator will warn.

### 4. `learnings/` metadata sanity

```bash
for m in skills/<your-skill>/learnings/*/metadata.json; do
  jq -e '.title and .summary and (.references | length > 0)' "$m" \
    > /dev/null || { echo "FAIL: $m missing required fields"; exit 1; }
done
```

`references[]` provenance is required for marketplace-published learnings —
the schema enforces it on receive.

### 5. Frontmatter sanity in `SKILL.md`

```bash
python3 -c "
import yaml, sys
with open('skills/<your-skill>/SKILL.md') as f:
    fm = yaml.safe_load(f.read().split('---')[1])
required = ['name', 'description']
missing = [k for k in required if k not in fm]
if missing:
    print('FAIL: missing frontmatter:', missing); sys.exit(1)
print('OK')
"
```

`description:` is the runtime trigger. The registry rejects packages with a
description shorter than a usable trigger sentence.

---

## Package metadata

**Nothing changes in your `package.yaml`.** The existing publish flow already
recursively tarballs your `skills/<name>/` tree, so adding `references/`,
`learnings/`, `registry/`, and `scripts/` happens automatically. The signing
manifest is content-addressed across all files, so new directories are
included in the hash and signature without you naming them.

Two soft conventions worth following:

- Bump the skill's `version:` in `SKILL.md` frontmatter by **minor** when you
  add the 4-layer directories to an existing skill. Additive change, no
  breaking interface. Bump **major** only if your `description:` trigger
  fundamentally changes.
- Bump each registry item's `version:` independently when you change a file
  inside that item. Don't piggy-back item bumps on a skill bump.

If your existing `package.yaml` declares an explicit file allowlist (some
vendors do, to keep tarballs small), add the new directories to the list:

```yaml
# package.yaml — only if you declare files explicitly
files:
  - "skills/**/SKILL.md"
  - "skills/**/client/schema.yaml"
  - "skills/**/references/**"      # ← add for Layer 2
  - "skills/**/learnings/**"       # ← add for Layer 3
  - "skills/**/registry/**"        # ← add for Layer 4
  - "skills/**/scripts/**"         # ← add for the optional fifth folder
  - "README.md"
  - "CHANGELOG.md"
```

Most vendors don't need this — the default tarball glob already picks
everything up.

---

## What goes wrong (top 3 gotchas)

These are the failure modes the OPVS team hit four times in a row while
shipping the in-house lighthouses. They're easy to avoid once you know about
them.

### 1. Regenerating the typed plugin (skip-the-step trap)

For OPVS-internal skills, `schema.yaml` edits are paired with a generated
TypeScript plugin under `apps/spideragent/packages/shared/src/plugins/skills/`.
If you change your `schema.yaml` and forget to regenerate the plugin, the
gateway loads stale tool descriptions. CI's "Generate & Verify" step catches
the drift, but on every wave we shipped, at least one PR was opened with the
regen step skipped.

For **external vendors**, this trap is structurally different — your skill
ships as a signed marketplace package; there is no in-tree plugin to
regenerate. But if your vendor build pipeline also produces a typed client
(an MCP server, an SDK, a CLI), the same rule applies: re-run the build after
every `schema.yaml` change.

**Mitigation:** wire a pre-publish hook that fails if `git diff --stat` shows
the generated artefact would change. Don't trust manual discipline.

### 2. Worktree-setup-as-STEP-0 (workspace trap)

Not a publishing trap exactly, but the most-cited one in OPVS internal
post-mortems. Whenever an agent task spawns a worker, **STEP 0 of the
worker's instructions must be `./scripts/wt.sh <slug> <branch>` + `cd
../OPVS-<slug>`**. If anything else comes first ("read these 10 files",
"claim the task"), the worker reads top-down and ends up coding in `/root/OPVS`
on `main`, which silently corrupts whatever else that directory is doing.

If your skill exposes a method that spawns workers, document the convention
in `references/<spawn-pattern>.md` and call it out in `SKILL.md`. The agent
gets to learn it once instead of bouncing off the gotcha each time.

### 3. `intent_aliases` left implicit (Q1 trap)

Q1 says: declare `intent_aliases` explicitly when any method on your skill
takes a textual search/query/intent parameter. The runtime **never guesses**.

The trap is shipping a method with a param named `query` (or `q`, or
`search_text`, or anything semantically intent-shaped) without listing it
under top-level `guidance.intent_aliases`. The generator hard-errors when it
can't map the param to canonical `intent`, and the registry rejects the
package.

Be explicit. If you take textual search input under any param name, alias it.
If you don't, omit the field — but don't ship it implicit.

---

## FAQ

**Do I need all four layers?**
No. Layer 1 (`SKILL.md` with frontmatter) is the only required layer. The
other three are recommended. A skill that ships only Layer 1 still publishes
and still loads. But agents drilling in have nowhere to go — they'll either
read your entire `SKILL.md` on every call or bounce off the surface. Ship at
least one `references/` file and your skill becomes useful at the first
drill-in.

**Where does the envelope contract live — inside one of the four layers?**
No — it lives in `client/schema.yaml`, alongside the rest of your API
definition. The four layers are content directories the agent reads. The
envelope contract is metadata the gateway proxy reads. They're sibling
concerns inside one skill folder.

**Can I publish a layers-only skill (no envelope contract)?**
Technically yes — your `schema.yaml` can omit the `guidance:` block. The
proxy will fall back to skill-name-as-guidance. Agents will function but
won't get the per-method `next` and `warn` hints. Strongly recommended: ship
the contract block.

**Can I publish an envelope-only skill (no layers beyond `SKILL.md`)?**
Yes — Layer 1 + the contract block is a valid minimum. You're publishing a
skill where the agent's only drill-in is to read `SKILL.md` itself. Works
for tiny skills (one method, no procedures, no installable assets). Bigger
skills need `references/` to stay usable.

**Do registry items have to live inside one skill?**
Yes. Each `registry-item.json` has a required `skill:` field naming the OPVS
skill it targets. No orphan items. A registry item that conceptually applies
to two skills should be authored as two separate items (one per target
skill) — sharing files between them is fine, but the manifest is per skill.

**Do `learnings/` files count as "documentation" for the security review's
required-docs check?**
No. The security review's required-docs check is for repo-root files:
`README.md`, `SKILL.md`, `CHANGELOG.md`, `LEARNINGS.md`. The `LEARNINGS.md`
at repo root is a vendor-level summary. `learnings/` directories inside each
skill are the structured KI shape — separate concern, not interchangeable.

---

## See also

- [`packaging.md`](packaging.md) — the `package.yaml` + tarball shape
- [`signing-and-submit.md`](signing-and-submit.md) — the publish CLI step
- [`security-review.md`](security-review.md) — the seven static checks
- [`versioning.md`](versioning.md) — semver + per-item versioning
- [`03-authoring/the-contract.md`](../03-authoring/the-contract.md) — the
  canonical on-demand contract pointer
- [`03-authoring/skill-anatomy.md`](../03-authoring/skill-anatomy.md) — the
  `SKILL.md` + `schema.yaml` field walk
- Canonical 4-layer spec:
  [`SKILLS-4-LAYER-FORMAT.md`](https://github.com/opvs-ai/OPVS/blob/main/docs/planning/opvsHUB/architecture/SKILLS-4-LAYER-FORMAT.md)
- Registry item JSON Schema:
  [`registry-item.schema.json`](https://github.com/opvs-ai/OPVS/blob/main/docs/planning/opvsHUB/architecture/registry-item.schema.json)
- Lighthouse worked example:
  [`agentboard` skill folder](https://github.com/opvs-ai/OPVS/tree/main/apps/spideragent/packages/shared/skills/agentboard)
- Third-party reference: [HeyGen Hyperframes](https://github.com/heygen-com/hyperframes)
