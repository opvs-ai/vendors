# Signing and submit

How the bytes leave your laptop and arrive on `market.opvs.ai`.

By the end you'll have run `opvs-skills publish` and seen your package live in the
registry. Everything here is the real flow — no placeholders.

---

## What "publish" actually does

One command (`opvs-skills publish`) runs six steps. Knowing them makes the
failures obvious:

```
1. VALIDATE     strict manifest check (mirrors the server's Pydantic schema,
                so --dry-run catches what the registry would reject)
2. KEY          load your ed25519 private key → derive its cert fingerprint
3. SIGN         tar the package → SHA-256 the tarball → ed25519-sign the digest
4. UPLOAD-URL   POST /api/v1/bundles/upload-url  → a pre-signed R2 PUT URL
5. PUT          upload the tarball to R2 at that URL
6. REGISTER     POST /api/v1/registry/packages  with the signed manifest +
                bundle_url + sha256
```

After step 6 the registry verifies your signature, checks your cert is on the
allowlist, validates the manifest server-side, and runs the security review. If
all pass, the row is published and brands can install it.

```
  your machine                         market.opvs.ai
  ────────────                         ──────────────
  validate ─ sign ─┐
                   ├─ upload-url ─────► (pre-signed R2 URL)
                   ├─ PUT tarball ────► R2 bucket
                   └─ POST package ───► registry
                                          ├─ verify ed25519 signature
                                          ├─ check cert on allowlist
                                          ├─ validate manifest (Pydantic)
                                          ├─ run 8-check security review
                                          └─ published ✅  (or 422 + findings)
```

---

## Prerequisites

- **A vendor identity + cert.** See [01-onboarding](../01-onboarding/). You have
  a vendor slug (e.g. `di-atomic`) and your ed25519 keypair is minted + on the
  allowlist.
- **Your private key, stored safely.** See [02-authentication](../02-authentication/).
  Never commit it. Reference it by path or `@env:VAR`.
- **Your partner key** (the HTTP admin key that gates publish). Also from
  [02-authentication](../02-authentication/).
- **The CLI.** Two ways:
  ```bash
  # internal (Verdaccio):
  npm install -g @opvs-ai/skills --registry https://npm.opvs.ai

  # external vendors (build from the public source mirror):
  git clone https://github.com/opvs-ai/opvsHUB.git
  cd opvsHUB/cli && npm install && npm run build && npm link
  opvs-skills --help
  ```
- **A package directory** authored in the [4-layer format](4-layer-and-envelope.md)
  with a valid `package.yaml` (manifest).

---

## Step 1 — Preflight with `--dry-run`

Always dry-run first. It validates the manifest with the **same** strict checks
the server runs, signs locally, and makes **zero** network calls. No partner key
needed.

```bash
opvs-skills publish ./my-package --dry-run \
  --private-key @env:DI_ATOMIC_SIGNING_KEY
```

