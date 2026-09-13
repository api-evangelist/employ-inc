---
name: employ-inc-search-content
description: Search and retrieve Employ Inc's published research, blog posts, glossary and press releases from its WordPress REST API.
api: employ-inc:employ-inc-content-api
generated: '2026-09-13'
method: generated
source: openapi/employ-inc-content-api-openapi.yml
operations:
  - listSearch
  - listPosts
  - listResources
  - listNews
  - listGlossary
  - listCategories
  - listTopics
---

# Search Employ published content

Employ, Inc.'s corporate site is fully readable as JSON at
`https://www.employinc.com/wp-json/wp/v2`. Reads on published content are anonymous.

## Steps

1. **Broad lookup** — `listSearch` (`GET /search?search={term}`) spans every public content type and
   returns `id`, `title`, `url`, `type` and `subtype` per hit. Use it when you do not know which
   content type holds the answer.
2. **Narrow by type** once `subtype` tells you where the answer lives:
   - `listPosts` (`GET /posts`) — the Employ blog. 32 records at last count.
   - `listResources` (`GET /resource`) — the gated research: Recruiter Nation Report, Job Seeker
     Nation Report, benchmarks, buyer's guides, analyst reports.
   - `listNews` (`GET /news_item`) — press releases.
   - `listGlossary` (`GET /glossary_terms`) — recruiting and TA definitions.
3. **Filter and page.** Every collection accepts `search`, `per_page` (max 100), `page`, `offset`,
   `order`, `orderby`, `after` and `before`. Taxonomy filters take term ids, not slugs: fetch
   `listCategories` or `listTopics` first and map the label to its `id`.
4. **Trim the payload.** Add `_fields=id,title,link,date,excerpt` — the default record is large
   because it carries rendered HTML. Add `_embed` only when you need the author or featured image
   inlined.

## Paging contract

Read `X-WP-Total` and `X-WP-TotalPages` from the response headers, and follow the RFC 8288 `Link`
header's `rel="next"`. Do not increment `page` blindly past `X-WP-TotalPages` — that returns a
`rest_post_invalid_page_number` error, not an empty list.

## Two things to expect

- `GET /users` returns **401** and `GET /comments` returns **403** on this installation. Author names
  are still reachable through `_embed` on a post; do not retry the closed collections.
- Read routes are served `cache-control: public, max-age=604800` behind Fastly, so a freshly
  published post can lag the API by up to a week. If freshness matters, cross-check
  `https://www.employinc.com/sitemap.xml`, which carries per-section `lastmod` dates.
