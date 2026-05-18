# Publishing tokens

The `@opvs-ai/skills` CLI does the heavy lifting at publish time — packaging your skill, signing it with your cert, calling the registry. To make that work, it reads a handful of values from your environment. This page is the one-page reference for what each one is, where it comes from, and where it ends up.

There are **two separate auth layers**. Get them straight up front:

```
@vendor/your-package (npm tarball)
       │
       ├── pushed to ──> npm.opvs.ai (Verdaccio)
       │                 auth: per-vendor Verdaccio user via ~/.npmrc
       │
       └── manifest registered ──> market.opvs.ai (Marketplace HTTP API)
                                   auth: X-Admin-Key (your partner key)
```

The Verdaccio token gets your *bytes* into the npm registry. The partner key gets your *manifest* into the marketplace registry. They are independent — leaking one does not give an attacker the other, and you rotate them separately.

## What the CLI reads at publish time

| Env var | What it is | Required for | Notes |
|---|---|---|---|
| `OPVS_MARKETPLACE_ADMIN_KEY` | Your partner key | `publish`, `revoke`, `doctor` (admin views) | See [partner-keys.md](partner-keys.md) |
| `OPVS_MARKETPLACE_URL` | Registry base URL | All commands | Defaults to `https://market.opvs.ai` |
| `OPVS_SIGNING_KEY` (path) | Filesystem path to your ed25519 private key | `publish` only | See [01-onboarding/apply-for-a-cert.md](../01-onboarding/apply-for-a-cert.md) |

The `OPVS_SIGNING_KEY` value is a **filesystem path** to a file containing the key. The CLI reads the file at publish time, signs the bundle hash, and discards the in-memory copy. The key itself is never passed as an env var value.

For the full CLI surface (every flag, every command), see [reference/cli.md](../reference/cli.md).

## What appears in headers vs what stays on disk

When you run `opvs-skills publish`, here is what crosses the wire and what does not.

```
On the wire                          On disk only
────────────                          ────────────
X-Admin-Key: <partner key>           Your private signing key file
Authorization: Bearer <npm token>    Your secrets manager creds
Bundle SHA-256                       Anything in ~/.npmrc
ed25519 signature of bundle hash     Your local .env
Manifest JSON (no secrets)
```

The signing key never leaves your machine. The CLI hashes the bundle, signs the hash, attaches the signature to the manifest, and ships the signed manifest. The registry verifies using your *public* key from the vendor allowlist — see the trust loop in [04-publishing/signing-and-submit.md](../04-publishing/signing-and-submit.md).

## npm/Verdaccio auth — separate layer

The marketplace stores manifests. The actual npm tarballs live on `npm.opvs.ai`, a Verdaccio instance. Verdaccio has its own auth model and you log in to it with your own per-vendor user account.

Each vendor scope gets a dedicated Verdaccio user:

```
@opvs-ai/*    →  user: opvs-ai-publisher
@spideriq/*   →  user: spideriq-publisher
@di-atomic/*  →  user: di-atomic-publisher
```

OPVS provisions your user account during onboarding and hands you the credentials the same way as a partner key — one-shot reveal, store it in your secrets manager, never commit it.

You add it to your shell or your CI as a `.npmrc`:

```ini
# ~/.npmrc — never commit this
//npm.opvs.ai/:_authToken=REDACTED_NPM_TOKEN
@opvs-ai:registry=https://npm.opvs.ai
@your-vendor:registry=https://npm.opvs.ai
```

Two important rules about this layer:

1. **Your Verdaccio user can only publish to *your* scope.** Even though Verdaccio's `htpasswd` plugin authenticates any logged-in user with `$authenticated`, the package policy at `infra/verdaccio/config.yaml` only allows local publish for the trusted scopes (`@opvs-ai/*`, `@di-atomic/*`, `@spideriq/*`). Every other pattern is `proxy: npmjs` + `publish: $authenticated_no_one` — so a leaked token cannot squat `@react/whatever` and shadow upstream packages. The hardening rationale is in `docs/planning/opvsHUB/LEARNINGS.md` (Shai-Hulud / W2.S.1).

2. **Bytes alone are not enough.** You can `npm publish` a tarball to `npm.opvs.ai` without ever talking to the marketplace — but a brand cannot install it. The marketplace install flow only resolves packages that have been registered via `POST /api/v1/marketplace/packages` (the call that needs your partner key). The two-step pattern is intentional: tarballs land in Verdaccio first, then the signed manifest registers them. If either step is missing, nothing happens to brands.

## A typical CI publish job

Here is the full set of secrets a CI job needs, written out so you can copy it into your pipeline config. Real values are redacted; replace with your own from your secrets manager.

```yaml
env:
  # Marketplace HTTP auth — your partner key
  OPVS_MARKETPLACE_ADMIN_KEY: ${{ secrets.OPVS_MARKETPLACE_ADMIN_KEY }}
  OPVS_MARKETPLACE_URL: "https://market.opvs.ai"

  # Signing — path to your ed25519 private key, written from a CI secret
  OPVS_SIGNING_KEY: "/tmp/opvs-signing.key"

  # Verdaccio auth — your per-vendor npm token (separate from partner key)
  NPM_TOKEN: ${{ secrets.OPVS_NPM_TOKEN }}

steps:
  - run: |
      # Materialize the signing key on disk (file deleted on cleanup)
      echo "${{ secrets.OPVS_SIGNING_KEY_PEM }}" > "$OPVS_SIGNING_KEY"
      chmod 600 "$OPVS_SIGNING_KEY"

      # Configure npm/Verdaccio
      echo "//npm.opvs.ai/:_authToken=$NPM_TOKEN" >> ~/.npmrc
      echo "@your-vendor:registry=https://npm.opvs.ai" >> ~/.npmrc

      # Publish — CLI reads everything from env + .npmrc
      npx @opvs-ai/skills publish
```

Three secrets, three layers, one command. If a publish ever fails with a credential error, the CLI will tell you which layer rejected the call — partner key, signing cert, or Verdaccio token. Each one is rotated independently per [credential-rotation.md](credential-rotation.md).

## If a token leaks

Treat any disclosed token as burned, regardless of where it was disclosed. The recovery is:

- **Partner key** — ask OPVS to revoke it; we mint a fresh one
- **Signing cert** — rotate via `/api/v1/marketplace/ca/mint-rotation` (covered in the next page)
- **Verdaccio token** — ask OPVS to invalidate your Verdaccio user and reissue

For the full rotation procedure across all three credential types, see [credential-rotation.md](credential-rotation.md).

For the wire-level HTTP details (endpoints, headers, error shapes), see [reference/api.md](../reference/api.md).
