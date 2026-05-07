---
tags: [rest-api]
---

# Schedules API: v2 → v3 Upgrade Guide

Side-by-side examples showing how common requests against PagerDuty's existing public Schedules API (v2) translate to the new shift-based schedules API (v3). The doc covers a few core endpoint comparisons (list schedules, get a single schedule, create an override) and worked end-to-end examples for common automation tasks (reading on-call across all schedules, post-PR-merge claim of coverage, offboarding a user from a rotation).

---

## Conventions

### Base URLs

| API | Base URL |
| :---- | :---- |
| v2 (existing public) | `https://api.pagerduty.com` |
| v3 (shift-based schedules) | `https://api.pagerduty.com` (same host; routes prefixed `/v3/schedules`) |

### Headers

v2 and v3 use the same standard PagerDuty REST API headers.

```
Authorization: Token token=YOUR_API_KEY
Accept: application/vnd.pagerduty+json;version=2
Content-Type: application/json
```

### Schedule IDs

v2 and v3 use the same opaque schedule IDs (e.g. `PL5FQHC`). A schedule that has been upgraded keeps its ID — references from escalation policies, teams, URLs, and integrations are preserved. Only the **shape** of the schedule changes.

### Type discriminators

Once a schedule is upgraded, its `type` flips:

| Schedule kind | `type` value | Detail endpoint to call |
| :---- | :---- | :---- |
| Layer-based (v2-shaped) | `schedule` | `https://api.pagerduty.com/schedules/{id}` |
| Shift-based (v3-shaped) | `schedule_v3` | `https://api.pagerduty.com/v3/schedules/{id}` |

In list responses, references use the corresponding `_reference` form: `schedule_reference` vs. `schedule_v3_reference`. Sub-resources (overrides, custom shifts, rotations, events) live under the same prefix as the schedule they belong to.

### When you don't know whether a schedule has been upgraded

If you have a schedule ID but don't yet know whether the schedule is layer-based or shift-based, query the v3 endpoint first and fall back to v2 on a `400` response. v3 returns `400` when the ID resolves to a schedule that hasn't been upgraded; v2 will then succeed for the same ID.

```py
import requests

def get_schedule(schedule_id, params, headers):
    """Fetch a schedule whose type isn't known in advance."""
    resp = requests.get(
        f"https://api.pagerduty.com/v3/schedules/{schedule_id}",
        headers=headers,
        params=params,
    )
    if resp.status_code == 400:
        # Schedule is still layer-based — fall back to v2.
        resp = requests.get(
            f"https://api.pagerduty.com/schedules/{schedule_id}",
            headers=headers,
            params=params,
        )
    resp.raise_for_status()
    return resp.json()
```

Once you have a successful response, branch on `schedule["type"]` for any shape-specific handling.

---

## 1. List schedules

### v2 — `GET /schedules`

```shell
curl -G https://api.pagerduty.com/schedules \
  -H 'Authorization: Token token=YOUR_API_KEY' \
  -H 'Accept: application/vnd.pagerduty+json;version=2' \
  --data-urlencode 'limit=25' \
  --data-urlencode 'offset=0' \
  --data-urlencode 'query=primary'
```

Response (abridged):

```json
{
  "schedules": [
    {
      "id": "PL5FQHC",
      "type": "schedule",
      "summary": "Primary On-Call",
      "self": "https://api.pagerduty.com/schedules/PL5FQHC",
      "html_url": "https://acme.pagerduty.com/schedules/PL5FQHC",
      "name": "Primary On-Call",
      "time_zone": "America/Los_Angeles",
      "description": "..."
    }
  ],
  "limit": 25,
  "offset": 0,
  "more": false,
  "total": null
}
```

### v3 — `GET /v3/schedules`

```shell
curl -G https://api.pagerduty.com/v3/schedules \
  -H 'Authorization: Token token=YOUR_API_KEY' \
  -H 'Accept: application/vnd.pagerduty+json;version=2' \
  --data-urlencode 'limit=100' \
  --data-urlencode 'offset=0'
```

Response (abridged):

```json
{
  "schedules": [
    {
      "id": "PL5FQHC",
      "type": "schedule_v3_reference",
      "summary": "Engineering On-Call",
      "self": "https://api.pagerduty.com/v3/schedules/PL5FQHC",
      "html_url": "https://acme.pagerduty.com/schedules/PL5FQHC"
    }
  ],
  "limit": 100,
  "offset": 0,
  "more": false
}
```

### What changed

- **List items are reference-shaped.** Each item carries only `id`, `type`, `summary`, `self`, and `html_url`. Fields like `name`, `description`, and `time_zone` move to the detail endpoint.
- **`type` is `schedule_v3_reference`** for items in a v3 list.
- **`self` URL points at the v3 path** (`/v3/schedules/{id}`).
- **No `query` parameter.** v2 supports `query=...` for substring filtering on the schedule name; v3's list endpoint does not. Filter client-side after fetching, or look up specific schedules by ID.
- **No `include_legacy` parameter.** v3 returns only shift-based schedules — there's no public flag to widen the set.
- **Pagination defaults differ.** v3's `limit` defaults to 100 (max 1000); v2's defaults to 25 (max 100). v3 also omits `total` from the response — only `limit`, `offset`, and `more` are returned.

### What stayed the same

- IDs and `html_url` are stable across the two versions.
- `limit` and `offset` query parameters work the same way (just with different defaults and caps).
- Pagination semantics (`more` flag, `offset`/`limit` cursoring) are unchanged.

### Upgrade tip

There is no single endpoint that returns both layer-based and shift-based schedules. To see all schedules on the account, call both list endpoints:

```shell
# layer-based schedules
curl https://api.pagerduty.com/schedules ...

# shift-based schedules
curl https://api.pagerduty.com/v3/schedules ...
```

