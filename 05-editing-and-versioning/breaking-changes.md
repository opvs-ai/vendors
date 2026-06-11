# Breaking changes

A change is **breaking** when an agent that worked against your old version
would fail against the new one. The semver major bump exists for exactly these
cases. Everything additive is a minor; everything cosmetic is a patch. This page
draws the line, then covers how to ship a major without stranding the brands
already running you.

The companion read is [../04-publishing/versioning.md](../04-publishing/versioning.md),
which covers semver mechanics and immutability. This page is about the
*editorial* call — what counts, and what to do about it.

---

## Section 1 — What counts as breaking

A change is breaking if it removes or changes the contract an installed agent
already depends on:

- **Removing a method** — an agent calling `listDeals` stops working when you
  drop it.
- **Renaming a method or a required parameter** — same effect as removal; the
  old name resolves to nothing.
- **Changing a response shape** an agent parses — a field that moves, changes
  type, or disappears.
- **Adding a new required input** to an existing method — calls that didn't pass
  it now fail.
- **Narrowing or changing a required entitlement** — the install grant the brand
  gave no longer covers what the package asks for, so the install gate rejects.
- **Changing a credential's shape** — renaming the env var, switching
  `auth_type` (e.g. `api_key` → `oauth`), or making an optional credential
  required. The brand's stored credential no longer satisfies the manifest.

All of these require a **major** bump (`1.x → 2.0.0`).

---

## Section 2 — What does *not* count

These are safe to ship in a **minor** (additive) or **patch** (cosmetic):

| Change | Bump |
|---|---|
| Adding a new method | minor |
| Adding a new *optional* parameter | minor |
| Adding a new optional entitlement | minor |
| Richer guidance / better descriptions | minor or patch |
| Fixing a wrong default value | patch |
| Doc / changelog / typo fix | patch |

The test: *could an agent that worked yesterday break tomorrow purely from this
change?* If no, it's not breaking — minor or patch. If yes, it's a major.

---

## Section 3 — Shipping a major safely

A major doesn't get pushed onto anyone. Brands pin the version they installed and
stay there until they reinstall (see
[versioning.md § How brands install](../04-publishing/versioning.md#section-5--how-brands-install-pin-vs-float)).
That's your safety margin — use it:

```
1. Publish 2.0.0 (immutable, side-by-side with 1.x)
2. Leave 1.x published — installed brands keep running untouched
3. Announce: CHANGELOG entry on 2.0.0 + a direct customer notice
4. Brands reinstall to move to 2.0.0 at their own pace
5. Only revoke 1.x if it's actively unsafe (it usually isn't)
```

Because 1.x stays `published`, you get a natural dual-publish window: both majors
are installable at once, and brands migrate when ready. Don't revoke the old
major just to "force the upgrade" — revoke is for danger, not for housekeeping
(see [../04-publishing/revocation.md](../04-publishing/revocation.md)).

Your `CHANGELOG.md` for a major should spell out the migration: what was removed
or renamed, the new shape, and the minimum a brand has to do. Treat it as the
upgrade guide, because it's the one document every installer reads.

---

## Section 4 — Peer-dependency knock-on

If you declare a peer (e.g. every `@opvs-ai/*` skill peers
`@opvs-ai/opvs-foundation ^1.0.0`), a **major bump in the peer** can break you.
Your `^1.0.0` range accepts `1.x` but not `2.0.0`. When the peer ships a major:

1. The peer's vendor flags the major as a coordinated event.
2. You test against the new major.
3. You publish a **new version of your package** with the range pointed at the
   new major (`^2.0.0`).

The range field is your control surface here — see
[versioning.md § Peer dependencies — pinning](../04-publishing/versioning.md#section-2--peer-dependencies--pinning).
Pin with `^` so the peer's *minor* fixes flow to you automatically; only its
*major* requires you to act.

---

## Section 5 — The soft alternative

Sometimes a change is technically breaking but you'd rather not force a hard
cutover. The pattern is **deprecate the old, publish the new**: announce the new
method/version, keep the old one working for a grace period, and remove it in a
later major. That lets brands migrate on their own schedule instead of all at
once.

The full lifecycle (and the honest note that a platform-level soft-deprecation
flag isn't shipped yet) lives in
[deprecation.md](deprecation.md). Until that ships, "deprecate" in practice means
*publish the new version, document the old one as superseded, and don't revoke
it.*

---

## See also

- [../04-publishing/versioning.md](../04-publishing/versioning.md) — semver,
  immutability, peer pinning, how brands pin vs upgrade.
- [deprecation.md](deprecation.md) — the soft retire path.
- [../04-publishing/revocation.md](../04-publishing/revocation.md) — the hard
  pull, for unsafe versions only.
