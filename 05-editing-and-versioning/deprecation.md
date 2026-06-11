# Deprecation

Deprecation is the *soft* retire path — the gentle counterpart to
[revocation](../04-publishing/revocation.md). The idea: a version is superseded
but harmless, so instead of breaking everyone off it, you flag it "prefer the
newer one" and let brands migrate at their own pace.

**Be straight about the current state:** a platform-level soft-deprecation flag
is **not shipped today.** This page documents what you actually do now, and what
the planned feature will add. It does not invent a button that doesn't exist.

---

## Section 1 — What's shipped, what isn't

A package version has exactly three registry statuses, defined in
[`models/registry.py`](https://github.com/opvs-ai/OPVS/blob/main/apps/spideragent/packages/marketplace/api/app/models/registry.py):

```python
_PACKAGE_STATUSES = ("published", "revoked", "yanked")
```

Only two transitions have a write path a vendor can drive:

- **publish** → `published`
- **revoke** → `revoked` (the hard path — installed brands error out, the
  gateway picks up the flip within ~60s)

`yanked` is reserved in the schema but has **no endpoint that sets it** and no
dashboard surface for "installed but deprecated." So there is currently no way
to soft-flag a version as deprecated while leaving it installable. There is also
no recognised `deprecated:` field in `package.yaml` or `schema.yaml` — the
manifest validator doesn't know it, and the generator ignores it. Don't design
around either.

---

## Section 2 — How you deprecate today

Until the soft flag ships, deprecation is a *convention*, not a platform state.
It's three moves:

```
1. Publish the replacement version  (immutable, side-by-side)
2. Document the old one as superseded:
     - CHANGELOG entry: "X is superseded by Y, will be removed in 2.0.0"
     - schema.yaml method description / guidance warn: "prefer Y"
     - marketing overlay long_description: a migration note
3. Leave the old version published — installed brands keep working
```

New installs naturally land on the latest version, so the deprecated one stops
spreading. Existing installs keep running untouched — nothing pushes the new
version onto them. They migrate when they reinstall.

The one place an agent *will* see the deprecation at runtime is the method's
guidance text. If you route a "use the new method instead" note into the
schema's `guidance.warn:` (phrased as "Do NOT use X — use Y"), the runtime
surfaces it on every call to the old method. That's the closest thing to a live
deprecation signal available today. See
[../03-authoring/the-contract.md](../03-authoring/the-contract.md) for how
guidance reaches the agent.

---

## Section 3 — Deprecate vs. revoke

Pick by whether the old version is *dangerous* or merely *old*.

| | Deprecate (today's convention) | Revoke |
|---|---|---|
| Old version still works? | yes | no — errors out |
| Brand disruption | none | immediate (≤60s) |
| Use when | superseded but harmless | unsafe or wrong |
| Mechanism | publish-new + document | `POST .../revoke` |

If the old version is bug-free and you just have a better one, deprecate (the
convention above). If it leaks a credential or corrupts data, **revoke** — see
[../04-publishing/revocation.md](../04-publishing/revocation.md).

---

## Section 4 — A worked example

You ship `@acme/leads@1.4.0`. It has a `findLeads` method. You build a better
`searchLeads` and want to retire `findLeads` without breaking anyone.

```
1.5.0 (minor)   add searchLeads alongside findLeads.
                In findLeads' schema guidance.warn:
                  "Do NOT use findLeads for new work — use searchLeads."
                CHANGELOG: "findLeads is deprecated; migrate to searchLeads
                            before 2.0.0."
        ── grace period (you choose: weeks, a quarter, longer) ──
2.0.0 (major)   remove findLeads. This is a breaking change — see
                breaking-changes.md. Brands on 1.x keep running until they
                reinstall onto 2.0.0.
```

Note the bump sizes: **adding** `searchLeads` is additive → minor. **Removing**
`findLeads` is breaking → major. The deprecation lives in the minor; the removal
lives in the next major. Brands get a full version line of overlap to migrate.

---

## Section 5 — The planned soft-deprecation feature

When the soft flag ships, it's expected to add: a `yanked`-style status a vendor
can set without revoking, a dashboard banner on the brand's installed-packages
view, a [doctor](../reference/api.md) warning (not an error), and no agent-side
disruption. Until then, treat the Section 2 convention as the real mechanism —
it gets you the same outcome (brands migrate gradually, nobody breaks) using
only shipped primitives.

---

## See also

- [breaking-changes.md](breaking-changes.md) — the major bump that removes the
  deprecated surface.
- [../04-publishing/revocation.md](../04-publishing/revocation.md) — the hard
  path, for unsafe versions.
- [../04-publishing/versioning.md](../04-publishing/versioning.md#section-3--deprecating-a-version)
  — the same honest note from the versioning angle.