Each item's `type` tells you which detail endpoint to call next: `schedule_reference` → `GET /schedules/{id}`, `schedule_v3_reference` → `GET /v3/schedules/{id}`.

---

## 2. Get a single schedule

This is the operation where the response shape diverges most. The high-level rule:

Identity, metadata, teams/EPs, and iCal URLs all carry over unchanged. Anything that exposes the *internal layer structure* of a schedule will be replaced with rotations and events. The final-schedule view changes shape too — see below.

### v2 — `GET /schedules/{id}`

```shell
curl -G https://api.pagerduty.com/schedules/PL5FQHC \
  -H 'Authorization: Token token=YOUR_API_KEY' \
  -H 'Accept: application/vnd.pagerduty+json;version=2' \
  --data-urlencode 'time_zone=UTC' \
  --data-urlencode 'since=2026-04-29T00:00:00Z' \
  --data-urlencode 'until=2026-05-29T00:00:00Z'
```

Response (abridged):

```json
{
  "schedule": {
    "id": "PL5FQHC",
    "type": "schedule",
    "summary": "Primary On-Call",
    "self": "https://api.pagerduty.com/schedules/PL5FQHC",
    "html_url": "https://acme.pagerduty.com/schedules/PL5FQHC",
    "name": "Primary On-Call",
    "description": "...",
    "time_zone": "America/Los_Angeles",
    "escalation_policies": [ { "id": "PESC123", "type": "escalation_policy_reference" } ],
    "teams":               [ { "id": "PTEAM01", "type": "team_reference" } ],
    "http_cal_url": "https://acme.pagerduty.com/private/...",
    "web_cal_url":  "webcal://acme.pagerduty.com/private/...",

    "schedule_layers": [
      {
        "id": "PSLAYER1",
        "name": "Layer 1",
        "rotation_virtual_start": "2026-01-01T00:00:00-08:00",
        "rotation_turn_length_seconds": 604800,
        "users": [ { "user": { "id": "PUSER01", "type": "user_reference" } } ],
        "restrictions": [
          {
            "type": "weekly_restriction",
            "start_day_of_week": 1,
            "start_time_of_day": "09:00:00",
            "duration_seconds": 28800
          }
        ],
        "rendered_schedule_entries": [ /* rendered shifts for [since, until] */ ]
      }
    ],
    "overrides_subschedule": {
      "rendered_schedule_entries": [ /* override entries in [since, until] */ ]
    },
    "final_schedule": {
      "rendered_schedule_entries": [ /* merged final layer */ ]
    },

    "oncall": { "user": { "id": "PUSER01", "type": "user_reference" } },
    "users":  [ { "id": "PUSER01", "type": "user_reference" } ]
  }
}
```

### v3 — `GET /v3/schedules/{id}`

```shell
curl -G https://api.pagerduty.com/v3/schedules/PL5FQHC \
  -H 'Authorization: Token token=YOUR_API_KEY' \
  -H 'Accept: application/vnd.pagerduty+json;version=2' \
  --data-urlencode 'time_zone=UTC' \
  --data-urlencode 'since=2026-04-29T00:00:00Z' \
  --data-urlencode 'until=2026-05-29T00:00:00Z' \
  --data-urlencode 'include[]=final_schedule'
```

Response (abridged):

```json
{
  "schedule": {
    "id": "PL5FQHC",
    "type": "schedule_v3",
    "name": "Primary On-Call",
    "description": "...",
    "time_zone": "America/Los_Angeles",
    "self": "https://api.pagerduty.com/v3/schedules/PL5FQHC",
    "html_url": "https://acme.pagerduty.com/schedules/PL5FQHC",
    "escalation_policies": [ { "id": "PESC123", "type": "escalation_policy_reference" } ],
    "teams":               [ { "id": "PTEAM01", "type": "team_reference" } ],
    "http_cal_url": "https://acme.pagerduty.com/private/...",
    "web_cal_url":  "webcal://acme.pagerduty.com/private/...",

    "rotations": [
      {
        "id": "ABCDEFGHIJKLMNOPQRSTUVWXY2",
        "type": "schedule_rotation",
        "events": [
          {
            "id": "ABCDEFGHIJKLMNOPQRSTUVWXY3",
            "type": "schedule_event",
            "name": "Weekday business hours",
            "start_time": {
              "date_time": "2026-01-05T09:00:00-08:00",
              "time_zone": "America/Los_Angeles"
            },
            "end_time": {
              "date_time": "2026-01-05T17:00:00-08:00",
              "time_zone": "America/Los_Angeles"
            },
            "effective_since": "2026-01-05T17:00:00Z",
            "effective_until": null,
            "recurrence": ["RRULE:FREQ=WEEKLY;BYDAY=MO,TU,WE,TH,FR"],
            "assignment_strategy": {
              "type": "rotating_member_assignment_strategy",
              "shifts_per_member": 1,
              "members": [
                { "type": "user_member", "user_id": "PUSER01" }
              ]
            }
          }
        ]
      }
    ],

    "final_schedule": {
      "type": "final_schedule",
      "rendered_coverage_percentage": 100,
      "computed_shift_assignments": [
        {
          "type": "computed_shift_assignment",
          "start_time": "2026-04-29T16:00:00Z",
          "end_time":   "2026-04-30T00:00:00Z",
          "member":     { "type": "user_member", "user_id": "PUSER01" },
          "source": {
            "type": "schedule_rotation",
            "rotation_id": "ABCDEFGHIJKLMNOPQRSTUVWXY2"
          }
        }
      ]
    }
  }
}
```

### What stays the same

These fields are populated identically (down to the value) for the same underlying schedule:

- `id`, `name`, `description`
- `time_zone`
- `escalation_policies` and `teams` references
- `http_cal_url`, `web_cal_url`
- `html_url` (UI link is unchanged)

### What changes

