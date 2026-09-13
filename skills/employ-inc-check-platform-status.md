---
name: employ-inc-check-platform-status
description: Answer "is the Employ platform up?" from Employ Inc's own public status API, with the coverage caveat that matters.
api: employ-inc:employ-inc-status-api
generated: '2026-09-13'
method: generated
source: openapi/employ-inc-status-api-openapi.yml
operations:
  - getStatusIndicator
  - getStatusSummary
  - listComponents
---

# Check Employ platform status

Employ, Inc. publishes an anonymous Atlassian Statuspage API at
`https://status.employinc.com/api/v2`. No key, no account, no headers. This is the only Employ
surface an agent can call today with zero setup.

## Steps

1. For a one-line answer, call `getStatusIndicator` — `GET /status.json`. Read
   `status.indicator` (`none` | `minor` | `major` | `critical` | `maintenance`) and
   `status.description` (e.g. `All Systems Operational`). Branch on `indicator`, never on the
   English description.
2. For anything more, call `getStatusSummary` — `GET /summary.json` — once instead of fanning out.
   It returns `components[]`, `incidents[]` (unresolved only), `scheduled_maintenances[]` and the
   same rollup `status` in a single document.
3. To enumerate what is actually watched, call `listComponents` — `GET /components.json` — and read
   each `components[].name` and `components[].status`.

## Report this caveat, every time

**The page monitors exactly one component: `AI Interview Companion`.** JazzHR, Lever and Jobvite —
the platforms that carry Employ's customer traffic — are not on it. `All Systems Operational` from
this API means that one AI feature is healthy. It is **not** an answer about whether a customer's
ATS is reachable. If the question was about an ATS, say so and point at that brand's own status
surface instead.

## Conventions that apply

- Every response is wrapped in a `page` object identifying the status page (`id` `5t2j43rsvdfx`).
  The payload you want sits beside it.
- Nothing is paginated. Each endpoint returns its complete current collection.
- There is no rate-limit header and no published limit (`rate-limits/employ-inc-rate-limits.yml`).
  Poll politely — once a minute is plenty for a page with five incidents in its whole history.
- An unknown path returns a bare `404` with no body. There is no error envelope to parse.
