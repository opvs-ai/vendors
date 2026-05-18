# Revocation

A published version cannot be edited. If you ship a bug or a security issue, you revoke the version — that flips a flag in the registry and the gateway picks up the change within 60 seconds.

This page covers:

- When to revoke — security bug, data leak, breaking change shipped as a patch
- Who can revoke — you (your scope only) and platform admins (anything)
- The revoke endpoint — `POST /packages/{package_id}/revoke`
- What brands see — installed-but-revoked falls into an error state until they install a fixed version
- The 60-second propagation window — gateway revocation polling
- How to communicate the revoke — CHANGELOG, customer notification, what the dashboard shows

Note: revoking a *version* is different from rotating a *credential* (covered in [../02-authentication/credential-rotation.md](../02-authentication/credential-rotation.md)). One pulls a bundle; the other replaces a key.

<!-- TODO V-E: write content. Reference MARKETPLACE-CONCEPT.md §Signing & revocation step 4. Reference the live revoke endpoint at apps/api/app/api/v1/marketplace.py POST /packages/{id}/revoke. -->

---

> Filled in by content session **V-E**.
