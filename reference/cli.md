# `@opvs-ai/skills` CLI

The vendor-facing CLI. Wraps the HTTP API plus the signing flow.

This page covers every command, every flag, every env var the CLI reads.

Commands:

- `opvs-skills generate <name>` / `--all` — emit Tier 3 plugin, MCP tools, TS SDK, CLI subcommands, OpenAPI from `schema.yaml`
- `opvs-skills validate` — cross-tier consistency check (run before publish)
- `opvs-skills publish` — sign + upload bundle, with `--dry-run` for preflight
- `opvs-skills install <vendor>/<name>` — install a published package into a brand (admin-side, but documented here for parity)
- `opvs-skills doctor <skill>` — brand-side health check; you'll use this to verify your own brand-installed package works

Env vars:

- `OPVS_MARKETPLACE_ADMIN_KEY` — partner key for HTTP auth
- `OPVS_MARKETPLACE_URL` — registry base (default `https://market.opvs.ai`)
- Signing-key file path — convention TBD, planner to confirm

<!-- TODO V-H: write content. The canonical CLI is packages/opvs-skills/ in the OPVS monorepo, published to Verdaccio as @opvs-ai/skills (current version 0.4.2). -->

---

> Filled in by content session **V-H**.
