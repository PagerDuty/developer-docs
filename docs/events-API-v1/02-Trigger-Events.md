---
tags: [events-api-v1]
---

# Send a v1 Event

## Endpoint

```http
https://events.pagerduty.com/generic/2010-04-15/create_event.json
```

## Parameters

Parameters                     | Type                | Description
------------------------------ | ------------------- | -------------
`service_key`                  | String              | A version 4 UUID expressed as a 32-digit hexadecimal number. This is the Integration Key for an integration on any given service.
`event_type`                   | String              | The type of event. Can be `trigger`, `acknowledge` or `resolve`.
`incident_key`                 | String              | Identifies the incident to `trigger`, `acknowledge`, or `resolve`. Required unless the event_type is `trigger`. The maximum permitted length of this property is 255 characters.
`description`                  | String              | Text that will appear in the incident's log associated with this event. Required for trigger events. The maximum permitted length of this property is 1024 characters.
`details`                      | Object              | An arbitrary JSON object containing any data you'd like included in the incident log.
`client`                       | String              | The name of the monitoring client that is triggering this event. (This field is only used for trigger events.)
`client_url`                   | String              | The URL to view the analogous event in the UI of the monitoring client, if any. (This field is only used for `trigger` events.)
`contexts`                     | Array of Objects    | Contexts to be included with the incident trigger such as links to graphs or images (see format below). (This field is only used for `trigger` events.)

## Using the Events API

Your monitoring tools should send PagerDuty a `trigger` event to report a new or ongoing problem. When PagerDuty receives a `trigger` event, it will either open a new incident, or add a new trigger [log entry](https://api-reference.pagerduty.com/#!/Log_Entries/get_log_entries) to an existing incident, depending on the provided `incident_key`.

Read more about event types and incident de-duplication in the [Events API v1 Overview](./01-Overview.md).

## Example Request Payloads

<!--
type: tab
title: Trigger Event
-->

```json
/*
  This example shows how to send a trigger event without an incident_key.
  In this case, PagerDuty will automatically assign a random and unique key
  and return it in the response object.
  You should store this key in case you want to send an acknowledge or resolve
  event to this incident in the future.
*/

{
  "service_key": "e93facc04764012d7bfb002500d5d1a6",
  "event_type": "trigger",
  "description": "FAILURE for production/HTTP on machine srv01.acme.com",
  "client": "Sample Monitoring Service",
  "client_url": "https://monitoring.service.com",
  "details": {
    "ping time": "1500ms",
    "load avg": 0.75
  },
  "contexts": [
    {
      "type": "link",
      "href": "http://acme.pagerduty.com",
      "text": "View in custom tool"
    },
    {
      "type": "link",
      "href": "http://acme.pagerduty.com",
      "text": "View the incident on PagerDuty"
    },
    {
      "type": "image",
      "src": "https://chart.googleapis.com/chart?chs=600x400&chd=t:6,2,9,5,2,5,7,4,8,2,1&cht=lc&chds=a&chxt=y&chm=D,0033FF,0,0,5,1"
    }
  ]
}
```
<!--
type: tab
title: Acknowledge Event
-->

```json
/*
  This example acknowledges an existing open incident with the incident key 'srv01/HTTP'.
  If there is no open incident with that service key, no action will be taken.
*/

{
  "service_key": "e93facc04764012d7bfb002500d5d1a6",
  "event_type": "acknowledge",
  "incident_key": "srv01/HTTP"
}
```

<!--
type: tab
title: Resolve Event
-->

```json
/*
  This example resolves an existing open incident with the incident key 'srv01/HTTP'.
  If there is no open incident with that service key, no action will be taken.
*/

{
  "service_key": "e93facc04764012d7bfb002500d5d1a6",
  "event_type": "resolve",
  "incident_key": "srv01/HTTP"
}
```

<!-- type: tab-end -->

## Response Format

If the event is improperly formatted, a `400 Bad Request` will be returned.

There is a limit on the number of events that a service can accept at any given time. Depending on the behavior of the incoming traffic and how many incidents are being created at once, we reduce our throttle dynamically to ensure that important events continue to be processed.

If the service has received too many events, a `403 Forbidden` will be returned. If it is vital that all events your monitoring tool sends be received, be sure to retry on a 403 response code (preferably with a backoff period).

<!--
type: tab
title: 200 Event Processed
-->

```json
{
  "status": "success",
  "message": "Event processed",
  "incident_key": "73af7a305bd7012d7c06002500d5d1a6"
}
```

<!--
type: tab
title: 400 Invalid Event
-->

```json
{
    "status": "invalid event",
    "message": "Event object is invalid",
    "errors": [ // This errors array indicates why the event was invalid
        "event_type is invalid (must be one of: trigger acknowledge resolve)"
    ]
}
```

<!--
type: tab
title: 403 Rate Limited
-->


<!-- type: tab-end -->

## Contexts

The `contexts` field is a JSON array of informational assets that can be attached to the incident. Every element of the array is a JSON object referred to as a `context`.

Every `context` must have a `type`. There are a few different types of contexts supported; the fields allowed and required depend on the context type.

### Link Context

The `link` type is used to attach hyperlinks to an incident.

Name   | Required | Description
------ | -------- | -----------
`type` | Yes      | The type of context being attached. For link contexts, this must be `link`.
`href` | Yes      | The link being attached to the incident.
`text` | No       | Plain text that describes the purpose of the link, and can be used as the link's text.


### Image Context

The `image` type is used to attach images to an incident. Images must be served via HTTPS.

Name   | Required | Description
------ | -------- | -----------
`type` | Yes      | The type of context being attached to the incident. For image contexts, this must be `image`.
`src`  | Yes      | The source (URL) of the image being attached to the incident. This image must be served via HTTPS.
`href` | No       | Optional link for the image.
`alt`  | No       | Optional alternative text for the image.


