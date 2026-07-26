---
name: Track EDDA Technology news and press releases
description: Read EDDA Technology's press releases and news announcements from its public content API, with correct pagination and date filtering.
api: openapi/edda-wordpress-openapi.yml
operations:
  - listPosts
  - getPost
  - listCategories
  - listUsers
generated: '2026-07-20'
method: generated
---

# Track EDDA Technology news

EDDA Technology, Inc. (Princeton, NJ) publishes its press releases and media coverage through the
public WordPress REST API at `https://www.eddatech.com/wp-json`. The surface is read-only and
anonymous — send no `Authorization` header.

## Base

```
https://www.eddatech.com/wp-json
```

No credential is required. Any route that returns `401 rest_forbidden` is administrative and out of
scope: EDDA issues no public developer credentials, so do not attempt to authenticate.

## Steps

1. **List the most recent announcements** — `listPosts`
   `GET /wp/v2/posts?per_page=20&orderby=date&order=desc`
   There were 55 posts at last count. Read `X-WP-Total` and `X-WP-TotalPages` from the response
   headers rather than guessing how much is left.

2. **Page through the archive** — `listPosts`
   Increment `page`. `per_page` is capped at 100; asking for more returns
   `400 rest_invalid_param` with the exact bound in `data.details.per_page.message`. Alternatively
   follow the `rel="next"` URL in the `Link` response header.

3. **Filter to a window** — `listPosts`
   `GET /wp/v2/posts?after=2025-01-01T00:00:00&orderby=date&order=asc`
   `after`, `before`, `modified_after` and `modified_before` take ISO 8601 values.

4. **Keep responses small** — `listPosts`
   `GET /wp/v2/posts?_fields=id,date,slug,link,title&per_page=50`
   `_fields` trims the payload; add `_embed` only when you actually need the author and featured
   image inlined under `_embedded`.

5. **Read one announcement in full** — `getPost`
   `GET /wp/v2/posts/{id}`
   `title.rendered` and `content.rendered` contain HTML, not plain text — strip or render it before
   quoting. An unknown or unpublished ID returns `404 rest_post_invalid_id`.

6. **Resolve supporting references** — `listCategories`, `listUsers`
   Post objects carry `categories` (integer IDs) and `author` (integer ID). Resolve them with
   `GET /wp/v2/categories` and `GET /wp/v2/users`, or skip the round trips by adding `_embed` in
   step 5.

## Rules

- Errors use the WordPress envelope `{"code","message","data":{"status"}}`, **not** RFC 9457. Branch
  on `code`, not on message text. See `errors/edda-problem-types.yml`.
- There is no idempotency contract and no documented rate limit. All operations here are `GET`, so
  retries are safe — still back off on repeated failures rather than hammering the host.
- Never write. `POST`/`PUT`/`DELETE` on these routes will be rejected, and attempting them against a
  production hospital-facing vendor site is inappropriate.
- Dates are returned in site-local time with `gmt_offset: 0`; `date_gmt` is authoritative.
