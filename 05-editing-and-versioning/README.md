# 05. Editing & versioning

Your package is live. What happens when something changes?

Three things move differently after publish:

- **Manifest content** (entitlements, signature, tier) — immutable. You ship a new version.
- **Marketing content** (hero image, screenshots, long description) — mutable via an admin-controlled overlay. The signed manifest doesn't change.
- **Lifecycle state** (deprecated, revoked) — flags that flip without bumping a version.

This section covers all three so you know what you can change yourself, what OPVS controls, and what forces a new version.

## Pages

- [breaking-changes.md](breaking-changes.md) — what counts as breaking, when to bump a major
- [marketing-overlay.md](marketing-overlay.md) — what's mutable after publish, who controls it, what stays signed
- [deprecation.md](deprecation.md) — softly retiring a version without yanking it

## What's next

You're maintaining a live package well. Time to make people *find* it — see [06-promotion/](../06-promotion/).

---

> Pages in this section are filled in by content session **V-F**.
