# Glossary

Short definitions of every term that shows up across these docs.

This page will define:

- **Cert / signing cert** — your ed25519 keypair used to sign package bundles
- **Cert fingerprint** — SHA-256 of your public key; appears on every package detail page
- **Partner key** — long-lived HTTP API auth token scoped to your vendor
- **Vendor / vendor scope** — your namespace on the marketplace (e.g. `@spideriq`, `@opvs-ai`)
- **Package** — versioned signed bundle of one or more skills under your vendor scope
- **Skill** — one capability inside a package, defined by `SKILL.md` + `schema.yaml`
- **Manifest / `package.yaml`** — declarative metadata at the package root; signed, immutable
- **Marketing overlay** — admin-controlled mutable layer of marketing copy and media on top of the manifest
- **Entitlement** — granular capability declared in manifest, granted by brand at install
- **Trust tier** — `internal | beta | verified | community`; controls badging and gating
- **Doctor** — `skill_doctor` self-check; tells you whether a skill is installed and healthy for a given brand
- **Revoke** — pull a published version; gateway picks up within 60s
- **Deprecate** — soft-flag a version; existing installs keep working, new ones see a warning
- **Security review** — seven static checks the registry runs before accepting a publish
- **On-demand contract** — the locked response shape every skill follows (`_guidance` sibling, `telemetry_signal` enum, `intent` param)

<!-- TODO V-H: write content. Cross-link each term to the page in this repo where it's covered in depth. -->

---

> Filled in by content session **V-H**.
