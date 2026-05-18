# Security review

Before your package goes live, the marketplace runs a seven-check static review. This page lists every check, what it catches, and how to fix a failure.

The seven checks (in order):

1. **Manifest schema** — does `package.yaml` validate against the JSON schema
2. **Signature verify** — does the ed25519 signature match your registered public key
3. **Schema lint** — does every method in `schema.yaml` declare an entitlement
4. **Blocklist** — no raw `shell`, `eval`, or unscoped `network.egress.public` when narrower would fit
5. **Auth scheme** — auth type on the approved list (`pat / oauth / oauth_pkce / bearer / hmac / none`)
6. **Rate-limit declarations** — every outbound method declares a rate limit
7. **Required docs** — `README.md`, `SKILL.md`, `CHANGELOG.md`, `LEARNINGS.md` present

Plus the meta check for Verified-tier candidates: integration tests present in `tests/`.

This page covers:

- Each of the seven checks in plain English
- What "blocking" means (publish fails with a 422 + finding codes)
- How to read the finding code structure
- Worked examples of common failures and the one-line fixes
- The score model — every check passed = 100/100

<!-- TODO V-E: write content. Reference apps/spideragent/packages/marketplace/api/app/services/security_review_service.py for the canonical implementation. Reference VENDOR-GUIDE.md §Security checks for the existing field-by-field walkthrough. -->

---

> Filled in by content session **V-E**.
