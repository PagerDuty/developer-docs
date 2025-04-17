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

### incident.role.assigned

`data.type` is [`incident_role_assignment`](#incident_role_assignment)

Sent when an incident role is assigned or unassigned.

### service.custom_field_values.updated

`data.type` is [`service_field_values`](#service_field_values)

Sent when an service's custom fields values are updated.

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
  }
}
```

### incident_role_assignment
```json
{
    "type": "incident_role_assignment",
    "incident_role_assignments": [
      {
        "assignee": {
          "html_url": "https://acme.pagerduty.com/users/P75B6QD",
          "id": "P75B6QD",
          "self": "https://api.pagerduty.com/users/P75B6QD",
          "summary": "User 1810194",
          "type": "user_reference"
        },
        "id": "af64b84c-137e-40c6-875c-5dd30a2afaaa",
        "incident": {
          "html_url": "https://acme.pagerduty.com/incidents/PBAZLIU",
          "id": "PBAZLIU",
          "self": "https://api.pagerduty.com/incidents/PBAZLIU",
          "summary": null,
          "type": "incident_reference"
        },
        "old_assignee": null,
        "role": {
          "id": "P8PQO4R",
          "summary": "Role Display Name",
          "type": "role_reference"
        },
        "status": "active",
        "type": "role_assignment_reference"
      }
    ]
  }
```

### service_field_values
```json
{
  "service": {
    "html_url": "https://acme.pd-staging.com/services/PY0TW31",
    "id": "PY0TW31",
    "self": "https://api.pd-staging.com/services/PY0TW31",
    "summary": null,
    "type": "service_reference"
  },
  "custom_fields": [
    {
      "data_type": "string",
      "field_type": "multi_value",
      "id": "P0CU101",
      "name": "string_multi_example_1",
      "type": "field_value",
      "value": [
        "1",
        "2"
      ]
    },
    {
      "data_type": "string",
      "field_type": "single_value",
      "id": "P7DNIMB",
      "name": "example_field",
      "type": "field_value",
      "value": "Some new value"
    }
  ],
  "changed_custom_fields": [
    {
      "data_type": "string",
      "field_type": "single_value",
      "id": "P7DNIMB",
      "name": "example_field",
      "type": "field_value",
      "value": "Some old value"
    }
  ],
  "type": "service_field_values"
}
```
