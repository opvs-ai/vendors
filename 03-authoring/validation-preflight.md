# Validation — preflight

Run the validator on your laptop before you submit. The marketplace's security review repeats the same checks server-side, so if your local run is clean, the publish is almost always clean.

This page covers:

- `opvs-skills generate <name>` — generates the Tier 3 plugin so you can see what the gateway will load
- `opvs-skills validate` — cross-tier consistency check
- `opvs-skills publish --dry-run` — runs the same Pydantic + manifest checks the registry will run, without uploading
- The seven server-side security checks (manifest validation, signature, schema lint, blocklist, auth scheme, rate-limit declarations, doc presence) — what each one catches
- How to read validator findings (the `_findings: [...]` audit array on the doctor response)
- Common preflight failures and quick fixes — short pointers to [common-gotchas.md](common-gotchas.md)

<!-- TODO V-D: write content. Reference apps/spideragent/packages/marketplace/api/app/services/security_review_service.py for the canonical 7 checks. Reference packages/opvs-skills/ for the local CLI surface. -->

---

> Filled in by content session **V-D**.
