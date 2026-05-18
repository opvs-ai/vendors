# Breaking changes

When you change a skill, sometimes existing installs keep working and sometimes they don't. The semver major bump exists exactly for the cases where they don't.

This page covers:

- What counts as a breaking change in OPVS — removing a tool, renaming a tool, changing the contract response shape, narrowing an entitlement
- What does *not* count as breaking — adding a tool, adding an optional parameter, improving guidance copy, changing pricing (Phase 3)
- How to coordinate a major-version bump — release notes, customer communication, dual-publish window
- Peer-dependency knock-on — if your major bump breaks `^X.0.0` for downstream packages, you flag it
- The "deprecate the old, publish the new" pattern — soft path that lets brands migrate at their own pace

<!-- TODO V-F: write content. Cross-link 04-publishing/versioning.md and 05-editing-and-versioning/deprecation.md. -->

---

> Filled in by content session **V-F**.
