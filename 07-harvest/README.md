# 07 — Harvest: learn from how your skill is actually used

The vendor lifecycle doesn't end at "published." Once brands install your skill and their
agents start using it, the highest-signal way to improve it is to look at **what actually
happens in production** — not to guess from the outside.

**Harvest** is a read-only, vendor-scoped, audited surface that lets you read the deployed-agent
experience your skills produced: which agents run your skill, their task history, their tool-call
traces, and — with the brand's permission — the agent's workspace files and memories. You drive it
from your own machine with your partner key; no OPVS operator is involved.

> This is the **supply side** improving its own product. It is scoped to *your* footprint — the
> brands that installed *your* skills — and nothing else.

## The five reads

| Method | Returns |
|---|---|
| `listAgents` | The agents on a brand that can run your skill |
| `exportAgentTaskHistory` | Task cards those agents worked (titles, status, result summaries, custom fields) |
| `gateTracesByAgent` | Tool-call traces for your skills (timestamp, method, latency, error code) |
| `readAgentWorkspace` | An agent's workspace files (SOUL / AGENTS / MEMORY / …) — **raw, opt-in** |
| `dumpAgentMemory` | An agent's structured memories — **raw, opt-in** |

## The one rule that surprises vendors: the RAW split

The five methods are **not** equally sensitive, so consent is split:

- **Non-raw** — `listAgents`, `exportAgentTaskHistory`, `gateTracesByAgent`. These are metadata
  or already scoped to your own skills. They work **by default** on any brand in your footprint.
- **Raw** — `readAgentWorkspace`, `dumpAgentMemory`. These return **whole-persona** content that
  can't be attributed to a single skill. They return **403** until the brand explicitly opts your
  package in.

**A 403 on `readAgentWorkspace` or `dumpAgentMemory` is the expected default — it is not a bug.**
It means the brand hasn't opted your package into raw access. The right response is to ask the
brand to opt in (they do it in their dashboard, or via `PUT /harvest/consent`), **not** to file a
support ticket. The 403 body tells you exactly this, and reminds you that non-raw harvest is still
available.

## Scope: your footprint, and nothing outside it

You can only harvest a brand that has an **active install of one of your packages**. Ask about any
other brand and you get:

```
404  { "detail": "brand not found" }
```

This is **identical** to the answer for a brand id that doesn't exist. That's deliberate: it denies
you the ability to probe which brand ids are real. Don't build anything that treats 404 as "exists
but denied" — you cannot tell the two apart, by design.

## What you get back is redacted, and every read is logged

- **Redaction is server-side and fail-closed.** Credentials, API keys, tokens, and PII are stripped
  before anything leaves OPVS. A brand can add its own extra redaction pattern. If redaction can't
  process a value safely, it drops it — you will never receive a secret because a filter had a bug.
- **Every call is audited** — allowed *and* refused — and the brand can review exactly what you
  read, and revoke your access at any time. Treat the audit log as a feature: it's what makes brands
  comfortable leaving harvest on.
- **Rate limit:** 100 calls / hour / brand, 5 MB per call.

## Re-export policy (non-negotiable)

You may fold what you learn into your skill — better guidance, better references, a new version.
You may **not** re-export raw agent memories or messages to third parties. Aggregate patterns only.
This is a condition of holding a partner key.

## Using it

Everything runs through the vendor CLI (`@opvs-ai/skills`, ≥ 0.5.10):

```bash
export OPVS_PARTNER_KEY=opvs_pk_…

# non-raw — works on any brand in your footprint
opvs-skills harvest agents  --brand 5
opvs-skills harvest tasks   --brand 5 --agent lead-brand-5
opvs-skills harvest traces  --brand 5 --agent lead-brand-5

# raw — 403 until the brand opts your package in
opvs-skills harvest workspace --brand 5 --agent lead-brand-5 --args '{"paths":["SOUL.md","MEMORY.md"]}'
opvs-skills harvest memory    --brand 5 --agent lead-brand-5 --args '{"query":"pricing","limit":50}'
```

`--partner-key` (or `OPVS_PARTNER_KEY`) authenticates; `--api-url` (or `OPVS_API_URL`) targets the
gateway; `--json` prints the raw body. A denied or rate-limited call exits non-zero, so it's safe in
CI.

## The loop

```
  publish  →  brands install  →  their agents use your skill  →  HARVEST what worked
                                                                        │
                                                        fold real usage into guidance/references
                                                                        │
                                                                  publish a better version
```

That's Wave-3: your skill improves from grounded production signal instead of authoring intuition.

## What's next

You've walked the full vendor lifecycle — onboarding through harvest. The [reference/](../reference/)
section holds the API surface, CLI commands, and glossary for quick lookup.

---

> Added 2026-07-20 alongside the platform `@opvs-ai/agent-inspector` surface. The authoritative
> platform-side spec (consent table, scope resolver, redaction, audit schema) lives in the OPVS repo
> at `docs/planning/Marketplace/architecture/AGENT-INSPECTOR.md`.
