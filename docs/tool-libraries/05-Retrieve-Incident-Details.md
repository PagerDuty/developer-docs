---
tags: [tool-libraries]
---

# Retrieve Incident Details

The original details of any event sent from your monitoring tools to PagerDuty, whether an email sent through an email integration or JSON-encoded data through an API-based integration, can be retrieved through the REST API. This process requires the use of our Log Entries API with channels included, which will expose the original data received through the integration.

## Introduction

As noted on our [Log Entries API](https://api-reference.pagerduty.com/#!/Log_Entries/get_log_entries) documentation:

> PagerDuty keeps a log of all the events that happen to an incident. These are exposed as log entries. Log entries give you more insight into how your team or organization is handling your incidents.

Log entry data includes details about the event(s) that affected the incident throughout its lifecycle, such as:

* the data contained in events sent by the integration
* what users were notified and when
* how they were notified
* what user(s) acknowledged or resolved the incident
* any automatic actions that occurred to the incident
* other manual user actions, such as a reassignment or a note

### How to obtain the data

To obtain the event data sent through the integration, we'll be retrieving a specific type of log entry called the trigger log entry. For this, you will need the ID (a string of alphanumeric characters that starts with `P`, `Q`, or `R`) or the number of the incident that the event triggered.

In both cases, we'll be making requests to the log entries API. We will need to append the `channels` value to the array-type URL parameter `include[]` when obtaining log entries. In other words, the query part of the URL (after `?`) will contain `include[]=channels`, to enable including this feature in the response.

The two ways of getting the trigger log entry are as follows:

1. **With just the incident number (or ID):**
    1. Obtain the first trigger log entry (which initially triggered the incident) by [retrieving the incident via `GET /incidents/{id}`](https://api-reference.pagerduty.com/#!/Incidents/get_incidents_id), where `{id}` could be the ID or the incident number.
    2.  From the `first_trigger_log_entry` property in the response, make a second `GET` request to the URL in its `self` property, which should lead to [the endpoint to retrieve an individual log entry, `/log_entries/{id}`](https://api-reference.pagerduty.com/#!/Log_Entries/get_log_entries_id)
2. **With the incident ID:** You can obtain all trigger log entries in one or more API calls by:
    1. Use the [incident log entries endpoint at `/incidents/{id}/log_entries`](https://api-reference.pagerduty.com/#!/Incidents/get_incidents_id_log_entries), where `{id}` is the incident ID.
    2. Use the first log entry in the array (`log_entries` property in the response), which should contain a `log_entry` type object whose `type` value is `trigger_log_entry`.

Note that using the second method, it is possible to obtain subsequent alert data sent by the monitoring tool (after the incident was initially triggered) by iterating through all log entries and selecting the entries whose `type` value is `trigger_log_entry`.

## The channel property of log entries

Each log entry object returned will then contain a `channel`, if `channels` was in the `include` parameter. This property will itself contain an object with the triggering event data. Note the following about this property:

* The structure of the object in the channel property will vary, but it will always contain a property named `type`.
* For trigger log entries, the type property will be `api`, `email` or `web_trigger`.
* The overall structure of the `channel` object depends on the `type` value.

<!-- theme:info -->
> ### Disclaimer
> The description of the structure of the `log_entry.channel` produced here only reflects what it was observed to be as of this writing. A full specification of the `log_entry.channel` property's schema has not yet been published in our API Reference because it may be subject to change until a permanent structure is decided upon.",

Channel Type   | Description
-------------- | ------------
`api`          | It will contain the properties of the event that was received through the integration. **Note:** This type appears for both Event and REST API.
`email`        | It will contain aptly-named properties `subject`, `body`, `from` and `to`, in addition to a property `summary` containing the short event description extracted from the email per management rules (if nothing, this will be the subject header of the email).
`website`      | Will contain `subject` (short description / incident title), `details` (long description entered in the "New Incident" form) and `summary` (usually same as the title)
`mobile`       | It will contain the `summary` which is identical to the subject and details.

