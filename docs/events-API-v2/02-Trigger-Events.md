---
tags: [events-API-v2]
---

# Send an Alert Event

## Endpoint

```http
https://events.pagerduty.com/v2/enqueue
```

### Parameters

| Parameters                           | Type             | Description                                                                                                                                                          |
| ------------------------------------ | ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `routing_key`<br/> **Required**      | String           | This is the 32 character Integration Key for an integration on a service or on a global ruleset.                                                                     |
| `event_action`<br/> **Required**     | String           | The type of event. Can be `trigger`, `acknowledge` or `resolve`.                                                                                                     |
| `dedup_key`<br/>                     | String           | Deduplication key for correlating triggers and resolves. The maximum permitted length of this property is 255 characters.                                            |
| `payload.summary`<br/> **Required**  | String           | A brief text summary of the event, used to generate the summaries/titles of any associated alerts. The maximum permitted length of this property is 1024 characters. |
| `payload.source`<br/> **Required**   | String           | The unique location of the affected system, preferably a hostname or FQDN.                                                                                           |
| `payload.severity`<br/> **Required** | String           | The perceived severity of the status the event is describing with respect to the affected system. This can be `critical`, `error`, `warning` or `info`.              |
| `payload.timestamp`                  | timestamp        | The time at which the emitting tool detected or generated the event.                                                                                                 |
| `payload.component`                  | String           | Component of the source machine that is responsible for the event, for example `mysql` or `eth0`                                                                     |
| `payload.group`                      | String           | Logical grouping of components of a service, for example `app-stack`                                                                                                 |
| `payload.class`                      | String           | The class/type of the event, for example `ping failure` or `cpu load`                                                                                                |
| `payload.custom_details`             | Object           | Additional details about the event and affected system                                                                                                               |
| `images`                             | Array of Objects | List of images to include.                                                                                                                                           |
| `links`                              | Array of Objects | List of links to include.                                                                                                                                            |

### Alert De-Duplication

Every alert event has a `dedup_key`: a string which identifies the alert triggered for the given event. The `dedup_key` can be specified when submitting the first trigger event that creates an alert. If omitted, it will be generated automatically by PagerDuty and returned in the Events API v2 response.

Submitting subsequent events with the same `dedup_key` will result in those events being applied to an open alert matching that `dedup_key`. Once the alert is resolved, any further events with the same `dedup_key` will create a new alert (for `trigger` events) or be dropped (for `acknowledge` and `resolve` events). Alerts will only be created for events with an `event_action` of `trigger`. `Acknowledge` or `resolve` events without a currently open alert will not create a new one.

Subsequent events for the same `dedup_key` will only apply to the open alert if the events are sent via the same `routing_key` as the original trigger event. Subsequent acknowledge or resolve events sent via a different `routing_key` from the original will be dropped.

A trigger event sent without a `dedup_key` will always generate a new alert because the automatically generated `dedup_key` will be a unique [UUID](../../docs/REST-API/06-Types.md#uuid).

Alerts can be further grouped into Incidents, for centralizing incident response and context. For more information on the way events, alerts, and incidents interact, please see [this knowledge base article](https://support.pagerduty.com/docs/alerts).

### Event Action Behavior

Three actions can be specified in the `event_action` parameter. Each represents a different type of activity.

| Event Action  | Behavior in PagerDuty                                                                                                                                                                                                                                                                                                                                                                                                                |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `trigger`     | A new alert is opened or a trigger log entry is created on an existing alert if one already exists with the same `dedup_key`.<br/> <br/>Use this event action when a new problem has been detected. Additional triggers _may_ be sent when a previously detected problem has occurred again.                                                                                                                                         |
| `acknowledge` | The incident referenced with the `dedup_key` will enter the acknowledged state.<br/> <br/>While an incident is acknowledged, it won't generate any additional notifications, even if it receives new trigger events.<br/> <br/>Use this event action to indicate that someone is presently working on the problem.                                                                                                                   |
| `resolve`     | The incident referenced with the `dedup_key` will enter the resolved state.<br/> <br/>Once an incident is resolved, it won't generate any additional notifications. New trigger events with the same `incident_key` / `dedup_key`as a resolved incident won't re-open the incident. Instead, a new incident will be created.<br/> <br/> Use this event action when the problem that caused the initial trigger event has been fixed. |

### Example Request Payloads

<!--
type: tab
title: Trigger
-->

```json
/*
  This example shows how to send a trigger event without a dedup_key.
  In this case, PagerDuty will automatically assign a random and unique key
  and return it in the response object.
  You should store this key in case you want to send an acknowledge or resolve
  event to this incident in the future.
*/
{
  "payload": {
    "summary": "Example alert on host1.example.com",
    "timestamp": "2015-07-17T08:42:58.315+0000",
    "source": "monitoringtool:cloudvendor:central-region-dc-01:852559987:cluster/api-stats-prod-003",
    "severity": "info",
    "component": "postgres",
    "group": "prod-datapipe",
    "class": "deploy",
    "custom_details": {
      "ping time": "1500ms",
      "load avg": 0.75
    }
  },
  "routing_key": "samplekeyhere",
  "dedup_key": "samplekeyhere",
  "images": [
    {
      "src": "https://www.pagerduty.com/wp-content/uploads/2016/05/pagerduty-logo-green.png",
      "href": "https://example.com/",
      "alt": "Example text"
    }
  ],
  "links": [
    {
      "href": "https://example.com/",
      "text": "Link text"
    }
  ],
  "event_action": "trigger",
  "client": "Sample Monitoring Service",
  "client_url": "https://monitoring.example.com"
}
```

<!--
type: tab
title: Acknowledge
-->

```json
{
  "routing_key": "routingkeyhere",
  "dedup_key": "dedupkeyhere",
  "event_action": "acknowledge"
}
```

<!--
type: tab
title: Resolve
-->

```json
{
  "routing_key": "routingkeyhere",
  "dedup_key": "dedupkeyhere",
  "event_action": "resolve"
}
```

<!-- type: tab-end -->

### Responses and Limits

See the [Events API v2 Overview page](../../docs/events-API-v2/01-Overview.md#response-codes--retry-logic) for response codes and limits.

### Context Properties

These properties can be used to attach informational assets to the incident record. Each element of these arrays is an [object](../../docs/REST-API/06-Types.md#object).

#### The `images` Property

This property is used to attach images to the incident. Each object in the list has the following properties:

| Name   | Required | Description                                                                                        |
| ------ | -------- | -------------------------------------------------------------------------------------------------- |
| `src`  | Yes      | The source (URL) of the image being attached to the incident. This image must be served via HTTPS. |
| `href` | No       | Optional URL; makes the image a clickable link.                                                    |
| `alt`  | No       | Optional alternative text for the image.                                                           |

#### The `links` Property

This property is used to attach text links to the incident. Each object in the list has the following properties:

| Name   | Required | Description                                                                            |
| ------ | -------- | -------------------------------------------------------------------------------------- |
| `href` | Yes      | URL of the link to be attached.                                                        |
| `text` | No       | Plain text that describes the purpose of the link, and can be used as the link's text. |

## Try it Out

The Events API v2 requires a `routing_key`. You can [create an Events API v2 integration on any PagerDuty service](https://support.pagerduty.com/docs/services-and-integrations#section-events-API-v2) in order to get a routing key that will route an event to that service. You can also use an integration key from a [ruleset](https://support.pagerduty.com/docs/rulesets#section-global-rulesets) to send alert events.

[Try it out here](/send-event-form/)