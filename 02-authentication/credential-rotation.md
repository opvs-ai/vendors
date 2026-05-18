# Credential rotation

Keys leak. People leave teams. Certs reach the end of their useful life. This page covers how to swap any credential OPVS issued to you, with zero downtime where the design allows it.

Three credentials, three rotation paths:

```
Partner key  ────  instant swap, no downtime
Signing cert ────  dual-trust window, instant in practice
Verdaccio    ────  user reissue, short downtime per publisher
```

> A note on scope. This page is about rotating credentials. Revoking a *published package version* is different and lives in [04-publishing/revocation.md](../04-publishing/revocation.md). One pulls a bundle from circulation. The other replaces a credential.

## When to rotate

Rotate on a schedule **and** on signal.

**Scheduled:**

| Credential | Default cadence | Why |
|---|---|---|
| Partner key | Every 90 days | Matches OPVS internal token-hygiene policy ([docs/planning/Ops/TOKEN-HYGIENE.md](https://github.com/opvs-ai/marketplace) in the platform repo) |
| Verdaccio user token | Every 90 days | Same policy, same reason |
| Signing cert | No fixed expiry today | Cadence is set by OPVS as the trust tier matures — TBD per vendor |

**On signal — rotate immediately when:**

- A team member with access to the credential leaves
- A credential appears in any log, screenshot, transcript, or chat thread (even your own)
- A laptop or CI runner with the credential on disk is lost, stolen, or decommissioned
- A `git push` lands a credential in source — even if you force-push the history afterwards
- OPVS sends you a notice that your credential is implicated in a wider incident

If you are unsure whether something counts as a leak, treat it as one. Rotation is cheap. Investigation after a real leak is not.

## Rotating a partner key

This is the simplest rotation. The flow is mint-new → switch CI → revoke-old, in that order, and it is zero-downtime if you keep the order.

```
1. Ask OPVS admin to mint a new key (label: "rotation 2026-08-15, replaces <prefix>")
2. Admin clicks Mint, copies raw key into secure channel, hands it to you
3. You paste the new key into your CI secret (OPVS_MARKETPLACE_ADMIN_KEY)
4. You trigger a test publish OR doctor call to confirm the new key works
5. Ask OPVS to revoke the old key by prefix
6. Done — next publish uses the new key end-to-end
```

The reason we mint *before* revoking: a partner key is HTTP-API auth for the live registry. Revoking before the new key is in CI means any concurrent publish job fails with `403 partner-key invalid`. With this order, there is always at least one valid key in your possession.

If you have multiple CI runners or environments (staging, prod), update them all between steps 3 and 4, and do one combined test publish from each environment before asking for revocation.

**Self-service:** not yet shipped. Today the mint and the revoke both go through an OPVS admin. Self-service is on the roadmap.

**HTTP endpoints** (admin-side reference, not for vendor use):

```
POST /api/v1/marketplace/vendors/{vendor_id}/keys
GET  /api/v1/marketplace/vendors/{vendor_id}/keys
POST /api/v1/marketplace/vendors/{vendor_id}/keys/{key_id}/revoke
```

These require `ADMIN_API_KEY`, not your partner key. They are documented here so you know the surface; you cannot call them yourself.

## Rotating a signing cert

The signing cert is the ed25519 keypair from [01-onboarding/apply-for-a-cert.md](../01-onboarding/apply-for-a-cert.md). It signs your package bundles at publish time. Rotation runs through a separate endpoint that swaps the *active* key for your vendor while keeping the previous fingerprint trusted long enough for in-flight installs to verify.

```
1. Ask OPVS admin to call /api/v1/marketplace/ca/mint-rotation for your vendor
2. Admin receives the new private key (one-shot reveal) and hands it to you
3. You replace OPVS_SIGNING_KEY (path to file on disk) with the new key
4. You publish a fresh package version signed by the new key
5. OPVS marks the old fingerprint as superseded; brands installing future
   versions verify against the new fingerprint
6. The old fingerprint stays in the allowlist as "trusted, superseded" so
   already-installed packages keep verifying — they don't break
```

The **dual-trust window** is the period between step 5 and the day the old fingerprint is dropped. During this window, both your old and new fingerprints verify successfully. Brands that installed a package signed by the old key are unaffected. Brands installing your *latest* version verify against the new key.

The window length is a platform-level operational decision (defaults to "until no live install references the old fingerprint"). OPVS notifies you before dropping any old fingerprint from the allowlist.

**Endpoint (admin):**

```
POST /api/v1/marketplace/ca/mint-rotation
Body: {"vendor_id": "<your-vendor-uuid>", "deliver": "direct"}
```

`deliver: "direct"` returns the new private key in the response body. `deliver: "invite"` mints a `credential_invites` row instead and sends an email link so you can claim the key without it crossing an admin's screen. For production rotations, prefer the invite mode — it leaves less of a trail.

**Difference from mint:** `/ca/mint-rotation` returns `422` if your vendor has no active cert to rotate from. Use `/ca/mint` for the first cert; use `/ca/mint-rotation` for every cert after that.

## Rotating a Verdaccio user

The `@vendor-publisher` user account on `npm.opvs.ai` is provisioned by OPVS staff via the `htpasswd` plugin. Rotation is:

```
1. Ask OPVS to invalidate your current Verdaccio user token
2. OPVS reissues a fresh token using `npm adduser` on the registry
3. You update your CI ~/.npmrc with the new _authToken
4. Subsequent npm publishes use the new token
```

This rotation has a small downtime window — between invalidation and the new token being in CI, your publish jobs will fail with `401 Unauthorized` from Verdaccio. Plan rotations for low-traffic windows, or use a second Verdaccio user as a rolling spare if your publish cadence is critical.

The Verdaccio side cannot do dual-trust like the signing cert. There is only one current valid token per user. The mitigation is to schedule the swap during a quiet hour, or coordinate with OPVS to mint a second user account temporarily.

## If a key leaks

This is the emergency path. Run it the moment a leak is plausible, not after you have confirmed it.

```
Minute 0  ──  Notify OPVS via your existing escalation channel
              Subject: "Partner-key leak suspected — vendor <name>"
              Include: which credential, when you noticed, blast radius
              if known (logs? git? screenshot?)
Minute 5  ──  OPVS revokes the credential. Next API call carrying it
              gets 403. Effect is instant — no cache, no delay.
Minute 10 ──  OPVS mints a replacement, hands it to you via the same
              secure channel as the original onboarding flow
Minute 15 ──  You install the replacement in CI/secrets manager,
              trigger a verification call (doctor or test publish)
Day 0     ──  OPVS provides an audit report on activity attributed to
              the leaked credential since its issue date
Day 1+    ──  Joint root-cause review if the leak involved a third
              party, your CI provider, or a published package
```

Two things to avoid in a panic:

- **Do not delete the credential file or env var before notifying OPVS.** Revocation has to happen on the OPVS side to actually invalidate it. Removing it from your side just stops *you* from using it.
- **Do not force-push to clean git history before notifying OPVS.** Force-pushes can clear the credential from the visible tree but leave it in reflogs, mirrors, CI build caches, and GitHub's API cache. Rotation is the only real recovery.

## Expiry policy

Partner keys do not expire by default — they live until revoked. The 90-day scheduled rotation cadence above is operational hygiene, not enforced by the registry.

Signing certs also do not have a fixed expiry today. The cadence is set by OPVS per vendor as your trust tier evolves. We notify you ahead of any forced rotation.

If either of these policies changes, OPVS will update this page and email all vendors with at least 30 days lead time.

## Auditing rotations

Every rotation event is logged on the OPVS side. For partner keys, the `marketplace_partner_keys` table preserves `created_at`, `last_used_at`, `revoked_at`, and `revoked_reason` indefinitely — even revoked keys keep their row. For cert rotations, the `ca_certificates` table records each mint and the fingerprint of the cert it superseded.

To request an audit report, email your OPVS account owner with:

- Your vendor name and ID
- The time window you care about
- What you want included (mint events, revoke events, last-used timestamps, publish events attributed to a specific key prefix)

OPVS turns these around within one business day. The platform-level audit log is append-only; nothing in it can be edited or deleted by anyone, including OPVS staff.
