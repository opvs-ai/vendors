# Security review

Every publish runs through a static security review before the registry accepts
it. The review is **eight checks** run in a fixed order. If any check emits a
*blocking* finding, the publish fails with a `422` and a list of finding codes;
nothing reaches the registry. If every check passes, the package is accepted
with a score of `100`.

This page lists all eight checks, what each one catches, how the score works,
and how to read and fix a `422`. The authority for everything here is the check
registry at
[`security_checks/__init__.py`](https://github.com/opvs-ai/OPVS/blob/main/apps/spideragent/packages/marketplace/api/app/services/security_checks/__init__.py)
and the orchestrator at
[`security_review.py`](https://github.com/opvs-ai/OPVS/blob/main/apps/spideragent/packages/marketplace/api/app/services/security_review.py).

---

## Section 1 — The eight checks, in order

The order matters: `manifest_schema` runs first and short-circuits the pipeline
on a blocking schema error, because every later check reads the typed manifest
it produces.

| # | Check | Catches | Severity on failure |
|---|---|---|---|
| 1 | `manifest_schema` | `package.yaml` doesn't validate against the manifest schema | **blocking** |
| 2 | `signature_verify` | ed25519 signature doesn't match your active cert | **blocking** (warn if skipped) |
| 3 | `meta_coherence` | package declares no skills, no events, no peers — nothing to install | **blocking** |
| 4 | `entitlement_lint` | an entitlement isn't known, or `network.egress.public` is over-broad | **blocking** |
| 5 | `blocklist` | raw `shell(` / `eval(` / `exec(` in bundle source | **blocking** |
| 6 | `auth_scheme` | a credential's `auth_type` isn't on the vendor allowlist | **blocking** |
| 7 | `rate_limits` | an outbound HTTP method declares no rate limit | blocking / warning by tier |
| 8 | `required_docs` | `README.md` / `SKILL.md` / `CHANGELOG.md` missing from the bundle | **blocking** |

---

## Section 2 — What each check actually does

**1. `manifest_schema`** — runs your `package.yaml` through the same
`PackageManifest` validator that
[packaging.md](packaging.md) documents field by field. A failure here stops the
whole review; the other checks can't run without a valid manifest. Finding code:
`manifest.schema`, with the failing field path attached.

**2. `signature_verify`** — looks up your vendor's active certificate and
verifies the ed25519 signature against the bundle hash. It distinguishes three
cases so you can self-diagnose: unknown vendor, revoked cert, bad signature. A
**dev-mode** publish that omits both `signature` and `bundle_sha256` doesn't
block — it emits a `signature.skipped` **warning** so the admin knows the check
didn't run against a real signature.

**3. `meta_coherence`** — blocks the one degenerate shape: `skills: []` **and**
`inbound_events: []` **and** `peer_dependencies: []`. A package with none of the
three does nothing and wastes a registry slot. Finding code:
`meta.empty_useless`. (Declaring peers with no skills is fine — that's a valid
meta-package.)

**4. `entitlement_lint`** — every required/optional entitlement must be a known
name: in the fixed registry, or a parametric prefix
(`network.egress.provider.<name>`, `credits.consume.<name>`, or the vendor-scoped
`vendor.<vendor>.<surface>.<verb>`). It also flags `network.egress.public` when
all your skills' outbound URLs share a single host — the narrower
`network.egress.provider.<host>` would fit, so the broad grant is rejected as
too-broad.

**5. `blocklist`** — scans the text files in the bundle (`.ts`, `.js`, `.py`,
`.yaml`, `.md`) for literal `shell(`, `eval(`, and `exec(` calls. These hard
block. `exec(` is allowed **only** if the package declares an `exec.*`
entitlement (an explicit audit signal). Attribute-shaped `.shell*(` access is a
soft **warning**, not a block.

**6. `auth_scheme`** — your credential `auth_type` values must be on the
vendor-approved allowlist: `pat`, `api_key`, `oauth`, `oauth_pkce`,
`hmac_sha256`. The enum also contains `device_code` and `cert_invite`, but those
are reserved for internal flows and **block** on a vendor submission. Finding
code: `credentials.auth_type_unknown`.

**7. `rate_limits`** — walks each skill's `schema.yaml` and checks that every
outbound `methods.<name>.http` block has a `rate_limit` (set on the method, or a
package-wide default in `provider_config.rate_limit`). On `internal` tier a
miss **blocks**; on `beta` tier it's a **warning**. (When you publish without
attaching a bundle, there are no schemas to inspect and the check stays silent.)

**8. `required_docs`** — the bundle must ship `README.md`, `SKILL.md`, and
`CHANGELOG.md` at the tarball root. Missing any of them blocks. For the
`verified` tier (Phase 2), `LEARNINGS.md` and a non-empty `tests/` directory
are additionally expected — warnings today, requirements once the promotion
policy ships.

---

## Section 3 — The score

Scoring is deterministic. Start at `100`, subtract per finding, floor at `0`:

```
start          100
each blocking   -25
each warning     -5
```

A clean publish is `100`. The score is informational when `passed` is false —
the publish is rejected by the *presence of any blocking finding*, not by the
number. One blocking finding (`75`) fails just as hard as four (`0`). A package
with only warnings still passes; the score just dips.

The score is stored on the published version (`registry_packages.security_score`)
and shown on the package detail page, so brands can see it.

---

## Section 4 — Reading a `422`

A failed publish returns the findings as a list. Each finding has a `severity`,
a dotted `code`, a `message`, and (for schema findings) the `path` to the field:

```json
{
  "passed": false,
  "score": 75,
  "findings": [
    {
      "severity": "blocking",
      "code": "credentials.auth_type_unknown",
      "message": "credential 'BLOTATO_API_KEY' uses auth_type 'device_code' which is not in the marketplace-approved set",
      "path": "credentials.required[0].auth_type"
    }
  ]
}
```

Read the `code` first — it tells you which check fired. The `message` names the
exact field and value. Fix it, bump the version (you can't republish a version
in place), and publish again.

---

## Section 5 — Common failures and the one-line fix

| Finding code | Cause | Fix |
|---|---|---|
| `manifest.schema` | unknown top-level key, bad SemVer, missing `trust_domain` | match the field rules in [packaging.md](packaging.md) |
| `signature.bad` | wrong private key, or cert rotated | re-sign with the key matching your active cert |
| `meta.empty_useless` | empty skills + events + peers | add a skill, or declare peers to make it a meta-package |
| `entitlement.unknown` | typo or non-registry entitlement | use a registry name or `vendor.<you>.<surface>.<verb>` |
| `entitlement.too_broad` | `network.egress.public` over one host | switch to `network.egress.provider.<host>` |
| `blocklist.eval` | a literal `eval(` in source | remove it; the runtime sandbox forbids it anyway |
| `credentials.auth_type_unknown` | reserved/unknown `auth_type` | use `pat`/`api_key`/`oauth`/`oauth_pkce`/`hmac_sha256` |
| `docs.readme_missing` | `README.md` not in the bundle | add it at the package root |

---

## Section 6 — Running the review without publishing

You don't have to publish to see findings. The standalone introspection
endpoint runs the same eight checks and returns the findings without writing
anything:

```bash
curl -X POST \
  "https://app.opvs.ai/api/v1/marketplace/security/review" \
  -H "X-Admin-Key: $OPVS_MARKETPLACE_ADMIN_KEY" \
  -H "Content-Type: application/json" \
  -d @review-request.json
```

It proxies to the marketplace service's `POST /api/v1/security/review`. Use it
in CI to gate a release before it ever reaches `publish`. The CLI's
`publish --dry-run` runs the local half (validate + sign) without network calls;
the review endpoint runs the server-side half.

---

## See also

- [packaging.md](packaging.md) — the manifest fields `manifest_schema` validates.
- [signing-and-submit.md](signing-and-submit.md) — what produces the signature
  `signature_verify` checks.
- [../03-authoring/validation-preflight.md](../03-authoring/validation-preflight.md)
  — catching these locally before you publish.
- [../reference/api.md](../reference/api.md) — the publish + review endpoints.
