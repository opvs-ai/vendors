# Marketing overlay

Your `package.yaml` is signed and immutable — change one byte and the signature
breaks. But the things that sell a package — the hero image, the screenshots,
the long description — need to change without a republish every time you swap a
screenshot. The **marketing overlay** is the mutable presentation layer that
sits on top of the signed manifest.

The split is clean: the manifest stays the authority for anything that *runs*
(entitlements, signature, tier, skills); the overlay holds only what's
*displayed*. The overlay can change freely; the manifest can't change at all.

---

## Section 1 — Where it lives

The overlay is a JSONB column, `registry_packages.marketing_overlay`, added in
the marketplace migration
[`047_marketing_overlay.py`](https://github.com/opvs-ai/OPVS/blob/main/apps/spideragent/packages/marketplace/api/alembic/versions/047_marketing_overlay.py).
It is **per package**, not per version — editing it doesn't touch any signed
bundle and doesn't bump a version.

That's the whole point: a typo fix in your long description is a JSONB update,
not a re-sign-and-republish.

---

## Section 2 — The six overlay fields

The overlay holds exactly these presentation fields:

| Field | What it is |
|---|---|
| `hero_image` | the banner on the package detail page |
| `screenshots` | a gallery of product shots |
| `demo_video` | an embedded demo URL |
| `long_description` | the full marketing write-up (the manifest `description` is the short one) |
| `features` | a bulleted capability list |
| `author` | the author/maker card |

Anything not in this list is **not** overlay-editable. Entitlements, the
signature, the tier, the skill surface, credentials — those live in the signed
manifest and only change by publishing a new version.

---

## Section 3 — The four rules

1. **Marketing fields only.** The overlay can touch the six fields above and
   nothing else. It cannot grant an entitlement, change a tier, or alter what
   the package does. Presentation, not capability.

2. **Overlay wins on conflict.** Where a field exists in both the manifest and
   the overlay (the manifest's `media`/`marketing` blocks vs. the overlay), the
   **overlay value is what the catalog UI shows.** The manifest value is the
   fallback for when no overlay is set.

3. **Admin-controlled today.** The edit endpoint is gated by
   `require_admin_or_session` — a platform admin, not vendor self-service.
   OPVS curates the launch surfaces; vendor self-serve overlay editing is a
   later phase. Until then, you supply the copy and OPVS applies it.

4. **Every edit is audited.** Overlay changes are recorded — there's no silent
   rewrite of how your package is presented.

---

## Section 4 — Editing the overlay

A single PATCH, against the api-gateway marketplace proxy:

```bash
curl -X PATCH \
  "https://app.opvs.ai/api/v1/marketplace/packages/{package_id}/marketing" \
  -H "X-Admin-Key: $OPVS_MARKETPLACE_ADMIN_KEY" \
  -H "Content-Type: application/json" \
  -d '{
        "long_description": "AgentBoard brings task management to your agents...",
        "features": ["Kanban + table views", "Bulk task create", "Activity feed"]
      }'
```

The proxy forwards to the admin route
`/api/v1/admin/marketplace/packages/{package_id}/marketing`. Image and video
assets upload separately:

```bash
curl -X POST \
  "https://app.opvs.ai/api/v1/marketplace/packages/{package_id}/media" \
  -H "X-Admin-Key: $OPVS_MARKETPLACE_ADMIN_KEY" \
  -F "file=@hero.png"
```

A partial PATCH only touches the fields you send — omit a field and its current
value stays.

---

## Section 5 — Manifest vs. overlay: what to put where

Practical guidance for the split:

| Put in `package.yaml` (signed) | Put in the overlay (mutable) |
|---|---|
| the one-line `description` | the long marketing write-up |
| skills, entitlements, credentials | hero image, screenshots, demo video |
| tier, peers, signing block | feature bullets, author card |

Ship a sensible baseline in the manifest's optional `media`/`marketing` blocks
(see [../04-publishing/packaging.md § The rest](../04-publishing/packaging.md#the-rest-optional-with-defaults))
so the detail page looks complete the moment you publish. Then iterate on the
overlay — that's the layer you'll touch often, and it never costs a republish.

If an overlay edit ever goes wrong, the manifest values are the fallback:
clearing the overlay field reverts the detail page to whatever the signed
manifest declared. The signed package is always the floor.

---

## See also

- [../04-publishing/packaging.md](../04-publishing/packaging.md) — the manifest
  `media` / `marketing` blocks that the overlay overrides.
- [../06-promotion/featured-placement.md](../06-promotion/featured-placement.md)
  — a strong overlay is what makes a package feature-worthy.
- [breaking-changes.md](breaking-changes.md) — the changes that *do* require a
  republish (because they're in the signed manifest, not the overlay).
