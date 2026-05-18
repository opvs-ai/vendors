# Signing and submit

How the bytes leave your laptop and arrive on `market.opvs.ai`.

This page covers:

- The signing step — ed25519, what gets hashed, what gets signed
- The CLI command — `opvs-skills publish` end-to-end
- What headers the CLI sends — partner key (auth) + signature + cert fingerprint
- What the registry does on receive — signature verify, cert allowlist check, manifest validation, security review trigger
- The bundle URL — where your signed tarball ends up in R2
- What "published" actually means — the row is in the registry, the bundle is in R2, brands can install

<!-- TODO V-E: write content. Reference MARKETPLACE-CONCEPT.md §Signing & revocation step 2 (publishing). Reference the live publish endpoint at apps/api/app/api/v1/marketplace.py POST /packages. -->

---

> Filled in by content session **V-E**.
