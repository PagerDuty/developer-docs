---
tags: [webhooks]
---

# v3 Overview

V3 webhooks provide the foundation for the future of PagerDuty webhooks. When compared with previous versions, they provide additional event types to signal changes to incident priorities and incident responders. They also provide additional filtering capabilities.

To get started with v3 webhooks, create a _webhook subscription_ using the [Webhook Subscriptions API](https://developer.pagerduty.com/api-reference/b3A6MjkyNDc4NA-create-a-webhook-subscription).

## Webhook Subscriptions

V3 webhooks are configured by creating a _webhook subscription_ which contains three main components:

- The set of _[outbound event types](#event-types)_ that are of interest
- A _filter_ or scope of the events to be delivered
- The _delivery method_ that should be used to send the matching events as a [webhook](#webhook-payload)

**Sample Create Webhook Subscription Request**

```json
{
  "webhook_subscription": {
    "delivery_method": {
      "type": "http_delivery_method",
      "url": "https://example.com/receive_a_pagerduty_webhook",
      "custom_headers": [
        {
          "name": "your-header-name",
          "value": "your-header-value"
        }
      ]
    },
    "description": "Sends PagerDuty v3 webhook events somewhere interesting.",
    "events": [
      "incident.acknowledged",
      "incident.annotated",
      "incident.delegated",
      "incident.escalated",
      "incident.priority_updated",
      "incident.reassigned",
      "incident.reopened",
      "incident.resolved",
      "incident.responder.added",
      "incident.responder.replied",
      "incident.status_update_published",
      "incident.triggered",
      "incident.unacknowledged"
    ],
    "filter": {
      "id": "P393ZNQ",
      "type": "service_reference"
    },
    "type": "webhook_subscription"
  }
}
```

### Custom Headers

The `custom_headers` of a webhook subscription define any optional headers that will be passed along with the payload to the destination URL. The header values are redacted in GET requests, but are not redacted on the webhook when delivered to the webhook's endpoint. All header names must be unique within a webhook subscription.

### Events

The `events` of a webhook subscription define which [event types](#event-types) will produce webhooks for the subscription. A subset of all event types may be provided if the destination is only interested in a limited set of events.

### Filter

The `filter` of a webhook subscription determines which events will match and produce a webhook. There are currently three types of filters that can be applied to v3 webhooks: `service_reference`, `team_reference` and `account_reference`.

In the case of incident events, the different filter types will only produce webhooks for the incidents that are associated with the filter object. For example: a webhook subscription with a service filter would produce webhooks for all incidents belonging to the specified service.

### More Details

Please see the full [Webhook Subscriptions API Reference](https://developer.pagerduty.com/api-reference/b3A6MjkyNDc4NA-create-a-webhook-subscription) for more details.

## Webhook Payload

V3 webhook payloads are based around the concept of [outbound events](#event-types). V3 webhooks provide a mechanism to deliver these events to your server or application.

Each webhook payload contains a single event object. This event object contains high-level information common to all outbound events. The inner `event.data` payload differs based on the `event.event_type` and contains detailed information about the change that occurred.

| Field Name            | Type     | Description                                                                                                     |
| --------------------- | -------- | --------------------------------------------------------------------------------------------------------------- |
| `event`               | Object   | The event that triggered the webhook.                                                                           |
| `event.id`            | String   | The unique id of the event.                                                                                     |
| `event.event_type`    | String   | The type of the event. This usually provides a description of what happened (e.g. `incident.priority_updated`). |
| `event.resource_type` | String   | The primary resource this event affected (e.g. `incident`).                                                     |
| `event.occurred_at`   | DateTime | An ISO 8601 datetime indicating when the event occurred.                                                        |
| `event.agent`         | Object   | Indicates who or what initiated the event.                                                                      |
| `event.client`        | Object   | Information about where the event was triggered.                                                                |
| `event.data`          | Object   | Data specific to the `event_type` that occurred.                                                                |

An example webhook payload for an `incident.priority_updated` event is shown below.

```json
{
  "event": {
    "id": "5ac64822-4adc-4fda-ade0-410becf0de4f",
    "event_type": "incident.priority_updated",
    "resource_type": "incident",
    "occurred_at": "2020-10-02T18:45:22.169Z",
    "agent": {
      "html_url": "https://acme.pagerduty.com/users/PLH1HKV",
      "id": "PLH1HKV",
      "self": "https://api.pagerduty.com/users/PLH1HKV",
      "summary": "Tenex Engineer",
      "type": "user_reference"
    },
    "client": {
      "name": "PagerDuty"
    },
    "data": {
      "id": "PGR0VU2",
      "type": "incident",
      "self": "https://api.pagerduty.com/incidents/PGR0VU2",
      "html_url": "https://acme.pagerduty.com/incidents/PGR0VU2",
      "number": 2,
      "status": "triggered",
      "incident_key": "d3640fbd41094207a1c11e58e46b1662",
      "created_at": "2020-04-09T15:16:27Z",
      "title": "A little bump in the road",
      "service": {
        "html_url": "https://acme.pagerduty.com/services/PF9KMXH",
        "id": "PF9KMXH",
        "self": "https://api.pagerduty.com/services/PF9KMXH",
        "summary": "API Service",
        "type": "service_reference"
      },
      "assignees": [
        {
          "html_url": "https://acme.pagerduty.com/users/PTUXL6G",
          "id": "PTUXL6G",
          "self": "https://api.pagerduty.com/users/PTUXL6G",
          "summary": "User 123",
          "type": "user_reference"
        }
      ],
      "escalation_policy": {
        "html_url": "https://acme.pagerduty.com/escalation_policies/PUS0KTE",
        "id": "PUS0KTE",
        "self": "https://api.pagerduty.com/escalation_policies/PUS0KTE",
        "summary": "Default",
        "type": "escalation_policy_reference"
      },
      "teams": [
        {
          "html_url": "https://acme.pagerduty.com/teams/PFCVPS0",
          "id": "PFCVPS0",
          "self": "https://api.pagerduty.com/teams/PFCVPS0",
          "summary": "Engineering",
          "type": "team_reference"
        }
      ],
      "priority": {
        "html_url": "https://acme.pagerduty.com/account/incident_priorities",
        "id": "PSO75BM",
        "self": "https://api.pagerduty.com/priorities/PSO75BM",
        "summary": "P1",
        "type": "priority_reference"
      },
      "urgency": "high",
      "conference_bridge": {
        "conference_number": "+1 1234123412,,987654321#",
        "conference_url": "https://example.com"
      },
      "resolve_reason": null
    }
  }
}
```

An example webhook payload for a `service.updated` event is shown below.

```json
{
  "event": {
    "id": "01BRB6ZP4M6T8ZG4X6BP63ZB9O",
    "event_type": "service.updated",
    "resource_type": "service",
    "occurred_at": "2021-03-02T13:35:11.682Z",
    "agent": null,
    "client": null,
    "data": {
      "html_url": "https://acme.pagerduty.com/services/PF9KMXH",
      "id": "PF9KMXH",
      "self": "https://api.pagerduty.com/services/PF9KMXH",
      "summary": "testing service updates",
      "alert_creation": "create_alerts_and_incidents",
      "teams": [
        {
          "html_url": "https://acme.pagerduty.com/teams/PFCVPS0",
          "id": "PFCVPS0",
          "self": "https://api.pagerduty.com/teams/PFCVPS0",
          "summary": "Engineering",
          "type": "team_reference"
        }
      ],
      "type": "service"
    }
  }
}
```

## Event Types

Outbound events are created when PagerDuty resources change in interesting ways. Each outbound event is usually associated with some other PagerDuty resource. For example, the `incident.priority_updated` event is generated whenever the priority of an incident is changed. The following event types are available to v3 webhooks. Additional event types may be added to this list over time.

### incident.acknowledged

`data.type` is [`incident`](#incident)

Sent when an incident is acknowledged.

### incident.annotated

`data.type` is [`incident_note`](#incident_note)

Sent when a note is added to an incident.

### incident.delegated

`data.type` is [`incident`](#incident)

Sent when an incident has been reassigned to another escalation policy.

### incident.escalated

`data.type` is [`incident`](#incident)

Sent when an incident has been escalated to another user in the same escalation level.

### incident.priority_updated

`data.type` is [`incident`](#incident)

Sent when the priority of an incident has changed.

### incident.reassigned

`data.type` is [`incident`](#incident)

Sent when an incident has been reassigned to another user.

### incident.reopened

`data.type` is [`incident`](#incident)

Sent when an incident is reopened.

### incident.resolved

`data.type` is [`incident`](#incident)

Sent when an incident has been resolved.

### incident.responder.added

`data.type` is [`incident_responder`](#incident_responder)

Sent when a responder has been added to an incident.

### incident.responder.replied

`data.type` is [`incident_responder`](#incident_responder)

Sent when a responder replies to a request.

### incident.status_update_published

`data.type` is [`incident_status_update`](#incident_status_update)

Sent when a status update is added to an incident.

### incident.triggered

`data.type` is [`incident`](#incident)

Sent when an incident is newly created/triggered.

### incident.unacknowledged

`data.type` is [`incident`](#incident)

Sent when an incident is unacknowledged.

### service.created

`data.type` is [`service`](#service)

Sent when a service is created.

### service.deleted

`data.type` is [`service`](#service)

Sent when a service is deleted.

### service.updated

`data.type` is [`service`](#service)

Sent when a service is updated.

## Event Data Types

Depending on the `event.event_type`, of the webhook payload, the `event.data` field will contain one of the objects described in this section.

### incident

```json
{
  "id": "PGR0VU2",
  "type": "incident",
  "self": "https://api.pagerduty.com/incidents/PGR0VU2",
  "html_url": "https://acme.pagerduty.com/incidents/PGR0VU2",
  "number": 2,
  "status": "triggered",
  "incident_key": "d3640fbd41094207a1c11e58e46b1662",
  "created_at": "2020-04-09T15:16:27Z",
  "title": "A little bump in the road",
  "service": {
    "html_url": "https://acme.pagerduty.com/services/PF9KMXH",
    "id": "PF9KMXH",
    "self": "https://api.pagerduty.com/services/PF9KMXH",
    "summary": "API Service",
    "type": "service_reference"
  },
  "assignees": [
    {
      "html_url": "https://acme.pagerduty.com/users/PTUXL6G",
      "id": "PTUXL6G",
      "self": "https://api.pagerduty.com/users/PTUXL6G",
      "summary": "User 123",
      "type": "user_reference"
    }
  ],
  "escalation_policy": {
    "html_url": "https://acme.pagerduty.com/escalation_policies/PUS0KTE",
    "id": "PUS0KTE",
    "self": "https://api.pagerduty.com/escalation_policies/PUS0KTE",
    "summary": "Default",
    "type": "escalation_policy_reference"
  },
  "teams": [
    {
      "html_url": "https://acme.pagerduty.com/teams/PFCVPS0",
      "id": "PFCVPS0",
      "self": "https://api.pagerduty.com/teams/PFCVPS0",
      "summary": "Engineering",
      "type": "team_reference"
    }
  ],
  "priority": {
    "html_url": "https://acme.pagerduty.com/account/incident_priorities",
    "id": "PSO75BM",
    "self": "https://api.pagerduty.com/priorities/PSO75BM",
    "summary": "P1",
    "type": "priority_reference"
  },
  "urgency": "high",
  "conference_bridge": {
    "conference_number": "+1 1234123412,,987654321#",
    "conference_url": "https://example.com"
  },
  "resolve_reason": null
}
```

### incident_note

```json
{
  "incident": {
    "html_url": "https://acme.pagerduty.com/incidents/PGR0VU2",
    "id": "PGR0VU2",
    "self": "https://api.pagerduty.com/incidents/PGR0VU2",
    "summary": "A little bump in the road",
    "type": "incident_reference"
  },
  "id": "P2LA89X",
  "content": "I sure am glad we are using PagerDuty!",
  "trimmed": false,
  "type": "incident_note"
}
```

### incident_status_update

```json
{
  "incident": {
    "html_url": "https://acme.pagerduty.com/incidents/PGR0VU2",
    "id": "PGR0VU2",
    "self": "https://api.pagerduty.com/incidents/PGR0VU2",
    "summary": "A little bump in the road",
    "type": "incident_reference"
  },
  "id": "P2LA89X",
  "message": "A fix for this incident is being developed",
  "trimmed": false,
  "type": "incident_status_update"
}
```

### incident_responder

```json
{
  "incident": {
    "html_url": "https://acme.pagerduty.com/incidents/PGR0VU2",
    "id": "PGR0VU2",
    "self": "https://api.pagerduty.com/incidents/PGR0VU2",
    "summary": "A little bump in the road",
    "type": "incident_reference"
  },
  "user": {
    "html_url": "https://acme.pagerduty.com/users/PVMGSML",
    "id": "PVMGSML",
    "self": "https://api.pagerduty.com/users/PVMGSML",
    "summary": "Maeve",
    "type": "user_reference"
  },
  "escalation_policy": {
    "html_url": "https://acme.pagerduty.com/escalation_policies/PJFWPEP",
    "id": "PJFWPEP",
    "self": "https://api.pagerduty.com/escalation_policies/PJFWPEP",
    "summary": "The Policy",
    "type": "escalation_policy_reference"
  },
  "message": "Please help me make the tests pass",
  "state": "pending",
  "type": "incident_responder"
}
```

### service

```json
{
  "html_url": "https://acme.pagerduty.com/services/PF9KMXH",
  "id": "PF9KMXH",
  "self": "https://api.pagerduty.com/services/PF9KMXH",
  "summary": "testing service updates",
  "alert_creation": "create_alerts_and_incidents",
  "teams": [
    {
      "html_url": "https://acme.pagerduty.com/teams/PFCVPS0",
      "id": "PFCVPS0",
      "self": "https://api.pagerduty.com/teams/PFCVPS0",
      "summary": "Engineering",
      "type": "team_reference"
    }
  ],
  "type": "service"
}
```
