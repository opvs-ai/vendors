# Required upgrades (MUST_UPGRADE)

Sometimes a published version isn't just *superseded* — it's *bad*, and you need
every brand off it. But you don't want to strand them. **MUST_UPGRADE** is the
tool for that: you flag a version, and on the next auto-update pass every install
sitting on it is moved **forward** to the newest version their pin allows —
automatically, regardless of how tightly they pinned.

It's the forward-moving counterpart to [revocation](revocation.md). Revoke
*stops* brands (they break and must reinstall by hand). MUST_UPGRADE *carries*
them (they move to your fix without lifting a finger).

This page covers when to use it, the one call that sets it, who's allowed, what
brands experience, and how it differs from revoke and from a plain new release.

---

## Section 1 — MUST_UPGRADE vs. revoke vs. publish-newer

Three ways to retire a version. Pick by *what you want to happen to brands on it*.

| Action | What happens to installs on the old version | They keep working? | When |
|---|---|---|---|
| **Publish a newer version** | nothing until they choose (or their pin auto-bumps them) | yes | the new one is simply better; the old one is harmless |
| **MUST_UPGRADE** | force-moved to the newest allowed version on the next pass | yes — on the fix | a shipped version is wrong, but a safe newer version exists to land them on |
| **Revoke** | disabled; skill calls stop resolving within ~60s | no | the version is dangerous and stopping is better than running, even briefly |

The deciding question: **is there a safe version to move brands *to*?**

- **Yes, and running the bad one a little longer is tolerable** → MUST_UPGRADE.
  Brands glide to the fix on the next auto-update pass, no breakage, no manual
  reinstall.
- **No safe landing spot, or running it even briefly is unacceptable** →
  [revoke](revocation.md). Brands break immediately; that's the point.

MUST_UPGRADE needs a newer version to exist. **Publish the fix first**, then flag
the bad one — exactly like the healthy revoke sequence.

---

## Section 2 — When to set it

MUST_UPGRADE is mild compared to revoke (nobody breaks), but it still overrides
the brand's explicit pin — so reserve it for "this version should not keep
running" cases:

- **A correctness bug** that a newer version fixes — wrong output, a
  mis-mapped upstream, a silent failure.
- **A behaviour change you need universal** — a corrected entitlement, a fixed
  default, a security hardening that every brand should have.
- **A deprecation you want enforced, not suggested** — the old version still
  *works*, but you don't want anyone left on it.

If the new version is just *nicer* and the old one is fine, don't flag it. Brands
on `^`/`~`/`latest` pins pick up minor and patch releases on their own; only
brands pinned to an exact old version stay put, and that's their choice to make.
MUST_UPGRADE is for overriding that choice when staying put is a problem.

---

## Section 3 — Who can set it

Two parties, same scope rules as revoke:

- **You (the vendor)** — within your own scope. A partner key may flag any
  version of a package under your vendor slug. Flagging someone else's package
  returns `403`.
- **Platform operators** — anything, via the platform admin key, in a safety or
  support situation.

The package's vendor is looked up **before** any state changes, so a denied
request never touches the flag.

---

## Section 4 — How to set it

One call against the registry on `market.opvs.ai`, authenticated with your
**partner key** (the same HTTP admin key you publish with — not the platform
master key):

```bash
curl -X POST \
  "https://market.opvs.ai/api/v1/registry/packages/{package_id}/must-upgrade" \
  -H "X-Admin-Key: $YOUR_VENDOR_PARTNER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"reason": "1.3.0 mis-handles empty result sets — 1.3.1 fixes it"}'
```

`reason` is **mandatory** (1–1024 characters). It's audited on every flip and
surfaced to brand admins, so a forced upgrade is never silent. Write it for the
brand admin who'll read it: what was wrong, which version is safe.

Find the `package_id` from the registry detail read:

```bash
curl -s "https://market.opvs.ai/api/v1/registry/packages/@your-vendor/your-skill" \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['id'])"
```

The endpoint is dual-auth (`require_admin_or_partner`): your partner key on your
own scope, or a platform-admin key on anything. Responses you'll see:

- `200` — flag set; the package detail comes back with `must_upgrade: true`.
- `403` — partner key scope mismatch (not your package).
- `404` — no such package.
- `409` — the version is already **revoked**; you can't force-upgrade *onto* a
  tombstoned version. (Flagging a *different, live* version is fine.)

Platform admins and the dashboard can also reach the same operation through the
api-gateway proxy at
`https://app.opvs.ai/api/v1/marketplace/packages/{package_id}/must-upgrade`,
which attaches the server-side admin key. Vendors hit the registry directly, the
same way you publish.

---

## Section 5 — What the flip does, and what brands experience

Setting the flag records `must_upgrade` (plus your `reason` and a timestamp) on
the registry version, and emits a `package.must_upgrade` webhook event (the
mirror of `package.published`). Brands subscribed to webhooks hear about it
immediately; everyone else is caught by the next poll.

```
must-upgrade call
   │
   ├─► registry: version flagged must_upgrade = true   (+ reason, + timestamp)
   │
   ├─► package.must_upgrade webhook fired               (instant, for subscribers)
   │
   └─► next auto-update pass (daily 03:00 UTC, or on api-gateway boot):
         every install on the flagged version
              │
              ▼
         moved to the newest version its pin allows
         — pin is OVERRIDDEN; an exact pin does not block a required upgrade
              │
              ▼
         the move is recorded in the brand's auto-update log
```

Two things to know about how brands feel it:

- **The pin is overridden.** A brand pinned to the exact bad version is still
  moved forward. That's the whole point — MUST_UPGRADE is how you reach the
  brands who would otherwise stay put.
- **It's not instant.** The forced move lands on the next scheduled pass (daily,
  plus whenever an api-gateway restarts). Brands with webhooks can react sooner;
  there's no hard cutoff like revoke's 60-second window.

### Entitlements on a forced upgrade

If the version you're moving brands *to* needs a permission the brand hadn't
granted, a **required** upgrade grants it automatically and records it in the
audit log — the upgrade is mandatory, so the permission comes with it. (A
brand's *voluntary* upgrade in the same situation is instead held until the admin
consents.) So be deliberate: only flag MUST_UPGRADE when you genuinely need every
brand on the new version, permissions included.

---

## Section 6 — Communicate it anyway

Brands don't break, but they *do* get moved off a version they chose — and the
target may pull in a new permission. Don't let the `reason` field be the only
notice:

1. **`CHANGELOG.md`** on the new version — what was wrong, why the upgrade is
   required, what (if anything) changes for them.
2. **A direct customer notice** for anything that alters behaviour or grants a
   new permission — the audit trail is for you; the heads-up is for them.

The healthy sequence mirrors revoke: **publish the fix → announce → flag the old
version MUST_UPGRADE.** Never flag a version with nowhere safe to land.

---

## See also

- [revocation.md](revocation.md) — the *hard* retire: stop brands instead of
  carrying them forward.
- [versioning.md](versioning.md) — pins (`1.0.0` / `~1.0.0` / `^1.0.0` /
  `latest`) and how brands choose the pace MUST_UPGRADE overrides.
- [signing-and-submit.md](signing-and-submit.md) — your partner key and the
  registry base URL, reused here.
