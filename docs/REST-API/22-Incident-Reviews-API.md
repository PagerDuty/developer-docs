---
tags: [rest-api]
---

# Incident Reviews API

A post-incident review captures what happened during an incident: a written narrative broken into
sections, a timeline of key moments, and the action items the review produced. The Incident Reviews
API lets you create and manage those reviews programmatically, so that a review can be opened,
filled in, and completed by your own tooling rather than only through the PagerDuty web UI.

This API is part of [REST API v2](../../docs/REST-API/01-Overview.md) and follows the conventions of
our other [RESTful API endpoints](../../docs/REST-API/05-Endpoints.md).

## The Review is Addressed by Incident

An incident has at most one review, so a review has no ID of its own in the URL — you address it
through the incident it belongs to:

```
https://api.pagerduty.com/incidents/{incident_id}/review
```

`{incident_id}` is the incident's ID, for example `P1234ABC`. Because the review is a singleton
under its incident, there is no index endpoint and no `POST` to a collection: creating a review is a
`POST` to the review path itself.

## Authentication and Scopes

All requests require [authentication](../../docs/REST-API/02-Authentication.md). The account is
always resolved from your credentials, so there is no account field in any request body.

| Requests | Scoped OAuth scope |
|:---------|:-------------------|
| `GET` | `incident_reviews.read` |
| `POST`, `PUT`, `DELETE` | `incident_reviews.write` |

<!-- theme:info -->
> A review that does not exist and a review belonging to a different account are both reported as
> `404 Not Found`. This is deliberate: it means the API cannot be used to discover whether an
> incident ID exists on another account.

## Endpoints

| Method | Endpoint | Description |
|:-------|:---------|:------------|
| `POST` | `/incidents/{incident_id}/review` | Create a review for an incident |
| `GET` | `/incidents/{incident_id}/review` | Retrieve a review and its sections |
| `PUT` | `/incidents/{incident_id}/review` | Update a review's title or status |
| `DELETE` | `/incidents/{incident_id}/review` | Delete a review |
| `POST` | `/incidents/{incident_id}/review/markers` | Add a marker, with notes, to the timeline |
| `GET` | `/incidents/{incident_id}/review/action_items` | List the review's action items |

Each response is wrapped in a key naming the resource it carries — `incident_review`, `marker`, or
`action_items` — as described under [Wrapped Entities](../../docs/REST-API/12-Wrapped-Entities.md).

## The Incident Review Object

| Field | Type | Description |
|:------|:-----|:------------|
| `id` | string | The review's unique identifier. |
| `incident_id` | string | ID of the incident being reviewed. |
| `title` | string | The review's title. |
| `status` | string | One of `started`, `in_review`, or `completed`. |
| `sections` | array | The review's sections, ordered by `priority`. |
| `generation_status` | string | `idle`, or `generating` while an AI draft is being written into the review. |
| `ai_generated` | boolean | Whether the review has been through the AI draft flow. |
| `template_id` | string | ID of the template the review follows, when it follows one. |
| `has_events` | boolean | Whether the timeline has at least one event on it. |
| `action_item_count` | integer | Total number of action items on the review. |
| `completed_action_item_count` | integer | How many of those action items are complete. |
| `created_by` | string | ID of the user who created the review, when known. |
| `created_at` | string | When the review was created. |
| `updated_at` | string | When the review was last updated. |
| `completed_at` | string | When the review was completed, if it has been. |

Each entry in `sections` has a `title`, a `priority` controlling display order (lower sorts first),
a `required` flag, a `completed_at` timestamp, and `content` holding the section's body as Tiptap
editor JSON. `content` is null or empty for a section nobody has written or generated yet.

<!-- theme:info -->
> `generation_status` reports `generating` while an AI draft is being written into the review's
> sections. Content you read in that state may be overwritten when generation finishes, so re-fetch
> the review before treating its sections as final.

## Creating a Review

`POST` to the review path with a title:

```json
{
  "title": "Database outage review"
}
```

A successful request returns `201 Created` and the new review. A new review starts in `started`
status with `generation_status` of `idle`.

Creating a review also begins importing the incident's own events onto the review's timeline, so
`has_events` may become true shortly after creation without you adding anything.

An incident can only have one review. Creating a review for an incident that already has one is
rejected with `400 Bad Request`, so if you don't know whether a review exists yet, `POST` first and
fall back to `GET` when you receive a `400`.

## Updating a Review

`PUT` accepts `title`, `status`, or both. It is a partial update: a field you omit keeps its current
value.

```json
{
  "status": "completed"
}
```

`status` must be `started`, `in_review`, or `completed`; any other value is rejected with
`400 Bad Request`.

<!-- theme:warning -->
> A review cannot be moved to `completed` while any section marked `required` is still unfilled.
> Attempting it returns `409 Conflict`. Read the `sections` array and check `required` against
> `content` to find what is outstanding.

## Deleting a Review

`DELETE` removes the review and returns `204 No Content`. Deleting a review that is already gone
returns `404 Not Found` and changes nothing, so a delete that times out is safe to retry.

## Adding a Marker to the Timeline

A marker is a labelled point on the review's timeline — a key moment — with one or more notes
attached as supporting evidence. `POST` to the `markers` path:

```json
{
  "type": "Key Moment",
  "title": "Customer-reported timeline update",
  "notes": [
    "Customer confirmed impact started at 10:42 UTC",
    "Support ticket #4821 opened at 10:55 UTC"
  ]
}
```

| Field | Required | Description |
|:------|:---------|:------------|
| `title` | ✓ | The marker's title. |
| `notes` | ✓ | One or more non-empty notes. Each becomes its own event on the timeline, all attached to this marker. |
| `type` | | The name of an existing marker type. Defaults to `Key Moment` when omitted. |

A successful request returns `201 Created` and the marker, including the note events created from
`notes` with their IDs and timestamps. The incident must already have a review; if it does not, the
request is rejected with `404 Not Found`.

## Listing Action Items

`GET` the `action_items` path to retrieve every action item on the review:

```json
{
  "action_items": [
    {
      "id": "AGOBAMKTT5Z3RO7CR7T2ZGSMJA",
      "incident_id": "P1234ABC",
      "name": "Update runbook",
      "description": "Add failover steps to the runbook",
      "status": "incomplete",
      "priority": "high",
      "assignee": "PABC123",
      "due_at": "2026-09-30T17:00:00Z",
      "external_type": "Jira",
      "external_url": "https://example.atlassian.net/browse/OPS-1234",
      "ai_generated": false,
      "created_by": "PUSER123",
      "created_at": "2026-09-01T10:42:00Z",
      "updated_at": "2026-09-02T09:15:00Z",
      "completed_at": null
    }
  ]
}
```

A review with no action items returns `200 OK` and an empty `action_items` array. An incident with no
review at all returns `404 Not Found` — an empty array and a missing review are distinguishable.

Where an action item has been pushed to an external tracker, `external_type` and `external_url`
together are what you need to link back to it. `created_by` is null when the item was created by
PagerDuty Advance rather than by a person, and `status` is the item's review-specific status rather
than any status held in the external tracker.

## Errors

This API uses the standard [error responses](../../docs/REST-API/14-Errors.md) and
[rate limits](../../docs/REST-API/04-Rate-Limits.md) of the REST API. The status codes that carry
meaning specific to incident reviews are:

| Code | Meaning in this API |
|:-----|:--------------------|
| `400` | The request body was invalid, or a review already exists for this incident. |
| `404` | No review exists for this incident on your account. |
| `409` | The review cannot be completed because a required section is unfilled. |
