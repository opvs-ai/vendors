# Vendor types

OPVS recognises three kinds of vendors. The differences affect how you get approved, what scope you publish under, what trust tier your packages start in, and what level of support you can expect.

This page walks through each in turn, then ends with a side-by-side table for quick comparison.

## At a glance

```
  Internal teams          ──►  @opvs-ai/*           tier: internal
       │                       OPVS staff. Default for platform-owned skills.
       │
  Partner teams           ──►  @spideriq/*,         tier: internal / beta
       │                       @di-atomic/*         Established collaborators,
       │                                            own org, own cert.
       │
  Third-party developers  ──►  @yourvendor/*        tier: beta / community
                               Independent builders, not yet open in Phase 1.
```

You typically know which row you are in already. If you're staff at OPVS, you're internal. If you run a partner company OPVS works with regularly, you're partner. If you're neither, you're a third-party developer — and in Phase 1 that means "wait list."

## Internal teams

**Who they are.** Engineers and ops staff inside OPVS itself. The folks who built and maintain the platform.

**Scope.** Everything published under `@opvs-ai/*`. That namespace is reserved for OPVS-owned work — `@opvs-ai/platform-skills`, `@opvs-ai/agentboard`, `@opvs-ai/agentdocs`, and the rest of the Wave 1 packages.

**Cert flow.** Internal-team certs are minted the same way external certs are minted (ed25519 keypair, one-shot reveal). The difference is that the private key lives in OPVS's own CI secrets, not on an external machine. Same shape, same security model — just the recipient is us.

**Default trust tier.** `internal`. Bypasses the more cautious `beta` defaults that external vendors start with. Used because OPVS-published packages are the bedrock the rest of the marketplace runs on.

**Support model.** Internal channels. No formal SLA — internal teams talk to each other.

**What's expected.** Same security review as everyone else. Internal status is *not* a free pass on the seven security checks at publish time. Every package gets reviewed, every signature gets verified. The only difference is the trust tier badge on the customer-facing detail page.

## Partner teams

**Who they are.** External organisations OPVS has an established relationship with. Today there are two: **SpiderIQ** and **Di-Atomic**. Each runs their own engineering team, their own product, their own GitHub org.

**Scope.** One scope per partner — `@spideriq/*`, `@di-atomic/*`. Scopes are reserved. Another vendor cannot publish into someone else's scope, and the marketplace enforces this in two places: at cert mint (scope tied to vendor row) and at publish time (partner key checked against package manifest's `vendor_id`).

**Cert flow.** Standard ed25519 mint via the OPVS CA. Same one-shot reveal as everyone else, sent to the partner's primary contact email. The partner stores the private key in their own CI secrets. OPVS does not keep a copy.

**Partner-key flow.** In addition to the signing cert, each partner gets a partner key (bcrypt-hashed in the marketplace DB, scoped to their vendor row). This is the credential their CLI uses when calling `market.opvs.ai` to publish. A single partner can have multiple active partner keys at once — for example one per CI environment, plus one for a human operator's laptop. Each is independently revocable.

**Default trust tier.** `beta` on a new package, with promotion to `internal` once the partner is stable. `verified` is a Phase 2 surface that requires runtime telemetry and human review across multiple package versions.

**Support model.** Direct line to OPVS ops. Partners get faster responses than third-party developers will when that surface opens. There is no formal SLA in Phase 1, but the practical expectation is "we talk regularly, we coordinate on incidents."

**What's expected.** Partners are expected to operate their own infra responsibly. If your skill calls into SpiderIQ's API, OPVS does not maintain SpiderIQ's API — that's on SpiderIQ. The marketplace surfaces the signed package; the partner runs everything downstream of it. Partners are also expected to keep their cert and partner key secure, rotate on a sensible cadence, and notify OPVS promptly on any suspected compromise.

## Third-party developers

