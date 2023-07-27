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

### incident.custom_field_values.updated

`data.type` is [`incident_field_values`](#incident_field_values)

Sent when an incident's custom fields values are updated.

### incident.conference_bridge.updated

`data.type` is [`incident_conference_bridge`](#incident_conference_bridge)

Sent when an incident's conference bridge number and/or conference bridge url is updated.

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

### incident_field_values

```json
{
  "incident": {
    "html_url": "https://acme.pagerduty.com/incidents/PBAZLIU",
    "id": "PBAZLIU",
    "self": "https://api.pagerduty.com/incidents/PBAZLIU",
    "summary": null,
    "type": "incident_reference"
  },
  "custom_fields": [
    {
      "data_type": "string",
      "id": "PICFVXX",
      "name": "environment",
      "type": "field_value",
      "value": "production",
      "field_type": "single_value"
    },
    {
      "data_type": "string",
      "id": "PF3UUI7",
      "name": "region",
      "type": "field_value",
      "value": ["US"],
      "field_type": "multi_value_fixed"
    }
  ],
  "changed_custom_fields": [
    {
      "data_type": "string",
      "id": "PICFVXX",
      "name": "environment",
      "type": "field_value",
      "value": "staging",
      "field_type": "single_value_fixed"
    }
  ],
  "type": "incident_field_values"
}
```

### incident_conference_bridge

```json
{
  "incident": {
    "html_url": "https://acme.pagerduty.com/incidents/PGR0VU2",
    "id": "PGR0VU2",
    "self": "https://api.pagerduty.com/incidents/PGR0VU2",
    "summary": "Major incident - customer shopping cart screen",
    "type": "incident_reference"
  },
  "conference_numbers": [
    {
      "label": "",
      "number": "+1-555-555-5555"
    }
  ],
  "conference_url": "https://example.com",
  "type": "incident_conference_bridge"
}
```
