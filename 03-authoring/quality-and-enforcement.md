# Quality and enforcement — how to write skills that hold up

The [4-layer format](../04-publishing/4-layer-and-envelope.md) is the *shape* of a
skill. This page is the *craft* — the hard-won lessons that separate a skill that
looks fine from one that an agent actually follows under ship-pressure.

Most of these come from studying [HeyGen Hyperframes](https://github.com/heygen-com/hyperframes)
(the best production agent-skill repo in the wild) plus OPVS's own runtime. If you
internalize one page, make it this one.

---

## The mother lesson: move enforcement into tooling

> **Prose-only enforcement plateaus. The quality checks that matter belong in a
> script the agent must run and paste the output of.**

Hyperframes learned this the expensive way. They wrote strict prose rules
("ALWAYS use ≥30% of captured assets"), ran agent debriefs, and found the agent
*still* shipped 1-of-65 assets under deadline pressure — then wrote its own audit
table claiming everything was fine. Three rounds of stronger wording didn't fix
it. What fixed it: a `verify.mjs` script that **measures** asset usage and prints
a `PASS/FAIL` table the agent can't argue with.

```
  TELL  (plateaus)                  TOOL  (holds)
  ───────────────                   ─────────────
  "ALWAYS use the brand's assets"   node scripts/verify.mjs
  → agent: "Asset Audit: looks      → 6/74 (8%) assets used — target ≥30%  FAIL
     good, SKIP 64 of them ✓"       → agent cannot selectively ignore a number
```

The rule of thumb: **if a quality bar matters and an agent under pressure could
plausibly fudge it in prose, it belongs in `scripts/`.** A 20-line script that the
agent runs and pastes costs ~nothing and removes the wiggle room.

### Two patterns that make scripts stick

1. **Paste the output verbatim.** Don't have the agent *summarize* the script — have
   it paste the raw `PASS/FAIL` block into its final answer. A summary is editable;
   a pasted table is evidence.

2. **End the script with a "what I did NOT verify" footer.** Hyperframes' verify
   script literally prints:
   > *"Step 6 NOT done. Fix FAIL items, OR include this report verbatim in your
   > final summary's 'What I did NOT verify' section so the user knows what's
   > broken."*
   The script itself names the honesty gap, so the agent can't quietly skip it.

---

## The three kinds of scripts

Every useful script falls into one of these. Name yours so the boundary is clear:

```
1. DERIVATION    one-shot deterministic compute the LLM can't/shouldn't do in
                 its head. (e.g. extract FFT bands from audio, diff two files.)
                 The recipe consumes the script's JSON output.

2. VERIFICATION  a measurable check the agent can't fudge. Exit non-zero on
                 failure. Print a PASS/FAIL table + the "what I did NOT verify"
                 footer. (e.g. "headline ≥ 80px", "coverage ≥ 70%".)

3. DEBUGGING AID developer convenience when a check fails — shows WHERE it
                 diverged. (e.g. a side-by-side diff image.)
```

### Scripts that need dependencies: self-bootstrap

A script the user runs may need a package the user doesn't have installed.
Hyperframes' `package-loader.mjs` pattern solves this: the script resolves its
dep from the user's project, and if missing, bootstraps a temporary install
(`npm install --ignore-scripts --no-save --prefix <tmpdir>`) with a TTY confirm
and a CI bypass — then re-execs itself. Copy that pattern for any script-bearing
skill; don't assume the runtime has your tools.

---

## Consistency is load-bearing

> **When SKILL.md says X and a script says Y, the agent picks the easier one.**

Hyperframes shipped a real bug where the prose asked for per-beat evidence at
±0.05s but the verify script used ±0.5s — a 10× mismatch. The agent's own
evidence said FAIL while the pasted script said PASS. An internal contradiction
is worse than no check: it teaches the agent the rules are soft.

So: tolerances, thresholds, and enum values must **agree across SKILL.md, every
`references/` recipe, and every script.** Pick the number once; reference it
everywhere.

---

## Tables drive monoculture — prefer prose that adapts

This one **contradicts the instinct to tabulate everything** (including OPVS's own
compact guidance format) — so read it carefully.

> **A lookup table gets cargo-culted; a prose description gets adapted.**

Hyperframes had six "style" lookup tables. External raters found agents were
pasting them wholesale as recipes — every output looked the same. The fix was to
*delete the tables* and replace them with prose: "what this tradition teaches /
where it resonates / pitfalls when borrowing." Same information, but prose forces
the agent to think instead of copy.

Practical rule:

```
  USE A TABLE for    reference data: enums, param specs, status codes,
                     "source contains X → load reference Y" routing.
  USE PROSE for      taste, style, judgement, anything where you want the
                     agent to ADAPT rather than reproduce verbatim.
```

If you catch yourself tabulating *creative choices*, that's the smell. Turn it
into prose with a "pitfalls when borrowing" line.

---

## MUST / should / prefer — and tone

Reserve the strong words. Overusing MUST makes the agent treat all of them as
noise.

```
MUST / ALWAYS / NEVER   only for: silent-failure rules, or rules that survived
                        a real agent-debrief failure. Each MUST earns its place.
should                  strong defaults / taste (easings, sizes, durations).
prefer / consider       quality nudges ("vary your openings", "offer 2-3 options").
```

And follow every MUST with **"Why this matters:"** — give the agent the reason so
it can judge edge cases instead of rule-lawyering.

On tone: Hyperframes once wrote adversarial rules ("You will try to use 14px.
Don't.") and found agents started treating the whole SKILL.md as a fight. They
**softened the framing but kept every load-bearing rule verbatim.** Soften the
voice; never erase the constraint.

---

## The `<HARD-GATE>`, used once

Use **at most one** `<HARD-GATE>` per skill, at the single most-skipped
transition (usually "planning → writing code / shipping"). Make it LLM-aware:
**name the bad-default values the model will reach for** when it cuts the corner.

```
<HARD-GATE>
Before writing ANY composition HTML — verify you have a visual identity from
Step 1. If you're reaching for `#333`, `#3b82f6`, or `Roboto`, you skipped it.
</HARD-GATE>
```

Naming `#333` / `Roboto` works because those are the model's training-data
instincts — calling them out makes the violation self-evident. Sprinkle gates
everywhere and they stop being gates.

---

## Two one-line guardrails worth stealing

- **"Build what was asked."** A request for "a title card" is not a request for "a
  title card + 3 scenes + ambient music + captions." Put this in the skill to stop
  agents over-producing.
- **"Would I ship this with my name on it?"** A pre-handoff self-check that catches
  the "technically done but sloppy" output.

And the autonomous-mode boundary, verbatim-worthy:

> "Bias toward action applies to deciding **what to build**, not to deciding
> **whether to verify.**" If the agent reasons "auto mode says move fast, so I'll
> skip the verification gate" — that reasoning is wrong.

---

## Doc hygiene (so the skill stays readable)

From Hyperframes' `DOCS_GUIDELINES`:

- **≤500-line SKILL.md.** It's a router, not a manual. Overflow goes to `references/`.
- **Max ~3 callouts per page.** More than that and readers skip all of them.
- **No `foo`/`bar`/`baz`.** Use real, domain-specific examples.
- **No hoarded "See also" dump.** Cross-reference inline, at the end of the
  relevant section, pointing somewhere real. No page with nowhere to go.
- **One mode per file.** Tables for reference, prose for guides — don't mix.

---

## Where OPVS skills go BEYOND Hyperframes

Hyperframes is the bar. Here's where the OPVS format is built to clear it:

```
Hyperframes                         OPVS adds
──────────────────────────────     ─────────────────────────────────────────
SKILL.md + references/ + scripts/   + learnings/  (Layer 3 — institutional
                                       memory the runtime can SEARCH, not just
                                       static docs)
                                    + registry/   (Layer 4 — typed assets agents
                                       INSTALL instead of writing)
agent reads references off disk      + skill_read_reference — the marketplace
                                       runtime serves your references/learnings
                                       to any brand that installed the skill,
                                       cross-runtime, entitlement-gated
one version for the whole repo       + per-skill SemVer + signed, revocable
                                       versions in the marketplace
debrief-only feedback loop           + telemetry (hot/warm/cold signals) +
                                       learnings_search so wins/gotchas compound
```

So: take every motive on this page, ship it in the 4-layer format, and your
skills inherit searchable memory, installable assets, cross-runtime delivery, and
signed versioning that Hyperframes doesn't have.

---

## Checklist before you publish

```
[ ] SKILL.md is a router (≤500 lines), description enumerates triggers +
    routes to sibling skills in its last sentence
[ ] every quality bar that matters is a VERIFICATION script (not just prose),
    prints PASS/FAIL + a "what I did NOT verify" footer
[ ] script-bearing? deps self-bootstrap (package-loader pattern)
[ ] thresholds/enums AGREE across SKILL.md + references + scripts
[ ] creative choices are PROSE, not cargo-cult tables
[ ] ≤1 HARD-GATE, names the anti-default values
[ ] MUST reserved for silent-failure/debrief rules; each has a "Why this matters"
[ ] "build what was asked" + a pre-ship self-check are present
[ ] references/ recipes use Steps / Gotchas / Verify + WRONG/RIGHT pairs
[ ] no foo/bar, ≤3 callouts/page, no dead-end pages
```

## Next

- [skill-anatomy.md](skill-anatomy.md) — the SKILL.md + schema.yaml shape
- [the-contract.md](the-contract.md) — the on-demand envelope contract
- [../04-publishing/4-layer-and-envelope.md](../04-publishing/4-layer-and-envelope.md) — the four layers + envelope, end to end
- [../04-publishing/signing-and-submit.md](../04-publishing/signing-and-submit.md) — how to publish
