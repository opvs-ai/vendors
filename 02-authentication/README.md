# 02. Authentication

You have a vendor account and a signing cert from [01-onboarding/](../01-onboarding/). Before you can call the marketplace HTTP API to publish, revoke, or upload a bundle, you need a **partner key**.

Two credentials, two jobs:

- **Cert** (from onboarding) — signs your package bundle so the registry trusts the bytes
- **Partner key** (this section) — authenticates your HTTP API calls to `market.opvs.ai`

Both are scoped to your vendor. Neither gives you admin power over anyone else's packages.

## Pages

- [partner-keys.md](partner-keys.md) — what a partner key is, how to mint one, where to store it
- [publishing-tokens.md](publishing-tokens.md) — the env vars and headers the CLI expects at publish time
- [credential-rotation.md](credential-rotation.md) — rotating both keys and certs when they leak or expire

## What's next

Authenticated and ready to build. Head to [03-authoring/](../03-authoring/) to learn what a skill actually looks like on disk.

---

> Pages in this section are filled in by content session **V-C**.
