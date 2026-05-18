# Partner keys

A partner key is a long-lived secret OPVS issues to your vendor account. It authenticates HTTP API calls to `market.opvs.ai` and scopes them to *your* vendor — you can publish under your own scope, you cannot touch other vendors' packages.

This page covers:

- What a partner key looks like (prefix + body, ~64 chars)
- How to request one (today: platform admin mints via the admin UI)
- The one-shot reveal — raw key shown once, then OPVS stores only a bcrypt hash
- Where to store the key safely (CI secrets, never the repo)
- What "scoped to vendor X" means for HTTP API endpoints
- Why this is different from `ADMIN_API_KEY` (platform-master, never given to vendors)

<!-- TODO V-C: write content. Reference MKT-PARTNER-KEYS-BRIEF.md (the canonical spec, shipped 2026-04-26, opvsHUB#1 Option B). Cover the prefix-lookup + bcrypt-verify pattern at a high level — vendors don't need internals, but they need to know that revocation is soft-delete and instant. -->

---

> Filled in by content session **V-C**.
