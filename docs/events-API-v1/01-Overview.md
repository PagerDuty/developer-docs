---
tags: [events-api-v1]
---

# Events API v1

The PagerDuty Events API is used to add PagerDuty's advanced incident management functionality to any system that can make an HTTP API call. You can now add phone, SMS, email, and mobile push notifications to your monitoring tools, ticketing systems, and custom software.

## Description

Events API v1 was designed to allow you to easily integrate a monitoring system with a [service](https://support.pagerduty.com/docs/services-and-integrations) in PagerDuty. Monitoring systems generally send out events when problems are detected and when these problems have been resolved (fixed). Some more advanced systems also understand the concept of acknowledgements: problems can be acknowledged by an engineer to signal that he or she is working on fixing the issue.

Since monitoring systems emit events, the Events API is based around accepting events. Incoming events are routed to a PagerDuty service and processed. They may result in a new incident being created, or an existing incident being acknowledged or resolved.

The same event-based API can also be used to integrate a PagerDuty service with ticketing systems and various other software tools.

To accomplish customized integrations with PagerDuty, you may be interested in [building an app or integration with PagerDuty](../../docs/app-integration-development/03-Register-An-App.md)

## Getting Started

The Events API can be used with any service that has an Events API v1 integration, or by [creating a new service or integration](https://support.pagerduty.com/docs/services-and-integrations#section-create-a-generic-events-api-integration) with the option **Use our API directly**.

## Making a Request

To make an Events API v1 request, POST a JSON body (size limited to **512 KB**) representing the event to the following URL:

```
https://events.pagerduty.com/generic/2010-04-15/create_event.json
```

<!-- theme:warning -->
> ### Note
> `2010-04-15` is a version identifier. Do not change the URL to today's date, or any other date.

Head on over to [Create Event](../../docs/events-api-v1/02-Trigger-Events.md) to see the full details of the event body format.

<!-- theme:info -->
> ### Tip
> Want a simpler way to send events to PagerDuty? The [PagerDuty Agent](../../docs/tool-libraries/03-On-Premise-Agent.md) provides an installable command-line utility for Linux-based systems that handles all the details of event queueing and sending.

## Event Types

There are three types of events that PagerDuty recognizes, and are used to represent different types of activity in your monitored systems.

Event Type    | Description
------------- | ------------
`trigger`     | When PagerDuty receives a `trigger` event, it will either open a new incident, or add a new `trigger` log entry to an existing incident, depending on the provided `incident_key`.<br/> <br/> If there is no one on call for the service's escalation policy when the event is triggered, an incident will not be created.<br/> <br/>Your monitoring tools should send PagerDuty a trigger event to report a new or ongoing problem.
`acknowledge` | `acknowledge` events cause the referenced incident to enter the acknowledged state.<br/> <br/>While an incident is acknowledged, it won't generate any additional notifications, even if it receives new trigger events.<br/> <br/>Your monitoring tools should send PagerDuty an `acknowledge` event when they know someone is presently working on the problem.
`resolve`     | `resolve` events cause the referenced incident to enter the resolved state.<br/> <br/>Once an incident is resolved, it won't generate any additional notifications. New trigger events with the same `incident_key` as a resolved incident won't re-open the incident. Instead, a new incident will be created.<br/> <br/> Your monitoring tools should send PagerDuty a `resolve` event when the problem that caused the initial trigger event has been fixed.


## Incident De-duplication and incident_key

Every incident has an `incident_key`: a string which identifies the incident. The `incident_key` can be specified when submitting the first `trigger` event that creates an incident. If omitted, it will be generated automatically by PagerDuty and returned in the Events API response.

Submitting subsequent events for the same `incident_key` will result in those events being applied to an open incident matching that `incident_key`. Once the incident is `resolved`, any further events with the same `incident_key` will create a new incident (for `trigger` events) or be dropped (for `acknowledge` and `resolve` events).

Subsequent events for the same `incident_key` will only apply to the open incident if the events are sent via the same `service_key` as the original `trigger` event. Subsequent `acknowledge` or `resolve` events sent via a different `service_key` from the original will be dropped. Incidents created within the PagerDuty web application are inaccessible via the Events API.

If a `trigger` event is sent with no `incident_key`, it will always generate a new incident because the key is a [UUID](../../docs/rest-api/06-Types.md#uuid).

## API Response Codes & Retry Logic

The following table shows the possible results of the API request and if you need to retry the API call for that result:

Response Code    | Description                                                                                   | Retry?
---------------- | --------------------------------------------------------------------------------------------- | ---
200              | Event was received and processed successfully                                                 | No
400              | Invalid event format (i.e. JSON parse error) or incorrect event structure                     | No
403              | Rate limit reached (too many Events API requests)                                             | Yes - retry after some time
500 or other 5XX | Internal Server Error - the PagerDuty server experienced an error while processing the event. | Yes - retry after some time
Network Error    | Error while trying to communicate with PagerDuty servers.                                     | Yes - retry after some time