| v2 field | v3 equivalent | Notes |
| :---- | :---- | :---- |
| `type: "schedule"` | `type: "schedule_v3"` | Discriminator. Branch on this if you handle both. |
| `self: /schedules/{id}` | `self: /v3/schedules/{id}` | Path prefix differs. |
| `schedule_layers[]` | `rotations[]` | Whole structure replaced; see below. |
| `overrides_subschedule` | (separate sub-resource) | Overrides moved to `GET /v3/schedules/{id}/overrides`. Not returned inline. |
| `final_schedule.rendered_schedule_entries[]` | `final_schedule.computed_shift_assignments[]` | Flat list of "who is on when" is preserved; field names differ. Returned only with `include[]=final_schedule`. The `final_schedule` itself now also carries `type: "final_schedule"` and `rendered_coverage_percentage` (0–100). |
| `oncall` (singular) | — | Not part of the v3 schedule response. To recover "who's on call right now," pass `since=now` and `until=now+1s` with `include[]=final_schedule` and read the first entry from `final_schedule.computed_shift_assignments[]`. |
| `oncalls[]` | — | Not part of the v3 schedule response. Same recovery as above (use a narrow `final_schedule` window). |
| `since` / `until` query params | Same | Behavior is unchanged: bound the time window for the computed `final_schedule`. |

### Layers vs. rotations: the shape change

The most material change in the response. v2 modeled a schedule as a stack of *layers*, each with its own *restrictions* (time-of-day / day-of-week filters). v3 models it as a list of *rotations*, each containing one or more *events* (think iCal `VEVENT` with an `RRULE`).

| v2 concept | v3 concept |
| :---- | :---- |
| A `schedule_layer` with one or more `restrictions` | A `rotation` with one or more `events`, each carrying an `RRULE` |
| `rotation_virtual_start` + `rotation_turn_length_seconds` | Encoded in the event's `start_time`/`end_time` and `RRULE` |
| `users[]` on the layer (round-robin order) | `assignment_strategy` on the event (`rotating_member_assignment_strategy` or `every_member_assignment_strategy`, with `members[]`) |
| `rendered_schedule_entries[]` per layer | `final_schedule.computed_shift_assignments[]` (merged across rotations) |

Two shape details worth flagging:

- An event's `start_time` and `end_time` are **objects**, not strings: `{"date_time": "2026-01-05T09:00:00-08:00", "time_zone": "America/Los_Angeles"}`. The `time_zone` is part of the value, not a separate query parameter.
- Each `final_schedule.computed_shift_assignments[]` entry has a `source` object (`{type, rotation_id?, shift_id?, override_id?}`) that tells you which rotation, custom shift, or override the shift came from. `source.type` is one of `schedule_rotation`, `custom_shift`, `schedule_rotation_override`, `custom_shift_override`. Per-layer rendered entries are not exposed in v3 — group by `source.rotation_id` if you need per-rotation breakdowns.

### `include[]` behavior

v3 makes more fields opt-in than v2 did. The only valid `include[]` value is **`final_schedule`**.

```
?include[]=final_schedule
```

v3 has no `exclude[]` parameter on the GET endpoint. Fields like `teams`, `escalation_policies`, `http_cal_url`, and `web_cal_url` are returned by default when the schedule has them.

### The `final_schedule` field

Both v2 and v3 expose a `final_schedule` field that flattens "who's on call when" over the requested `[since, until]` window. In v2 it's the merged top-of-stack output of the rotation layers and the override layer; in v3 it's the merged result of stacking rotations, custom shifts, and overrides. The conceptual purpose is the same; the shape and semantics shift in a few notable ways:

|  | v2 | v3 |
| :---- | :---- | :---- |
| Returned by default? | Yes (whenever `since` / `until` are set) | No — must pass `include[]=final_schedule` |
| Wrapper discriminator | (none) | `type: "final_schedule"` |
| Coverage summary | (none) | `rendered_coverage_percentage` (0–100) |
| List field | `rendered_schedule_entries[]` | `computed_shift_assignments[]` |
| Per-entry user data | `entry.user` (full user reference, name inline) | `entry.member` (only `type` and `user_id`) |
| Source attribution | Override entries surfaced separately under `overrides_subschedule` | Each entry's `source` object names the rotation, custom shift, or override it came from |
| Concurrent on-calls | Always one user at a time (the merged final layer collapses) | Can be multiple — separate entries with overlapping windows |
| Empty windows | Omitted from the list | Present, with `member.type = "empty_member"` |

Iterating the field in v2:

```py
schedule = resp.json()["schedule"]
for entry in schedule["final_schedule"]["rendered_schedule_entries"]:
    print(entry["start"], entry["end"], entry["user"]["summary"])
```

The same loop in v3:

```py
schedule = resp.json()["schedule"]
for a in schedule["final_schedule"]["computed_shift_assignments"]:
    if a["member"]["type"] == "empty_member":
        continue
    print(a["start_time"], a["end_time"], a["member"]["user_id"])
```

Same conceptual shape — a flat list of "who is on when" — with different field names and a richer per-entry structure (source attribution and concurrent-on-call support) on the v3 side. Note that v3 returns the on-call user as a `user_id` only; if you need the user's name, fetch it from `GET /users/{user_id}`.

### Upgrade tip

A typical v2 → v3 read path looks like:

1. Call `GET /v3/schedules/{id}?since=...&until=...&include[]=final_schedule`.
2. If you previously relied on `schedule_layers[*].rendered_schedule_entries[]`, replace with `final_schedule.computed_shift_assignments[]` (and group by `source.rotation_id` if you need per-rotation breakdowns).
3. If you previously relied on `oncall` (singular) or `oncalls[]`, request a narrow `final_schedule` window (`since=now`, `until=now+1s`) and read `computed_shift_assignments[].member` to find the active on-call user(s).
4. If you previously read `overrides_subschedule.rendered_schedule_entries[]`, fetch `GET /v3/schedules/{id}/overrides?since=...&until=...` separately.

