# Credential rotation

Keys leak. People leave teams. Certs expire. This page covers how to rotate every credential OPVS issues to you, with zero downtime where possible.

This page covers:

- **Rotating a partner key** — mint a new one, switch CI to use it, ask OPVS to revoke the old one. Soft-delete is instant.
- **Rotating a signing cert** — mint-rotation flow, dual-signing window during the swap, when the old fingerprint stops being trusted
- **What to do if a key leaks** — emergency revoke checklist
- **Expiry policy** — partner keys don't expire by default; certs do (TBD interval, planner to confirm)
- **Auditing rotations** — where the audit trail lives, how to ask OPVS for a report

<!-- TODO V-C: write content. Cross-link 01-onboarding/apply-for-a-cert.md and 04-publishing/revocation.md. Note: 04-publishing/revocation.md is about revoking a published *package version* — different concept from revoking a key or cert. -->

---

> Filled in by content session **V-C**.
