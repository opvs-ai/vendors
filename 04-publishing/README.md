# 04. Publishing

Your skill is authored and validates locally. This is the section about getting it onto `market.opvs.ai` so brands can install it.

The path is:

```
package on disk  →  signed tarball  →  registry upload  →  security review  →  published
```

Each step is a separate page. Versioning and revocation are also covered here, because both are part of the publish lifecycle even if they don't happen on day one.

## Pages

- [packaging.md](packaging.md) — `package.yaml` manifest, the standard package layout, the tarball
- [signing-and-submit.md](signing-and-submit.md) — ed25519 sign, `opvs-skills publish`, what the registry does on receive
- [security-review.md](security-review.md) — the seven static checks the registry runs before accepting your bundle
- [versioning.md](versioning.md) — semver, peer dependencies, deprecating old versions
- [revocation.md](revocation.md) — pulling a published version after a bug or security finding
- [4-layer-and-envelope.md](4-layer-and-envelope.md) — how to ship the 4-layer skill format together with the on-demand envelope contract in one signed package (unblocks SpiderIQ + Di-Atomic and any third-party vendor)

## What's next

Your package is live. People are installing it. Now you maintain it — see [05-editing-and-versioning/](../05-editing-and-versioning/) for what changes mid-stream.

---

> Pages in this section are filled in by content session **V-E**.
