# Marketing overlay

Your package manifest is signed and immutable. But the hero image, the screenshots, the long-form description — those need to evolve without forcing a republish for every typo. The **marketing overlay** is the mutable layer on top.

This page covers:

- The four rules of the overlay — admin-only, marketing-fields-only, overlay-wins-on-conflict, all-edits-audited
- The six overlay fields — `hero_image_url`, `screenshots`, `demo_video_url`, `long_description`, `features`, `author`
- Why overlay edits are admin-controlled today — OPVS curates Wave 1 surfaces; vendor self-service is Phase 2
- How your shipped manifest interacts with overlay edits — manifest is fallback, overlay wins
- What to ship in `package.yaml` vs what to leave for overlay — practical guidance
- The "Reset to manifest" admin action — what triggers it, what you can expect

<!-- TODO V-F: write content. Reference MARKETING-OVERLAY-POLICY.md in OPVS planning docs for the four rules verbatim. Note: vendors should NOT plan around self-service editing in v1. -->

---

> Filled in by content session **V-F**.
