# 01. Onboarding

Your first stop. You're new to OPVS, you've heard you can publish a skill on `market.opvs.ai`, and you want to know what that takes.

Onboarding is about becoming a recognised vendor. It does not yet cover writing a skill (that's [03-authoring/](../03-authoring/)) or authenticating ongoing API calls (that's [02-authentication/](../02-authentication/)). It answers three questions: who are you, what kind of vendor are you, and how do you get the signing cert you need before you can publish anything?

By the end of this section you should have a vendor account approved, an ed25519 signing cert in your CI secrets, and a clear next step toward your first publish.

## Pages

| Page | What it covers |
|---|---|
| [become-a-vendor.md](become-a-vendor.md) | The journey from "I'm interested" to "my first skill published." Who can apply, what info OPVS needs, what your first week looks like. |
| [vendor-types.md](vendor-types.md) | Internal teams, partner teams, third-party developers — what each is, what scope they publish under, and how their default trust tier differs. |
| [apply-for-a-cert.md](apply-for-a-cert.md) | The ed25519 cert mint flow — CA mint endpoint, Fernet-encrypted one-shot reveal, where the private key lives, and how rotation works. |

## What's next

Once you have a vendor account and a cert, head to [02-authentication/](../02-authentication/). That's where the **partner key** lives — the HTTP API credential your CLI uses for every publish call. The cert signs the bytes; the partner key authenticates the upload request. You need both.

---

> Filled in by content session **V-B**.
