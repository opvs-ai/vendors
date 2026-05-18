# Packaging

The marketplace packages skills into a signed tarball. This page describes the shape that tarball expects.

This page covers:

- The `package.yaml` manifest — every field, what it controls
- Standard package layout — top-level `README.md`, `SKILL.md`, `CHANGELOG.md`, `LEARNINGS.md`, `skills/`, `schemas/`, `tests/`, `docs/`
- Skill IDs inside a package — namespacing within your vendor scope
- Peer dependencies — when one of your skills calls another vendor's skill (e.g. `@spideriq/core-skills` needs `@opvs-ai/platform-skills`)
- What goes IN the tarball — and what the marketplace deliberately leaves out (CI secrets, .env files, signing keys)

<!-- TODO V-E: write content. Reference MARKETPLACE-CONCEPT.md §Standard package layout + §Package manifest schema. Reference VENDOR-GUIDE.md in the OPVS docs for the field-by-field manifest walkthrough. -->

---

> Filled in by content session **V-E**.
