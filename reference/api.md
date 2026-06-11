# HTTP API reference

The on-the-wire surface. The [CLI](cli.md) wraps most of this, so day to day you
won't call these directly — but they're here for CI integrations, one-off curl,
and understanding what the CLI does under the hood.

Every endpoint below is traceable to code:
[`registry.py`](https://github.com/opvs-ai/OPVS/blob/main/apps/api/app/api/v1/registry.py)
(public reads + review writes),
[`marketplace.py`](https://github.com/opvs-ai/OPVS/blob/main/apps/api/app/api/v1/marketplace.py)
(admin writes),
[`credential_invites.py`](https://github.com/opvs-ai/OPVS/blob/main/apps/api/app/api/v1/credential_invites.py)
(invites). Nothing here is invented — if it's not in those files, it's not on
this page.

---

## Hosts and auth

```
market.opvs.ai   registry — public reads, no auth
app.opvs.ai      api-gateway — /marketplace/* (admin), /registry/* writes (session),
                 /skills/*/doctor, /credential-invites/*
localhost:8600   marketplace service direct — the CLI publishes here
```

| Auth | Header / mechanism | Used on |
|---|---|---|
| none | — | registry public reads |
| partner key | `X-Admin-Key: <key>` | `/marketplace/*` admin writes (proxy attaches the server key) |
| brand session | cookie session (+ brand pin) | ratings, reviews, install |
| admin + brand | `X-Admin-Key` + `X-Brand-ID` | doctor (service-to-service) |

The gateway `/marketplace/*` routes are gated by `require_admin_or_session` — a
partner key **or** a platform-admin session.

---

## Registry — public reads (`/api/v1/registry`)

No auth. Served from `market.opvs.ai`.

| Method | Path | Returns |
|---|---|---|
| GET | `/packages` | published package list (filter `vendor` / `category` / `tier`) |
| GET | `/search?q=` | full-text + vector search with usage telemetry |
| GET | `/packages/{vendor}/{name}` | package detail (score, `avg_rating`, `rating_count`) |
| GET | `/packages/{vendor}/{name}/versions` | version history |
| GET | `/packages/{vendor}/{name}/{version}` | one version |
| GET | `/packages/{vendor}/{name}/{version}/changelog` | changelog for a version |
| GET | `/packages/{vendor}/{name}/reviews` | public reviews (hidden filtered, newest first) |
| GET | `/manifest` | full manifest dump (gateway revocation-poll) |

```bash
curl -s "https://market.opvs.ai/api/v1/registry/packages/@opvs-ai/agentboard/versions"
```

---

## Registry — brand-scoped writes (`/api/v1/registry`)

Brand session required. Served from `app.opvs.ai`.

| Method | Path | Notes |
|---|---|---|
| POST | `/packages/{package_id}/ratings` | star `1–5`, upsert, 30/hr/brand, no install gate → `201` |
| POST | `/packages/{package_id}/reviews` | prose, install-gated (else `403`), 10/hr/brand → `201` |
| PATCH | `/packages/{package_id}/reviews/{review_id}` | edit own review, 3/hr/user |

See [../06-promotion/ratings-and-reviews.md](../06-promotion/ratings-and-reviews.md).

---

## Marketplace admin (`/api/v1/marketplace`)

`require_admin_or_session`. Served from `app.opvs.ai`. The proxy forwards to the
marketplace service and attaches the server-side admin key.

**Publish + lifecycle**

| Method | Path | Notes |
|---|---|---|
| POST | `/packages` | publish a signed package → `201` (8-check review, `422` on block) |
| POST | `/packages/{package_id}/revoke` | tombstone a version |
| POST | `/security/review` | run the 8 checks without publishing |
| GET | `/stats` | platform counts + 24h publish rate + revocations |

**Certificates (CA)**

| Method | Path | Notes |
|---|---|---|
| POST | `/ca/mint` | mint an ed25519 keypair (one-shot reveal) → `201` |
| POST | `/ca/mint-rotation` | mint a rotation key → `201` |
| POST | `/ca/certificates/{certificate_id}/revoke` | revoke a cert |
| GET | `/ca/certificates` | list certs |
| GET | `/ca/certificates/{fingerprint}` | public-key lookup |

**Vendors + partner keys**

| Method | Path | Notes |
|---|---|---|
| POST | `/vendors` | seed a vendor namespace → `201` |
| GET | `/vendors` · `/vendors/{vendor_id}` | list / fetch |
| POST | `/vendors/{vendor_id}/keys` | mint a partner key → `201` |
| GET | `/vendors/{vendor_id}/keys` | list partner keys |
| POST | `/vendors/{vendor_id}/keys/{key_id}/revoke` | revoke a partner key |

**Marketing overlay + moderation**

| Method | Path | Notes |
|---|---|---|
| PATCH | `/packages/{package_id}/marketing` | edit overlay fields |
| POST | `/packages/{package_id}/media` | upload hero/screenshot → `201` |
| DELETE | `/media` | remove a media asset |
| PATCH / DELETE | `/reviews/{review_id}/hide` | hide / un-hide a review |
| POST | `/reviews/{review_id}/vendor-reply` | admin-mediated vendor reply → `201` |

**Brand installs**

| Method | Path | Notes |
|---|---|---|
| POST | `/packages/{package_id}/install` | install onto a brand → `201` |
| GET | `/installed` · `/installed/{install_id}` | list / fetch installs |
| PATCH | `/installed/{install_id}/entitlements` | grant / revoke entitlements |
| PATCH | `/installed/{install_id}/agents` | per-agent opt-out → `204` |
| DELETE | `/installed/{install_id}` | uninstall → `204` |

---

## Doctor (`/api/v1/skills`)

| Method | Path | Auth |
|---|---|---|
| GET | `/skills/{skill_id}/doctor` | session (brand auto-pinned) **or** `X-Admin-Key` + `X-Brand-ID` |

Returns `installed`, `entitlements` {required/granted/missing}, `credentials`,
`upstream` {status/latency/last_health_check}, and `ready: bool`. Fails open —
no telemetry gives `upstream.status="unknown"`, not `ready:false`.

---

## Credential invites (`/api/v1/credential-invites`)

The token-IS-auth flow for collecting a brand's credentials. See
[../02-authentication/](../02-authentication/).

| Method | Path | Notes |
|---|---|---|
| POST | `` | create an invite (returns a `/contribute/{token}` URL) |
| GET | `` | list invites |
| GET | `/{invite_id}` | invite detail |
| POST | `/{invite_id}/resend` | resend |
| POST | `/{invite_id}/cancel` | cancel |

The public `/contribute/{token}/*` path takes **no auth** — the token is the
auth. 10/brand/24h, 72h token expiry.

---

## Error shape

A blocked publish returns `422` with the security-review findings — `severity`,
dotted `code`, `message`, optional `path`. See
[../04-publishing/security-review.md § Reading a 422](../04-publishing/security-review.md#section-4--reading-a-422).

---

## See also

- [cli.md](cli.md) — the CLI that wraps these endpoints.
- [../04-publishing/security-review.md](../04-publishing/security-review.md) —
  what gates the publish endpoint.
- [glossary.md](glossary.md) — terms used above.
