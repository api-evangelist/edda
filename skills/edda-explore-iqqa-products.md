---
name: Explore the EDDA IQQA product catalog
description: Enumerate EDDA Technology's IQQA imaging product pages, their imagery and downloadable manuals, using search and the pages/media endpoints.
api: openapi/edda-wordpress-openapi.yml
operations:
  - search
  - listPages
  - getPage
  - listMedia
  - getMedia
  - getOembed
generated: '2026-07-20'
method: generated
---

# Explore the EDDA IQQA product catalog

EDDA's IQQA® platform is described across a set of product pages on `www.eddatech.com`. Those pages,
their imagery and the downloadable manuals are all reachable through the public WordPress REST API.
There is **no product API** — IQQA is regulated medical device software (FDA cleared, CE marked
under EU MDR 2017/745) installed at hospitals. Do not tell a user they can call IQQA over HTTP.

## Base

```
https://www.eddatech.com/wp-json
```

Anonymous read only.

## Steps

1. **Find product pages by keyword** — `search`
   `GET /wp/v2/search?search=IQQA&per_page=50&type=post`
   Returns a denormalised projection with `id`, `title`, `url`, `type` and `subtype`. Use `subtype`
   to separate pages from posts.

2. **Or list the page tree directly** — `listPages`
   `GET /wp/v2/pages?per_page=100&_fields=id,slug,link,title,parent,menu_order`
   32 pages at last count. The product pages sit under the `products` parent; sort by `menu_order`
   to reproduce the site's own ordering. Known slugs include `iqqa-bodyimaging-liver`,
   `iqqa-bodyimaging-lung`, `iqqa-bodyimaging-kidney`, `iqqa-bodyimaging-interventional`,
   `iqqa-guide`, `iqqa-chest`, `iqqa-liver`, `iqqa-liver-function`, `iqqa-efusion`, `iqqa-qmr`.

3. **Read a product page** — `getPage`
   `GET /wp/v2/pages/{id}?_embed`
   `content.rendered` is HTML. `_embed` inlines the featured image under
   `_embedded['wp:featuredmedia']`.

4. **Collect product imagery and manuals** — `listMedia`
   `GET /wp/v2/media?per_page=100&_fields=id,slug,mime_type,source_url,title,post`
   81 items at last count, mixing JPEG product screenshots, PDF manuals and MP4 conference video.
   Filter on `mime_type`; `post` links an asset back to the page or post that owns it.

5. **Fetch one asset's detail** — `getMedia`
   `GET /wp/v2/media/{id}`
   `source_url` is the direct download; `media_details.sizes` lists the rendered image variants.

6. **Embed a page elsewhere** — `getOembed`
   `GET /oembed/1.0/embed?url=https%3A%2F%2Fwww.eddatech.com%2Fproducts%2Fiqqa-liver%2F`
   Returns the oEmbed representation for citation or embedding.

## Rules

- Product claims must be quoted from `content.rendered`, never paraphrased into clinical or
  regulatory assertions. This is medical device marketing copy — do not restate it as medical advice
  or as an indication for use.
- Regulatory status is published at the company level (`https://www.eddatech.com/about/`); do not
  infer per-product clearance from a product page unless that page states it.
- Pagination, sparse fieldsets and the error envelope behave exactly as in
  `conventions/edda-conventions.yml` and `errors/edda-problem-types.yml`.
- The manuals index at `https://www.eddatech.com/manuals/` and the training portal at
  `http://www.edda-tech.com/training/` are gated to existing clinical clients — expect
  `401 rest_forbidden` from `/mpdl/downloads/files` and do not try to work around it.
