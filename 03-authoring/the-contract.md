# The on-demand contract

> **This page is a pointer.** The canonical contract spec lives in [`opvs-ai/marketplace/reference/on-demand-contract-spec.md`](https://github.com/opvs-ai/marketplace/blob/main/reference/on-demand-contract-spec.md). Every runtime (OpenClaw, Claude Code, Cursor, Windsurf, langchain, and the rest) consumes the same contract, so its single source of truth lives there. We keep this summary here so you don't have to leave the authoring path blind.
>
> If anything below disagrees with the marketplace spec, **marketplace wins**.

## What the contract is

Every skill on OPVS follows the same response shape so the platform can present skills consistently to any agent runtime and so agents can discover what to do next without reading documentation.

If you follow the contract, your skill plugs into the on-demand discovery system for free — `skill_search`, `skill_describe`, `skill_call`, telemetry-driven ranking, all of it. If you don't, your skill still works, but agents lose the affordances that make discovery feel native.

## The four locked rules

These are the four things the contract enforces. Each one has a one-line summary here. The full reasoning, edge cases, and examples are in the marketplace spec.

### 1. Sibling `_guidance`, not wrapper

`_guidance` is a peer key on your existing response root. It is **not** a wrapper that nests your response under `data`.

```yaml
# CORRECT
{ id: "abc", status: "done", _guidance: "..." }

# WRONG
{ data: { id: "abc", status: "done" }, _guidance: "..." }
```

Wrapping breaks every flat-response consumer (dashboard, CLI, SSR site, generated SDK) on the next deploy. This pattern was rolled out across seven internal services in May 2026; five shipped wrapper-shape and broke production. Don't repeat it.

### 2. Closed `telemetry_signal` enum

Exactly four values: `absent | cold | warm | hot`.

No `"warmish"`, no `"unknown"`, no `null` when you mean `absent`. The on-demand model uses this enum to rank skills in agent context; non-enum values silently drop you out of consideration.

### 3. `intent` over `query`

The canonical input field for "what does the agent want to do" is named `intent`. Not `query`. Not `q`. Not `prompt`.

This is conventional, not enforced — your skill will still load with a different name — but `intent` is what every other skill uses and what the on-demand layer searches against. Mixing names hurts discoverability.

### 4. `skill_call` routes by manifest `id`

Agents invoke other skills through the always-on `skill_call` tool. The `skill` parameter it expects is the **sub-skill id from `manifest.skills[].id`**, not the `@vendor/name` package coordinate.

Your manifest's `skills[].id` field is the routing key. Pick one that is short, lowercase, and unique within your package.

## Optional but recommended

### `_findings` audit array

If your method produces diagnostics — health checks, validator results, security findings — surface them as a sibling `_findings: [...]` array. Doctor endpoints and the marketplace UI both render `_findings` if present.

This is optional; if you do not have findings to report, omit the field entirely. Empty arrays are fine but noisier than just omitting.

### `additional_filters` for structured params

Instead of inventing top-level fields per method (`assignee_id`, `priority`, `due_before`, ...), lift filters into a single structured `additional_filters: { ... }` map. Easier on the agent, easier on you when you add new filters later.

## Read the full spec

Everything above is the high-level summary. For the field-by-field schema, version compatibility rules, the rationale behind each locked rule, and the example responses for each contract variant, read:

[**opvs-ai/marketplace/reference/on-demand-contract-spec.md**](https://github.com/opvs-ai/marketplace/blob/main/reference/on-demand-contract-spec.md)

That is the canonical home. This page is just a doormat.

## What happens if you ignore it

Your skill still works. The gateway loads it, the plugin registers its tools, agents can call those tools directly by name.

What you lose:

- Your skill stops appearing in `skill_search` results because the on-demand ranking layer can't read its signals
- Agents can't `skill_describe` your tools and get a useful answer
- Telemetry-driven UI features (badges, healthy/cold indicators, "agents typically use this with X" hints) all show as `absent`
- Your skill works in isolation but not as part of the ecosystem

Most vendors decide that following the contract is cheap and worth it. The four rules are small and the codegen does most of the work for you.

## What's next

Once your `schema.yaml` matches the contract, validate locally before you submit — see [validation-preflight.md](validation-preflight.md).

For the practical "how do I ship both the contract block and the 4-layer skill directories in one signed package" walkthrough, jump to [`04-publishing/4-layer-and-envelope.md`](../04-publishing/4-layer-and-envelope.md).
