# Introduction

PagerDuty provides a number of ways to programmatically interact with a PagerDuty account.

Whether you are looking to alter an account's configuration, pipe events from a monitoring tool into PagerDuty, update other systems when actions take place in PagerDuty, or retrieve information about activity on a PagerDuty account, there are APIs that expose this functionality to your code.

The details of how to interact with any given API are accessible in documentation guides listed in the left column navigation.

## REST API
The [REST API](../docs/rest-api/01-Overview.md) is used for accessing and manipulating data concerning all of the entities on a PagerDuty account. It is designed around RESTful principles and provides a standard suite of CRUD actions for most entities: create, read, update, and delete.

Using the REST API, you can do things like:
- add and configure users on a PagerDuty account, including how they're notified by PagerDuty
- set up workflows for responding to incidents
- find out who is on call and when they're being notified
- display a list of open or recent incidents for a team
- manually create an incident without an associated monitoring tool

<!-- theme:info -->
> The REST API should not be used to create incidents originating from monitoring systems or other automated tools - for that, use the [Events API](../docs/events-api-v2/01-Overview.md) instead. [Read more](../docs/rest-api/15-Incident-Creation-API.md) about the synchronous Incident Creation API.

Ready to dive in? Read the [REST API Overview](../docs/rest-api/01-Overview.md) to get started and consult the [API Reference](/api-reference/) for comprehensive details about each endpoint and to try it out right from the documentation.

## Events API
The Events APIs, [v2](../docs/events-api-v2/01-Overview.md) and [v1](../docs/events-api-v1/01-Overview.md) are asynchronous APIs for connecting your systems to PagerDuty via various monitoring tools. Through the Events APIs, events — which represent occurrences in the services managed by an account — are sent to PagerDuty to be processed.

If you're looking to leverage PagerDuty's Event Management features to turn data gathered from monitoring tools into actionable incidents, the Events APIs are the gateway to doing so.

### Which version of the Events API should I use?

Both versions of the Events API are designed to handle machine-generated monitoring and event data, such as infrastructure monitoring (Nagios, SignalFX, Datadog), application performance monitoring (New Relic, AppDynamics), and external site checks (Pingdom, Wormly).

However, Events API v2 provides a direct interface to set [PD-CEF fields](https://support.pagerduty.com/docs/pd-cef) in the PagerDuty alerts. This makes it easier to generate rich alert data in PagerDuty for better classification, filtering and operational intelligence.

Hence, if you are creating a new or custom integration, we strongly recommend using the Events API v2.

## Webhooks
Via [Webhooks](../docs/webhooks/01-v3-Overview.md), PagerDuty will send information about actions taking place within PagerDuty to any software of your choice that can accept an HTTP request.

Check out the [Webhooks Overview](../docs/webhooks/01-v3-Overview.md) to learn how to parse and interpret webhooks that are sent to your software, as well as the different types of webhooks that your software can receive.
