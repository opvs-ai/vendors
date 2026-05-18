# Marketplace HTTP API

Vendor-relevant endpoints on `market.opvs.ai`. This is the on-the-wire view; the CLI wraps these so you usually don't call them directly, but they're documented for CI integrations and one-off curl.

This page covers:

- **Publish** — `POST /api/v1/marketplace/packages`
- **Revoke** — `POST /api/v1/marketplace/packages/{package_id}/revoke`
- **Package list** — `GET /api/v1/registry/packages` (public read)
- **Package detail** — `GET /api/v1/registry/packages/{vendor}/{name}`
- **Version history** — `GET /api/v1/registry/packages/{vendor}/{name}/versions`
- **Doctor** — `GET /api/v1/skills/{skill_id}/doctor` (your brand-installed health check)
- **Marketing overlay PATCH** — admin-only, included so vendors know what changes mid-stream
- **Auth headers** — `X-Admin-Key` (partner key) on every write; cert signature header on publish; brand-pin handling on doctor

<!-- TODO V-H: write content. The canonical reference is apps/api/app/api/v1/marketplace.py (proxy) + apps/api/app/api/v1/registry.py (registry reads) in the OPVS monorepo. Include error response shapes (especially 422 with finding codes). -->

---

> Filled in by content session **V-H**.
