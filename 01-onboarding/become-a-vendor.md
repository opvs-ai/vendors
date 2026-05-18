# Become a vendor

The path from "I'm interested in publishing on `market.opvs.ai`" to "my first skill is live and brands can install it."

This page covers who can be a vendor, what the application looks like in practice, and what your first week ends up being about. If you're already approved and just looking for the cert mint flow, skip to [apply-for-a-cert.md](apply-for-a-cert.md).

## The short version

```
  I'm interested
       │
       ▼
  Reach out to OPVS ops
       │  (informal — email or DM is fine in Phase 1)
       ▼
  Vendor account approved          ◄── manual review today
       │
       ▼
  ed25519 signing cert minted      ◄── one-shot reveal of private key
       │
       ▼
  Partner key issued for HTTP API  ◄── separate from the cert
       │
       ▼
  Author your first skill          ◄── 03-authoring/
       │
       ▼
  Publish + security review        ◄── 04-publishing/
       │
       ▼
  My first skill is live
```

Three named things land in your lap on day one — a vendor scope, a signing cert, and a partner key. Each does a different job and they are covered in separate pages.

## Who can be a vendor

OPVS recognises three vendor types today. The full breakdown lives in [vendor-types.md](vendor-types.md), but at a glance:

- **Internal teams** — OPVS staff publishing under the `@opvs-ai` scope. This is where the first 8 marketplace packages came from.
- **Partner teams** — established collaborators with their own org and their own scope. SpiderIQ (`@spideriq`) and Di-Atomic (`@di-atomic`) are the two live examples.
- **Third-party developers** — independent builders publishing under their own scope. This category is real on paper but not yet open in practice (Phase 2).

In Phase 1, vendor approval is a manual conversation with OPVS ops. There is no public sign-up form yet. You reach out, we talk, we mint your credentials. The CA flow and the partner-key mint are both admin-driven.

This will become self-serve over time. For now it is on purpose: every vendor on the marketplace today is someone OPVS has spoken to.

## What OPVS needs from you

Before we mint your cert and your partner key, we need a small amount of structured info.

- **Vendor scope name.** The npm-style namespace you want to publish under — for example `@spideriq`. We check it isn't already taken and doesn't conflict with reserved names.
- **A primary contact email.** This is where the one-shot reveal link for the signing key gets sent. Use a shared inbox or a real human, not a noreply.
- **An ops contact for incidents.** A different email or a Slack handle is fine. Used for revocation, security incidents, and "your latest publish is failing security review."
- **A short description of what you'll publish.** One paragraph is enough. Helps us pick the right starting trust tier (see [vendor-types.md](vendor-types.md)).
- **GitHub org or username.** So we know where your source repo lives and can attribute revocations to a sensible identity if needed.

That is the whole intake form. No legal docs, no payments, no SLAs in Phase 1. The marketplace is internal-only at this stage.

## How long approval takes

In Phase 1, hours not days. The bottleneck is human review — there is no automated approval pipeline. If you reach OPVS ops in working hours, you can typically have a vendor scope, a signing cert reveal link, and a partner key within the same conversation.

The mint operations themselves are instant. The CA mints an ed25519 keypair in well under a second. The partner-key mint is a database insert with a bcrypt hash. The slow part is the human deciding to do it.

## What you get on day one

After approval, you walk away with three things.

A **vendor scope** registered on `market.opvs.ai`. That is the `@yourvendor` prefix you'll publish under. Other vendors cannot publish under your scope; you cannot publish under theirs. The marketplace enforces this at publish time.

A **signing cert** — ed25519 keypair. The public half is added to the OPVS CA allowlist. The private half is revealed to you exactly once via a Fernet-encrypted link and never stored by OPVS again after that reveal expires. You sign every package release with the private key; the marketplace verifies the signature against the public key on file. Full detail in [apply-for-a-cert.md](apply-for-a-cert.md).

A **partner key** — a long-lived API secret scoped to your vendor. This authenticates your CLI's HTTP calls to `market.opvs.ai` (publish, revoke, list-your-own-packages). Partner keys live in `02-authentication/partner-keys.md` because the conversation about authentication is bigger than just onboarding.

The signing cert and the partner key are two different things. The cert signs the package bytes. The partner key authenticates the HTTP request that uploads the signed package. Both pieces are required to publish.

## What your first week looks like

Day one is mostly setup. Stand up the signing cert in your CI secrets, drop the partner key in `OPVS_MARKETPLACE_ADMIN_KEY`, install the `@opvs-ai/skills` CLI from `npm.opvs.ai`, and run `opvs-skills doctor` to confirm both credentials work end-to-end.

```bash
npm config set @opvs-ai:registry https://npm.opvs.ai
npm install -g @opvs-ai/skills
export OPVS_MARKETPLACE_ADMIN_KEY=<your partner key>
opvs-skills doctor
```

Day two and three are usually authoring. You pick one of your existing capabilities — an API you already run, a tool you already maintain — and write the `schema.yaml` + `SKILL.md` pair that describes it to the marketplace. The contract for what a skill actually is lives in [03-authoring/](../03-authoring/).

By day four or five you typically have a first publish candidate. You sign the bundle, push it to `market.opvs.ai`, and the security review runs. The review is automated — seven static checks today, more later. If anything fails, the marketplace returns structured findings and the publish does not land. You fix, re-sign, re-push.

By the end of the first week most new vendors have one or two skills live in the `beta` tier. Promotion to `verified` is slower (Phase 2), and that is fine — `beta` is the right place to launch.

## What is intentionally manual right now

A few flows are deliberately admin-only in Phase 1.

**Vendor approval.** No public sign-up. We talk first.

**Cert mint.** Platform admin runs the CA mint. The vendor receives the one-shot link.

**Partner-key mint.** Platform admin runs `POST /api/v1/marketplace/vendors/{id}/keys`. The vendor copies the raw key from the one-shot reveal page in the admin UI.

**Tier promotion.** `internal` and `beta` are issued automatically by the type of vendor you are. `verified` requires human review and is a Phase 2 surface.

None of this is permanent. The plan is to open vendor sign-up, self-serve key rotation, and an automated review-to-verified path as the marketplace grows. The manual gate today is the right call for the volume we have.

## What happens if something goes wrong later

A few escape hatches are worth knowing about up front.

If your signing key is compromised, OPVS can revoke the cert by fingerprint. Gateways across the fleet pick up the revocation within 60 seconds and refuse to load any package signed with that key. You mint a new keypair and republish.

If your partner key leaks, OPVS soft-deletes it (sets `revoked_at`). New publish attempts with that key 403 immediately. You request a new key, plug it into CI, you're back.

If a published package itself is broken, OPVS can revoke that specific version. Brands with that version installed see it disabled within 60 seconds and can install a fixed version when you publish one.

Both the cert mint and the partner-key mint can be re-run as often as needed. There is no "you only get one key, ever" rule.

## Next steps

- Read [vendor-types.md](vendor-types.md) to figure out which category you fall into and what default tier that gives you.
- Read [apply-for-a-cert.md](apply-for-a-cert.md) for the ed25519 mint flow and the one-shot reveal.
- Once you have a vendor account and a cert, head to [02-authentication/](../02-authentication/) for the partner key and the rotation policy.

---

> Filled in by content session **V-B**.
