---
tags: [webhooks]
---


# Overview

Our latest version of Webhooks is V3. 

V3 webhooks provide the foundation for the future of PagerDuty webhooks. When compared with previous versions, they provide additional event types to signal changes to incident priorities and incident responders. They also provide additional filtering capabilities.

To get started with v3 webhooks, create a _webhook subscription_ using the [Webhook Subscriptions API](https://developer.pagerduty.com/api-reference/b3A6MjkyNDc4NA-create-a-webhook-subscription).

<!-- theme: info -->
> ### Migrate to V3 Webhook Subscriptions now!
> If you are currently using V1/V2 webhook extensions and need to migrate them to V3 webhook subscriptions, please follow our migration guide.
>
> V1 webhook extensions became unsupported on November 13, 2021 and will lose functionality in October, 2022.
>
> V2 webhooks extensions will become unsupported in October, 2022 and will lose functionality in March, 2023.

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

`data.type` is [`incident`](##incident)

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

<!--
type: tab
title: v2
-->


# v2 Overview

<!-- theme: warning -->
> ### Migrate to V3 Webhook Subscriptions
> If you are currently using V1/V2 webhook extensions and need to migrate them to V3 webhook subscriptions, please follow our migration guide.
>
> V2 webhooks extensions will become unsupported in October, 2022 and will lose functionality in March, 2023.

Webhooks let you receive HTTP callbacks when interesting events happen within your PagerDuty account. Details surrounding the interesting event will be sent via HTTP POST to a URL that you specify.

PagerDuty currently supports incident-based webhooks. After adding a webhook URL to a PagerDuty service, the triggering of new incidents on that service will cause outgoing webhook messages to be sent to that URL. In addition, certain interesting changes to an incident's state will cause other types of incident webhook messages to be sent. Generally, any change to the `status` or `assignees` of an incident will cause an outgoing message to be sent.

### V2 Webhook Payload

Webhook recipients receive a payload containing a single `messages` array that may contain multiple `message` elements if webhook firing actions occurred in quick succession. Each `message` in the array consists of four fields:

Field Name    | Type     | Description
------------- | -------- | ------------
`id`          | UUID     | Uniquely identifies this outgoing webhook message; can be used for idempotency when processing the messages.
`event`       | String   | The webhook event type (see below).
`created_on`  | DateTime | The date/time when the incident changed state.
`incident`    | Object   | The incident details at the time of the state change.
`webhook`     | Object   | The webhook configuration which resulted in this message.
`log_entries` | Array    | Log entries (see below) that correspond to the action this Webhook is reporting. There will be only one `log_entry` type object in the array for `incident.trigger`, `incident.acknowledge` and `incident.resolve` type webhooks. For `incident.escalate` webhooks where there were more than one target, there will be one entry in this array for each escalation target.



### Incident Details

For incident webhooks, the `incident` field contains a representation of the incident associated with the action that caused this webhook message.

The fields below can be expected to be found in the incident data. Refer to the webhook examples below to see what the data looks like in practice.

Field Name                     | Type                | Description
------------------------------ | ------------------- | -------------
`id`                           | String              | The incident's id, which can be used to find it in the REST API.
`alerts`                       | Array               | A list of alert keys associated with this incident (called `dedup_key` in the Events API format). More than one alert may be associated if grouping is enabled on a service. Use these keys to correlate an event sent to PagerDuty with a webhook event. See [Time-Based Alert Grouping](https://support.pagerduty.com/docs/time-based-alert-grouping) or [Intelligent Alert Grouping](https://support.pagerduty.com/docs/intelligent-alert-grouping) for more info on grouping.
`incident_number`              | Integer             | The number of the incident. This is unique across the account.
`title`                        | String              | A succinct description of the nature, symptoms, cause, or effect of the incident.
~~`description`~~ (deprecated) | String              | This field is deprecated. Please use `title` instead. The contents will be identical. For incidents manually created in PagerDuty, the description is available in the first log entry in `channel.details`.
`created_at`                   | DateTime            | The date/time the incident was first triggered.
`status`                       | String              | The current status of the incident. One of `triggered`, `acknowledged`, or `resolved`
`incident_key`                 | String              | The incident's de-duplication key.
`html_url`                     | URL                 |
`pending_actions`              | Array               | The list of `pending_actions` on the incident. A `pending_action` object contains a type of action which can be `escalate`, `unacknowledge`, `resolve` or `urgency_change`. A `pending_action` object contains `at`, the time at which the action will take place. An `urgency_change` `pending_action` will contain `to`, the urgency that the incident will change to.
`service`                      | Object              | A representation of the PagerDuty service that the incident belongs to. See the webhook examples below for what the `service` object looks like.
`assignments`                  | Array               | A list of users assigned to the incident at the time of the webhook action. Each entry in the array indicates when the user was assigned.
`acknowledgements`             | Array               | List of all acknowledgements for this incident.
`last_status_change_at`        | DateTime            | The time at which the status of the incident last changed.
`last_status_change_by`        | Object              | The user or service which is responsible for the incident's last status change. If the incident is in the acknowledged or resolved status, this will be the user that took the first acknowledged or resolved action. If the incident was automatically resolved (say through the Events API), or if the incident is in the triggered state, this will be the incident's service.
`first_trigger_log_entry`      | Object              | The first trigger log entry for the incident.
`escalation_policy`            | Object              | The escalation policy that the incident is currently following.
`teams`                        | Array               | The teams involved in the incident's lifecycle.
`priority`                     | Object              | The priority of the incident.
`urgency`                      | String              | The incident's urgency (`high` or `low`).
`resolve_reason`               | Nullable<br/>String | The reason the incident was resolved. Currently the only valid values are `null` and `"merged"` with plans to introduce additional reasons in the future.
`alert_counts`                 | Object              | A summary of the number of alerts by status.
`metadata`                     | Object              | The metadata saved on the service.



### Log Entries

PagerDuty keeps a log of all the events that happen to an incident. These are exposed as log entries.

Log entries give you more insight into how your team or organization is handling your incidents. Log entry data includes details about the event(s) that affected the incident throughout its lifecycle, such as:

- the data contained in events sent by the integration
- which users were notified and when
- how they were notified
- which user(s) acknowledged or resolved the incident
- any automatic actions that occurred to the incident
- other manual user actions, such as a reassignment or a note

The payload contains the log entry for the action(s) that triggered the webhook to be sent. The fields below can be expected to be found in each log entry object. Refer to the webhook examples below to see what the data looks like in practice.

Field Name      | Type     | Description
--------------- | -------- | ------------
`created_at`    | DateTime | Time at which the log entry was created.
`channel`       | Object   | The channel field represents the means by which the action was carried out.<br/><br/> For example, in `incident.trigger` webhooks, this will represent the original event data sent to PagerDuty that triggered the incident.
`agent`         | Object   | The agent (user, service or integration) that created or modified the log entry.
`note`          | String   | Optional field containing a note, if one was included with the log entry.
`contexts`      | Array    | Contexts to be included with the trigger such as links to graphs or images.
`incident`      | Object   | Reference to the incident.
`service`       | Object   | Reference to the service associated with the incident.
`teams`         | Array    | References to teams associated with the service.
`event_details` | Object   | Additional details about the event.



### Webhook Types

A webhook's `event` field indicates what action caused it to be sent.

Type                     | Description
------------------------ | ------------
`incident.trigger`       | Sent when an incident is newly created/triggered.
`incident.acknowledge`   | Sent when an incident is acknowledged by a user.
`incident.unacknowledge` | Sent when an incident is unacknowledged due to its acknowledgement timing out.
`incident.resolve`       | Sent when an incident has been resolved.
`incident.assign`        | Sent when an incident has been assigned to another user. Often occurs in concert with an `acknowledge`.
`incident.escalate`      | Sent when an incident has been escalated to another user in the same escalation chain.
`incident.delegate`      | Sent when an incident has been reassigned to another escalation policy.
`incident.annotate`      | Sent when a note is created on an incident.

More webhook types may be added in the future.


### V2.Examples


### incident.trigger


```json
{
  "messages": [
    {
      "event": "incident.trigger",
      "log_entries": [
        {
          "id": "R2XGXEI3W0FHMSDXHDIBQGBQ5E",
          "type": "trigger_log_entry",
          "summary": "Triggered through the website",
          "self": "https://api.pagerduty.com/log_entries/R2XGXEI3W0FHMSDXHDIBQGBQ5E",
          "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY/log_entries/R2XGXEI3W0FHMSDXHDIBQGBQ5E",
          "created_at": "2017-09-26T15:14:36Z",
          "agent": {
            "id": "P553OPV",
            "type": "user_reference",
            "summary": "Laura Haley",
            "self": "https://api.pagerduty.com/users/P553OPV",
            "html_url": "https://webdemo.pagerduty.com/users/P553OPV"
          },
          "channel": {
            "type": "web_trigger",
            "summary": "My new incident",
            "subject": "My new incident",
            "details": "Oh my gosh",
            "details_omitted": false
          },
          "service": {
            "id": "PN49J75",
            "type": "service_reference",
            "summary": "Production XDB Cluster",
            "self": "https://api.pagerduty.com/services/PN49J75",
            "html_url": "https://webdemo.pagerduty.com/services/PN49J75"
          },
          "incident": {
            "id": "PRORDTY",
            "type": "incident_reference",
            "summary": "[#33] My new incident",
            "self": "https://api.pagerduty.com/incidents/PRORDTY",
            "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY"
          },
          "teams": [
            {
              "id": "P4SI59S",
              "type": "team_reference",
              "summary": "Engineering",
              "self": "https://api.pagerduty.com/teams/P4SI59S",
              "html_url": "https://webdemo.pagerduty.com/teams/P4SI59S"
            }
          ],
          "contexts": [],
          "event_details": {
            "description": "My new incident"
          }
        }
      ],
      "webhook": {
        "endpoint_url": "https://requestb.in/18ao6fs1",
        "name": "V2 wabhook",
        "description": null,
        "webhook_object": {
          "id": "PN49J75",
          "type": "service_reference",
          "summary": "Production XDB Cluster",
          "self": "https://api.pagerduty.com/services/PN49J75",
          "html_url": "https://webdemo.pagerduty.com/services/PN49J75"
        },
        "config": {},
        "outbound_integration": {
          "id": "PJFWPEP",
          "type": "outbound_integration_reference",
          "summary": "Generic V2 Webhook",
          "self": "https://api.pagerduty.com/outbound_integrations/PJFWPEP",
          "html_url": null
        },
        "accounts_addon": null,
        "id": "PKT9NNX",
        "type": "webhook",
        "summary": "V2 wabhook",
        "self": "https://api.pagerduty.com/webhooks/PKT9NNX",
        "html_url": null
      },
      "incident": {
        "incident_number": 33,
        "title": "My new incident",
        "description": "My new incident",
        "created_at": "2017-09-26T15:14:36Z",
        "status": "triggered",
        "pending_actions": [
          {
            "type": "escalate",
            "at": "2017-09-26T15:44:36Z"
          },
          {
            "type": "resolve",
            "at": "2017-09-26T19:14:36Z"
          }
        ],
        "incident_key": null,
        "service": {
          "id": "PN49J75",
          "name": "Production XDB Cluster",
          "description": "This service was created during onboarding on July 5, 2017.",
          "auto_resolve_timeout": 14400,
          "acknowledgement_timeout": 1800,
          "created_at": "2017-07-05T17:33:09Z",
          "status": "critical",
          "last_incident_timestamp": "2017-09-26T15:14:36Z",
          "teams": [
            {
              "id": "P4SI59S",
              "type": "team_reference",
              "summary": "Engineering",
              "self": "https://api.pagerduty.com/teams/P4SI59S",
              "html_url": "https://webdemo.pagerduty.com/teams/P4SI59S"
            }
          ],
          "incident_urgency_rule": {
            "type": "constant",
            "urgency": "high"
          },
          "scheduled_actions": [],
          "support_hours": null,
          "escalation_policy": {
            "id": "PINYWEF",
            "type": "escalation_policy_reference",
            "summary": "Default",
            "self": "https://api.pagerduty.com/escalation_policies/PINYWEF",
            "html_url": "https://webdemo.pagerduty.com/escalation_policies/PINYWEF"
          },
          "addons": [],
          "privilege": null,
          "alert_creation": "create_alerts_and_incidents",
          "integrations": [
            {
              "id": "PUAYF96",
              "type": "generic_events_api_inbound_integration_reference",
              "summary": "API",
              "self": "https://api.pagerduty.com/services/PN49J75/integrations/PUAYF96",
              "html_url": "https://webdemo.pagerduty.com/services/PN49J75/integrations/PUAYF96"
            },
            {
              "id": "P90GZUH",
              "type": "generic_email_inbound_integration_reference",
              "summary": "Email",
              "self": "https://api.pagerduty.com/services/PN49J75/integrations/P90GZUH",
              "html_url": "https://webdemo.pagerduty.com/services/PN49J75/integrations/P90GZUH"
            }
          ],
          "metadata": {},
          "type": "service",
          "summary": "Production XDB Cluster",
          "self": "https://api.pagerduty.com/services/PN49J75",
          "html_url": "https://webdemo.pagerduty.com/services/PN49J75"
        },
        "assignments": [
          {
            "at": "2017-09-26T15:14:36Z",
            "assignee": {
              "id": "P553OPV",
              "type": "user_reference",
              "summary": "Laura Haley",
              "self": "https://api.pagerduty.com/users/P553OPV",
              "html_url": "https://webdemo.pagerduty.com/users/P553OPV"
            }
          }
        ],
        "acknowledgements": [],
        "last_status_change_at": "2017-09-26T15:14:36Z",
        "last_status_change_by": {
          "id": "PN49J75",
          "type": "service_reference",
          "summary": "Production XDB Cluster",
          "self": "https://api.pagerduty.com/services/PN49J75",
          "html_url": "https://webdemo.pagerduty.com/services/PN49J75"
        },
        "first_trigger_log_entry": {
          "id": "R2XGXEI3W0FHMSDXHDIBQGBQ5E",
          "type": "trigger_log_entry_reference",
          "summary": "Triggered through the website",
          "self": "https://api.pagerduty.com/log_entries/R2XGXEI3W0FHMSDXHDIBQGBQ5E",
          "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY/log_entries/R2XGXEI3W0FHMSDXHDIBQGBQ5E"
        },
        "escalation_policy": {
          "id": "PINYWEF",
          "type": "escalation_policy_reference",
          "summary": "Default",
          "self": "https://api.pagerduty.com/escalation_policies/PINYWEF",
          "html_url": "https://webdemo.pagerduty.com/escalation_policies/PINYWEF"
        },
        "privilege": null,
        "teams": [
          {
            "id": "P4SI59S",
            "type": "team_reference",
            "summary": "Engineering",
            "self": "https://api.pagerduty.com/teams/P4SI59S",
            "html_url": "https://webdemo.pagerduty.com/teams/P4SI59S"
          }
        ],
        "alert_counts": {
          "all": 0,
          "triggered": 0,
          "resolved": 0
        },
        "impacted_services": [
          {
            "id": "PN49J75",
            "type": "service_reference",
            "summary": "Production XDB Cluster",
            "self": "https://api.pagerduty.com/services/PN49J75",
            "html_url": "https://webdemo.pagerduty.com/services/PN49J75"
          }
        ],
        "is_mergeable": true,
        "basic_alert_grouping": null,
        "alert_grouping": null,
        "metadata": {},
        "external_references": [],
        "importance": null,
        "incidents_responders": [],
        "responder_requests": [],
        "subscriber_requests": [],
        "urgency": "high",
        "id": "PRORDTY",
        "type": "incident",
        "summary": "[#33] My new incident",
        "self": "https://api.pagerduty.com/incidents/PRORDTY",
        "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY",
        "alerts": [
          {
            "alert_key": "c24117fc42e44b44b4d6876190583378"
          }
        ]
      },
      "id": "69a7ced0-a2cd-11e7-a799-22000a15839c",
      "created_on": "2017-09-26T15:14:36Z"
    }
  ]
}
```


### incident.acknowledge


```json
{
  "messages": [
    {
      "event": "incident.acknowledge",
      "log_entries": [
        {
          "id": "RRPP3746OFFZZ742NSP1G67AWC",
          "type": "acknowledge_log_entry",
          "summary": "Acknowledged by Laura Haley",
          "self": "https://api.pagerduty.com/log_entries/RRPP3746OFFZZ742NSP1G67AWC",
          "html_url": null,
          "created_at": "2017-09-26T15:15:17Z",
          "agent": {
            "id": "P553OPV",
            "type": "user_reference",
            "summary": "Laura Haley",
            "self": "https://api.pagerduty.com/users/P553OPV",
            "html_url": "https://webdemo.pagerduty.com/users/P553OPV"
          },
          "channel": {
            "type": "website"
          },
          "service": {
            "id": "PN49J75",
            "type": "service_reference",
            "summary": "Production XDB Cluster",
            "self": "https://api.pagerduty.com/services/PN49J75",
            "html_url": "https://webdemo.pagerduty.com/services/PN49J75"
          },
          "incident": {
            "id": "PRORDTY",
            "type": "incident_reference",
            "summary": "[#33] My new incident",
            "self": "https://api.pagerduty.com/incidents/PRORDTY",
            "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY"
          },
          "teams": [
            {
              "id": "P4SI59S",
              "type": "team_reference",
              "summary": "Engineering",
              "self": "https://api.pagerduty.com/teams/P4SI59S",
              "html_url": "https://webdemo.pagerduty.com/teams/P4SI59S"
            }
          ],
          "contexts": [],
          "acknowledgement_timeout": 1800,
          "event_details": {}
        }
      ],
      "webhook": {
        "endpoint_url": "https://requestb.in/18ao6fs1",
        "name": "V2 wabhook",
        "description": null,
        "webhook_object": {
          "id": "PN49J75",
          "type": "service_reference",
          "summary": "Production XDB Cluster",
          "self": "https://api.pagerduty.com/services/PN49J75",
          "html_url": "https://webdemo.pagerduty.com/services/PN49J75"
        },
        "config": {},
        "outbound_integration": {
          "id": "PJFWPEP",
          "type": "outbound_integration_reference",
          "summary": "Generic V2 Webhook",
          "self": "https://api.pagerduty.com/outbound_integrations/PJFWPEP",
          "html_url": null
        },
        "accounts_addon": null,
        "id": "PKT9NNX",
        "type": "webhook",
        "summary": "V2 wabhook",
        "self": "https://api.pagerduty.com/webhooks/PKT9NNX",
        "html_url": null
      },
      "incident": {
        "incident_number": 33,
        "title": "My new incident",
        "description": "My new incident",
        "created_at": "2017-09-26T15:14:36Z",
        "status": "acknowledged",
        "pending_actions": [
          {
            "type": "unacknowledge",
            "at": "2017-09-26T15:45:17Z"
          },
          {
            "type": "resolve",
            "at": "2017-09-26T19:14:36Z"
          }
        ],
        "incident_key": null,
        "service": {
          "id": "PN49J75",
          "name": "Production XDB Cluster",
          "description": "This service was created during onboarding on July 5, 2017.",
          "auto_resolve_timeout": 14400,
          "acknowledgement_timeout": 1800,
          "created_at": "2017-07-05T17:33:09Z",
          "status": "warning",
          "last_incident_timestamp": "2017-09-26T15:14:36Z",
          "teams": [
            {
              "id": "P4SI59S",
              "type": "team_reference",
              "summary": "Engineering",
              "self": "https://api.pagerduty.com/teams/P4SI59S",
              "html_url": "https://webdemo.pagerduty.com/teams/P4SI59S"
            }
          ],
          "incident_urgency_rule": {
            "type": "constant",
            "urgency": "high"
          },
          "scheduled_actions": [],
          "support_hours": null,
          "escalation_policy": {
            "id": "PINYWEF",
            "type": "escalation_policy_reference",
            "summary": "Default",
            "self": "https://api.pagerduty.com/escalation_policies/PINYWEF",
            "html_url": "https://webdemo.pagerduty.com/escalation_policies/PINYWEF"
          },
          "addons": [],
          "privilege": null,
          "alert_creation": "create_alerts_and_incidents",
          "integrations": [
            {
              "id": "PUAYF96",
              "type": "generic_events_api_inbound_integration_reference",
              "summary": "API",
              "self": "https://api.pagerduty.com/services/PN49J75/integrations/PUAYF96",
              "html_url": "https://webdemo.pagerduty.com/services/PN49J75/integrations/PUAYF96"
            },
            {
              "id": "P90GZUH",
              "type": "generic_email_inbound_integration_reference",
              "summary": "Email",
              "self": "https://api.pagerduty.com/services/PN49J75/integrations/P90GZUH",
              "html_url": "https://webdemo.pagerduty.com/services/PN49J75/integrations/P90GZUH"
            }
          ],
          "metadata": {},
          "type": "service",
          "summary": "Production XDB Cluster",
          "self": "https://api.pagerduty.com/services/PN49J75",
          "html_url": "https://webdemo.pagerduty.com/services/PN49J75"
        },
        "assignments": [
          {
            "at": "2017-09-26T15:14:36Z",
            "assignee": {
              "id": "P553OPV",
              "type": "user_reference",
              "summary": "Laura Haley",
              "self": "https://api.pagerduty.com/users/P553OPV",
              "html_url": "https://webdemo.pagerduty.com/users/P553OPV"
            }
          }
        ],
        "acknowledgements": [
          {
            "at": "2017-09-26T15:15:17Z",
            "acknowledger": {
              "id": "P553OPV",
              "type": "user_reference",
              "summary": "Laura Haley",
              "self": "https://api.pagerduty.com/users/P553OPV",
              "html_url": "https://webdemo.pagerduty.com/users/P553OPV"
            }
          }
        ],
        "last_status_change_at": "2017-09-26T15:15:17Z",
        "last_status_change_by": {
          "id": "P553OPV",
          "type": "user_reference",
          "summary": "Laura Haley",
          "self": "https://api.pagerduty.com/users/P553OPV",
          "html_url": "https://webdemo.pagerduty.com/users/P553OPV"
        },
        "first_trigger_log_entry": {
          "id": "R2XGXEI3W0FHMSDXHDIBQGBQ5E",
          "type": "trigger_log_entry_reference",
          "summary": "Triggered through the website",
          "self": "https://api.pagerduty.com/log_entries/R2XGXEI3W0FHMSDXHDIBQGBQ5E",
          "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY/log_entries/R2XGXEI3W0FHMSDXHDIBQGBQ5E"
        },
        "escalation_policy": {
          "id": "PINYWEF",
          "type": "escalation_policy_reference",
          "summary": "Default",
          "self": "https://api.pagerduty.com/escalation_policies/PINYWEF",
          "html_url": "https://webdemo.pagerduty.com/escalation_policies/PINYWEF"
        },
        "privilege": null,
        "teams": [
          {
            "id": "P4SI59S",
            "type": "team_reference",
            "summary": "Engineering",
            "self": "https://api.pagerduty.com/teams/P4SI59S",
            "html_url": "https://webdemo.pagerduty.com/teams/P4SI59S"
          }
        ],
        "alert_counts": {
          "all": 0,
          "triggered": 0,
          "resolved": 0
        },
        "impacted_services": [
          {
            "id": "PN49J75",
            "type": "service_reference",
            "summary": "Production XDB Cluster",
            "self": "https://api.pagerduty.com/services/PN49J75",
            "html_url": "https://webdemo.pagerduty.com/services/PN49J75"
          }
        ],
        "is_mergeable": true,
        "basic_alert_grouping": null,
        "alert_grouping": null,
        "metadata": {},
        "external_references": [
          {
            "id": "PLF5V5G",
            "type": "incident_external_reference",
            "summary": "Slack",
            "self": null,
            "html_url": null,
            "external_id": "1506438878.000302",
            "external_url": "http://webdemo.slack.com/archives/C6975FLAZ/p1506438878.000302",
            "sync": false,
            "webhook": {
              "id": "PRXV5AH",
              "type": "webhook_reference",
              "summary": "Slack M2M",
              "self": "https://api.pagerduty.com/webhooks/PRXV5AH",
              "html_url": null
            }
          }
        ],
        "importance": null,
        "incidents_responders": [],
        "responder_requests": [],
        "subscriber_requests": [],
        "urgency": "high",
        "id": "PRORDTY",
        "type": "incident",
        "summary": "[#33] My new incident",
        "self": "https://api.pagerduty.com/incidents/PRORDTY",
        "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY",
        "alerts": [
          {
            "alert_key": "c24117fc42e44b44b4d6876190583378"
          },
          {
            "alert_key": "2d6ee0a6b58040f7996a4047ae5edcd7"
          }
        ]
      },
      "id": "82a33960-a2cd-11e7-8f69-22000affca53",
      "created_on": "2017-09-26T15:15:17Z"
    }
  ]
}
```


### incident.resolve

```json
{
  "messages": [
    {
      "event": "incident.resolve",
      "log_entries": [
        {
          "id": "RNAJ8DLJGWHBXXYX1CC5G50C9O",
          "type": "resolve_log_entry",
          "summary": "Resolved by Laura Haley",
          "self": "https://api.pagerduty.com/log_entries/RNAJ8DLJGWHBXXYX1CC5G50C9O",
          "html_url": null,
          "created_at": "2017-09-26T15:18:09+00:00",
          "agent": {
            "id": "P553OPV",
            "type": "user_reference",
            "summary": "Laura Haley",
            "self": "https://api.pagerduty.com/users/P553OPV",
            "html_url": "https://webdemo.pagerduty.com/users/P553OPV"
          },
          "channel": {
            "type": "website"
          },
          "service": {
            "id": "PN49J75",
            "type": "service_reference",
            "summary": "Production XDB Cluster",
            "self": "https://api.pagerduty.com/services/PN49J75",
            "html_url": "https://webdemo.pagerduty.com/services/PN49J75"
          },
          "incident": {
            "id": "PRORDTY",
            "type": "incident_reference",
            "summary": "[#33] My new incident",
            "self": "https://api.pagerduty.com/incidents/PRORDTY",
            "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY"
          },
          "teams": [
            {
              "id": "P4SI59S",
              "type": "team_reference",
              "summary": "Engineering",
              "self": "https://api.pagerduty.com/teams/P4SI59S",
              "html_url": "https://webdemo.pagerduty.com/teams/P4SI59S"
            }
          ],
          "contexts": [],
          "event_details": {}
        }
      ],
      "webhook": {
        "endpoint_url": "https://requestb.in/18ao6fs1",
        "name": "V2 wabhook",
        "description": null,
        "webhook_object": {
          "id": "PN49J75",
          "type": "service_reference",
          "summary": "Production XDB Cluster",
          "self": "https://api.pagerduty.com/services/PN49J75",
          "html_url": "https://webdemo.pagerduty.com/services/PN49J75"
        },
        "config": {},
        "outbound_integration": {
          "id": "PJFWPEP",
          "type": "outbound_integration_reference",
          "summary": "Generic V2 Webhook",
          "self": "https://api.pagerduty.com/outbound_integrations/PJFWPEP",
          "html_url": null
        },
        "accounts_addon": null,
        "id": "PKT9NNX",
        "type": "webhook",
        "summary": "V2 wabhook",
        "self": "https://api.pagerduty.com/webhooks/PKT9NNX",
        "html_url": null
      },
      "incident": {
        "incident_number": 33,
        "title": "My new incident",
        "description": "My new incident",
        "created_at": "2017-09-26T15:14:36Z",
        "status": "resolved",
        "pending_actions": [],
        "incident_key": null,
        "service": {
          "id": "PN49J75",
          "name": "Production XDB Cluster",
          "description": "This service was created during onboarding on July 5, 2017.",
          "auto_resolve_timeout": 14400,
          "acknowledgement_timeout": 1800,
          "created_at": "2017-07-05T17:33:09Z",
          "status": "active",
          "last_incident_timestamp": "2017-09-26T15:14:36Z",
          "teams": [
            {
              "id": "P4SI59S",
              "type": "team_reference",
              "summary": "Engineering",
              "self": "https://api.pagerduty.com/teams/P4SI59S",
              "html_url": "https://webdemo.pagerduty.com/teams/P4SI59S"
            }
          ],
          "incident_urgency_rule": {
            "type": "constant",
            "urgency": "high"
          },
          "scheduled_actions": [],
          "support_hours": null,
          "escalation_policy": {
            "id": "PINYWEF",
            "type": "escalation_policy_reference",
            "summary": "Default",
            "self": "https://api.pagerduty.com/escalation_policies/PINYWEF",
            "html_url": "https://webdemo.pagerduty.com/escalation_policies/PINYWEF"
          },
          "addons": [],
          "privilege": null,
          "alert_creation": "create_alerts_and_incidents",
          "integrations": [
            {
              "id": "PUAYF96",
              "type": "generic_events_api_inbound_integration_reference",
              "summary": "API",
              "self": "https://api.pagerduty.com/services/PN49J75/integrations/PUAYF96",
              "html_url": "https://webdemo.pagerduty.com/services/PN49J75/integrations/PUAYF96"
            },
            {
              "id": "P90GZUH",
              "type": "generic_email_inbound_integration_reference",
              "summary": "Email",
              "self": "https://api.pagerduty.com/services/PN49J75/integrations/P90GZUH",
              "html_url": "https://webdemo.pagerduty.com/services/PN49J75/integrations/P90GZUH"
            }
          ],
          "metadata": {},
          "type": "service",
          "summary": "Production XDB Cluster",
          "self": "https://api.pagerduty.com/services/PN49J75",
          "html_url": "https://webdemo.pagerduty.com/services/PN49J75"
        },
        "assignments": [
          {
            "at": "2017-09-26T15:16:05Z",
            "assignee": {
              "id": "PFBSJ2Z",
              "type": "user_reference",
              "summary": "Wiley Jacobson",
              "self": "https://api.pagerduty.com/users/PFBSJ2Z",
              "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z"
            }
          }
        ],
        "acknowledgements": [],
        "last_status_change_at": "2017-09-26T15:18:09Z",
        "last_status_change_by": {
          "id": "P553OPV",
          "type": "user_reference",
          "summary": "Laura Haley",
          "self": "https://api.pagerduty.com/users/P553OPV",
          "html_url": "https://webdemo.pagerduty.com/users/P553OPV"
        },
        "first_trigger_log_entry": {
          "id": "R2XGXEI3W0FHMSDXHDIBQGBQ5E",
          "type": "trigger_log_entry_reference",
          "summary": "Triggered through the website",
          "self": "https://api.pagerduty.com/log_entries/R2XGXEI3W0FHMSDXHDIBQGBQ5E",
          "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY/log_entries/R2XGXEI3W0FHMSDXHDIBQGBQ5E"
        },
        "escalation_policy": {
          "id": "PINYWEF",
          "type": "escalation_policy_reference",
          "summary": "Default",
          "self": "https://api.pagerduty.com/escalation_policies/PINYWEF",
          "html_url": "https://webdemo.pagerduty.com/escalation_policies/PINYWEF"
        },
        "privilege": null,
        "teams": [
          {
            "id": "P4SI59S",
            "type": "team_reference",
            "summary": "Engineering",
            "self": "https://api.pagerduty.com/teams/P4SI59S",
            "html_url": "https://webdemo.pagerduty.com/teams/P4SI59S"
          }
        ],
        "alert_counts": {
          "all": 0,
          "triggered": 0,
          "resolved": 0
        },
        "impacted_services": [
          {
            "id": "PN49J75",
            "type": "service_reference",
            "summary": "Production XDB Cluster",
            "self": "https://api.pagerduty.com/services/PN49J75",
            "html_url": "https://webdemo.pagerduty.com/services/PN49J75"
          }
        ],
        "is_mergeable": true,
        "basic_alert_grouping": null,
        "alert_grouping": null,
        "metadata": {},
        "external_references": [
          {
            "id": "PLF5V5G",
            "type": "incident_external_reference",
            "summary": "Slack",
            "self": null,
            "html_url": null,
            "external_id": "1506438878.000302",
            "external_url": "http://webdemo.slack.com/archives/C6975FLAZ/p1506438878.000302",
            "sync": false,
            "webhook": {
              "id": "PRXV5AH",
              "type": "webhook_reference",
              "summary": "Slack M2M",
              "self": "https://api.pagerduty.com/webhooks/PRXV5AH",
              "html_url": null
            }
          }
        ],
        "importance": null,
        "resolve_reason": null,
        "incidents_responders": [],
        "responder_requests": [],
        "subscriber_requests": [],
        "urgency": "high",
        "id": "PRORDTY",
        "type": "incident",
        "summary": "[#33] My new incident",
        "self": "https://api.pagerduty.com/incidents/PRORDTY",
        "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY",
        "alerts": [
          {
            "alert_key": "c24117fc42e44b44b4d6876190583378"
          },
          {
            "alert_key": "2d6ee0a6b58040f7996a4047ae5edcd7"
          }
        ]
      },
      "id": "e82195c0-a2cd-11e7-8f69-22000affca53",
      "created_on": "2017-09-26T15:18:09Z"
    }
  ]
}
```


### incident.assign


```json
{
  "messages": [
    {
      "event": "incident.assign",
      "log_entries": [
        {
          "id": "R6XNGC35VF6U1TUSVIE2DWXD4Z",
          "type": "assign_log_entry",
          "summary": "Assigned to Wiley Jacobson by Laura Haley",
          "self": "https://api.pagerduty.com/log_entries/R6XNGC35VF6U1TUSVIE2DWXD4Z",
          "html_url": null,
          "created_at": "2017-09-26T15:16:05Z",
          "agent": {
            "id": "P553OPV",
            "type": "user_reference",
            "summary": "Laura Haley",
            "self": "https://api.pagerduty.com/users/P553OPV",
            "html_url": "https://webdemo.pagerduty.com/users/P553OPV"
          },
          "channel": {
            "type": "website"
          },
          "service": {
            "id": "PN49J75",
            "type": "service_reference",
            "summary": "Production XDB Cluster",
            "self": "https://api.pagerduty.com/services/PN49J75",
            "html_url": "https://webdemo.pagerduty.com/services/PN49J75"
          },
          "incident": {
            "id": "PRORDTY",
            "type": "incident_reference",
            "summary": "[#33] My new incident",
            "self": "https://api.pagerduty.com/incidents/PRORDTY",
            "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY"
          },
          "teams": [
            {
              "id": "P4SI59S",
              "type": "team_reference",
              "summary": "Engineering",
              "self": "https://api.pagerduty.com/teams/P4SI59S",
              "html_url": "https://webdemo.pagerduty.com/teams/P4SI59S"
            }
          ],
          "contexts": [],
          "assignees": [
            {
              "id": "PFBSJ2Z",
              "type": "user_reference",
              "summary": "Wiley Jacobson",
              "self": "https://api.pagerduty.com/users/PFBSJ2Z",
              "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z"
            }
          ]
        }
      ],
      "webhook": {
        "endpoint_url": "https://requestb.in/18ao6fs1",
        "name": "V2 wabhook",
        "description": null,
        "webhook_object": {
          "id": "PN49J75",
          "type": "service_reference",
          "summary": "Production XDB Cluster",
          "self": "https://api.pagerduty.com/services/PN49J75",
          "html_url": "https://webdemo.pagerduty.com/services/PN49J75"
        },
        "config": {},
        "outbound_integration": {
          "id": "PJFWPEP",
          "type": "outbound_integration_reference",
          "summary": "Generic V2 Webhook",
          "self": "https://api.pagerduty.com/outbound_integrations/PJFWPEP",
          "html_url": null
        },
        "accounts_addon": null,
        "id": "PKT9NNX",
        "type": "webhook",
        "summary": "V2 wabhook",
        "self": "https://api.pagerduty.com/webhooks/PKT9NNX",
        "html_url": null
      },
      "incident": {
        "incident_number": 33,
        "title": "My new incident",
        "description": "My new incident",
        "created_at": "2017-09-26T15:14:36Z",
        "status": "triggered",
        "pending_actions": [
          {
            "type": "resolve",
            "at": "2017-09-26T19:14:36Z"
          }
        ],
        "incident_key": null,
        "service": {
          "id": "PN49J75",
          "name": "Production XDB Cluster",
          "description": "This service was created during onboarding on July 5, 2017.",
          "auto_resolve_timeout": 14400,
          "acknowledgement_timeout": 1800,
          "created_at": "2017-07-05T17:33:09Z",
          "status": "critical",
          "last_incident_timestamp": "2017-09-26T15:14:36Z",
          "teams": [
            {
              "id": "P4SI59S",
              "type": "team_reference",
              "summary": "Engineering",
              "self": "https://api.pagerduty.com/teams/P4SI59S",
              "html_url": "https://webdemo.pagerduty.com/teams/P4SI59S"
            }
          ],
          "incident_urgency_rule": {
            "type": "constant",
            "urgency": "high"
          },
          "scheduled_actions": [],
          "support_hours": null,
          "escalation_policy": {
            "id": "PINYWEF",
            "type": "escalation_policy_reference",
            "summary": "Default",
            "self": "https://api.pagerduty.com/escalation_policies/PINYWEF",
            "html_url": "https://webdemo.pagerduty.com/escalation_policies/PINYWEF"
          },
          "addons": [],
          "privilege": null,
          "alert_creation": "create_alerts_and_incidents",
          "integrations": [
            {
              "id": "PUAYF96",
              "type": "generic_events_api_inbound_integration_reference",
              "summary": "API",
              "self": "https://api.pagerduty.com/services/PN49J75/integrations/PUAYF96",
              "html_url": "https://webdemo.pagerduty.com/services/PN49J75/integrations/PUAYF96"
            },
            {
              "id": "P90GZUH",
              "type": "generic_email_inbound_integration_reference",
              "summary": "Email",
              "self": "https://api.pagerduty.com/services/PN49J75/integrations/P90GZUH",
              "html_url": "https://webdemo.pagerduty.com/services/PN49J75/integrations/P90GZUH"
            }
          ],
          "metadata": {},
          "type": "service",
          "summary": "Production XDB Cluster",
          "self": "https://api.pagerduty.com/services/PN49J75",
          "html_url": "https://webdemo.pagerduty.com/services/PN49J75"
        },
        "assignments": [
          {
            "at": "2017-09-26T15:16:05Z",
            "assignee": {
              "id": "PFBSJ2Z",
              "type": "user_reference",
              "summary": "Wiley Jacobson",
              "self": "https://api.pagerduty.com/users/PFBSJ2Z",
              "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z"
            }
          }
        ],
        "acknowledgements": [],
        "last_status_change_at": "2017-09-26T15:16:05Z",
        "last_status_change_by": {
          "id": "PN49J75",
          "type": "service_reference",
          "summary": "Production XDB Cluster",
          "self": "https://api.pagerduty.com/services/PN49J75",
          "html_url": "https://webdemo.pagerduty.com/services/PN49J75"
        },
        "first_trigger_log_entry": {
          "id": "R2XGXEI3W0FHMSDXHDIBQGBQ5E",
          "type": "trigger_log_entry_reference",
          "summary": "Triggered through the website",
          "self": "https://api.pagerduty.com/log_entries/R2XGXEI3W0FHMSDXHDIBQGBQ5E",
          "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY/log_entries/R2XGXEI3W0FHMSDXHDIBQGBQ5E"
        },
        "escalation_policy": {
          "id": "PINYWEF",
          "type": "escalation_policy_reference",
          "summary": "Default",
          "self": "https://api.pagerduty.com/escalation_policies/PINYWEF",
          "html_url": "https://webdemo.pagerduty.com/escalation_policies/PINYWEF"
        },
        "privilege": null,
        "teams": [
          {
            "id": "P4SI59S",
            "type": "team_reference",
            "summary": "Engineering",
            "self": "https://api.pagerduty.com/teams/P4SI59S",
            "html_url": "https://webdemo.pagerduty.com/teams/P4SI59S"
          }
        ],
        "alert_counts": {
          "all": 0,
          "triggered": 0,
          "resolved": 0
        },
        "impacted_services": [
          {
            "id": "PN49J75",
            "type": "service_reference",
            "summary": "Production XDB Cluster",
            "self": "https://api.pagerduty.com/services/PN49J75",
            "html_url": "https://webdemo.pagerduty.com/services/PN49J75"
          }
        ],
        "is_mergeable": true,
        "basic_alert_grouping": null,
        "alert_grouping": null,
        "metadata": {},
        "external_references": [
          {
            "id": "PLF5V5G",
            "type": "incident_external_reference",
            "summary": "Slack",
            "self": null,
            "html_url": null,
            "external_id": "1506438878.000302",
            "external_url": "http://webdemo.slack.com/archives/C6975FLAZ/p1506438878.000302",
            "sync": false,
            "webhook": {
              "id": "PRXV5AH",
              "type": "webhook_reference",
              "summary": "Slack M2M",
              "self": "https://api.pagerduty.com/webhooks/PRXV5AH",
              "html_url": null
            }
          }
        ],
        "importance": null,
        "incidents_responders": [],
        "responder_requests": [],
        "subscriber_requests": [],
        "urgency": "high",
        "id": "PRORDTY",
        "type": "incident",
        "summary": "[#33] My new incident",
        "self": "https://api.pagerduty.com/incidents/PRORDTY",
        "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY",
        "alerts": [
          {
            "alert_key": "c24117fc42e44b44b4d6876190583378"
          },
          {
            "alert_key": "2d6ee0a6b58040f7996a4047ae5edcd7"
          }
        ]
      },
      "id": "9e7850d0-a2cd-11e7-8f69-22000affca53",
      "created_on": "2017-09-26T15:16:05Z"
    }
  ]
}
```

<!--
type: tab
title: v1
-->

# v1 Overview


<!-- theme: warning -->
> ### Migrate to V3 Webhook Subscriptions
> If you are currently using V1/V2 webhook extensions and need to migrate them to V3 webhook subscriptions, please follow our migration guide.
>
> V1 webhook extensions became unsupported on November 13, 2021 and will lose functionality in October, 2022.


Webhooks let you receive HTTP callbacks when interesting events happen within your PagerDuty account. Details surrounding the interesting event will be sent via HTTP POST to a URL that you specify.

PagerDuty currently supports incident-based webhooks. After adding a webhook URL to a PagerDuty service, the triggering of new incidents on that service will cause outgoing webhook messages to be sent to that URL. In addition, certain interesting changes to an incident's state will cause other types of incident webhook messages to be sent. Generally, any change to the `status` or `assignees` of an incident will cause an outgoing message to be sent.


### Webhook Payload

Webhook recipients receive a payload containing a single `messages` array that may contain multiple `message` elements if webhook firing actions occurred in quick succession. Each `message` in the array consists of four fields:

<!-- TODO link to types page -->

Field Name   | Type     | Description
------------ | -------- | ------------
`id`         | UUID     | Uniquely identifies this outgoing webhook message; can be used for idempotency when processing the messages.
`type`       | String   | The webhook message type (see below).
`created_on` | DateTime | The date/time when the incident changed state.
`data`       | Object   | The incident details at the time of the state change.


### Webhook Incident Details

For incident webhooks, the `data` field contains a representation of the incident associated with the action that caused this webhook message.

The fields below can be expected to be found in the incident data. Refer to the [webhook examples](#examples) to see what the data looks like in practice.

Field Name                 | Type     | Description
-------------------------- | -------- | -----------
`id`                       | String   | The incident's id, which can be used to find it in the REST API.
`incident_number`          | Integer  | The number of the incident. This is unique across the account.
`created_on`               | DateTime | The date/time the incident was first triggered.
`status`                   | String   | The current status of the incident.
`html_url`                 | URL      |
`service`                  | Object   | A representation of the PagerDuty service that the incident belongs to. See the webhook examples for what the `service` object looks like.
`assigned_to_user`         | Object   | _Deprecated_. Use `assigned_to` instead, as the incident can have multiple assignees.
`assigned_to`              | Array    | A list of users assigned to the incident at the time of the webhook action. Each entry in the array indicates when the user was assigned.
`trigger_summary_data`     | Object   | Contains details about the event that triggered the incident.
`trigger_details_html_url` | URL      |
`last_status_change_on`    | DateTime | The time at which the status of the incident last changed.
`last_status_change_by`    | Object   | A representation of the PagerDuty user that is responsible for the incident's last status change.
`number_of_escalations`    | Integer  | Number of times the incident has been escalated.
`urgency`                  | String   | The incident's urgency.


### Webhook Types

A webhook's `type` field indicates what action caused it to be sent.

Type                     | Description
------------------------ | ------------
`incident.trigger`       | Sent when an incident is newly created/triggered.
`incident.acknowledge`   | Sent when an incident is acknowledged by a user.
`incident.unacknowledge` | Sent when an incident is unacknowledged due to its acknowledgement timing out.
`incident.resolve`       | Sent when an incident has been resolved.
`incident.assign`        | Sent when an incident has been assigned to another user. Often occurs in concert with an `acknowledge`.
`incident.escalate`      | Sent when an incident has been escalated to another user in the same escalation chain.
`incident.delegate`      | Sent when an incident has been reassigned to another escalation policy.


### V1.Examples


### incident.trigger


```json
{
  "messages": [
    {
      "type": "incident.trigger",
      "data": {
        "incident": {
          "id": "PRORDTY",
          "incident_number": 2126,
          "created_on": "2016-02-22T21:02:55Z",
          "status": "triggered",
          "pending_actions": [
            {
              "type": "escalate",
              "at": "2016-02-22T13:07:55-08:00"
            },
            {
              "type": "resolve",
              "at": "2016-02-22T17:02:55-08:00"
            }
          ],
          "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY",
          "incident_key": "17a02d0d370d4add8e53132199614121",
          "service": {
            "id": "PDS1SN6",
            "name": "Production XDB Cluster",
            "html_url": "https://webdemo.pagerduty.com/services/PDS1SN6",
            "deleted_at": null,
            "description": "Primary production datastore."
          },
          "escalation_policy": {
            "id": "P5ARF12",
            "name": "Database Team",
            "deleted_at": null
          },
          "assigned_to_user": {
            "id": "P553OPV",
            "name": "Laura Haley",
            "email": "laura.haley@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/P553OPV"
          },
          "trigger_summary_data": {
            "subject": "CPU Load High on xdb_production_echo"
          },
          "trigger_details_html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY/log_entries/Q2AIXW2ZIMCI4P",
          "trigger_type": "web_trigger",
          "last_status_change_on": "2016-02-22T21:02:55Z",
          "last_status_change_by": null,
          "number_of_escalations": 0,
          "assigned_to": [
            {
              "at": "2016-02-22T21:02:55Z",
              "object": {
                "id": "P553OPV",
                "name": "Laura Haley",
                "email": "laura.haley@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/P553OPV",
                "type": "user"
              }
            }
          ],
          "urgency": "high"
        }
      },
      "id": "a52d3f80-d9a7-11e5-8db3-22000ad5aec9",
      "created_on": "2016-02-22T21:02:55Z"
    }
  ]
}
```


### incident.acknowledge


```json
{
  "messages": [
    {
      "type": "incident.acknowledge",
      "data": {
        "incident": {
          "id": "PRORDTY",
          "incident_number": 2126,
          "created_on": "2016-02-22T21:02:55Z",
          "status": "acknowledged",
          "pending_actions": [
            {
              "type": "unacknowledge",
              "at": "2016-02-22T13:23:44-08:00"
            },
            {
              "type": "resolve",
              "at": "2016-02-22T17:02:55-08:00"
            }
          ],
          "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY",
          "incident_key": "17a02d0d370d4add8e53132199614121",
          "service": {
            "id": "PDS1SN6",
            "name": "Production XDB Cluster",
            "html_url": "https://webdemo.pagerduty.com/services/PDS1SN6",
            "deleted_at": null,
            "description": "Primary production datastore."
          },
          "escalation_policy": {
            "id": "P5ARF12",
            "name": "Database Team",
            "deleted_at": null
          },
          "assigned_to_user": {
            "id": "PFBSJ2Z",
            "name": "Wiley Jacobson",
            "email": "wiley.jacobson@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z"
          },
          "trigger_summary_data": {
            "subject": "CPU Load High on xdb_production_echo"
          },
          "trigger_details_html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY/log_entries/Q2AIXW2ZIMCI4P",
          "trigger_type": "web_trigger",
          "last_status_change_on": "2016-02-22T21:13:44Z",
          "last_status_change_by": {
            "id": "PFBSJ2Z",
            "name": "Wiley Jacobson",
            "email": "wiley.jacobson@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z"
          },
          "number_of_escalations": 0,
          "assigned_to": [
            {
              "at": "2016-02-22T21:03:21Z",
              "object": {
                "id": "PFBSJ2Z",
                "name": "Wiley Jacobson",
                "email": "wiley.jacobson@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z",
                "type": "user"
              }
            },
            {
              "at": "2016-02-22T21:13:21Z",
              "object": {
                "id": "P553OPV",
                "name": "Laura Haley",
                "email": "laura.haley@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/P553OPV",
                "type": "user"
              }
            }
          ],
          "acknowledgers": [
            {
              "at": "2016-02-22T21:13:44Z",
              "object": {
                "id": "PFBSJ2Z",
                "name": "Wiley Jacobson",
                "email": "wiley.jacobson@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z",
                "type": "user"
              }
            }
          ],
          "urgency": "high"
        }
      },
      "id": "286890b0-d9a9-11e5-bbf4-22000ae21df7",
      "created_on": "2016-02-22T21:13:44Z"
    }
  ]
}
```


### incident.unacknowledge


```json
{
  "messages": [
    {
      "type": "incident.unacknowledge",
      "data": {
        "incident": {
          "id": "PRORDTY",
          "incident_number": 2126,
          "created_on": "2016-02-22T21:02:55Z",
          "status": "triggered",
          "pending_actions": [
            {
              "type": "resolve",
              "at": "2016-02-22T17:02:55-08:00"
            }
          ],
          "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY",
          "incident_key": "17a02d0d370d4add8e53132199614121",
          "service": {
            "id": "PDS1SN6",
            "name": "Production XDB Cluster",
            "html_url": "https://webdemo.pagerduty.com/services/PDS1SN6",
            "deleted_at": null,
            "description": "Primary production datastore."
          },
          "escalation_policy": {
            "id": "P5ARF12",
            "name": "Database Team",
            "deleted_at": null
          },
          "assigned_to_user": {
            "id": "PFBSJ2Z",
            "name": "Wiley Jacobson",
            "email": "wiley.jacobson@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z"
          },
          "trigger_summary_data": {
            "subject": "CPU Load High on xdb_production_echo"
          },
          "trigger_details_html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY/log_entries/Q2AIXW2ZIMCI4P",
          "trigger_type": "web_trigger",
          "last_status_change_on": "2016-02-22T21:13:21Z",
          "last_status_change_by": null,
          "number_of_escalations": 0,
          "assigned_to": [
            {
              "at": "2016-02-22T21:03:21Z",
              "object": {
                "id": "PFBSJ2Z",
                "name": "Wiley Jacobson",
                "email": "wiley.jacobson@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z",
                "type": "user"
              }
            },
            {
              "at": "2016-02-22T21:13:21Z",
              "object": {
                "id": "P553OPV",
                "name": "Laura Haley",
                "email": "laura.haley@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/P553OPV",
                "type": "user"
              }
            }
          ],
          "urgency": "high"
        }
      },
      "id": "1acf45c0-d9a9-11e5-9ad4-22000a1798ef",
      "created_on": "2016-02-22T21:13:21Z"
    }
  ]
}
```


### incident.resolve


```json
{
  "messages": [
    {
      "type": "incident.resolve",
      "data": {
        "incident": {
          "id": "PRORDTY",
          "incident_number": 2126,
          "created_on": "2016-02-22T21:02:55Z",
          "status": "resolved",
          "pending_actions": [],
          "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY",
          "incident_key": "17a02d0d370d4add8e53132199614121",
          "service": {
            "id": "PDS1SN6",
            "name": "Production XDB Cluster",
            "html_url": "https://webdemo.pagerduty.com/services/PDS1SN6",
            "deleted_at": null,
            "description": "Primary production datastore."
          },
          "escalation_policy": {
            "id": "P55N6KH",
            "name": "Database Executive",
            "deleted_at": null
          },
          "assigned_to_user": null,
          "trigger_summary_data": {
            "subject": "CPU Load High on xdb_production_echo"
          },
          "trigger_details_html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY/log_entries/Q2AIXW2ZIMCI4P",
          "trigger_type": "web_trigger",
          "last_status_change_on": "2016-02-22T21:38:30Z",
          "last_status_change_by": {
            "id": "PFBSJ2Z",
            "name": "Wiley Jacobson",
            "email": "wiley.jacobson@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z"
          },
          "number_of_escalations": 4,
          "resolved_by_user": {
            "id": "PFBSJ2Z",
            "name": "Wiley Jacobson",
            "email": "wiley.jacobson@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z"
          },
          "assigned_to": [],
          "urgency": "high"
        }
      },
      "id": "9dd14c90-d9ac-11e5-9ad4-22000a1798ef",
      "created_on": "2016-02-22T21:38:30Z"
    }
  ]
}
```


### incident.assign


```json
{
  "messages": [
    {
      "type": "incident.assign",
      "data": {
        "incident": {
          "id": "PRORDTY",
          "incident_number": 2126,
          "created_on": "2016-02-22T21:02:55Z",
          "status": "acknowledged",
          "pending_actions": [
            {
              "type": "unacknowledge",
              "at": "2016-02-22T21:13:21Z"
            },
            {
              "type": "resolve",
              "at": "2016-02-23T01:02:55Z"
            }
          ],
          "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY",
          "incident_key": "17a02d0d370d4add8e53132199614121",
          "service": {
            "id": "PDS1SN6",
            "name": "Production XDB Cluster",
            "html_url": "https://webdemo.pagerduty.com/services/PDS1SN6",
            "deleted_at": null,
            "description": "Primary production datastore."
          },
          "escalation_policy": {
            "id": "P5ARF12",
            "name": "Database Team",
            "deleted_at": null
          },
          "assigned_to_user": {
            "id": "PFBSJ2Z",
            "name": "Wiley Jacobson",
            "email": "wiley.jacobson@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z"
          },
          "trigger_summary_data": {
            "subject": "CPU Load High on xdb_production_echo"
          },
          "trigger_details_html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY/log_entries/Q2AIXW2ZIMCI4P",
          "trigger_type": "web_trigger",
          "last_status_change_on": "2016-02-22T21:03:21Z",
          "last_status_change_by": {
            "id": "PFBSJ2Z",
            "name": "Wiley Jacobson",
            "email": "wiley.jacobson@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z"
          },
          "number_of_escalations": 0,
          "assigned_to": [
            {
              "at": "2016-02-22T21:02:55Z",
              "object": {
                "id": "P553OPV",
                "name": "Laura Haley",
                "email": "laura.haley@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/P553OPV",
                "type": "user"
              }
            },
            {
              "at": "2016-02-22T21:03:21Z",
              "object": {
                "id": "PFBSJ2Z",
                "name": "Wiley Jacobson",
                "email": "wiley.jacobson@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z",
                "type": "user"
              }
            }
          ],
          "acknowledgers": [
            {
              "at": "2016-02-22T21:03:21Z",
              "object": {
                "id": "PFBSJ2Z",
                "name": "Wiley Jacobson",
                "email": "wiley.jacobson@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z",
                "type": "user"
              }
            }
          ],
          "urgency": "high"
        }
      },
      "id": "b4dbd5e0-d9a7-11e5-9ad4-22000a1798ef",
      "created_on": "2016-02-22T21:03:21Z"
    }
  ]
}
```


### incident.escalate


```json
{
  "messages": [
    {
      "type": "incident.escalate",
      "data": {
        "incident": {
          "id": "PRORDTY",
          "incident_number": 2126,
          "created_on": "2016-02-22T21:02:55Z",
          "status": "triggered",
          "pending_actions": [
            {
              "type": "resolve",
              "at": "2016-02-22T17:02:55-08:00"
            }
          ],
          "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY",
          "incident_key": "17a02d0d370d4add8e53132199614121",
          "service": {
            "id": "PDS1SN6",
            "name": "Production XDB Cluster",
            "html_url": "https://webdemo.pagerduty.com/services/PDS1SN6",
            "deleted_at": null,
            "description": "Primary production datastore."
          },
          "escalation_policy": {
            "id": "P5ARF12",
            "name": "Database Team",
            "deleted_at": null
          },
          "assigned_to_user": {
            "id": "PGLPTJ3",
            "name": "Nina Gulgowski",
            "email": "nina.gulgowski@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/PGLPTJ3"
          },
          "trigger_summary_data": {
            "subject": "CPU Load High on xdb_production_echo"
          },
          "trigger_details_html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY/log_entries/Q2AIXW2ZIMCI4P",
          "trigger_type": "web_trigger",
          "last_status_change_on": "2016-02-22T21:28:45Z",
          "last_status_change_by": null,
          "number_of_escalations": 1,
          "assigned_to": [
            {
              "at": "2016-02-22T21:28:45Z",
              "object": {
                "id": "PGLPTJ3",
                "name": "Nina Gulgowski",
                "email": "nina.gulgowski@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/PGLPTJ3",
                "type": "user"
              }
            }
          ],
          "urgency": "high"
        }
      },
      "id": "41163e80-d9ab-11e5-bbf4-22000ae21df7",
      "created_on": "2016-02-22T21:28:45Z"
    }
  ]
}
```


### incident.delegate


```json
{
  "messages": [
    {
      "type": "incident.delegate",
      "data": {
        "incident": {
          "id": "PRORDTY",
          "incident_number": 2126,
          "created_on": "2016-02-22T21:02:55Z",
          "status": "triggered",
          "pending_actions": [
            {
              "type": "resolve",
              "at": "2016-02-23T01:37:49Z"
            }
          ],
          "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY",
          "incident_key": "17a02d0d370d4add8e53132199614121",
          "service": {
            "id": "PDS1SN6",
            "name": "Production XDB Cluster",
            "html_url": "https://webdemo.pagerduty.com/services/PDS1SN6",
            "deleted_at": null,
            "description": "Primary production datastore."
          },
          "escalation_policy": {
            "id": "P55N6KH",
            "name": "Default",
            "deleted_at": null
          },
          "assigned_to_user": {
            "id": "PFBSJ2Z",
            "name": "Wiley Jacobson",
            "email": "wiley.jacobson@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z"
          },
          "trigger_summary_data": {
            "subject": "CPU Load High on xdb_production_echo"
          },
          "trigger_details_html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY/log_entries/Q2AIXW2ZIMCI4P",
          "trigger_type": "web_trigger",
          "last_status_change_on": "2016-02-22T21:37:49Z",
          "last_status_change_by": null,
          "number_of_escalations": 4,
          "assigned_to": [
            {
              "at": "2016-02-22T21:37:49Z",
              "object": {
                "id": "PFBSJ2Z",
                "name": "Wiley Jacobson",
                "email": "wiley.jacobson@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z",
                "type": "user"
              }
            }
          ],
          "urgency": "high"
        }
      },
      "id": "85973630-d9ac-11e5-bbf4-22000ae21df7",
      "created_on": "2016-02-22T21:37:49Z"
    }
  ]
}
```

<!-- type: tab-end -->