---

## 3. Create an override (and when to use a custom shift instead)

This is the operation with the largest semantic shift between v2 and v3. Skim "When an override is **not** the right tool in v3" below before translating v2 override calls.

### v2 — `POST /schedules/{id}/overrides`

In v2, an override is a flat statement: "during `[start, end)`, the on-call person for schedule `{id}` is **this user**, regardless of what the rotation layers say." The override layer has top priority and replaces whoever else would have been on-call.

```shell
curl -X POST https://api.pagerduty.com/schedules/PL5FQHC/overrides \
  -H 'Authorization: Token token=YOUR_API_KEY' \
  -H 'Accept: application/vnd.pagerduty+json;version=2' \
  -H 'Content-Type: application/json' \
  -d '{
    "overrides": [
      {
        "start": "2026-05-10T09:00:00-07:00",
        "end":   "2026-05-10T17:00:00-07:00",
        "user":  { "id": "PUSER02", "type": "user_reference" },
        "time_zone": "America/Los_Angeles"
      }
    ]
  }'
```

Response (abridged):

```json
{
  "overrides": [
    {
      "id": "PXXXXXX",
      "start": "2026-05-10T16:00:00Z",
      "end":   "2026-05-11T00:00:00Z",
      "user":  { "id": "PUSER02", "type": "user_reference" }
    }
  ]
}
```

### v3 — `POST /v3/schedules/{id}/overrides`

In v3, an override is **scoped to a specific source**: either a `rotation_id` or a `custom_shift_id`. The override replaces the assigned member of that source's shifts during the time window — **not** the schedule's on-call as a whole.

```shell
curl -X POST https://api.pagerduty.com/v3/schedules/PL5FQHC/overrides \
  -H 'Authorization: Token token=YOUR_API_KEY' \
  -H 'Accept: application/vnd.pagerduty+json;version=2' \
  -H 'Content-Type: application/json' \
  -d '{
    "overrides": [
      {
        "type": "override_shift",
        "rotation_id": "ABCDEFGHIJKLMNOPQRSTUVWXY2",
        "start_time":  "2026-05-10T16:00:00Z",
        "end_time":    "2026-05-11T00:00:00Z",
        "overridden_member": { "type": "user_member", "user_id": "PUSER01" },
        "overriding_member": { "type": "user_member", "user_id": "PUSER02" }
      }
    ]
  }'
```

Response (abridged):

```json
{
  "overrides": [
    {
      "id": "ABCDEFGHIJKLMNOPQRSTUVWXY4",
      "type": "override_shift",
      "rotation_id": "ABCDEFGHIJKLMNOPQRSTUVWXY2",
      "start_time": "2026-05-10T16:00:00Z",
      "end_time":   "2026-05-11T00:00:00Z",
      "overridden_member": { "type": "user_member", "user_id": "PUSER01" },
      "overriding_member": { "type": "user_member", "user_id": "PUSER02" }
    }
  ]
}
```

### Field-by-field mapping

| v2 field | v3 field | Notes |
| :---- | :---- | :---- |
| `start` | `start_time` | Renamed; ISO-8601 with offset still accepted. |
| `end` | `end_time` | Renamed. |
| `user` | `overriding_member` | The replacement person. v3 uses the `user_member` shape: `{"type": "user_member", "user_id": "PUSER..."}`. |
| (implicit — "whoever is on-call") | **`overridden_member`** | **Required** in v3. Identifies the specific member being replaced (relevant when the source rotation has multiple concurrent members). |
| `time_zone` | (none) | Send absolute UTC (or offset-bearing) timestamps in `start_time` / `end_time`. v3 has no per-override `time_zone`. |
| (none) | **`rotation_id` or `custom_shift_id`** | One of these must be set; setting both is rejected. |
| (none) | **`type`** | Must be `"override_shift"` on each override. |

### The semantic difference (read this)

In **v2**, an override is unconditionally global to the schedule. There's a single override layer, and any override on it wins over every rotation layer for the override window.

In **v3**, a schedule can have multiple rotations producing overlapping coverage. An override must therefore declare *which source* it modifies:

- `rotation_id: ...` → "during `[start_time, end_time)`, replace the assigned member of shifts coming from this rotation." Other rotations in the same window are unaffected.
- `custom_shift_id: ...` → "during `[start_time, end_time)`, replace the assigned member of this specific custom shift."

For a simple, single-rotation schedule (which is what most schedules upgraded from v2 look like immediately after the upgrade), this distinction is invisible — `rotation_id` will be the schedule's only rotation, and the v3 override behaves the same as the v2 override.

It starts to matter once you (or your customer) take advantage of multi-rotation schedules, or once you try to express something v2 overrides could only fudge.

### When an override is **not** the right tool in v3

If you find yourself writing an override that has no good answer to "which `rotation_id` or `custom_shift_id` does this target?", an override probably isn't what you want. Consider a **custom shift** instead in these cases:

#### 1. Filling an empty period

