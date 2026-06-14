# Revocation

A published version is immutable — you can't edit it, and you can't take it
back by re-uploading. When a version is dangerous (a bug, a leaked credential,
a wrong entitlement that shipped), the way out is **revocation**: a one-way
state flip that tombstones that version so brands can no longer install it, and
brands already on it lose access within ~60 seconds.

This page covers when to revoke, who can, what the flip actually does, the
60-second propagation window, and how to communicate it.

---

## Section 1 — Revoke vs. delete vs. deprecate

Three different things, often confused. Pick the right one.

| Action | What it does | Reversible | When |
|---|---|---|---|
| **Revoke** | tombstone a version; existing installs error out | no | a shipped version is unsafe or wrong |
| **Delete** | physically remove the row | n/a | almost never — orphans brand installs |
| **Deprecate** | soft-flag "prefer the newer one," installs keep working | — | superseded but harmless |

**Delete is not the tool.** Brands install a specific version, and the server
records it on their `installed_packages` row. Deleting the registry row behind
a live install leaves that brand pointing at nothing. The platform does not
expose a vendor-facing delete for this reason. If a version is bad, you
**revoke** it — that's the designed path, and it leaves the audit trail intact.

Deprecation is the *soft* counterpart and is not a shipped platform feature
today — see [../05-editing-and-versioning/deprecation.md](../05-editing-and-versioning/deprecation.md)
and [versioning.md § Deprecating a version](versioning.md#section-3--deprecating-a-version).
Until it ships, your two real levers are *publish-a-new-version* (soft) and
*revoke* (hard).

---

## Section 2 — When to revoke

Revoke is disruptive — every brand on that version breaks. Reserve it for cases
where breaking them is **better than leaving them running**:

- **A security issue** — a leaked key in the bundle, an over-broad entitlement
  that shipped, a tool that does more than it declared.
- **A correctness bug bad enough that running is worse than stopping** — data
  corruption, calls hitting the wrong upstream, silent failures.
- **A mistaken publish** — wrong version number, wrong tier, a draft that
  escaped.

If the new version is simply *better* and the old one is *harmless*, do **not**
revoke. Publish the new version, announce it, and let brands move at their own
pace (they reinstall to pick it up). Revocation is for "get off this now," not
"please upgrade."

**Revoke vs. MUST_UPGRADE.** Revoke *stops* brands — they break and must
reinstall by hand. [MUST_UPGRADE](must-upgrade.md) *carries* them — on the next
auto-update pass every install on the bad version is force-moved forward to the
newest version their pin allows, with no breakage. The deciding question is
whether a safe version exists to move brands *to*. If yes, and running the bad
one a little longer is tolerable, prefer MUST_UPGRADE. Reach for revoke only when
there's no safe landing spot, or when running the bad version even briefly is
unacceptable.

---

## Section 3 — Who can revoke

Two parties:

- **You (the vendor)** — within your own scope. You can revoke any version of a
  package under your vendor slug.
- **Platform operators** — anything, including your packages, in a safety
  emergency.

The manifest's `revocable_by` field lists who's authorised beyond the defaults
(your vendor slug + `opvs-operator`). See
[packaging.md § The rest](packaging.md#the-rest-optional-with-defaults).

---

## Section 4 — How to revoke

One call, against the api-gateway marketplace proxy on port 3000:

```bash
curl -X POST \
  "https://app.opvs.ai/api/v1/marketplace/packages/{package_id}/revoke" \
  -H "X-Admin-Key: $OPVS_MARKETPLACE_ADMIN_KEY" \
  -H "Content-Type: application/json" \
  -d '{"reason": "leaked credential in bundle — superseded by 1.2.1"}'
```

The proxy forwards to the registry's
`POST /api/v1/registry/packages/{package_id}/revoke` and attaches the
server-side admin key, so your partner key never reaches the registry directly.
The route is gated by `require_admin_or_session` — a partner key with revoke
rights on your scope, or a platform-admin session.

Find the `package_id` from the registry detail read:

```bash
curl -s "https://market.opvs.ai/api/v1/registry/packages/@your-vendor/your-skill" \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['id'])"
```

---

## Section 5 — What the flip does (and the 60-second window)

Revoke flips registry state to `revoked`. Two things follow:

```
revoke call
   │
   ├─► registry: version state → 'revoked'     (immediate — no new installs)
   │
   └─► per brand on that version:
         installed_packages.state → 'disabled'  (the authority)
              │
              ▼
         gateway grant-cache TTL expires (≤60s)
              │
              ▼
         agent calls to that skill stop resolving
```

Two facts worth pinning:

- **`installed_packages.state = 'disabled'` is the authority**, not anything on
  the gateway. The gateway is a cache.
- **Propagation is bounded by the gateway's 60-second grant-cache TTL.** A
  revoked package keeps resolving for up to a minute on already-warm gateways,
  then stops. Don't expect instantaneous cutoff; do expect it within a minute.

New installs are blocked the instant the registry state flips — the ≤60s window
only applies to brands that *already* had it warm.

---

## Section 6 — What brands see, and how to tell them

A brand on a revoked version falls into an **error state**: the package shows
`disabled`, [doctor](../reference/api.md) reports it unhealthy, and agent tool
calls to its skills stop resolving. There is no automatic migration — the brand
must install a fixed version to recover.

So revocation without communication strands your users. Do all three:

1. **`CHANGELOG.md`** — a clear entry on the new version: what was wrong, which
   version is safe, what to do.
2. **A direct customer notice** — email or the marketing overlay long
   description, telling installed brands to reinstall the fixed version.
3. **Ship the fix first.** Publish the good version *before* you revoke the bad
   one, so brands have somewhere to land the moment they break.

The healthy sequence is: **publish fix → announce → revoke bad version.** Never
revoke first and scramble for a fix while brands are down.

---

## See also

- [must-upgrade.md](must-upgrade.md) — the *soft* forced retire: carry brands
  forward to a fix instead of breaking them.
- [versioning.md](versioning.md) — why versions are immutable; publish-new as
  the soft alternative to revoke.
- [../05-editing-and-versioning/deprecation.md](../05-editing-and-versioning/deprecation.md)
  — the soft retire path (planned).
- [../02-authentication/credential-rotation.md](../02-authentication/credential-rotation.md)
  — rotating a *key* is a different operation from revoking a *version*.
- [../reference/api.md](../reference/api.md) — the revoke endpoint in the full
  API table.
