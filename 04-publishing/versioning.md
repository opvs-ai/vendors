# Versioning

The marketplace tracks every version you publish. A published
`(vendor, name, version)` is **immutable** — you never edit it in place, you
publish a new version. Old versions stay installable until you revoke them.
Brands pin the version they installed.

This page covers semver, peer-dependency pinning, the truth about
deprecation today, tier transitions across versions, how brands pin vs upgrade,
and a real version-history walkthrough.

---

## Section 1 — Semver expectations

`version` must parse as SemVer. The publish validator runs it through a real
semver parser — `0.1` or `v1` is rejected; `0.1.0` is fine.

| Bump | When | Example |
|---|---|---|
| **Major** (`1.x → 2.0.0`) | breaking change: a removed method, a renamed method, a changed response shape, a narrowed entitlement | dropping a tool an agent relied on |
| **Minor** (`1.2 → 1.3.0`) | additive: a new method, a new optional input, richer guidance | adding a `searchTasks` method |
| **Patch** (`1.2.3 → 1.2.4`) | bug fix, docs, no API-surface change | fixing a wrong default |

You **cannot republish a version in place.** To ship any change — even a typo
fix — bump the version and publish again. The old version keeps serving brands
that have it until they upgrade or you revoke it. For the full rules on what
counts as breaking, see
[../05-editing-and-versioning/breaking-changes.md](../05-editing-and-versioning/breaking-changes.md).

---

## Section 2 — Peer dependencies — pinning

