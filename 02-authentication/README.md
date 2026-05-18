# 02. Authentication

You have a vendor account and a signing cert from [01-onboarding/](../01-onboarding/). Before you can call the marketplace HTTP API to publish, revoke, or upload a bundle, you need a **partner key**.

Two credentials, two jobs:

- **Cert** (from onboarding) — signs your package bundle so the registry trusts the bytes
- **Partner key** (this section) — authenticates your HTTP API calls to `market.opvs.ai`

Both are scoped to your vendor. Neither gives you admin power over anyone else's packages.

## Pages

| Page | What it covers |
|---|---|
| [partner-keys.md](partner-keys.md) | What a partner key is, how OPVS mints one for you, where to store it, what "scoped to vendor X" means in practice |
| [publishing-tokens.md](publishing-tokens.md) | The env vars and headers the CLI reads at publish time, and the separate npm/Verdaccio token layer for per-vendor publish accounts |
| [credential-rotation.md](credential-rotation.md) | 90-day rotation cadence, the dual-trust window for cert rotation, and the emergency path when a credential leaks |

## What's next

Authenticated and ready to build. Head to [03-authoring/](../03-authoring/) to learn what a skill actually looks like on disk.
