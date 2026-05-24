# Changelog

All notable changes to the `opvs-ai/vendors` portal docs are recorded here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Added

- **Publishing section: 4-layer + envelope publish guide** — new page [`04-publishing/4-layer-and-envelope.md`](04-publishing/4-layer-and-envelope.md) walks through shipping the 4-layer skill format (Layer 1 `SKILL.md` + Layer 2 `references/` + Layer 3 `learnings/` + Layer 4 `registry/`) together with the on-demand envelope contract block (`guidance:` in `client/schema.yaml`) in one signed marketplace package. Cites Q1/Q3/Q4 locked decisions verbatim. Uses the in-house `agentboard` skill as the worked example with one snippet per layer. Unblocks SpiderIQ + Di-Atomic to publish per Initiative A (envelopes) + Initiative B (4-layer).
- Cross-links from [`04-publishing/README.md`](04-publishing/README.md) and [`03-authoring/the-contract.md`](03-authoring/the-contract.md) into the new guide.
- Authoring section filled — skill anatomy, contract pointer, validation preflight, common gotchas. ([content/V-D-authoring](03-authoring/))
- Authentication section filled — partner keys, publishing tokens, credential rotation.
- Onboarding section filled — vendor journey, vendor types, cert apply flow.

## 2026-05-18

### Added

- Initial information architecture (redesign-A). Seven directories with 25 page stubs, ready for content sessions V-B through V-H. ([#1](https://github.com/opvs-ai/vendors/pull/1))
