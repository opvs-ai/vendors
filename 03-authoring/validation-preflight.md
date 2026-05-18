# Validation — preflight

Run the validator on your laptop before you submit. The marketplace's security review repeats the same checks server-side, so if your local run is clean, the publish is almost always clean.

This page walks you through the four-step preflight and tells you the one verification step that is non-negotiable: a real LLM completion against your skill, **not** a gateway log line.

## The four preflight steps

```
1. opvs-skills generate <name>      ← see what the gateway will load
2. opvs-skills validate             ← cross-tier consistency check
3. opvs-skills publish --dry-run    ← server-side checks, no upload
4. Pre-merge LLM completion         ← non-negotiable; see §End-to-end below
```

Steps 1-3 catch the easy mistakes. Step 4 catches the one failure mode that the easy checks miss.

## Step 1 — generate

```bash
opvs-skills generate my-skill
```

This reads `client/schema.yaml` and emits the Tier 3 gateway plugin into a temp directory. Open the generated `.ts` file and skim it.

What you are looking for:

- **One typed tool per method** in your `schema.yaml`. If a method is missing from the plugin, your YAML probably has a parse error the generator silently dropped.
- **Tool names match your method names** — no rewrites, no prefixes you didn't ask for.
- **No tool name contains a dot, `@`, or `/`.** This is the showstopper. The regex is `^[a-zA-Z0-9_-]+$` and every output name has to pass.

You will not normally hand-edit the generated plugin — it is regenerated on every publish. You read it to confirm your `schema.yaml` says what you think it says.

## Step 2 — validate

```bash
opvs-skills validate
```

Cross-tier consistency check. It verifies that:

- Every method declared in `schema.yaml` has a description
- Every parameter has a type
- Auth declarations are well-formed
- The `guidance:` block is present and matches the shape the generator expects
- Tool names pass the `^[a-zA-Z0-9_-]+$` regex (the validator catches this; you don't have to trust step 1's eyeballing)
- The response contract holds — sibling `_guidance`, closed `telemetry_signal` enum

Findings come back as a `_findings: [...]` array on the validator response. Each finding has a code, a severity, and a one-line description. Fix anything at severity `blocking` before you publish; the server-side review will reject the same finding.

## Step 3 — publish --dry-run

```bash
opvs-skills publish --dry-run
```

This runs the same Pydantic schema validation, manifest checks, and signature steps the marketplace registry runs on submission — but stops before the actual upload.

What it adds on top of step 2:

- Manifest shape (`package.json`, `manifest.yaml`, `SKILL.md` frontmatter all agree)
- Version coherence (you bumped `version:` in every place it appears)
- Signing key present and not revoked
- Bundle is under the package size limit
- All declared entitlements parse cleanly

If `--dry-run` is clean, the real publish is almost always clean too. The one class of failure that dry-run cannot reproduce is **end-to-end LLM behaviour** — see the next section.

## The seven server-side security checks

When you actually publish, the marketplace runs seven checks before accepting the bundle. `--dry-run` runs all of them locally. They are:

| # | Check | What it catches |
|---|-------|-----------------|
| 1 | Manifest validation | Missing fields, wrong types, version mismatches across files |
| 2 | Signature | Bundle is signed by a key that matches a known vendor cert |
| 3 | Schema lint | Tool name regex, contract shape, response schema |
| 4 | Blocklist | No banned dependencies (license-incompatible or known-malicious npm packages) |
| 5 | Auth scheme | Auth declaration matches one of the supported patterns (PAT, partner key, OAuth) |
| 6 | Rate-limit declarations | Methods that hit external APIs declare their rate-limit budget |
| 7 | Doc presence | `SKILL.md` exists, has frontmatter, has a non-empty description |

A failure on any check returns `HTTP 422` with the failing check code in the response body. Fix the issue, bump the version, re-run `--dry-run`, re-publish.

## Step 4 — end-to-end LLM completion (non-negotiable)

This is the one check that gateway log markers cannot replace.

**A line in the gateway log that says `Registered 8 tools` is NOT proof that the agent can call those tools.**

The gateway is permissive about tool names. The LLM provider (OpenAI, in particular) is not. A tool with a dot in its name passes gateway loading, shows up in the log as "registered", and then crashes the agent on the next LLM call with `HTTP 400: Invalid 'tools[N].name': string does not match pattern`. The entire `tools` array is poisoned by one bad name — the agent's session dies, and so does every other agent that loaded the same skill.

This happened in production in May 2026. Three tools with dotted names shipped past a "verification" that consisted of reading the gateway log. The first agent to drive an OpenAI completion crashed; every agent with the same skill loaded would have crashed in turn until the rename shipped.

The fix is to actually drive a completion before you call your publish done:

```bash
# Send a real prompt through the gateway to an agent that has your
# skill loaded. Check that the LLM call returns (no HTTP 400) and
# that at least one tool from your skill appears in the response.
```

What you are checking:

1. The completion returns at all (no `HTTP 400` from the LLM provider)
2. Your skill's tools appear in the tools array sent to the model
3. The model can name-call at least one of them when asked

If you are publishing to the marketplace, the registry will run an LLM smoke test on its end too — but doing it locally first means you find the regex failure in five minutes instead of after a public rollback.

## How to read validator findings

The validator's `_findings` array uses three severities:

| Severity | Meaning | Action |
|----------|---------|--------|
| `blocking` | Will reject your publish | Fix before continuing |
| `warning` | Allowed but flagged on the marketplace listing | Worth fixing if cheap |
| `info` | Informational, no action needed | Skim and ignore |

Common fixes are short pointers in [common-gotchas.md](common-gotchas.md).

## What's next

If your preflight is clean, you are ready to publish. Head to [04-publishing/](../04-publishing/) for the submission flow.

If you hit something the validator didn't explain, check [common-gotchas.md](common-gotchas.md) — most first-time-author issues are listed there with one-line fixes.
