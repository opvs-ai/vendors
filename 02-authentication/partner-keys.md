# Partner keys

A partner key is a long-lived secret OPVS issues to your vendor account. You attach it to every HTTP API call you make to `market.opvs.ai`. The registry checks it on the way in and pins your call to *your* vendor scope — you can publish, revoke, and upload bundles under your own packages, and nothing else.

This is the credential that authenticates *you the vendor* over HTTP. It is not the same as the ed25519 signing cert from [01-onboarding/apply-for-a-cert.md](../01-onboarding/apply-for-a-cert.md). The cert signs the *bytes* of your bundle so the registry trusts the artifact. The partner key signs the *request* so the registry trusts the caller.

Both must check out for a publish to land.

## What it looks like

A partner key is a base64url string around 64 characters long. It has a stable 8-character prefix and a secret body:

```
opvs_pk_a1b2c3d4_REDACTED_BODY_NEVER_SHOWN_AGAIN
└────┬───┘└──┬───┘└──────────────┬──────────────┘
   tag    prefix              secret body
```

OPVS only stores the prefix in clear text. The body is bcrypt-hashed and discarded — even platform admins cannot read it back.

The prefix is what the registry uses for fast lookup. The bcrypt hash is what proves the body matches. This is the same pattern GitHub uses for its `ghp_*` tokens and AWS uses for `AKIA*` keys.

## How you get one

Today, platform admins mint partner keys for you through the admin UI at `/admin/marketplace/vendors/<your-vendor>/keys`. Self-service minting is on the roadmap but not yet shipped.

To request a key, contact your OPVS account owner with two pieces of information:

1. A short label describing where the key will live (e.g. `"ops@acme.com — production publish CI"`)
2. Whether this is a first key or a rotation of an existing key

The admin clicks "Mint key" and the UI shows the raw key **exactly once**. They paste it into a secure channel — 1Password, AWS Secrets Manager, Vault — and hand it to you. After that screen closes, nobody can read the full key again.

If the key is ever lost, the only recovery is to mint a new one and revoke the old one. That is the rotation flow described in [credential-rotation.md](credential-rotation.md).

## Where to store it

Treat a partner key like a database password.

```
OK   GitHub Actions secret                  -> OPVS_MARKETPLACE_ADMIN_KEY
OK   GitLab CI variable (masked + protected)
OK   Your secrets manager (Vault, AWS SM, 1Password)
OK   Local dev: a gitignored .env file outside the repo
NO   Hardcoded in source
NO   Committed to git, even briefly
NO   Pasted into Slack, email, or any chat tool
NO   Logged by your CI runner
```

If you commit a key by accident, treat it as compromised even if you force-push the history. Mint a new one and revoke the old one. See [credential-rotation.md → If a key leaks](credential-rotation.md#if-a-key-leaks).

## What "scoped to vendor X" means

Every HTTP API call to `market.opvs.ai` authenticated with a partner key is automatically pinned to the vendor that owns the key. Two consequences.

**You can do this:**

- Publish a new version of `@your-vendor/anything`
- Revoke any version you previously published
- Upload a bundle file to your vendor's R2 storage
- Read package metadata (everyone can; public endpoints don't need auth)

**You cannot do this, even if you ask nicely in the URL:**

- Publish under another vendor's scope — the registry rejects the manifest with `partner-key vendor scope mismatch`
- Revoke another vendor's package
- Manage install grants for any brand
- Mint or revoke other partner keys (yours or anyone else's)
- Run the security review pipeline on someone else's submission

Platform admins use a separate secret called `ADMIN_API_KEY` that is *not* vendor-scoped. They use it for cross-vendor operations like minting your key in the first place. That secret never leaves OPVS staff — you will never have a copy of it, and you should not ask for one.

## Example — signing a publish request

The CLI handles this for you, but here is the underlying HTTP call so you know what is on the wire.

```bash
# OPVS_MARKETPLACE_ADMIN_KEY is your partner key, loaded from a secrets manager.
# OPVS_MARKETPLACE_URL defaults to https://market.opvs.ai.
# bundle.json contains the manifest + signature + bundle URL.

curl -X POST "$OPVS_MARKETPLACE_URL/api/v1/marketplace/packages" \
  -H "X-Admin-Key: $OPVS_MARKETPLACE_ADMIN_KEY" \
  -H "Content-Type: application/json" \
  -d @bundle.json
```

The header is `X-Admin-Key`. The value is the partner key. The registry resolves the prefix, bcrypt-verifies the body, looks up the vendor scope, then enforces it against the manifest's `vendor_id`.

If anything fails — wrong header, unknown prefix, bcrypt mismatch, revoked key, scope mismatch — you get a `403` with a one-line reason. No information leaks about whether the key exists, just whether *this* call is authorized.

## Revocation is instant

Soft-delete by design. When an admin clicks Revoke (or you ask for an emergency revoke), OPVS sets `revoked_at = NOW()` on the key row. The next API call carrying that key sees a `403 partner-key invalid` immediately — no propagation delay, no caching window.

The bcrypt hash stays in the database with the prefix and the `revoked_reason`. You can audit later who revoked which key and why, even years after the fact.

For the rotation playbook, see [credential-rotation.md](credential-rotation.md).
