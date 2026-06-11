# Ratings and reviews

Ratings and reviews are **shipped**. Brands can star-rate any package and write
prose reviews on packages they've installed. The design is deliberately
cold-start-resistant: prose reviews are install-gated, so every reviewer has
actually used the thing.

The backing tables are `package_ratings` and `package_reviews` (marketplace
migrations
[`045_package_ratings`](https://github.com/opvs-ai/OPVS/blob/main/apps/spideragent/packages/marketplace/api/alembic/versions/045_package_ratings.py)
and
[`046_package_reviews`](https://github.com/opvs-ai/OPVS/blob/main/apps/spideragent/packages/marketplace/api/alembic/versions/046_package_reviews.py));
the endpoints live in
[`registry.py`](https://github.com/opvs-ai/OPVS/blob/main/apps/api/app/api/v1/registry.py).

---

## Section 1 — Ratings vs. reviews

Two separate things with different bars.

| | Rating | Review |
|---|---|---|
| Shape | a star score, `1–5` | prose: `title` + `body` |
| Install required? | **no** | **yes** — must have an `installed_packages` row |
| One-per | user, per brand, per package (upsert) | per user (editable) |
| Rate limit | 30/hr/brand | 10/hr/brand |
| Endpoint | `POST /registry/packages/{id}/ratings` | `POST /registry/packages/{id}/reviews` |

Ratings are cheap and spam-resistant by construction — one per user per brand
per package, upserted, so re-rating just overwrites. Reviews carry more weight,
so they cost more: the caller's brand must have installed the package (any
state — even uninstalled counts, because they used it once), or the write
returns `403 only_installed_brands_can_review`.

---

## Section 2 — Submitting

Ratings (no install gate, 30/hr/brand):

```bash
curl -X POST \
  "https://app.opvs.ai/api/v1/registry/packages/{package_id}/ratings" \
  -b session.jar \
  -H "Content-Type: application/json" \
  -d '{"rating": 5}'
```

Reviews (install-gated, 10/hr/brand):

```bash
curl -X POST \
  "https://app.opvs.ai/api/v1/registry/packages/{package_id}/reviews" \
  -b session.jar \
  -H "Content-Type: application/json" \
  -d '{"title": "Solid for task sync", "body": "Wired it into our SDR board in an afternoon."}'
```

Both are brand-scoped session calls — the brand and user are resolved
server-side from the session, not passed in the body, so a reviewer can't
impersonate another brand.

A brand can edit its own review (`PATCH .../reviews/{review_id}`, 3/hr/user).
Ownership is enforced by comparing the review's `user_id`; a cross-user edit
returns `404`.

---

## Section 3 — Reading reviews

The list is public — no auth, hidden rows filtered, newest first:

```bash
curl -s "https://market.opvs.ai/api/v1/registry/packages/@your-vendor/your-skill/reviews"
```

This is what the package detail page renders. The aggregate star average and
distribution come from `package_ratings`.

---

## Section 4 — Moderation

Moderation is **soft** and post-hoc — there's no pre-publish queue. A review
goes live the moment it's submitted. A super-admin can **hide** a review with a
recorded reason:

```
PATCH  /api/v1/marketplace/reviews/{review_id}/hide     → hidden_at + hidden_reason set
DELETE /api/v1/marketplace/reviews/{review_id}/hide     → un-hide
```

Hidden rows stay in the database (a partial index `WHERE hidden_at IS NULL`
keeps the public list fast) — hiding removes a review from view, it doesn't
delete it, so the action is auditable and reversible.

---

## Section 5 — Vendor replies

You can reply to a review, but today it's **admin-mediated**. The reply is
written through an admin route and stored in the review's `vendor_reply_body`:

```
POST /api/v1/marketplace/reviews/{review_id}/vendor-reply
```

First-party vendor replies (you replying directly, authenticated as your vendor)
arrive when vendor-auth ships. Until then, route your reply text through OPVS and
it's attached to the review.

---

## Section 6 — What's not in the model yet

Be aware of the edges so you don't design around them:

- No helpful/unhelpful voting on reviews.
- No review images or attachments.
- No "installed for N days" reviewer segmentation.
- No automated spam detection — moderation is human, post-hoc, and reason-logged.

If you hit abuse (a competitor review-bombing, off-topic content), flag it to
OPVS — the hide action with a recorded reason is the lever, and it's a human
call, not an algorithm.

---

## See also

- [badges.md](badges.md) — tier and security score, the other trust signals on
  the detail page.
- [analytics.md](analytics.md) — the aggregate rating distribution as a metric.
- [../reference/api.md](../reference/api.md) — the full ratings/reviews endpoint
  table.
