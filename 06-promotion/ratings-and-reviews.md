# Ratings and reviews

Reviews are install-gated: only brands that have actually installed your package can post one. That's a deliberate cold-start design — the signal-to-noise is high because every reviewer has skin in the game.

This page covers:

- Who can review — only brands with an `installed_packages` row, regardless of current state
- Who can rate (numeric only) — any brand member, no install gate, 30/hr rate limit
- How the moderation model works — reviews go live immediately, super-admins can hide with a recorded reason, no pre-publish queue
- Vendor replies — today admin-mediated, eventually first-party once vendor-auth ships
- What's *not* in the v1 model — spam detection, helpful/unhelpful voting, review images, "installed for X days" segmentation
- How abuse escalation works on your side — what to flag, how to contact OPVS

<!-- TODO V-G: write content. Reference REVIEWS-MODERATION-POLICY.md verbatim for the four rules. -->

---

> Filled in by content session **V-G**.
