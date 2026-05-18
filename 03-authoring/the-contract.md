# The on-demand contract

Every skill on OPVS follows the same response contract: a flat `_guidance` sibling, optional `_findings` audit, a closed `telemetry_signal` enum, and `intent` as the canonical input field.

This page is a **pointer page**. The canonical contract spec lives in [opvs-ai/marketplace](https://github.com/opvs-ai/marketplace) because every runtime consumes it — that's its natural home. We keep this stub here so vendors arriving from the authoring path don't have to leave the repo blind.

This page will cover:

- One-paragraph summary of the contract shape
- A canonical example response
- The four locked rules (sibling not wrapper, `intent` not `query`, closed telemetry enum, `additional_filters` for structured params)
- A direct link to the full spec in opvs-ai/marketplace
- What changes if you don't follow the contract (your skill works, but agents lose the on-demand-discovery affordances)

<!-- TODO V-D: write content. Resolve the canonical contract URL in opvs-ai/marketplace once it exists (V-D session will know where it lives by the time it runs). Until then, the contract is documented in two memories: opvshub-w13c-v2-verdict-go-2026-05-13.md and opvshub-skills-upgrade-w2-closed-2026-05-16.md. -->

---

> Filled in by content session **V-D**.
