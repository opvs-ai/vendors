# Apply for a cert

Every package published to `market.opvs.ai` is signed by its vendor's ed25519 keypair. OPVS runs a small centralized certificate authority: it mints the keypair, registers the public half in the vendor allowlist, and hands you the private half exactly once via a Fernet-encrypted one-shot link.

This page covers the full flow — what the cert is for, how the mint works, where the private key lives, and how rotation works when keys age out or get compromised.

It does **not** cover ongoing HTTP API authentication. That uses a separate credential called the **partner key**, which lives in [02-authentication/partner-keys.md](../02-authentication/partner-keys.md). The cert signs your package bytes. The partner key authenticates your CLI's HTTP request that uploads the signed bytes. Two different keys, two different jobs.

## What the cert is for

The cert is an ed25519 keypair. The private half stays with you and signs every package bundle you publish. The public half stays in the OPVS CA allowlist and verifies your signatures at three points:

1. **At publish time.** The marketplace registry verifies the signature on the uploaded tarball matches your registered public key. Mismatched signatures reject before any storage write.
2. **At gateway install.** When a brand installs your package, the gateway downloads the tarball, recomputes the SHA-256, and verifies the ed25519 signature against the cached public key. Mismatches refuse to load.
3. **At rolling re-check.** The gateway polls the revocation list every 60 seconds. If your cert fingerprint shows up as revoked, every loaded copy of your packages is disabled until brands install a version signed with a fresh key.

This is the only signing mechanism on the marketplace. There is no "skip the signature for now" fallback.

## The mint flow

The full path from "I want a cert" to "my private key is in CI."

```
   Vendor                                  OPVS ops              OPVS CA
     │                                        │                     │
     │  email or DM: "I need a cert"          │                     │
     │ ─────────────────────────────────────► │                     │
     │                                        │  POST /ca/mint      │
     │                                        │ ──────────────────► │
     │                                        │                     │
     │                                        │   ◄── generate ed25519 keypair
     │                                        │   ◄── store public + fingerprint
     │                                        │   ◄── Fernet-encrypt private
     │                                        │                     │
     │                                        │  short_code, one_shot_url
     │                                        │ ◄────────────────── │
     │  email with one_shot_url               │                     │
     │ ◄───────────────────────────────────── │                     │
     │                                        │                     │
     │  GET /ca/reveal/{short_code}                                  │
     │  (token IS auth)                                              │
     │ ─────────────────────────────────────────────────────────────►
     │                                                               │
     │  raw private key (one time only)                              │
     │ ◄─────────────────────────────────────────────────────────────│
     │                                                               │
     │  copy into CI secrets                                         │
     │  link expires → key is now unrecoverable from OPVS            │
```

Five things happen in that flow worth calling out individually.

**Mint is admin-driven in Phase 1.** You do not call the CA mint endpoint yourself. An OPVS ops engineer calls `POST /api/v1/marketplace/ca/mint` with your `vendor_id` and an optional label.

**Keypair generation is in-process and instant.** ed25519 is a small fixed-size curve — 32 bytes for the public key, 64 bytes for the private. There are no parameters to choose. Generation takes well under a millisecond.

**The private key is encrypted at rest with Fernet before any database write.** The encryption key for that Fernet wrapper lives in the marketplace service's environment, not in the DB. The encrypted blob sits in a short-TTL row.

**The one-shot reveal URL is the same pattern SpiderIQ uses for credential invites.** Token IS auth. No login required at `/ca/reveal/{short_code}` — the URL itself is the credential. The URL expires after 72 hours, or on first successful read, whichever comes first.

**After the reveal is consumed, OPVS no longer has your private key.** The Fernet-encrypted blob is deleted from the DB once the reveal is read. OPVS keeps only the public key and the fingerprint. There is no "send me my key again" path — if you lose the key before storing it in CI, you mint a new keypair.

## Cert fingerprints

Every cert has a SHA-256 fingerprint of its public key. This is the short stable identifier OPVS uses everywhere — in the package manifest, on the admin UI, in the revocation list, in audit logs.

```
SHA256:9f8a73c1e2b4d6e8f0a1b3c5d7e9f1a3b5c7d9e1f3a5b7c9d1e3f5a7b9c1d3e5
```

A few places you'll see it:

- **In your `package.yaml`.** The `cert_fingerprint:` field on every package you publish. The marketplace cross-checks this against the cert that signed the upload. Mismatches reject.
- **On the package detail page.** The "Information" block on every package lists the cert fingerprint. Brands installing your package can verify it matches the vendor profile.
- **On the vendor profile.** Your vendor page shows all cert fingerprints ever associated with you, with their state (`active`, `revoked`, etc.) and dates.
- **In revocation events.** When a cert is revoked, the fingerprint is what gets added to the revocation list, and that's what every gateway polls for.

Fingerprints are stable across the life of the cert. They change only when you mint a new keypair.

## Where your private key actually lives

After the one-shot reveal, the only copy of your private key is the one you stored. There are a few places that copy usually goes.

**CI secrets** — the default. GitHub Actions, GitLab CI, CircleCI, whatever your release pipeline runs in. The CI runner reads the key into the `opvs-skills publish` command, which signs the bundle locally and uploads. The key never leaves the runner.

