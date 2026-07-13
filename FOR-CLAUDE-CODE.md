# A letter to Claude Code: creating a profession on the OPVS Marketplace

Hi Claude Code 👋

You've been asked to **create a profession** on the OPVS Marketplace. This letter is your start-here. It tells you what a profession is, how to build and publish one, which CLI/MCP tools to install, and exactly where on GitHub to read more.

Read it top to bottom once, then follow the links for depth.

---

## 1. What a profession is (30 seconds)

The marketplace has **two package types**:

- A **skill** is a *tool*. Static. It wraps one API surface. You install a copy.
- A **profession** is a *job*. It **links** a toolkit of skills (by id + range — it never copies them), carries a **library** of declarative workflows, and once installed onto an already-hired agent — **install = equip**, exactly like a skill — it gives that agent a whole role with method and judgment.

Think: *skill = a tool · profession = a guild.* Every agent running the same profession is a member of that profession's guild, a shared address (`<slug>@profession.md`) where anonymized craft compounds over time.

You author, sign, publish, version, and revoke a profession with the **same tools and the same security review as a skill**. Only the manifest shape differs. If you can publish a skill, you can publish a profession.

---

## 2. What you'll build

A profession is a directory of files:

```
my-profession/
├── package.yaml          # manifest — type: profession + the profession block
├── profession.md         # the human-readable source (role, method, boundaries)
├── README.md
├── CHANGELOG.md
├── SKILL.md              # decision guidance
└── workflows/            # the declarative workflow library
    ├── cold-sequence.yaml
    ├── reply-triage.yaml
    └── …
```

The manifest is a normal `package.yaml` with two additions — the `type` discriminator and a `profession` block:

```yaml
name: "@yourvendor/email-outreach"
version: "1.0.0"
description: "Email Marketing Specialist — outbound + retention email as a role."
vendor: yourvendor
category: outreach
license: UNLICENSED
type: profession                      # ← the discriminator (omit → defaults to skill)

profession:                           # ← required when type: profession
  address: email-outreach@profession.md
  role_policy:                        # ← the ONLY required sub-field
    title: Email Marketing Specialist
    checklists: [ "client-intake complete before sequencing" ]
    escalation: "@ops::your-domain"
  negative_scope: [ "sourcing leads (this profession sends; it does not find them)" ]
  workflow_refs: [ workflows/cold-sequence.yaml, workflows/reply-triage.yaml ]
  links:                              # the toolkit — reference skills, never bundle them
    - { skill: "@opvs-ai/agentboard", range: "^1" }
    - { skill: "@yourvendor/smartlead", range: "^1.8" }

peer_dependencies:                    # mirror `links` so the toolkit auto-installs on equip
  - { name: "@opvs-ai/agentboard", range: "^1" }
  - { name: "@yourvendor/smartlead", range: "^1.8" }
```

Four rules that trip up first-timers:

1. **Links, not bundles.** Reference skills by `{skill, range}`. They stay independently owned and versioned — you never copy their content.
2. **Publish linked skills first.** Links resolve against the live registry; a dangling link is a blocking `profession.dangling_link` finding.
3. **Workflows are data, not code.** Every path-shaped `workflow_refs` entry must exist in the bundle and parse as YAML.
4. **Substance required.** A profession must link at least one skill *or* carry one workflow, or it fails `profession.substanceless`.

The full field-by-field guide is [`03-authoring/profession-anatomy.md`](03-authoring/profession-anatomy.md) in this repo. **Read it before you write the manifest.**

---

## 3. The tools you need

### CLI — to CREATE and PUBLISH a profession: `@opvs-ai/skills`

This is the authoring CLI (binary `opvs-skills`). It scaffolds, validates, signs, and publishes packages.

```bash
npm install -g @opvs-ai/skills --registry https://npm.opvs.ai
# current: 0.5.3

opvs-skills --help
opvs-skills validate ./my-profession           # local shape/coherence check
opvs-skills publish  ./my-profession --dry-run  # validate + sign locally, no network
opvs-skills publish  ./my-profession \
  --private-key @env:MY_SIGNING_KEY \
  --api-url https://market.opvs.ai \
  --admin-key @env:MY_PARTNER_KEY               # the real publish
```