**Who they are.** Independent developers and small organisations who want to publish to the marketplace but don't have an existing relationship with OPVS.

**Scope.** Their own — `@yourvendor/*`. Same naming rules as partner scopes.

**Status today.** Not open. Phase 1 of the marketplace is intentionally internal + partner only. The infrastructure works for third-party vendors today — there is nothing technically blocking it — but OPVS has chosen to gate this surface behind a manual sign-up flow that is not yet built.

**When it opens.** Phase 2, alongside the public catalog at `opvs.ai/marketplace`. The plan is a real sign-up form, automated baseline checks, and a path from `community` → `verified` driven by review queue + runtime telemetry.

**Default trust tier when it opens.** `community` on the first publish, with manual promotion to `verified` available once the package has 30 days of stable runtime telemetry and a human review of the source and the security posture.

**Support model.** Documentation-first. Self-serve sign-up, self-serve cert mint, self-serve key rotation. OPVS ops will not be involved in every publish, the way they currently are for partners.

**What's expected.** Same security review every other tier gets. Plus: a clear contact and support URL on the vendor profile (Phase 2 makes this mandatory for any `verified` upgrade — see Google Play's rule, OPVS adopts the same model).

If you fall in this bucket today, the practical advice is to subscribe to the marketplace announcements channel and wait. Phase 2 opens this surface.

## Side-by-side comparison

| Aspect | Internal | Partner | Third-party |
|---|---|---|---|
| Scope prefix | `@opvs-ai/*` | `@spideriq`, `@di-atomic` (one each) | `@yourvendor/*` |
| Phase 1 open? | ✓ | ✓ | ✗ (Phase 2) |
| Cert mint | OPVS CA, one-shot reveal | OPVS CA, one-shot reveal | OPVS CA, one-shot reveal |
| Partner key | Yes | Yes | Yes (when Phase 2 opens) |
| Default tier | `internal` | `beta` → `internal` | `community` → `verified` |
| Security review | Same 7 checks | Same 7 checks | Same 7 checks |
| Approval flow | Internal | Manual conversation with OPVS ops | Self-serve sign-up (Phase 2) |
| Support | Internal channels | Direct to OPVS ops | Documentation-first |
| Revocation authority | OPVS ops + your own team | OPVS ops + your own team | OPVS ops + your own team |
| Live today? | 8 packages live | 6 SpiderIQ packages live | 0 |

## Trust tiers, briefly

Tier and vendor type are related but not the same. Tier is a per-package badge. Vendor type is a per-vendor classification.

The four tiers today are:

- **`internal`** — OPVS-owned or partner-owned, signed, reviewed. Grey badge.
- **`beta`** — Published and signed, but not yet stable for general use. Yellow badge.
- **`community`** — Auto-checks only. No human review yet. Grey badge. (Phase 2 surface.)
- **`verified`** — Human reviewed, full docs present, 30 days of stable telemetry. Blue checkmark. (Phase 2 surface.)

Every package on the marketplace today is `internal` or `beta`. The promotion path to `verified` is part of the Phase 2 roadmap.

For the full breakdown, including how runtime telemetry can downgrade a package back into `beta`, see the OPVS planning doc `MARKETPLACE-CONCEPT.md §Trust tiers`.

## Which type am I, in practice

Most readers know already. If you're not sure:

- If your paycheck comes from OPVS — internal.
- If your company has an OPVS scope reserved and you're publishing to it — partner.
- Otherwise — third-party, and that means waiting until Phase 2 opens.

The "small partner" vs "large third-party" distinction is essentially: have we had a conversation? Partners are vendors we already coordinate with. Third-party is the eventual open path.

## Next steps

- Read [apply-for-a-cert.md](apply-for-a-cert.md) for the cert mint flow. Same flow for all three vendor types.
- Read [02-authentication/partner-keys.md](../02-authentication/partner-keys.md) for partner-key issuance and rotation.

---

> Filled in by content session **V-B**.