If your package declares a peer (see
[packaging.md](packaging.md#section-4--peer-dependencies)), the `range` field is
how you pin it:

| Range | Means |
|---|---|
| `^1.0.0` | any `1.x` — accept minor + patch, not a major bump |
| `~1.2.0` | any `1.2.x` — patch only |
| `1.2.3` | exactly that version |
| `>=1.2.0 <2.0.0` | an explicit bounded range |

```yaml
peer_dependencies:
  - name: "@opvs-ai/opvs-foundation"
    range: "^1.0.0"
```

`^1.0.0` is the common choice — it lets the peer ship minor + patch fixes
without you republishing. Pin tighter (`~` or exact) only when you depend on
behaviour that a minor bump could change. If the peer ships a **major** bump
that breaks your `^X.0.0`, that's a coordinated event — the peer's vendor flags
it and you publish a new version pointing at the new major. See
[breaking-changes.md](../05-editing-and-versioning/breaking-changes.md#peer-dependency-knock-on).

---

## Section 3 — Deprecating a version

**Be aware: a soft "deprecated" flag is not a platform feature today.** It's
worth stating plainly so you don't design around something that isn't there.

A package version has exactly three statuses in the registry: `published`,
`revoked`, and `yanked`. Only two transitions have an endpoint a vendor can
drive:

- **publish** → `published`
- **revoke** → `revoked` (the hard path: brands on that version fall into an
  error state, the gateway picks up the flip within ~60s — see
  [revocation.md](revocation.md))

`yanked` is reserved in the schema but has no write path yet — there is no
endpoint that sets it, and no dashboard warning surface for "installed but
deprecated." So there is currently **no way to soft-flag a version as
deprecated** while leaving it installable.

What you do **today** when a version is superseded but not dangerous:

1. Publish the new version (immutable, side-by-side with the old one).
2. Leave the old version `published` — brands on it keep working untouched.
3. Announce the move: a `CHANGELOG.md` entry, and a customer notice (the
   marketing overlay / email). New installs naturally land on the latest.
4. Only **revoke** the old version if there's a security or correctness reason
   to force everyone off it. Reserve revoke for that — it's disruptive.

The soft-deprecation workflow (a warning banner, doctor notice, brand-paced
migration) is the planned home of
[../05-editing-and-versioning/deprecation.md](../05-editing-and-versioning/deprecation.md).
Until that ships, the publish-new + announce + optional-revoke pattern above is
the real mechanism.

---

## Section 4 — Tier transitions across versions

`tier` is a per-version manifest field — each published version carries its own
tier, and they can differ across the version history of one package.

| Tier | Visibility |
|---|---|
| `beta` (default) | visible + installable (where every live `@opvs-ai` / `@spideriq` package sits) |
| `internal` | hidden from the customer marketplace browse — a deliberate test tier |
| `community` | community-contributed |
| `verified` | formal OPVS-team vetting |

**The tier is your decision, and the default is `beta` — omit `tier` and your
package is customer-visible on publish.** Whatever your manifest declares is
honored verbatim; the platform never overrides it and there is **no promotion
step** — no OPVS operator flips your tier. Set `tier: internal` only when you
want a deliberate hidden cut to test privately first; to make it visible,
publish a new version declaring `beta` (or omit `tier`). Each version inherits
exactly the tier its own manifest declares. (Requesting `verified` — Phase 2 —
is a separate human-vetting step over an already-visible package, not a way to
un-hide an `internal` one; see [../06-promotion](../06-promotion/) and
[security-review.md](security-review.md).)

---

## Section 5 — How brands install: pin vs float

When a brand installs your package, the version is **pinned**. The server
records it in `installed_packages.version` (a non-null column) — the exact
version that was live at install time.

There is **no auto-upgrade.** A brand on `0.3.0` stays on `0.3.0` even after you
publish `0.4.0`. To move them forward, the brand re-runs install — that resolves
the current latest and updates the pinned version on their row. Nothing pushes a
new version onto an existing install.

Practically, that means:

- Publishing a new version is **safe** — it never disturbs existing installs.
- Getting brands onto a fix requires them to reinstall (or you revoke the bad
  version to force the issue).
- The version a brand runs is always the one on their `installed_packages` row,
  not necessarily your latest.

---

## Section 6 — Real example: `@spideriq/spiderflows`

`@spideriq/spiderflows` is a live multi-skill package. As of writing it has
shipped **6 versions**, current `0.4.0` (`tier: beta`, `security_score: 100`).
The history shows the patterns above in practice:

| Version | Date | Tier | What it shows |
|---|---|---|---|
| `0.1.0` | 2026-05-25 | `internal` | first publish, kept private for testing |
| `0.1.1` | 2026-05-26 | `beta` | made visible by republishing at a new version |
| `0.1.2` | 2026-05-26 | `beta` | patch — fix, no surface change |
| `0.2.0` | 2026-05-26 | `beta` | minor — additive surface |
| `0.3.0` | 2026-05-26 | `beta` | minor — more surface |
| `0.4.0` | 2026-05-27 | `beta` | current latest |

> Note: this history starts at `internal` because spiderflows shipped before
> the default became `beta`. Today you'd just publish `0.1.0` at `beta` (the
> default) and be visible from the first version — the `internal` first cut is
> optional, only for a deliberate private test.

Two things to read out of this:

- **Tier is per version.** `0.1.0` is `internal`; everything from `0.1.1` on is
  `beta`. The package didn't "switch" tier — each version declared its own.
- **`internal → beta` was a republish, not an edit.** `0.1.0` couldn't be flipped
  in place; `0.1.1` is a new immutable version that happens to be `beta`.

You can confirm any package's live version history yourself:

```bash
curl -s "https://market.opvs.ai/api/v1/registry/packages/@spideriq/spiderflows/versions" \
  | python3 -c "import sys,json; [print(v['version'], v['tier'], v['status']) for v in json.load(sys.stdin)['versions']]"
```

---

## Cross-links

- For breaking-change procedures (what counts as breaking, how to coordinate a
  major bump): [../05-editing-and-versioning/breaking-changes.md](../05-editing-and-versioning/breaking-changes.md).
- For when and how to revoke a version: [revocation.md](revocation.md).
- For the manifest fields referenced here (`version`, `tier`,
  `peer_dependencies`): [packaging.md](packaging.md).
