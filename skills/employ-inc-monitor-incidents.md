---
name: employ-inc-monitor-incidents
description: Read the Employ incident timeline, including per-update component transitions, and decide whether anything is currently broken.
api: employ-inc:employ-inc-status-api
generated: '2026-09-13'
method: generated
source: openapi/employ-inc-status-api-openapi.yml, examples/employ-inc-status-incidents.json
operations:
  - listUnresolvedIncidents
  - listIncidents
  - listScheduledMaintenances
  - listActiveMaintenances
---

# Monitor Employ incidents

## Steps

1. **Is anything broken right now?** Call `listUnresolvedIncidents` — `GET
   /incidents/unresolved.json`. An empty `incidents[]` is the normal, healthy answer. Do not treat
   an empty array as an error.
2. **What happened recently?** Call `listIncidents` — `GET /incidents.json`. Each incident carries
   `name`, `status` (`investigating` → `identified` → `monitoring` → `resolved` → `postmortem`),
   `impact` (`none` | `minor` | `major` | `critical`), `started_at`, `resolved_at` and a `shortlink`.
3. **What changed, and to what?** Walk `incident_updates[]` oldest-first. Each update has a `body`
   written by Employ and an `affected_components[]` array where every entry names a component and
   its `old_status` → `new_status` transition. That array — not the incident title — is where the
   actual impact lives.
4. **Anything planned?** Call `listScheduledMaintenances` or `listActiveMaintenances`. Both have
   always returned empty on this page.

## What the record actually looks like

Five incidents exist in total. The most recent, `Service Disruption Due to Cloudflare Outage`
(2026-06-22), resolved with `degraded_performance` → `operational` on `AI Interview Companion` and a
body stating that Employ's platform was **not** in fact impacted by the broader Cloudflare event.
Read the `body`; an incident being open does not always mean Employ is down.

## Push instead of poll

Employ's status page offers webhook, Microsoft Teams, email and SMS subscriptions, plus RSS, Atom
and JSON history feeds — see `asyncapi/employ-inc-status-webhooks.yml`. Employ publishes no schema
for the delivered webhook payload, so if you need a guaranteed shape, poll `/incidents.json` and
model it from `openapi/employ-inc-status-api-openapi.yml` instead.
