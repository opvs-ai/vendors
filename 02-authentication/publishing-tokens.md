# Publishing tokens

When the `@opvs-ai/skills` CLI runs `publish` (or `install`, or `doctor`), it expects a small set of env vars on your shell or your CI. This page is the one-page reference for which env var goes where.

This page covers:

- `OPVS_MARKETPLACE_ADMIN_KEY` — your partner key, used for HTTP auth on publish/revoke
- `OPVS_MARKETPLACE_URL` — registry endpoint (default `https://market.opvs.ai`)
- The signing key path — where the CLI expects your ed25519 private key on disk
- npm/Verdaccio auth — separate, see [reference/cli.md](../reference/cli.md) for the layered story
- What appears in headers vs what stays on disk

<!-- TODO V-C: write content. Cross-reference reference/cli.md for the CLI flag layer and reference/api.md for the HTTP-level view. -->

---

> Filled in by content session **V-C**.
