---
tags: [events-API-v2]
---

# Events API v2 Overview


The Events API v2 is a highly reliable, highly available **asynchronous API** that ingests machine events from monitoring tools and other systems like code repositories, observability platforms, automated workflow tools, and configuration management systems. Events sent to this API are ultimately routed to a [PagerDuty service](https://support.pagerduty.com/docs/services-and-integrations) and processed.

## Event Types

The Events API v2 can ingest multiple types of events. Each event type is described below.


Event Type       | Description | Example Events | Notifications can be sent? |
---------------- | ----------- | -------------- | ---------------------------|
[Alert](../../docs/events-API-v2/02-Trigger-Events.md)| A problem in a machine monitored system. <br/> <br/> Follow up events can be sent to acknowledge or resolve an existing alert. | High error rate <br/><br/>CPU usage exceeded limit <br/><br/>Deployment failed | Yes |
[Change](../../docs/events-API-v2/03-Change-Events.md)| A change in a system that does not represent a problem. | Pull request merged<br/><br/>Secret successfully rotated<br/><br/>Configuration update applied | No |

*Note on ITSM and Ticketing:*

Previously, the Events API was used to create PagerDuty incidents from ticketing systems and various other collaboration tools. Today, we recommend using the synchronous [Incidents API](https://api-reference.pagerduty.com/#!/Incidents/get_incidents) for this purpose.

## How Events Are Used In PagerDuty

### Alert Events

* Alert events create incidents on a service in PagerDuty. The incident will be assigned to the person on-call. This will generate a notification (phone call, SMS, email, or mobile push notification dependng on the on-call responder's preferences).
* If an alert already exists for a problem, it can be grouped into a single incident (see [Alert De-Duplication](../../docs/events-API-v2/02-Trigger-Events.md#alert-de-duplication))


[Try it out here](https://developer.pagerduty.com/api-reference/b3A6Mjc0ODI2Nw-send-an-event-to-pager-duty)

### Change Events

Change events provide context to responders when triaging an incident. Currently, they are shown in the PagerDuty web application in these places.
* [Service activity stream](https://support.pagerduty.com/docs/change-events#view-change-events-on-a-service)
* [Incident details page](https://support.pagerduty.com/docs/change-events#view-most-recent-change-events-on-an-incident) (most recent change events)
* [Recent changes page](https://support.pagerduty.com/docs/change-events#view-and-filter-all-change-events-across-services-in-the-service-directory)

[Learn more about utilizing change events in PagerDuty](https://support.pagerduty.com/docs/change-events#view-change-events)

[Try it out here!](https://developer.pagerduty.com/api-reference/b3A6Mjc0ODI2Ng-send-change-events-to-the-pager-duty-events-api)

## Getting Started

To quickly start using the Events API v2 with PagerDuty services:

1. [Create an integration on any PagerDuty service](https://support.pagerduty.com/docs/services-and-integrations#section-events-API-v2).
2. Select **Events API v2** as the Integration Type
3. Include the integration key for your new integration, as a `routing_key` in the event payload.
4. Send a [change-event](../../docs/events-API-v2/03-Change-Events.md) or [alert event](../../docs/events-API-v2/02-Trigger-Events.md) payload to the appropriate endpoint.

[View PagerDuty API client libraries.](../../docs/tool-libraries/01-Client-Libraries.md)

Service integration keys can be used to send any type of event. Currently, change events cannot be sent to ruleset integration keys (these keys begin with the character `R`).

<!-- theme:info -->
> ### Have you built an integration using the Events API?
> Let us know and [become a PagerDuty integration partner](https://pdconnect.wufoo.com/forms/contact-our-partnerships-team/)

## Response Codes & Retry Logic

The following table shows the possible results of the API request and if you need to retry the API call for that result:

Response Code    | Description | Retry?
---------------- | --------------------------------------------------------------------------------------------- | ---
202              | Accepted - The event has been accepted by PagerDuty.                                          | No
400              | Bad Request - Check that the JSON is valid.                                                   | No
429              | Too many API calls at a time.                                                                 | Yes - retry after some time
500 or other 5XX | Internal Server Error - the PagerDuty server experienced an error while processing the event. | Yes - retry after some time
Network Error    | Error while trying to communicate with PagerDuty servers.                                     | Yes - retry after some time


## API Limits

### Rate Limits

There is a limit on the number of events that a service can accept at any given time. Depending on the behaviour of the incoming traffic and how many incidents are being created at once, we dynamically adjust our throttle limits.

If each of the events your monitoring system is sending is important, be sure to retry on a 429 response code, preferably with a backoff of a few minutes.

### Size Limits

Events API payloads are limited to **512 KB**


## PagerDuty Common Event Format (PD-CEF)

The Events API v2 accepts data in the [PagerDuty Common Event Format](https://support.pagerduty.com/docs/pd-cef), a schema for operations data released by PagerDuty in 2016. PD-CEF helps PagerDuty provide a more useful and consistent experience for our customers across all their different monitoring systems. Across different integrations, event types, services, and teams, users can see and use data to rapidly assess emerging situations, customize event, alert, and incident workflow, and optimize their operational processes.

Field          | Description                                                                                                                                 | Type                 | Required?                                                       | Examples
-------------- | ------------------------------------------------------------------------------------------------------------------------------------------- | -------------------- | ----------------------------------------------------------------| ---------
Summary        | A high-level, text summary message of the event. Will be used to construct an alert's description.                                          | String               | Required                                                        | "PING OK - Packet loss = 0%, RTA = 1.41 ms"<br/>"Host 'acme-andromeda-sv1-c40 :: 179.21.24.50' is DOWN"
Severity       | How impacted the affected system is. Displayed to users in lists and influences the priority of any created incidents.                      | Enum                 | Required                                                        | Enum<br/>{info, warning, error, critical}
Source         | Specific human-readable unique identifier, such as a hostname, for the system having the problem.                                           | String               | Required                                                        | "prod05.theseus.acme-widgets.com"<br/>"171.26.23.22"<br/>"aws:elasticache:us-east-1:852511987:cluster/api-stats-prod-003"<br/>"9c09acd49a25"
Timestamp      | When the upstream system detected / created the event. This is useful if a system batches or holds events before sending them to PagerDuty. | Timestamp (ISO 8601) | Optional - Will be auto-generated by PagerDuty if not provided. | 2015-07-17T08:42:58.315+0000
Component      | The part or component of the affected system that is broken.                                                                                | String               | Optional                                                        | "keepalive"<br/>"webping"<br/>"mysql"<br/>"wqueue"
Group          | A cluster or grouping of sources. For example, sources "prod-datapipe-02" and "prod-datapipe-03" might both be part of "prod-datapipe"      | String               | Optional                                                        | "prod-datapipe"<br/>"www"<br/>"web_stack"
Class          | The class/type of the event                                                                                                                 | String               | Optional                                                        | "High CPU"<br/>"Latency"<br/>"500 Error"
Custom Details | Free-form details from the event                                                                                                            | Object               | Optional                                                        | {"ping time": "1500ms", "load avg": 0.75 }


For more information about PD-CEF and how it is used in PagerDuty, please see [this knowledge base article](https://support.pagerduty.com/docs/pd-cef).

As a PagerDuty monitoring partner, adapting the new PD-CEF format provides your customers with a more streamlined and useful user experience within PagerDuty. Customers can see the most important details about an event front-and-center during initial triage, enabling them to rapidly determine whether a problem needs further investigation within the monitoring system, or whether action can immediately be taken. If you are a monitoring partner and interested in converting your integration to use PD-CEF, please email us at <a href="mailto:partners@pagerduty.com">partners@pagerduty.com</a>.
