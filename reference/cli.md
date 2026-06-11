# `@opvs-ai/skills` CLI

The vendor-facing command line. It wraps the [HTTP API](api.md), the ed25519
signing flow, and the Tier-3 code generator. This is what you use to go from a
`schema.yaml` to a published package.

- **Package:** `@opvs-ai/skills` on Verdaccio at `npm.opvs.ai` (current `0.4.2`).
- **Source:** [`packages/opvs-skills/src/cli.ts`](https://github.com/opvs-ai/OPVS/blob/main/packages/opvs-skills/src/cli.ts).
- **Binary:** `opvs-skills` (or run the built `node dist/cli.js`).

```bash
npm install -g @opvs-ai/skills --registry https://npm.opvs.ai
opvs-skills --help
```

---

## Environment variables

| Var | Used by | Default |
|---|---|---|
| `OPVS_MARKETPLACE_URL` | `publish` / `install` / `doctor` `--api-url` | `http://localhost:8600` |
| `OPVS_MARKETPLACE_ADMIN_KEY` | `publish` `--admin-key` | — |

Both have a matching flag; the flag wins over the env var.

---

## `generate [name]`

Emit the Tier-3 plugin, MCP tools, SDK, CLI subcommands, and OpenAPI from a
skill's `schema.yaml`.

| Flag | Effect |
|---|---|
| `--all` | regenerate every skill that has a `schema.yaml` |
| `--skills-dir <dir>` | skills directory |
| `--plugins-dir <dir>` | output dir for generated plugins |
| `--dry-run` | print output, write nothing |
| `--bundle <path>` | also emit a single bundled `index.ts` |
| `--mcp <path>` | also emit MCP tool definitions |
| `--sdk <dir>` · `--cli <dir>` · `--openapi <dir>` | also emit SDK / commander subcommands / OpenAPI 3.1 |
| `--allowlist <names>` | comma-separated skills to include |
| `--external` | force PAT (`Authorization: Bearer`) auth instead of `X-Admin-Key` — for gateways outside the OPVS network |

```bash
opvs-skills generate agentboard --plugins-dir ./build
```

---

## `validate [name]`

Cross-tier consistency check — run it before every publish. Confirms the
`schema.yaml`, generated plugin, and `SKILL.md` agree.

| Flag | Effect |
|---|---|
| `--skills-dir <dir>` | skills directory |

```bash
opvs-skills validate agentboard
```

---

## `publish <package>`

Package the directory into a signed tarball, sign it, and upload. `<package>` is
a directory or a manifest path.

| Flag | Effect |
|---|---|
| `--private-key <arg>` | **required.** ed25519 key — a file path, an inline base64url string, or `@env:<NAME>` |
| `--api-url <url>` | marketplace API base (default `OPVS_MARKETPLACE_URL` or `localhost:8600`) |
| `--admin-key <key>` | marketplace admin key (or `OPVS_MARKETPLACE_ADMIN_KEY`) — required unless `--dry-run` |
| `--dry-run` | validate + sign locally, skip all network calls |

The `--admin-key` for the publish endpoint on the marketplace service is the
platform `MANAGEMENT_API_SECRET_KEY` — distinct from the partner key used on the
gateway proxy. Always `--dry-run` first to confirm the sign + manifest before a
real publish.

```bash
# Preflight — validate + sign, no upload
opvs-skills publish ./packages/opvs-ai-agentboard \
  --private-key /tmp/opvs-ai-privkey.txt --dry-run

# Real publish
opvs-skills publish ./packages/opvs-ai-agentboard \
  --private-key @env:OPVS_SIGNING_KEY \
  --api-url https://market.opvs.ai \
  --admin-key "$OPVS_MARKETPLACE_ADMIN_KEY"
```

On success it prints the `package_id`, bundle URL, sha256, and cert fingerprint.

---

## `install <spec>`

Fetch, verify, and extract a published package. `<spec>` is
`@vendor/name[@version]`. (This is the local-extract install for inspection —
installing *onto a brand* is the dashboard / admin-API flow, not this command.)

| Flag | Effect |
|---|---|
| `--to <dir>` | **required.** extracts to `<dir>/<vendor>/<name>/` |
| `--api-url <url>` | marketplace API base |
| `--public-key <arg>` | inline key, file, or a fingerprint-keyed JSON key ring |
| `--no-verify-signature` | skip signature check (sha256 integrity still enforced) |

```bash
opvs-skills install @opvs-ai/agentboard --to ./vendor-packages
```

---

## `doctor <spec>`

Show marketplace health for a package. `<spec>` is `@vendor/name` (no version).

| Flag | Effect |
|---|---|
| `--api-url <url>` | marketplace API base |
| `--no-color` | disable ANSI colour |

```bash
opvs-skills doctor @opvs-ai/agentboard
```

Exits non-zero when the package is `revoked`, `error`, or `missing` — so it
drops straight into a CI gate.

---

## Other commands

`list` (skills + tier status), `emit-runtime <runtime>` (emit runtime-native
bundles), and `check` / `fix` (drift detection + auto-repair) round out the CLI.
They're authoring-side tools rather than part of the publish path — run
`opvs-skills --help` for the current set.

---

## A typical publish session

```bash
opvs-skills validate my-skill                       # 1. consistency check
opvs-skills publish ./my-skill --private-key @env:KEY --dry-run   # 2. sign + preflight
opvs-skills publish ./my-skill --private-key @env:KEY \
  --api-url https://market.opvs.ai --admin-key "$KEY"            # 3. real publish
opvs-skills doctor @my-vendor/my-skill              # 4. confirm it's live
```

---

## See also

- [api.md](api.md) — the endpoints these commands call.
- [../04-publishing/signing-and-submit.md](../04-publishing/signing-and-submit.md)
  — what `publish` does with your private key.
- [../04-publishing/security-review.md](../04-publishing/security-review.md) —
  the checks `publish` triggers server-side.
