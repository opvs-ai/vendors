# Common gotchas

A short list of mistakes that catch first-time skill authors. Each one is a real incident the platform has actually hit. They are listed up front so you don't have to discover them the same way.

The contract-level gotchas (sibling `_guidance`, `intent` over `query`, closed `telemetry_signal` enum, `skill_call` routing by manifest id) are summarised in [the-contract.md](the-contract.md) and explained fully in the marketplace spec. This page covers the operational gotchas that bite during authoring and deployment.

---

## 1. Tool-name regex — `^[a-zA-Z0-9_-]+$`

**The single most important rule on this page.**

Every tool name registered through any plugin — generated, hand-written, MCP, native — must match `^[a-zA-Z0-9_-]+$`. No dots. No `@`. No `/`. No colons. Letters, digits, underscore, hyphen, that's it.

Why this matters more than it sounds:

OpenAI's chat-completions API enforces this regex on every entry in the `tools[].function.name` array. **One bad name poisons the entire array.** The API returns `HTTP 400` before any LLM reasoning runs and the agent loop dies on its first turn. Every other agent that loaded the same skill will hit the same `HTTP 400` on its next OpenAI call.

The gateway is permissive about names. The LLM provider is not. Your tool ships, the log says "registered", and the first agent to call it crashes. So does the next. So does the agent in another brand. The blast radius is everyone who loaded your skill.

**How to apply:** Validate names locally with `opvs-skills validate` (it checks this rule). Run a real LLM completion against your skill before you ship — log markers do not catch this. See [validation-preflight.md §Step 4](validation-preflight.md#step-4--end-to-end-llm-completion-non-negotiable).

This rule applies to: gateway plugin tools, codegen output names (`schema.yaml → methods.<name>`), MCP tool exports, hand-written plugin tools. Anthropic accepts dots; OpenAI does not. Assume OpenAI.

---

## 2. `requires.env` in SKILL.md frontmatter blocks resolution

**Symptom:** Skill is on disk, declared in `moltbot.json` with the right env vars provided by `opvs-foundation`, but the agent says "skill not loaded".

**Cause:** Your `SKILL.md` frontmatter declares `metadata.openclaw.requires.env: [OPVS_API_URL, OPVS_PAT]`. The gateway checks `requires.env` against the **own skill's** `skills.entries.{name}.env` block — not against all available env vars from other skills. Even though `opvs-foundation` provides those vars at runtime, your skill's resolution check fails because *your* entry doesn't list them.

**Fix:** Remove `requires.env` from the frontmatter when the vars live on another skill. Keep `primaryEnv` as documentation only — it does not block resolution.

```yaml
# BAD — blocks resolution even though opvs-foundation provides these
metadata:
  openclaw:
    requires:
      env: [OPVS_API_URL, OPVS_PAT]

# GOOD — documentation only
metadata:
  openclaw:
    primaryEnv: OPVS_PAT
```

---

## 3. config.yaml ↔ moltbot.json drift

**Symptom:** You PATCH an agent's skill/tool config through the Management API. Nothing happens at runtime.

**Cause:** The Management API writes `config.yaml`. The gateway reads `moltbot.json`. A startup doctor process merges `config.yaml` into `moltbot.json` — but **only at container start**. A PATCH to a running gateway is a no-op for runtime config.

**Fix:** For runtime config changes (skill enablement, tool allowlists, env vars), edit `moltbot.json` directly and recreate the gateway container:

```bash
docker compose up -d --force-recreate openclaw-gateway
```

Do **not** use `docker compose restart` for moltbot changes — `restart` does not re-read `.env` or trigger the doctor merge.

---

## 4. `docker compose restart` ≠ recreate

Closely related to #3. `restart` keeps the existing container, just stops and starts the process. It does **not**:

- Re-read `.env`
- Re-run the entrypoint's doctor merge
- Pick up `moltbot.json` changes
- Pick up bind-mount changes for files copied at entrypoint time

For any config change that matters, use `docker compose up -d <service>` to recreate the container. The 40-second startup penalty is worth not chasing phantom bugs.

---

## 5. publishConfig hygiene

**Symptom:** You publish to one registry and a package mysteriously appears on another.

**Cause:** Every package in a monorepo needs `publishConfig.registry` pinned in its own `package.json`. The npm default falls back to the user's global `.npmrc`, which can publish a private package to the public registry by accident.

**Fix:** Pin it explicitly per package:

```json
{
  "publishConfig": {
    "registry": "https://npm.opvs.ai"
  }
}
```

Public packages get `"registry": "https://registry.npmjs.org"`. Internal packages get the Verdaccio URL. Pin both — never rely on the user's `.npmrc`.

---

## 6. `validate.yml` runs against stale `dist/`

**Symptom:** CI validate passes. The actual deploy ships broken bundles.

**Cause:** The validator runs `node dist/cli.js`. If `dist/` was built from an older commit (because `npx tsc` saw matching mtimes and no-op'd), the validator runs old generator code against new source. Bugs the new generator fixes silently come back.

**Fix:** Make `rm -rf dist && npx tsc` the first step before any validate or generate run. The 10ms cost prevents a class of silently-broken deploys.

```bash
# Safe regen recipe
rm -rf dist && npx tsc && node dist/cli.js generate --all ...
```

---

## 7. `opvs-skills doctor --fix` hangs in non-interactive contexts

**Symptom:** Gateway container reports "Up". Health check times out forever. Logs end at "Skills status" panel; "Doctor complete" never appears.

**Cause:** The entrypoint runs `openclaw doctor --fix` without `--non-interactive`. The doctor blocks on a TTY prompt asking to confirm an auto-fix. Docker's stdin is closed, so the prompt waits forever. The `|| true` after the doctor call doesn't save you because the process never exits — it is signal-waiting, not error-exiting.

**Workaround for now:**

```bash
docker exec OPVS-openclaw-gateway sh -c 'kill -9 <doctor_pid>'
```

That kicks the entrypoint past the doctor and lets the gateway boot.

**Real fix:** Always pass `--non-interactive` to doctor in CI, container entrypoints, and any other unattended context.

---

## 8. Env-var injection requires filesystem discovery

**Symptom:** You added a skill entry to `moltbot.json → skills.entries.my-skill.env` with API keys. The agent's exec curl still says the key is empty.

**Cause:** A `skills.entries` entry alone is not enough. The skill must also be **discovered from disk** — `SKILL.md` has to exist in one of the configured `skills.load.extraDirs` paths. Without a filesystem match, `applySkillEnvOverrides()` skips your entry entirely.

**Fix:**

1. Confirm your skill's `SKILL.md` is on disk in one of: `/home/node/.openclaw/workspace/skills`, `/root/.openclaw/skills`, `/app/node_modules/@spideragent/shared/skills`.
2. Confirm the config field is `skills.load.extraDirs` (an array). It is **not** `skills.paths` — that field does not exist in the schema and is silently ignored.

If you are confused about which path wins, the answer is "first match." See the discovery priority in the platform's runtime docs.

---

## What's next

Now that you know what to avoid, run through the [preflight checks](validation-preflight.md) one more time. If the validator is happy and your end-to-end LLM completion returns cleanly, you are ready to head to [04-publishing/](../04-publishing/) and submit.
