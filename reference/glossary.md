# Glossary

Plain-English definitions for the terms that recur across these docs. Where a
term has a page of its own, the link points there.

---

**Brand** — a tenant on OPVS. Skills are installed *per brand*, and entitlements
+ credentials are granted per brand. A brand pins the package version it
installed; nothing auto-upgrades it. See
[../04-publishing/versioning.md § How brands install](../04-publishing/versioning.md#section-5--how-brands-install-pin-vs-float).

**Bundle** — the signed, gzipped tarball the marketplace ships. It contains the
`package.yaml`, docs, and the skill layers. Content-addressed by sha256; the
signature covers whatever's inside. See
[../04-publishing/packaging.md § What goes IN vs OUT](../04-publishing/packaging.md#section-5--what-goes-in-vs-out-of-the-tarball).

**Certificate (signing cert)** — your ed25519 keypair, registered with the
OPVS CA. You sign every bundle with the private half; the registry verifies
against the public half. See [../01-onboarding/apply-for-a-cert.md](../01-onboarding/apply-for-a-cert.md).

**Cert fingerprint** — `SHA256:...` of your public key. It appears on every
package detail page and in the manifest's `cert_fingerprint` field, so brands can
see who signed what.

**Credential** — a secret a brand must supply before a package runs (an API key,
an OAuth token). Declared in the manifest's `credentials` block; collected via a
credential invite. See [../02-authentication/](../02-authentication/).

**Credential invite** — a token-IS-auth link (`/contribute/{token}`) a brand
opens to hand over a credential without logging into your systems. 72h expiry,
10/brand/24h. See [../02-authentication/](../02-authentication/).

**Deprecate** — soft-retire a version: prefer the newer one, but the old keeps
working. A platform-level soft flag isn't shipped — today it's a publish-new +
document convention. See [../05-editing-and-versioning/deprecation.md](../05-editing-and-versioning/deprecation.md).

**Doctor** — the "is this skill installed and healthy for this brand?" check.
Returns entitlements, credentials, and upstream health. Endpoint
`GET /skills/{id}/doctor`; CLI `opvs-skills doctor`. See [api.md](api.md).

**Entitlement** — a granular capability a package declares and a brand grants at
install (e.g. `agentboard.task.write`). Must be a known registry name or a
parametric form. See
[../04-publishing/packaging.md § entitlements](../04-publishing/packaging.md#entitlements-optional).

**Envelope contract** — the locked response shape every skill returns: a
`_guidance` sibling, a `telemetry_signal` enum, an `intent` parameter. It's what
lets the runtime decorate every response consistently. See
[../04-publishing/4-layer-and-envelope.md](../04-publishing/4-layer-and-envelope.md).

**4-layer format** — the four content layers a skill ships:
`references/` (procedures), `learnings/` (institutional memory),
`registry/` (typed installable assets), plus the Layer-1 `SKILL.md` router. See
[../04-publishing/4-layer-and-envelope.md](../04-publishing/4-layer-and-envelope.md).

**Install** — adding a package to a brand: validate manifest, resolve
credentials, grant entitlements, set state `active`. Pins the version. Admin / 
dashboard flow, not the CLI `install` (which only extracts locally). See [api.md](api.md).

**Manifest (`package.yaml`)** — the declarative root document of a package:
identity, signing block, provider config, entitlements, skills, tier. Signed and
immutable. The validator forbids unknown top-level keys. See
[../04-publishing/packaging.md](../04-publishing/packaging.md).

**Marketing overlay** — the mutable presentation layer (hero image, screenshots,
long description) on top of the immutable manifest. Admin-edited today; overlay
wins over the manifest at the UI. See
[../05-editing-and-versioning/marketing-overlay.md](../05-editing-and-versioning/marketing-overlay.md).

**Meta-package** — a package with no skills and no events but non-empty
`peer_dependencies` — installing it expands to its peers ("install all of
$bundle"). Valid; a package with none of the three is rejected. See
[../04-publishing/packaging.md § Peer dependencies](../04-publishing/packaging.md#section-4--peer-dependencies).

**Package** — a versioned, signed bundle of one or more skills under your vendor
scope, named `@vendor/slug`.

**Partner key** — a long-lived HTTP API token scoped to your vendor, used as
`X-Admin-Key` on admin writes. External vendors use a per-vendor partner key, not
the platform master key. See [../02-authentication/partner-keys.md](../02-authentication/partner-keys.md).

**Peer dependency** — another package yours needs present, declared with a
`name` + semver `range`. Every `@opvs-ai/*` skill peers
`@opvs-ai/opvs-foundation`. See
[../04-publishing/versioning.md § Peer dependencies](../04-publishing/versioning.md#section-2--peer-dependencies--pinning).

**Provider config** — the manifest block telling the runtime how to call your
upstream API: base-URL env var, auth type + token env, timeout. Required even for
guidance-only skills. See
[../04-publishing/packaging.md § provider_config](../04-publishing/packaging.md#provider_config-required).

**Rating** — a `1–5` star score from a brand. No install gate, 30/hr/brand,
upserted one-per-user-per-brand. See
[../06-promotion/ratings-and-reviews.md](../06-promotion/ratings-and-reviews.md).

**Review** — a prose `title` + `body` from a brand that has *installed* the
package. Install-gated, 10/hr/brand. See
[../06-promotion/ratings-and-reviews.md](../06-promotion/ratings-and-reviews.md).

**Revoke** — tombstone a published version. Installs error out, the gateway
picks up the flip within ~60s. One-way. See
[../04-publishing/revocation.md](../04-publishing/revocation.md).

**Security review** — the eight static checks the registry runs on every
publish; any blocking finding is a `422`. See
[../04-publishing/security-review.md](../04-publishing/security-review.md).

**Security score** — the deterministic `0–100` from the security review (start
100, blocking −25, warning −5). Shown on the detail page.

**Signed package** — a bundle whose sha256 is signed by your ed25519 cert and
verified at publish. The signature is what makes the supply chain trustworthy.

**Skill** — one capability inside a package, defined by `SKILL.md` +
`schema.yaml`, addressed by its `id`. The `id` is independent of the package
name. See [../04-publishing/packaging.md § Skill IDs](../04-publishing/packaging.md#section-3--skill-ids-inside-a-package).

**`skill_layers`** — the JSONB column (`installed_packages.skill_layers`) the
install-time extractor populates with a package's 4-layer content, so the
runtime can serve references/learnings/registry on demand. The Initiative-C
runtime piece. Surfaced by skills-meta tools (`skill_describe`, `learnings_search`,
`skill_read_reference`).

**Tier 1 / 2 / 3** — the three-tier skill system. Tier 1 is discovery
(manifest, ~15 tokens/skill), Tier 2 is decision guidance (`SKILL.md`), Tier 3 is
the typed tools (generated plugin). See the root project docs.

**Trust tier** — a per-version manifest value: `internal | beta | community |
verified`. Controls visibility and badging. `internal` hides, `beta` lists. See
[../06-promotion/badges.md](../06-promotion/badges.md).

**Vendor / vendor scope** — your namespace on the marketplace (`@spideriq`,
`@opvs-ai`, `@di-atomic`). Everything you publish lives under it; your
entitlements use the `vendor.<you>.<surface>.<verb>` form. See
[../01-onboarding/](../01-onboarding/).

---

## See also

- [api.md](api.md) — the endpoints behind these terms.
- [cli.md](cli.md) — the commands.
- [../04-publishing/4-layer-and-envelope.md](../04-publishing/4-layer-and-envelope.md)
  — the deepest of the format definitions.