You want someone on-call during a window where no rotation is currently producing shifts (e.g. a one-off emergency coverage block on a holiday when the rotation is paused, or extending coverage past the rotation's `effective_until`).

- v2 way: just `POST /schedules/{id}/overrides` — there's nothing being replaced, but v2 doesn't care.
- v3 way: `POST /v3/schedules/{id}/custom_shifts`. There is no rotation to override against, so trying to express this as an override has no valid `rotation_id`.

```shell
curl -X POST https://api.pagerduty.com/v3/schedules/PL5FQHC/custom_shifts \
  -H 'Authorization: Token token=YOUR_API_KEY' \
  -H 'Accept: application/vnd.pagerduty+json;version=2' \
  -H 'Content-Type: application/json' \
  -d '{
    "custom_shifts": [
      {
        "type": "custom_shift",
        "start_time": "2026-12-25T00:00:00Z",
        "end_time":   "2026-12-26T00:00:00Z",
        "assignments": [
          {
            "type": "shift_assignment",
            "member": { "type": "user_member", "user_id": "PUSER02" }
          }
        ]
      }
    ]
  }'
```

A custom shift takes **exactly one** assignment (`minItems: 1, maxItems: 1`). To put two people on-call simultaneously, create two separate custom shifts with the same `start_time`/`end_time`.

#### 2. Adding coverage rather than swapping a person

You want an *additional* person on-call during a window without removing the existing on-call person — for example, a second responder during a deployment.

- v2 way: there is no clean way; v2 overrides replace the on-call user, they don't augment coverage. Customers often misuse overrides for this and end up with the wrong person paged.
- v3 way: `POST /v3/schedules/{id}/custom_shifts`. Custom shifts add to the schedule rather than replacing rotation output.

#### 3. The window doesn't correspond to a rotation's recurrence at all

You want to express "Alice covers the on-call line for this maintenance window" and the window doesn't line up with any single rotation's shifts (e.g. it spans rotation boundaries, or it's a one-off that has nothing to do with the regular rotation).

- v2 way: one override.
- v3 way: a custom shift is usually the cleaner expression. An override is still valid if you genuinely want to *replace the rotation's contribution* during the window — but if the rotation's contribution is incidental to what you're modeling, custom shift first.

### Quick decision guide

```
Did the v2 override replace a person that the rotation
would otherwise have produced for that exact window?
├── Yes — v3: override on that rotation's id (overriding_member).
└── No / not really — v3: custom shift.
```

A useful sanity check: if removing the override would leave the schedule with the *correct* on-call person, the override is doing replacement work — keep it as an override. If removing the override would leave a *gap* (or remove an extra responder you wanted), what you actually wanted was a custom shift.

### What stays the same

- The endpoint is still `POST` to `/.../{schedule_id}/overrides`.
- The request body still wraps an array under `"overrides"`, supporting batch creation.
- `DELETE /v3/schedules/{id}/overrides/{override_id}` mirrors v2's delete shape (just under the v3 path prefix).
- Permissions to manage overrides are unchanged: a user who could create v2 overrides on a schedule can create v3 overrides on the upgraded schedule.

---

## 4. Worked example: print the current on-call user for every schedule

A common script: List every schedule on the account and print who's on call right now. This is the kind of integration that breaks the day a schedule is upgraded from layer-based to shift-based, because the `/schedules/{id}/users` endpoint that v2 scripts lean on is layer-based and isn't the supported way to read on-call from a shift-based schedule.

We'll start with a v2-only version that works only for layer-based schedules, then evolve it into a hybrid version that handles both shapes.

All examples below assume `pip install requests` and `export PAGERDUTY_API_KEY=...`.

### v2-only: layer-based schedules

```py
#!/usr/bin/env python3
"""Print the current on-call user(s) for every schedule on the account.

Works against the v2 PagerDuty Schedules API. Returns nothing useful for
shift-based (v3-shaped) schedules.
"""
import os
from datetime import datetime, timedelta, timezone
import requests

API_KEY = os.environ["PAGERDUTY_API_KEY"]
BASE = "https://api.pagerduty.com"
HEADERS = {
    "Authorization": f"Token token={API_KEY}",
    "Accept": "application/vnd.pagerduty+json;version=2",
}


def list_schedules():
    """Yield every schedule on the account, one per item."""
    offset, limit = 0, 100
    while True:
        resp = requests.get(
            f"{BASE}/schedules",
            headers=HEADERS,
            params={"limit": limit, "offset": offset},
        )
        resp.raise_for_status()
        body = resp.json()
        yield from body["schedules"]
        if not body["more"]:
            return
        offset += limit


def current_oncall_users(schedule_id):
    """Return the users on call for the schedule right now.

    Asking for `final_schedule` over a 1-second window centered on now
    gives us the active assignment(s) — `final_schedule` is the merged,
    on-call-resolution view of the schedule.
    """
    now = datetime.now(timezone.utc)
    resp = requests.get(
        f"{BASE}/schedules/{schedule_id}",
        headers=HEADERS,
        params={
            "since": now.isoformat(),
            "until": (now + timedelta(seconds=1)).isoformat(),
            "include[]": "final_schedule",
        },
    )
    resp.raise_for_status()
    final = resp.json()["schedule"].get("final_schedule") or {}
    return [
        entry["user"]["summary"]
        for entry in final.get("rendered_schedule_entries", [])
    ]


for schedule in list_schedules():
    users = current_oncall_users(schedule["id"])
    label = ", ".join(users) if users else "(no one on call)"
    print(f"{schedule['summary']}: {label}")
```

This works against layer-based schedules. Shift-based schedules don't appear in the `/schedules` list response, and `GET /schedules/{id}` against a shift-based schedule's ID won't return the `final_schedule.rendered_schedule_entries` shape this script reads from. The hybrid script below dispatches each schedule to the right endpoint by `type`.

### v2 + v3: handles both layer-based and shift-based schedules

```py
#!/usr/bin/env python3
"""Print the current on-call user(s) for every schedule on the account.

Handles both layer-based and shift-based schedules by branching on
the `type` field returned by the schedule list.
"""
import os
from datetime import datetime, timedelta, timezone
import requests

API_KEY = os.environ["PAGERDUTY_API_KEY"]
BASE = "https://api.pagerduty.com"
HEADERS = {
    "Authorization": f"Token token={API_KEY}",
    "Accept": "application/vnd.pagerduty+json;version=2",
}


def list_schedules():
    """Yield every schedule on the account — layer-based AND shift-based.

    The v2 and v3 list endpoints each return only their own schedule
    type, so we call both and chain the results. Each item's `type`
    field tells us which detail endpoint to use later.
    """
    yield from _paginate(f"{BASE}/schedules")
    yield from _paginate(f"{BASE}/v3/schedules")


def _paginate(url):
    offset, limit = 0, 100
    while True:
        resp = requests.get(
            url,
            headers=HEADERS,
            params={"limit": limit, "offset": offset},
        )
        resp.raise_for_status()
        body = resp.json()
        yield from body["schedules"]
        if not body["more"]:
            return
        offset += limit


def layer_based_oncall_users(schedule_id):
    """Layer-based path: who's on this schedule right now."""
    now = datetime.now(timezone.utc)
    resp = requests.get(
        f"{BASE}/schedules/{schedule_id}",
        headers=HEADERS,
        params={
            "since": now.isoformat(),
            "until": (now + timedelta(seconds=1)).isoformat(),
            "include[]": "final_schedule",
        },
    )
    resp.raise_for_status()
    final = resp.json()["schedule"].get("final_schedule") or {}
    return [
        entry["user"]["summary"]
        for entry in final.get("rendered_schedule_entries", [])
    ]


def shift_based_oncall_users(schedule_id):
    """Shift-based path: who's on this schedule right now.

    The v3 GET endpoint returns the computed final schedule for the
    requested time window. A 1-second window centered on "now" gives
    us the active assignment(s).
    """
    now = datetime.now(timezone.utc)
    resp = requests.get(
        f"{BASE}/v3/schedules/{schedule_id}",
        headers=HEADERS,
        params={
            "since": now.isoformat(),
            "until": (now + timedelta(seconds=1)).isoformat(),
            "include[]": "final_schedule",
        },
    )
    resp.raise_for_status()
    final = resp.json()["schedule"].get("final_schedule") or {}
    return [
        a["member"]["user_id"]
        for a in final.get("computed_shift_assignments", [])
        if a["member"]["type"] == "user_member"
    ]


def current_oncall_users(schedule):
    if schedule["type"] == "schedule_v3_reference":
        return shift_based_oncall_users(schedule["id"])
    return layer_based_oncall_users(schedule["id"])


for schedule in list_schedules():
    users = current_oncall_users(schedule)
    label = ", ".join(users) if users else "(no one on call)"
    print(f"{schedule['summary']}: {label}")
```

### What changed and why

Four concrete changes between the v2-only and the hybrid script:

1. **List both endpoints to enumerate every schedule.** `GET /schedules` returns only layer-based schedules; `GET /v3/schedules` returns only shift-based schedules. There is no single endpoint that returns both, so the script paginates each in turn and chains the results. Each list item's `type` field (`schedule_reference` for layer-based, `schedule_v3_reference` for shift-based) is the dispatch signal we use in step 2.

2. **Branch on `schedule["type"]`.** Layer-based items have `type: "schedule_reference"` (or `"schedule"` in some contexts); shift-based items have `type: "schedule_v3_reference"`. The on-call query path differs by shape — there is no single endpoint that answers "who's on call" for both kinds — so you have to dispatch.

3. **Same request, different path and response field for shift-based schedules.** Both halves of the script ask "who is on call during this window?" by fetching `final_schedule` over a 1-second window centered on now. What differs:


|  | v2 (layer-based) | v3 (shift-based) |
| :---- | :---- | :---- |
| Path | `GET /schedules/{id}` | `GET /v3/schedules/{id}` |
| Window param | `since` / `until` | `since` / `until` |
| Include | `include[]=final_schedule` | `include[]=final_schedule` |
| Response field with assignments | `final_schedule.rendered_schedule_entries[]` | `final_schedule.computed_shift_assignments[]` |
| Per-assignment user data | `entry.user` (full user reference: `id`, `summary`, `type`, `html_url`, `self`) | `entry.member` (only `type` and `user_id`) |



   On v3, an assignment's `member` may carry `type: "empty_member"` if the slot is intentionally unassigned; the script filters those out.



4. **Per-entry user data is just an ID on v3.** In v2, each `rendered_schedule_entries[].user` is a full user reference with `id`, `summary`, and `type` inline. In v3, each `computed_shift_assignments[].member` carries only `type` and `user_id`. The script above prints whatever each path natively returns (names from v2, IDs from v3); a real integration that needs names on both sides would resolve the v3 `user_id` via `GET /users/{user_id}`.

Two subtler changes worth calling out:

- **More than one user can be on call at the same time on v3.** A v2 schedule's `final_schedule.rendered_schedule_entries[]` collapses to at most one user at any given moment — the merged final layer always picks a single on-call. A v3 schedule can have multiple rotations producing concurrent shifts, or an event with `every_member_assignment_strategy`, either of which puts more than one user on call simultaneously. Scripts that previously assumed "one user per schedule per moment" (`users[0]`, `oncall.user.summary`, alerting "the on-call person") need to handle a list. The hybrid script above already returns a list and joins names with a comma — if your downstream expects a single name, decide deliberately whether to take the first, list all of them, or surface the multi-coverage state to the operator.

- **No `oncall` (singular) field on v3.** A common v2 pattern is to call `GET /schedules/{id}` and read `schedule.oncall.user.summary`. The v3 schedule response has no `oncall` and no `oncalls` — the supported way to ask "who's on right now" is the narrow `final_schedule` query above. This is a direct consequence of the previous point: there isn't always a single `oncall` to return.

What stays unchanged:

- Pagination on `GET /schedules` (`limit`/`offset`/`more`). The defaults differ between v2 and v3 list endpoints (see the **List schedules** section above), but here we're using the v2 list, so v2 defaults apply.
- Authentication. Both endpoints accept the same `Authorization: Token token=...` header and the same scoped OAuth tokens.
- `GET /users/{id}`. User identity lookup is the same in both worlds; only the schedule endpoints fork.

---

## 5. Worked example: put a user on call for the next hour after a PR merge

A common automation pattern: after a pull request is merged, put the pull request author on call for the next hour so they're paged for any fallout from their change. On v2, this is an override; on v3, the same pattern fits a **custom shift** more cleanly than an override.

### v2: one override on the schedule

```py
#!/usr/bin/env python3
"""Put `user_id` on call for the next hour on a layer-based schedule."""
import os
from datetime import datetime, timedelta, timezone
import requests

API_KEY = os.environ["PAGERDUTY_API_KEY"]
BASE = "https://api.pagerduty.com"
HEADERS = {
    "Authorization": f"Token token={API_KEY}",
    "Accept": "application/vnd.pagerduty+json;version=2",
    "Content-Type": "application/json",
}


def put_oncall_for_one_hour(schedule_id, user_id):
    now = datetime.now(timezone.utc)
    end = now + timedelta(hours=1)
    resp = requests.post(
        f"{BASE}/schedules/{schedule_id}/overrides",
        headers=HEADERS,
        json={
            "overrides": [
                {
                    "start": now.isoformat(),
                    "end":   end.isoformat(),
                    "user":  {"id": user_id, "type": "user_reference"},
                }
            ]
        },
    )
    resp.raise_for_status()
    return resp.json()["overrides"][0]
```

### v3: one custom shift on the schedule

```py
#!/usr/bin/env python3
"""Put `user_id` on call for the next hour on a shift-based schedule."""
import os
from datetime import datetime, timedelta, timezone
import requests

API_KEY = os.environ["PAGERDUTY_API_KEY"]
BASE = "https://api.pagerduty.com"
HEADERS = {
    "Authorization": f"Token token={API_KEY}",
    "Accept": "application/vnd.pagerduty+json;version=2",
    "Content-Type": "application/json",
}


def put_oncall_for_one_hour(schedule_id, user_id):
    now = datetime.now(timezone.utc)
    end = now + timedelta(hours=1)
    resp = requests.post(
        f"{BASE}/v3/schedules/{schedule_id}/custom_shifts",
        headers=HEADERS,
        json={
            "custom_shifts": [
                {
                    "type": "custom_shift",
                    "start_time": now.isoformat(),
                    "end_time":   end.isoformat(),
                    "assignments": [
                        {
                            "type": "shift_assignment",
                            "member": {"type": "user_member", "user_id": user_id},
                        }
                    ],
                }
            ]
        },
    )
    resp.raise_for_status()
    return resp.json()["custom_shifts"][0]
```

### Why custom shift and not override

For the post-merge use case, **a custom shift fits the intent more directly than an override**:

- The intent is "add this person to the on-call set for the next hour," not "replace whoever is scheduled." The pull request author is the *additional* responder, on top of the regular rotation. An override would replace the rotation's on-call user, which is rarely what a post-merge hook actually wants.
- A custom shift doesn't need to look up `overridden_member` — there's nothing being replaced.
- A custom shift doesn't need a `rotation_id` — it stands on its own and survives even if the rotations change shape later.
- A v3 schedule can have any number of users on call simultaneously, so adding one more is a first-class operation, not a workaround.

Create an override on v3 only when you genuinely want to **replace** a specific rotation's assigned user (e.g. "Alice is sick, Bob covers her shift on the primary rotation"). For "add another responder," use custom shift instead.

### What changed and why

- **v2 implicitly replaces; v3 explicitly adds.** v2's override layer wins over rotation layers, so the override-layer entry becomes the on-call. The new user is the sole on-call during the window. v3's custom shift is additive: the rotation's regular on-call user remains on call, and the custom shift's user is *also* on call. v3 lets a schedule have multiple users on call at the same time, and this is the v3-native way to express "another person is also responsible."
- **Stacking behavior of repeated calls also differs.** If the post-merge hook fires three times in an hour on v2, the override layer ends up with three overrides for overlapping windows; the schedule's resolution rules pick one user as on-call. On v3, three custom shifts means all three users are on call simultaneously. If you want to keep that to one ad-hoc responder, see "Keeping at most one ad-hoc responder" below.
- **The `overridden_member` lookup goes away.** Custom shifts don't replace anything, so the `final_schedule` query that the override-based v3 example would need is gone.

### Keeping at most one ad-hoc responder

Sometimes you don't want post-merge custom shifts to pile up — only the most recent pull request author should be on call as the ad-hoc responder. v3 makes that decision explicit: list the existing custom shifts in the window, delete them, then create the new one.

```py
def put_oncall_for_one_hour_replacing_others(schedule_id, user_id):
    now = datetime.now(timezone.utc)
    end = now + timedelta(hours=1)

    # List custom shifts overlapping the window and delete them first.
    resp = requests.get(
        f"{BASE}/v3/schedules/{schedule_id}/custom_shifts",
        headers=HEADERS,
        params={"since": now.isoformat(), "until": end.isoformat()},
    )
    resp.raise_for_status()
    for shift in resp.json().get("custom_shifts", []):
        requests.delete(
            f"{BASE}/v3/schedules/{schedule_id}/custom_shifts/{shift['id']}",
            headers=HEADERS,
        ).raise_for_status()

    # Create the new one.
    return put_oncall_for_one_hour(schedule_id, user_id)
```

Two things worth emphasizing about this variant:

- **Deleting custom shifts only affects ad-hoc coverage.** Rotation-based on-call coverage is untouched; the regular rotation user remains on call regardless of how many custom shifts you delete. To replace the rotation user, you'd need an override on the rotation — a different operation.
- **The "keep or delete" choice didn't exist on v2.** The v2 override layer applied its own resolution rules across stacked overrides (most recent wins, in practice). v3 surfaces the choice as data: each custom shift is a real, addressable resource you can list, delete, or leave alone deliberately.

---

## 6. Worked example: offboarding a user from a rotation

A user is leaving the team and needs to come off a rotation. On v3, the natural shape of this is a two-phase flow built around the **unassigned slot** — a first-class concept where a member position in the rotation is intentionally vacant. The unassigned slot lets you remove a user without committing to a replacement in the same step, and without disturbing anyone else's place in the rotation.

1. **Now** — replace the user with an unassigned slot. They stop being paged immediately. The rotation's cadence stays exactly the same: the offboarded user's seat is just empty until you fill it, and every other member keeps the same on-call weeks they would have had.
2. **Later** — fill the unassigned slot with a replacement user once one is identified.

### v2: edit the rotation layer's `users` list

In v2, a rotation layer's members live in `schedule_layers[*].users[]`. Removing a user is a `PUT /schedules/{id}` with the user dropped from that list:

```py
schedule = requests.get(f"{BASE}/schedules/{schedule_id}", headers=HEADERS).json()["schedule"]

# Remove the offboarded user from each rotation layer
for layer in schedule["schedule_layers"]:
    layer["users"] = [u for u in layer["users"] if u["user"]["id"] != "PUSER01"]

requests.put(
    f"{BASE}/schedules/{schedule_id}",
    headers=HEADERS,
    json={"schedule": schedule},
)
```

v2 has no "unassigned slot" concept. Removing a user shrinks the rotation: the remaining users absorb the vacated shifts and the rotation cadence changes for everyone. A team that used to rotate weekly across three people now rotates weekly across two, so each person's on-call frequency goes up and the calendar slot they used to hold may shift to a different week. This churn — getting unrelated people moved around when one user leaves without an immediate replacement — is one of the rough edges of v2 rotation membership; v3 fixes it by treating the vacant slot as an explicit, first-class state.

### v3: delete the active event, create a successor

Members live on each event's `assignment_strategy.members[]`. The catch: an active event (one that's already producing shifts) can't have its members changed in place. Swapping members on a live rotation always means **terminating the current event** and **starting a new one** with the swapped members from the cutover time onward.

The simplest way to terminate an active event is to **delete** it. A `DELETE` on an event in v3 doesn't erase history: shifts the event already produced — past on-call assignments, paged users, escalations — remain on the schedule's record. What it does is stop the event from producing any further shifts, equivalent to capping it at the deletion time. So the offboarding flow per phase is two calls: `DELETE` the active event, then `POST` a successor with the new members.

#### Phase 1 — replace the offboarded user with `empty_member`

Delete the active event:

```py
from datetime import datetime, timezone

now = datetime.now(timezone.utc)

requests.delete(
    f"{BASE}/v3/schedules/{schedule_id}/rotations/{rotation_id}/events/{event_id}",
    headers=HEADERS,
)
```

Then `POST` a successor with the same shape but the offboarded user replaced by `empty_member`:

```py
requests.post(
    f"{BASE}/v3/schedules/{schedule_id}/rotations/{rotation_id}/events",
    headers=HEADERS,
    json={"event": {
        "name": "Primary on-call",
        "start_time": {"date_time": "...", "time_zone": "America/Los_Angeles"},
        "end_time":   {"date_time": "...", "time_zone": "America/Los_Angeles"},
        "effective_since": now.isoformat(),
        "recurrence": ["RRULE:FREQ=WEEKLY;BYDAY=MO,TU,WE,TH,FR"],
        "assignment_strategy": {
            "type": "rotating_member_assignment_strategy",
            "shifts_per_member": 1,
            "members": [
                {"type": "empty_member"},                              # was PUSER01
                {"type": "user_member", "user_id": "PUSER02"},
                {"type": "user_member", "user_id": "PUSER03"},
            ],
        },
    }},
)
```

`name`, `start_time`, `end_time`, and `recurrence` are typically copied from the event being replaced — fetch it first via `GET /v3/schedules/{id}/rotations/{rotation_id}/events/{event_id}`. The `members` list is the only thing that changes.

After these two calls, the original event still produced shifts up to `now`, and the successor takes over from `now` with the rotation including an unassigned slot. `final_schedule` reads for windows after `now` show `member.type = "empty_member"` for the gap.

#### Phase 2 — fill the unassigned slot with a new user

Same two-call pattern, against the now-active successor event from Phase 1. List the rotation's events to find the active `event_id`, then `DELETE` it and `POST` another successor whose `assignment_strategy.members` fills the slot:

```py
members = [
    {"type": "user_member", "user_id": "PUSER04"},                   # was empty
    {"type": "user_member", "user_id": "PUSER02"},
    {"type": "user_member", "user_id": "PUSER03"},
]
```

### What changed and why

- **Active event members are immutable.** v3 doesn't let you change the members of an event that's already producing shifts. To change *who's* on the rotation, you stop the running event and start a new one alongside it. `DELETE` is the cleanest way to stop the running event because it preserves history — shifts already produced keep their member assignments — and the deletion time becomes the implicit cutover.
- **`empty_member` keeps the rotation's cadence intact.** Marking a position vacant is a normal rotation state in v3, not a hack. The rotation's other members continue producing shifts on the same weeks they would have anyway — replacing a user with `empty_member` does not shrink the cycle or shuffle anyone else's schedule. `final_schedule.computed_shift_assignments[]` surfaces the empty windows as `member.type = "empty_member"` so reporting and dashboards can flag the gap explicitly. (Compare with v2: dropping a user from `users[]` shrinks the rotation and changes everyone's on-call frequency.)
- **Two phases, same shape.** Phase 1 swaps user → empty; Phase 2 swaps empty → user. Both use the same delete-then-POST pattern; the only difference is the `from` and `to` members in the new event's `members[]`.
- **Future events can be edited in place.** If the event being changed has an `effective_since` in the future (it hasn't started producing shifts yet), `PUT` its members directly — no delete-and-recreate needed. The pattern above is the safe default that works regardless of timing.
