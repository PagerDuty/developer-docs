---
tags: [webhooks]
---

# Early Access Webhook Events

<!-- theme: warning -->

> ### Early Access
>
> The features described on this page are in an Early Access state and are subject to change. Please reach out to
> us if you have any questions or need support.

Early Access webhook events are provided here for informational purposes, but are subject to change in availability
(may be removed) and design (contents and shape of response may change). They can be created via the
`/webhook_subscriptions` REST API endpoint.

## Event Types

### incident.action_invocation.created

`data.type` is [`incident_action_invocation`](#incident_action_invocation)

Sent when an incident action invocation is created.

### incident.action_invocation.updated

`data.type` is [`incident_action_invocation`](#incident_action_invocation)

Sent when an incident action invocation is updated.

### incident.action_invocation.terminated

`data.type` is [`incident_action_invocation`](#incident_action_invocation)

Sent when an incident action invocation is terminated.

### incident.task.created

`data.type` is [`incident_task`](#incident_task)

Sent when an incident task is created.

### incident.task.updated

`data.type` is [`incident_task`](#incident_task)

Sent when an incident task is updated.

### incident.task.completed

`data.type` is [`incident_task`](#incident_task)

Sent when an incident task is completed.

## Event Data Types

Depending on the `event.event_type`, of the webhook payload, the `event.data` field will contain one of the objects described in this section.

### incident_action_invocation

```json
{
  "id": "01CELD6T9C2JS745I7CAK0LRRF",
  "self": "https://api.pagerduty.com/automation/invocations/01CELD6T9C2JS745I7CAK0LRRF",
  "html_url": "https://acme.pagerduty.com/rundeck-actions/actions/01CDYN0IRV4VG991K5FR73YNTW/invocations/01CELD6T9C2JS745I7CAK0LRRF/report",
  "incident": {
    "html_url": "https://acme.pagerduty.com/incidents/PBAZLIU",
    "id": "PBAZLIU",
    "self": "https://api.pagerduty.com/incidents/PBAZLIU",
    "summary": "An Incident",
    "type": "incident_reference"
  },
  "action": {
    "html_url": "https://acme.pagerduty.com/rundeck-actions/actions/01CDYN0IRV4VG991K5FR73YNTW",
    "id": "01CDYN0IRV4VG991K5FR73YNTW",
    "self": "https://api.pagerduty.com/automation/actions/01CDYN0IRV4VG991K5FR73YNTW",
    "summary": "A Helpful Action",
    "type": "action_reference"
  },
  "state": "created",
  "type": "incident_action_invocation"
}
```

### incident_task

```json
{
  "id": "2dd39544-ccd2-491d-9c5b-e4fb8be4259d",
  "event_type": "incident.task.created",
  "resource_type": "incident",
  "occurred_at": "2021-06-01T21:30:42Z",
  "agent": {
    "html_url": "https://acme.pagerduty.com/users/PLH1HKV",
    "id": "PLH1HKV",
    "self": "https://api.pagerduty.com/users/PLH1HKV",
    "summary": "User 10",
    "type": "user_reference"
  },
  "client": null,
  "data": {
    "name": "A thing that needs to be done",
    "description": null,
    "id": "PGR0VU2",
    "summary": "A thing that needs to be done",
    "type": "incident_task",
    "status": "todo",
    "assignees": [
      {
        "html_url": "https://acme.pagerduty.com/users/PIV35G6",
        "id": "PIV35G6",
        "self": "https://api.pagerduty.com/users/PIV35G6",
        "summary": "User 661768438",
        "type": "user_reference"
      }
    ],
    "incident": {
      "html_url": "https://acme.pagerduty.com/incidents/Q0SDD3HB6SGFTI",
      "id": "Q0SDD3HB6SGFTI",
      "self": "https://api.pagerduty.com/incidents/Q0SDD3HB6SGFTI",
      "summary": null,
      "type": "incident_reference"
    },
    "changed_fields": []
  }
}
```