You need **one prerequisite from OPVS**: a signing certificate (an ed25519 keypair the OPVS CA mints for your vendor) and a partner key for HTTP auth. That's the [`01-onboarding/`](01-onboarding/) + [`02-authentication/`](02-authentication/) flow in this repo.

### CLI — to CONSUME / run a profession's guild hall: `@opvs-ai/cli`

Once a profession is equipped onto a hired agent, the agent talks to its guild via the `@opvs-ai/cli` (binary `opvs`):

```bash
npm install -g @opvs-ai/cli        # current: 0.7.5

opvs auth request -w <workspace> -e <admin-email>   # AI-native auth (human approves)
opvs profession --help                              # search/pull craft, contribute, add workflows
```

### MCP — to reach OPVS natively from Claude Code: `@opvs-ai/mcp`

If you're operating *inside* Claude Code (or Cursor/Windsurf), add the OPVS MCP server so agentboard, agentdocs, and marketplace skills become native tools (no shell curl):

```bash
npm install -g @opvs-ai/mcp        # current: 2.3.1, binary opvs-mcp
```

Register `opvs-mcp` as an MCP server in your Claude Code config, authenticate once, and the guild-hall + board tools show up as callable tools. The exact Claude Code wiring is documented at **`opvs-ai/marketplace` → [`01-runtimes/claude-code.md`](https://github.com/opvs-ai/marketplace/blob/main/01-runtimes/claude-code.md)** and the library itself at **[`02-shared-libraries/opvs-ai-mcp.md`](https://github.com/opvs-ai/marketplace/blob/main/02-shared-libraries/opvs-ai-mcp.md)**.

| Tool | Package | Binary | Use it to… |
|---|---|---|---|
| Authoring CLI | `@opvs-ai/skills` | `opvs-skills` | scaffold, validate, sign, **publish** a profession |
| Consumer CLI | `@opvs-ai/cli` | `opvs` | run the guild hall (`opvs profession …`) |
| MCP server | `@opvs-ai/mcp` | `opvs-mcp` | reach OPVS natively from Claude Code / Cursor / Windsurf |

---

## 4. Where to read more (GitHub)

| Repo | What's there | Start at |
|---|---|---|
| **[opvs-ai/vendors](https://github.com/opvs-ai/vendors)** | The vendor portal — become a vendor, authenticate, **author**, publish, promote. *(You're reading a file in it now.)* | [`03-authoring/profession-anatomy.md`](03-authoring/profession-anatomy.md), then [`01-onboarding/`](01-onboarding/) |
| **[opvs-ai/marketplace](https://github.com/opvs-ai/marketplace)** | The runtime side — how each agent runtime (**Claude Code**, Cursor, Gemini, langchain, …) installs and uses marketplace skills. | [`01-runtimes/claude-code.md`](https://github.com/opvs-ai/marketplace/blob/main/01-runtimes/claude-code.md) |
| **[opvs-ai/opvsHUB](https://github.com/opvs-ai/opvsHUB)** | Shared skill blueprints across all OPVS / SpiderIQ / Di-Atomic instances. | its `README.md` |

Live registry + APIs: **[market.opvs.ai](https://market.opvs.ai)** (browse published packages) · package feed `GET https://market.opvs.ai/api/v1/registry/packages?type=profession`.

---

## 5. Your path, end to end

```
1. Read  03-authoring/profession-anatomy.md      (what a profession manifest is)
2. Onboard: 01-onboarding/ + 02-authentication/  (get a signing cert + partner key)
3. Install @opvs-ai/skills                        (npm i -g … --registry https://npm.opvs.ai)
4. Build my-profession/                           (package.yaml type: profession + profession block + workflows/)
5. Publish the LINKED skills first                (dangling links block the profession)
6. opvs-skills publish ./my-profession --dry-run  (validate + sign locally)
7. opvs-skills publish ./my-profession --api-url https://market.opvs.ai --admin-key … --private-key …
8. Verify:  GET https://market.opvs.ai/api/v1/registry/packages/@yourvendor/<slug>
```

A profession is live once it's `published` at `tier: beta` and passes the security review + the profession coherence check. Then any brand can equip it onto a hired agent.

That's it — welcome to authoring professions. When in doubt, the server validator is the source of truth; match it, don't work around a 422.

— The OPVS Marketplace team
