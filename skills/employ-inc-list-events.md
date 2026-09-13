---
name: employ-inc-list-events
description: List Employ Inc events and webinars with venue and organizer inlined, and pick the right one of the three overlapping event contracts.
api: employ-inc:employ-inc-events-calendar-rest-api
generated: '2026-09-13'
method: generated
source: openapi/employ-inc-events-calendar-rest-api-openapi.json, openapi/employ-inc-tec-events-rest-api-openapi.json
operations:
  - GET /events
  - GET /events/{id}
  - GET /events/by-slug/{slug}
  - GET /venues
  - GET /organizers
  - GET /categories
---

# List Employ events and webinars

## Pick the contract first

The same events are served through **three** different contracts, and nothing on Employ's site says
which to use:

| Base | Contract | Use it when |
|---|---|---|
| `https://www.employinc.com/wp-json/tribe/events/v1` | OpenAPI 3.0.0 at `/doc` | **Default.** Richest read surface — by-slug addressing, categories and tags, venue and organizer inlined on each event. |
| `https://www.employinc.com/wp-json/tec/v1` | OpenAPI 3.0.4 at `/docs` | Newer generation, narrower: events, organizers, venues only. No by-slug, no categories or tags. |
| `https://www.employinc.com/wp-json/wp/v2/tribe_events` | derived | Only when you want the raw WordPress post envelope rather than the event envelope. |

Neither event contract references the other and neither is marked deprecated. Prefer
`tribe/events/v1` for reads, and re-check before building anything long-lived.

## Steps

1. `GET /events` on `tribe/events/v1`. Filter with `start_date`, `end_date`, `categories`, `tags`,
   `venue`, `organizer`, `search`, `page` and `per_page`. The response envelope carries `events[]`,
   `total`, `total_pages` and `rest_url`, plus `next_rest_url` / `previous_rest_url` for paging —
   follow those rather than constructing page URLs yourself.
2. Each event inlines `venue` and `organizer[]` as objects, so a listing call is usually enough — no
   N+1 fetch needed.
3. For a single event, prefer `GET /events/by-slug/{slug}` over `GET /events/{id}`. Slugs are stable
   and human-readable; WordPress ids are per-installation and are not.
4. `GET /venues`, `GET /organizers`, `GET /categories` and `GET /tags` enumerate the supporting
   collections when you need to build a filter.

## Before you write

`POST` and `DELETE` are declared on every event, venue and organizer path, but they require an
authenticated WordPress application password on Employ's own CMS — this is staff access, not a
customer API. If you do hold one:

- **There is no idempotency mechanism.** No `Idempotency-Key`, no client request id. A retried
  `POST` after a timeout creates a duplicate event. Read before you re-send.
- **`DELETE` is a soft delete** — it trashes the record unless you pass `force=true`, which is
  unrecoverable. Employ publishes **no** retention window for trashed records, so do not assume one.

## Contract-quality note

The published event contracts declare **no `operationId` on any operation**, which is why this skill
refers to operations by method and path. Bind by path, not by a generated id.
