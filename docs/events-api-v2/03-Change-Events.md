---
tags: [events-api-v2]
---

# Send a Change Event

## Endpoint
```http
https://events.pagerduty.com/v2/change/enqueue
```

## Parameters

Parameters                                                        | Type             | Description
----------------------------------------------------------------- | ---------------- | -----------
`routing_key`<br/> **Required**       | String           | This is the 32 character Integration Key for an Integration on a Service. The same Integration Key can be used for both alert and change events.
`payload.summary`<br/> **Required**   | String           | A brief text summary of the event. Displayed in PagerDuty to provide information about the change. The maximum permitted length of this property is 1024 characters.
`payload.source`                                                  | String           | The unique name of the location where the Change Event occurred.
`payload.timestamp`                                               | Timestamp        | The time at which the emitting tool detected or generated the event.
`payload.custom_details`                                          | Object           | Additional details about the event.
`links`                                                           | Array of Objects | List of links to include.


## Example Request Payload

```json
{
  "routing_key": "samplekeyhere",
  "payload": {
    "summary": "Build Success: Increase snapshot create timeout to 30 seconds",
    "timestamp": "2020-07-17T08:42:58.315+0000",
    "source": "acme-build-pipeline-tool-default-i-9999",
    "custom_details": {
      "build_state": "passed",
      "build_number": "2",
      "run_time": "1236s"
    }
  },
  "links": [
    {
      "href": "https://acme.pagerduty.dev/build/2",
      "text": "View more details in Acme!"
    }
  ]
}
```

You should send Change Events with a valid routing key attached to a Service.
PagerDuty will process Change Events when they are received and route them to the appropriate Service.


## Context Properties

These properties can be used to attach informational assets to the incident record. Each element of these arrays is an [object.](../../docs/rest-api/06-Types.md#object)
Currently, the Change Events API only supports `links` as a custom property.

### The `links` Property

This property is used to attach text links to the Change Event. Each object in the list has the following properties:

Name   | Required | Description
------ | -------- | -----------
`href` | Yes      | URL of the link to be attached.
`text` | No       | Plain text that describes the purpose of the link, and can be used as the link's text.


## Responses and Limits

See the [Events API v2 Overview](../../docs/events-api-v2/01-Overview.md#response-codes--retry-logic) for response codes and limits.


## Data Retention

PagerDuty will only keep the last 90 days of Change Events, as per our data retention guidelines.

## Try it Out

The Events API v2 requires a `routing_key`. You can [create an Events API v2 integration on any PagerDuty service](https://support.pagerduty.com/docs/services-and-integrations#section-events-api-v2) in order to get a routing key that will route an event to that service. Note that the same Integration Key can be used to send both alert and change events.

[Try it out here](https://developer.pagerduty.com/api-reference/reference/events-v2/openapiv3.json/paths/~1change~1enqueue/post)