Green dry-run → the manifest shape is correct and the bundle signs. Red →
fix the reported field before you touch the network. (Common messages: see
[Common 422s](#common-422s) below.)

> Tip: `opvs-skills validate` is the even-cheaper cross-tier consistency check —
> run it while authoring; run `--dry-run` right before publishing.

---

## Step 2 — Real publish

```bash
opvs-skills publish ./my-package \
  --private-key @env:DI_ATOMIC_SIGNING_KEY \
  --api-url https://market.opvs.ai \
  --admin-key @env:DI_ATOMIC_PARTNER_KEY
```

Flags:

| Flag | What | Notes |
|---|---|---|
| `--private-key` | your ed25519 signing key | path or `@env:VAR`. Used to sign the digest; the registry verifies against your allow-listed cert. |
| `--api-url` | registry base | `https://market.opvs.ai` for prod. |
| `--admin-key` | your **partner key** | gates `/bundles/upload-url` + `/registry/packages`. This is your HTTP auth, **not** the platform master key. |
| `--dry-run` | preflight only | validate + sign, no upload. |

On success the CLI prints the `bundle_sha256`, `bundle_url`, and
`cert_fingerprint`. The package is live.

---

## Step 3 — Verify it's live

```bash
# it's in the registry at the new version:
curl -s "https://market.opvs.ai/api/v1/registry/packages/@di-atomic/my-package" \
  | python3 -m json.tool | grep -E '"version"|"status"|"tier"|"security_score"'
# expect: status=published, your new version, tier as declared, score 100

# brand-side health check (after a brand installs it):
opvs-skills doctor @di-atomic/my-package
```

---

## Versioning — you cannot republish a version in place

A published `(vendor, name, version)` is **immutable**. To ship a change, bump
the version in `package.yaml` (SemVer) and publish again. `0.1.0 → 0.1.1`
(fix), `→ 0.2.0` (feature), `→ 1.0.0` (stable). The old version stays installed
for brands that have it until they upgrade. See [versioning.md](versioning.md).

## Tier — set in the manifest, not on the CLI

`tier` is a manifest field (`internal` / `beta` / `community` / `verified`).
`internal` is hidden from the customer marketplace browse; **`beta` is what makes
a package visible + installable** (where every live `@opvs-ai` / `@spideriq`
package sits). New versions inherit the tier you declare. See
[../06-promotion](../06-promotion/).

---

## What the registry checks (and how it can reject you)

After upload, before "published":

1. **Signature verify** — your ed25519 signature over the tarball digest must
   verify against your allow-listed cert. Mismatch → rejected.
2. **Cert allowlist** — your `cert_fingerprint` must be active (not revoked /
   superseded). See [01-onboarding/apply-for-a-cert.md](../01-onboarding/apply-for-a-cert.md).
3. **Manifest validation** — server-side Pydantic, stricter than you'd guess
   (see below).
4. **Security review** — 8 automated checks. A blocking finding → `422` with the
   findings listed. See [security-review.md](security-review.md).

### Common 422s

| Message (paraphrased) | Cause | Fix |
|---|---|---|
| `manifest missing required fields` | one of name/version/description/vendor/category/license absent | add it |
| `manifest.name scope must match "@"+vendor` | `name: "@acme/x"` but `vendor: di-atomic` | make them agree |
| `manifest.version must be valid SemVer` | e.g. `v1` or `0.1` | use `0.1.0` |
| `must declare at least one entry in skills/inbound_events/peer_dependencies` | empty package | declare your skill(s) |
| `provider_config` required | omitted it | **keep it even for guidance-only skills** — the server requires it; a guidance skill keeps a placeholder `provider_config` + a schema.yaml with **no `methods:`**. See [4-layer-and-envelope.md](4-layer-and-envelope.md). |
| `entitlements.unknown` | a `<skill>.<surface>.<verb>` entitlement | external skills use the parametric `vendor.<vendor>.<surface>.<verb>` form (exactly 3 segments after `vendor.`) |
| security-review blocking finding | see [security-review.md](security-review.md) | address the named check |

---

## The one rule that breaks skills silently

A `BACKED` skill (one that declares `methods:` in its `schema.yaml`) **must have a
real endpoint serving each method** — otherwise every `skill_call` 404s at
runtime even though publish succeeds. If your skill's value is methodology rather
than a callable API, ship it **guidance-only**: zero `methods:`, rich
`references/` recipes, reached via `skill_read_reference`. Never declare a method
you don't serve. Full detail: [4-layer-and-envelope.md](4-layer-and-envelope.md)
and [../03-authoring/quality-and-enforcement.md](../03-authoring/quality-and-enforcement.md).

---

## Next

- [versioning.md](versioning.md) — semver, deprecation, what's mutable after publish
- [security-review.md](security-review.md) — the 8 checks, finding codes, how to pass
- [revocation.md](revocation.md) — pulling a bad version
- [../06-promotion](../06-promotion/) — tier promotion, marketing overlay, ratings
