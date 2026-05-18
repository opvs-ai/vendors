# Apply for a cert

Every package on the marketplace is signed with an ed25519 keypair. OPVS runs a small centralized CA: we mint your keypair, register the public key in the vendor allowlist, and hand the private key to you once.

This page covers:

- What the cert is for (signing package bundles at publish time)
- The mint flow — how the request gets made, who approves, where the private key actually lives
- The one-shot reveal of the private key — you see it once, you store it, OPVS doesn't keep a copy
- Cert fingerprints — what they are and where they show up on your packages

The cert is separate from the **partner key** you'll need for HTTP API auth. The cert signs bundles. The partner key authenticates publish requests. Both are covered in [02-authentication/partner-keys.md](../02-authentication/partner-keys.md) and [02-authentication/credential-rotation.md](../02-authentication/credential-rotation.md).

<!-- TODO V-B: write content. Reference MARKETPLACE-CONCEPT.md §Signing & revocation steps 1-4 for the canonical ed25519 mint flow. Reference MKT-PARTNER-KEYS-BRIEF.md for how cert mint differs from partner-key mint — same one-shot-reveal pattern, but different surface. -->

---

> Filled in by content session **V-B**.
