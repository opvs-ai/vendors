# Packaging

The marketplace ships your skills as one signed, gzipped tarball. This page
describes the shape that tarball expects: the `package.yaml` manifest field by
field, the directory layout, how skill IDs work, peer dependencies, and exactly
what the publish CLI puts in the bundle versus what it leaves out.

Everything here is cross-checked against the server-side validator
([`package_manifest.py`](https://github.com/opvs-ai/OPVS/blob/main/apps/spideragent/packages/marketplace/api/app/schemas/package_manifest.py)).
If the validator rejects a field, your `package.yaml` gets a `422` at publish —
so the rules below are the ones that actually hold.

The worked example throughout is the in-house `@opvs-ai/agentboard` package.
Browse it live:
[`packages/opvs-ai-agentboard/package.yaml`](https://github.com/opvs-ai/OPVS/blob/main/packages/opvs-ai-agentboard/package.yaml).

---

## Section 1 — The `package.yaml` manifest

The manifest is the root document. The validator **forbids unknown top-level
keys** — only the fields below are accepted. A stray key (a typo, a field from
some other tool) fails the publish with `extra fields not permitted`.

### Identity (all required)

| Field | Format | Rule |
|---|---|---|
| `name` | `@vendor/slug` | lowercase, 3–214 chars. The `@vendor` part **must** equal `@` + the `vendor` field below. |
| `version` | SemVer string | must parse as semver (`0.1.0`, not `0.1` or `v1`). |
| `description` | string | 1–1024 chars. |
| `vendor` | slug | lowercase `[a-z0-9][a-z0-9_-]*`, no leading `@`. |
| `category` | string | 1–64 chars, free-form (`board`, `data-collection`, …). |
| `license` | string | 1–128 chars (`UNLICENSED`, `MIT`, …). |

```yaml
name: "@opvs-ai/agentboard"
version: "1.2.0"
description: >
  Task management with AgentBoard — get/assign/progress tasks, manage boards
  and columns, post comments and results.
vendor: opvs-ai
category: board
license: UNLICENSED
```

> **Gotcha:** `name`'s scope and `vendor` must agree. `name: "@acme/x"` with
> `vendor: di-atomic` is rejected client-side **and** server-side.

### Signing block (required keys, placeholder values)

You write placeholders; `opvs-skills publish` overwrites them with the real
signature. Leave them exactly as shown:

```yaml
signature: "ed25519:unsigned"
signed_at: "1970-01-01T00:00:00Z"
cert_fingerprint: "SHA256:unsigned"
```

See [signing-and-submit.md](signing-and-submit.md) for what publish does with
these.

### `provider_config` (required)

How the runtime calls your upstream API. Required even for a guidance-only
skill (use a placeholder block + a schema with no `methods:`).

| Field | Required | Rule |
|---|---|---|
| `base_url_env` | yes | env var name holding the base URL, `^[A-Z][A-Z0-9_]*$`. |
| `auth.type` | yes | one of `bearer`, `basic`, `header`, `none`. |
| `auth.token_env` | conditional | **required** when `type` is `bearer`, `basic`, or `header`; env var name. |
| `auth.header` | conditional | **required** when `type` is `header` (the custom header name). |
| `default_format` | no | `yaml` or `json`. |
| `timeout_ms` | no | default `30000`, range 1–600000. |

```yaml
provider_config:
  base_url_env: OPVS_API_URL
  auth:
    type: bearer
    token_env: OPVS_PAT
  default_format: yaml
  timeout_ms: 30000
```

> **The `token_env` trap (real incident).** `@opvs-ai/vibe-publish@1.0.0`
> shipped `auth.type: header` with **no `token_env`**. Publish accepted it, but
> at runtime the gateway sent an empty Blotato key — every call failed. It had
> to be hot-fixed as `@1.0.1`. The validator now rejects this shape: any
> secret-carrying auth type (`bearer`/`basic`/`header`) without `token_env` is
> a hard `422`. A `header` skill needs **both** `header:` and `token_env:`:
>
> ```yaml
> auth:
>   type: header
>   header: blotato-api-key
>   token_env: BLOTATO_API_KEY
> ```

### `credentials` (optional)

Keys the brand must supply before the package runs. Two buckets, `required` and
`optional`. Each entry: `name` (env var, `^[A-Z][A-Z0-9_]*$`), `auth_type` (one
of `pat`, `api_key`, `oauth`, `oauth_pkce`, `device_code`, `cert_invite`,
`hmac_sha256`), `provider` (lowercase slug), optional `help_url` + `description`.
Default is empty (`required: []`, `optional: []`).

### `entitlements` (optional)

What the package is allowed to touch. Two buckets, `required` and `optional`.
Every name must be a **known** entitlement — either in the fixed registry
(`agentboard.task.write`, `brand.read`, …) or a parametric form
(`network.egress.provider.<name>`, `credits.consume.<name>`, or the vendor-scoped
`vendor.<vendor>.<surface>.<verb>`). An unknown name is rejected. A name cannot
appear in both buckets.

```yaml
entitlements:
  required:
    - agentboard.board.read
    - agentboard.board.write
    - agentboard.task.read
    - agentboard.task.write
    - agentboard.comment.write
  optional:
    - agentboard.task.delete
```

> **External vendors:** declare entitlements over your own surface with the
> `vendor.<vendor>.<surface>.<verb>` form — exactly three segments after
> `vendor.`, e.g. `vendor.spideriq.leadgen.search`. The install gate checks the
> `<vendor>` segment matches your publishing slug; cross-vendor declarations are
> rejected.

### `skills` (the package's surface)

A list of the skills this package ships. See
[Section 3](#section-3--skill-ids-inside-a-package) for the `id` rules.

```yaml
skills:
  - id: agentboard
    schema: ./skills/agentboard/schema.yaml
    visibility: public
    entitlements:
      - agentboard.board.read
      - agentboard.board.write
```

A skill's `entitlements` must be a **subset** of the package-level
`entitlements` (required + optional). Requesting one the package didn't declare
fails validation.

### The rest (optional, with defaults)

| Field | Default | What it does |
|---|---|---|
| `inbound_events` | `[]` | webhook/event declarations (Zapier-style adapters). |
| `tier` | `internal` | `internal` / `beta` / `community` / `verified`. `internal` is hidden from browse; `beta` makes it visible + installable. See [versioning.md](versioning.md). |
| `trust_domain` | **required** | slug-shape, usually equal to your `vendor`. Despite being a non-identity field, the validator **requires** it — omitting it is a `422`. |
| `revocable_by` | `[]` | who may revoke (e.g. `opvs-operator`, your vendor slug). |
| `docs` | `{}` | relative paths: `readme`, `skill_guide`, `changelog`, `learnings`. |
| `peer_dependencies` | `[]` | see [Section 4](#section-4--peer-dependencies). |
| `pricing` | `none` | phase-3 metadata. |
| `media`, `marketing` | `{}` | optional detail-page copy (hero image, features, author card). |

---

## Section 2 — Standard package layout

```
my-skill-package/
├── package.yaml                  # the manifest (this page)
├── README.md                     # human-facing overview
├── CHANGELOG.md                  # version history
├── SKILL.md                      # Layer 1 router (flat single-skill packages)
├── client/
│   └── schema.yaml               # API contract + envelope `guidance:` block
└── skills/                       # multi-skill packages put each skill here
    └── <skill-name>/
        ├── SKILL.md
        ├── schema.yaml
        ├── references/           # Layer 2 — procedures
        ├── learnings/            # Layer 3 — institutional memory
        └── registry/             # Layer 4 — typed installable assets
```

Two shapes, picked by your `skills[].schema` path:

- **Flat** — one skill, schema at `./client/schema.yaml`. Smaller packages like
  `@opvs-ai/team` use this. `skills: [{ id: team, schema: ./client/schema.yaml }]`.
- **Nested** — one or more skills under `skills/<name>/`, schema at
  `./skills/<name>/schema.yaml`. `@opvs-ai/agentboard` and the multi-skill
  `@spideriq/spiderflows` use this.

Both are valid. The validator only cares that each `schema` path is a non-empty
string; the path is yours to organise. For the four layers (`references/`,
`learnings/`, `registry/`) read
[4-layer-and-envelope.md](4-layer-and-envelope.md) — that page is the deep dive
on what goes in each.

---

## Section 3 — Skill IDs inside a package

Each entry in `skills:` has an `id`. The rules:

- **Unique within the package.** Two skills with the same `id` is a `422`
  (`duplicate skill ids`).
- **Lowercase kebab/snake** — `^[a-z0-9][a-z0-9_-]*$`, 1–128 chars.
- **The runtime keys on it.** The installed package stores schemas under the
  `id` — pick a name agents will recognise.

> **Footgun — name the skill deliberately.** A skill's `id` is independent of
> the package name. `@opvs-ai/opvs-foundation` ships a skill whose `id` is
> **`foundation`**, not `opvs-foundation`. That's the name the runtime and other
> packages reference. Choose the `id` on purpose; don't assume it mirrors the
> package slug.

---

## Section 4 — Peer dependencies

`peer_dependencies` **is** supported. Use it when your package needs another
package present. Each entry is a `name` (`@vendor/slug`) plus a `range` (a
semver range string).

```yaml
peer_dependencies:
  - name: "@opvs-ai/opvs-foundation"
    range: "^1.0.0"
```

Every `@opvs-ai/*` skill declares `@opvs-ai/opvs-foundation` as a peer, so the
shared identity helpers install automatically alongside any of them.

Range syntax follows standard semver ranges — `^1.0.0` (compatible-with-1.x),
`~1.2.0` (patch-level within 1.2), an exact `1.2.3`, or a bounded
`>=1.2.0 <2.0.0`. The validator checks the `range` is a non-empty string; the
install-side resolver interprets it. For how a peer's major bump ripples to your
package, see [versioning.md](versioning.md#section-2--peer-dependencies--pinning).

> A package with **no** `skills` and **no** `inbound_events` but a non-empty
> `peer_dependencies` is a valid **meta-package** — installing it expands to its
> peers ("install all of $bundle"). A package with none of the three is empty
> and rejected.

---

## Section 5 — What goes IN vs OUT of the tarball

`opvs-skills publish` tars the **whole package directory** and strips a fixed
exclude list. There is no per-package include allowlist — and because the
manifest schema forbids unknown top-level keys, you cannot add a top-level
`files:` field to `package.yaml` (it would `422`). The exclude list is the
control surface, and it lives in the CLI
([`tarball.ts`](https://github.com/opvs-ai/OPVS/blob/main/packages/opvs-skills/src/lib/tarball.ts)).

| IN the tarball | OUT (always excluded) |
|---|---|
| `package.yaml` | `node_modules/` |
| `README.md`, `CHANGELOG.md` | `.git/` |
| `SKILL.md`, `client/`, `skills/` | `.env`, `.env.local` |
| `references/`, `learnings/`, `registry/`, `scripts/` | `dist/`, `build/`, `.turbo/`, `.next/` |
| any other tracked content files | `.DS_Store` |

The excludes cover the usual leak vectors — `.env` (secrets), `.git` (history +
credentials), build artefacts. **Anything not on the exclude list is shipped**,
so keep CI secrets, private signing keys, and `*.local` scratch files out of the
package directory entirely; don't rely on the excludes to catch a key you left
next to `package.yaml`. The bundle is content-addressed and signed, so whatever
ends up inside is what brands receive and what your signature covers.

---

## Section 6 — Worked example walk-through

Browse [`@opvs-ai/agentboard`](https://github.com/opvs-ai/OPVS/blob/main/packages/opvs-ai-agentboard/package.yaml)
as your reference. It exercises every section above:

- **Identity + signing** — `@opvs-ai/agentboard`, version `1.2.0`, placeholder
  signing block.
- **`provider_config`** — `bearer` auth, `OPVS_PAT` token, proxied to
  `/api/v1/board/*`.
- **Entitlements** — five `required` board/task/comment scopes, one `optional`
  (`agentboard.task.delete`).
- **Skills** — one nested skill, `id: agentboard`, schema under
  `./skills/agentboard/`, requesting a subset of the package entitlements.
- **Peer** — `@opvs-ai/opvs-foundation ^1.0.0`, installed automatically.
- **Tier** — `beta`, so it shows up in the customer marketplace and is
  installable.

When a brand installs it, the gateway extracts the tarball into the brand
workspace with `references/`, `learnings/<id>/`, and `registry/<item>/` intact,
reads the `guidance:` block from `client/schema.yaml` to decorate every
response, and registers the typed tools an agent calls. One signed tarball; the
whole skill surface comes online.

---

## See also

- [4-layer-and-envelope.md](4-layer-and-envelope.md) — what goes in
  `references/`, `learnings/`, `registry/`, and the envelope contract.
- [signing-and-submit.md](signing-and-submit.md) — sign + publish the package
  you just built.
- [versioning.md](versioning.md) — semver, peer pinning, tiers, how brands pin
  a version.
- [security-review.md](security-review.md) — the static checks the registry
  runs on receive.
