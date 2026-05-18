# Common gotchas

A short list of mistakes that catch first-time skill authors. Each one is a real incident the platform has actually hit; we surface them up front so you don't have to discover them the same way.

This page covers:

- **OpenAI tool-name regex** — every tool name must match `^[a-zA-Z0-9_-]+$`. No dots. No `@`. No `/`. Dots in tool names brick the entire `tools` array on the next OpenAI call.
- **Sibling `_guidance`, not wrapper** — the locked contract puts `_guidance` next to your data, not around it. Wrapping breaks on-demand discovery.
- **`intent`, not `query`** — canonical input field name across every migrated skill. Stick to it.
- **Closed `telemetry_signal` enum** — exactly four values: `absent | cold | warm | hot`. No "warmish", no "unknown".
- **`additional_filters` for structured params** — instead of inventing top-level fields per method, lift filters into the structured map.
- **`requires.env` in `SKILL.md` frontmatter** — blocks skill resolution if the env vars actually live on a different skill (e.g. `opvs-foundation`). Skip it or you'll be debugging "skill not loaded" for an hour.
- **Pre-merge LLM verification is mandatory** — gateway log "tools registered" does NOT prove the model can call them. Drive a real completion before you call a publish done.

<!-- TODO V-D: write content. Reference the foundational tool-name regex memory (feedback-openai-tool-name-regex.md), the contract memory (opvshub-skills-upgrade-w2-closed-2026-05-16.md), and LEARNINGS.md in the OPVS planning dir. -->

---

> Filled in by content session **V-D**.
