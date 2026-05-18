# Skill anatomy

A skill is a directory with two load-bearing files plus a few optional ones. Understanding the shape lets you author by example.

This page covers:

- The minimum viable skill — `SKILL.md` + `client/schema.yaml`
- `SKILL.md` frontmatter — name, version, description, category, triggers, requires_auth
- `schema.yaml` top-level — guidance block, methods block, base_url, auth declaration
- `methods:` shape — input params, response shape, telemetry signal, per-method guidance
- Optional files — `CHANGELOG.md`, `LEARNINGS.md`, `GUIDES/`, per-method `SKILL.md` subfiles
- What gets generated from your `schema.yaml` — Tier 3 plugin, MCP tools, TS SDK, CLI subcommands, OpenAPI

<!-- TODO V-D: write content. Reference apps/spideragent/packages/shared/skills/vayapin-search/ and vibe-publish/ in the OPVS monorepo as the two canonical contract examples. Reference docs/planning/opvsHUB/CLAUDE.md §Guidance Block for the guidance shape. -->

---

> Filled in by content session **V-D**.