**A password manager**, for break-glass recovery. Some vendors save a copy in 1Password or Bitwarden so that if the CI secret is lost (rotation, account migration, accidental delete), the team can recover without re-minting. This is a tradeoff — one more place the key sits is one more attack surface.

**Nowhere else.** In particular, the key should not live in:

- A repository, ever, encrypted or not — public repos leak, private repos get audited.
- A laptop file outside a password manager — laptops walk off.
- A Slack DM or email body — those get indexed, screenshotted, forwarded.
- The OPVS CA — explicitly designed to forget the key after the reveal.

If you find yourself emailing your private key to a colleague, stop. Revoke and re-mint.

## Rotation

Two reasons to rotate: scheduled hygiene, and incident response.

**Scheduled rotation** has no platform-mandated cadence in Phase 1. OPVS recommends annually as a sensible default. Some vendors rotate on every major release; some rotate when a team member leaves; some rotate on a calendar. All of these are fine. The platform does not enforce a rotation date.

**Incident rotation** is fast. The flow is:

```
   You suspect compromise
        │
        ▼
   OPVS ops revokes the old cert         ◄── admin marks cert as revoked
        │                                    in vendor_certificates table
        ▼
   Revocation list updates                ◄── gateways pick this up
        │                                    within 60 seconds, fleet-wide
        ▼
   OPVS mints a new keypair               ◄── same flow as initial mint
        │
        ▼
   You receive the one-shot reveal
        │
        ▼
   You re-sign + republish current versions
        │
        ▼
   Brands auto-pick-up new versions on next install poll
```

There is no rate limit on how often you can rotate. Mint as many fresh keypairs as you need.

The key thing to understand about revocation: **gateways pick up revocations within 60 seconds**. That's the polling interval on the revocation list. Compromised keys stop being usable across the entire fleet inside a minute, not inside a day. This is by design — the marketplace optimises for fast revocation propagation over publish-time convenience.

## Multiple certs per vendor

You can have multiple active certs at the same time. The `vendor_certificates` table allows it, and the publish endpoint accepts a signature from any allowlisted public key associated with your vendor.

This is useful for:

- **Zero-downtime rotation** — mint the new cert, switch CI to it, then revoke the old one. No window where publishes fail.
- **Multi-environment signing** — one cert per CI environment (staging vs production), each with a different label.
- **Multiple humans** — one cert per release engineer, so revoking on a team change is precise.

The minimum is one. You don't need to invent reasons for more than one if you don't have a use case.

## What is intentionally not in this flow

A few things you might expect from a more mature CA system are not implemented in Phase 1.

**No public CRL endpoint.** Only gateways polling the marketplace's revocation list. External tools can't query revocation state directly today.

**No OCSP-style real-time checks.** Revocation is poll-based with a 60-second TTL. Acceptable for the threat model in Phase 1; revisited in Phase 2 if higher-assurance use cases land.

**No cert chains.** The CA is flat. Each vendor cert is signed by the OPVS CA's root key; there are no intermediates. Phase 1 has fewer than 10 vendors — a chain is overkill.

**No automated cert renewal.** No "your cert expires in 14 days, here's the new one" pipeline. Rotation is human-driven. Will likely be automated when third-party vendor sign-up opens in Phase 2.

## Quick reference — the actual endpoints

For completeness, these are the relevant API endpoints. As a vendor you don't call most of them directly — OPVS ops does — but knowing they exist makes the rest of the docs easier to follow.

| Method | Endpoint | Who calls it |
|---|---|---|
| `POST` | `/api/v1/marketplace/ca/mint` | OPVS ops, on vendor approval |
| `POST` | `/api/v1/marketplace/ca/mint-rotation` | OPVS ops, on rotation request |
| `GET` | `/api/v1/marketplace/ca/reveal/{short_code}` | You, exactly once, via the one-shot URL |
| `GET` | `/api/v1/marketplace/ca/certificates/{fingerprint}` | Anyone — public-key lookup for signature verify |
| `POST` | `/api/v1/marketplace/ca/revoke` | OPVS ops, on revocation |

The full mint + reveal flow is documented in the OPVS planning doc `MARKETPLACE-CONCEPT.md §Signing & revocation`.

## Common mistakes

A few things that have actually tripped up vendors during partner onboarding.

**Treating the reveal URL like a normal link.** It expires on first read. Opening it on your phone "to check it loaded" consumes the one-shot, and now you can't paste the key into CI. Open it on the machine you're going to copy from.

**Confusing the cert with the partner key.** They are minted in different flows, stored in different tables, and used in different parts of the publish pipeline. Re-read the top of this page if you're not sure which is which.

**Storing the private key in the repo, even encrypted.** Encrypted-in-repo is still in-repo, and Git history is forever. Use CI secrets.

**Skipping the doctor check.** Always run `opvs-skills doctor` after setting up your cert and partner key in CI. It verifies both work end-to-end before your first publish.

## Next steps

- Cert in hand? Head to [02-authentication/partner-keys.md](../02-authentication/partner-keys.md) for the HTTP API auth piece.
- Both credentials in CI? Head to [03-authoring/](../03-authoring/) to write your first skill.
- Curious about rotation cadence and what the platform considers a sensible default? [02-authentication/credential-rotation.md](../02-authentication/credential-rotation.md) goes deeper.

---

> Filled in by content session **V-B**.
