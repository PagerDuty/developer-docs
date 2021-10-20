---
tags: [rest-api]
---

# Incident Creation API

The PagerDuty Incident Creation API endpoint is part of [REST API v2](../../docs/REST-API/01-Overview.md) and is usable by third parties. With this API, one can connect to a PagerDuty account and create or edit incidents on that account. This API is not to be used for connecting your monitoring tools to send events to PagerDuty; for that, use the [Events v2](../../docs/events-API-v2/01-Overview.md)

## What and Where

This API is designed around RESTful principles and conforms to the conventions of our other [RESTful API endpoints](../../docs/REST-API/05-Endpoints.md).

Unlike the Events APIs, the Incident Creation API is **heavily rate limited** on a per-account basis. It’s meant for the creation of events at "human speed" - in response to user action, rather than automated tooling.

This API is also **synchronous**, as it returns a fully-formed Incident object (rather than the incident key returned by the asynchronous Events APIs). The Events APIs accept arbitrary JSON objects, while the Incident creation API currently **only supports string Incident bodies**.

The Incident Creation API is analogous to clicking on the **Create Incident** button in PagerDuty’s web or mobile interfaces. It captures incident information from users and uses that to create a new incident, regardless of monitoring tool data.

The Incident Creation API can be used to create incidents on a service. In order to create an incident the user must pass a **type** ("incident"), **title**, and the affected **service**. Some request headers must also be specified with the request:

  * **Authorization**: All REST API calls require [Authentication](../../docs/REST-API/02-Authentication.md). In order to make successful requests to the API, you must provide a valid form of authorization.
  * **Content-Type:** `application/json`
  * **Accept**: `application/vnd.pagerduty+json;version=2`
  * **From**: The email address of a valid PagerDuty user on the account associated with the auth token.

## Making a Request

Incidents created with the Incident Creation API can be updated with the existing [Incidents API update calls](https://api-reference.pagerduty.com/#!/Incidents/post_incidents).

The Incident Creation API also accepts other optional parameters:

| Name | Description | To Note: |
|:-----|:------------|:---------|
| Body | Provides a detailed description of the incident, in addition to the summary provided in the title. | Details defined here will be included in the details of the web UI. |
| Incident Key | Similar to PagerDuty's Events APIs, the Incident Creation API allows you to pass a unique incident key to identify the incident. | If an incident exists on the provided service with the same incident key, a subsequent Incident Create request will be denied with a `400 Bad Request`. |
| Assignee | A list of users to assign to the incident. Cannot be provided if `escalation_policy` is already specified. Only one assignee is supported at this time. | When a user creates an incident assigned to themselves, the incident will be created in the [Acknowledged](https://support.pagerduty.com/docs/glossary#section-acknowledged-incident) state and will not notify the user.If an incident is created without an assigned user or escalation policy, the default escalation policy for the service will be used. |
| Escalation Policy | Assign the incident to an escalation policy instead of assigning directly to a user. Cannot be provided if assignments are already specified. | When included, the escalation policy defined will override the escalation policy defined on the service. |

## Example Requests

<!--
type: tab
title: Simple Request
-->

```json
{
  "incident": {
    "type": "incident",
    "title": "The server is on fire.",
    "service": {
      "id": "PWIXJZS",
      "type": "service_reference"
    }
  }
}
```

<!--
type: tab
title: Body Details
-->

```json
{
  "incident": {
    "type": "incident",
    "title": "Disk usage at 85%",
    "service": {
      "id": "PWIXJZS",
      "type": "service_reference"
    },
    "body": {
      "type": "incident_body",
      "details": "A disk is getting full on this machine. You should investigate what is causing the disk to fill, and ensure that there is an automated process in place for ensuring data is rotated (eg. logs should have logrotate around them). If data is expected to stay on this disk forever, you should start planning to scale up to a larger disk."
    }
  }
}
```

<!--
type: tab
title: Incident Key Details
-->

```json
{
  "incident": {
    "type": "incident",
    "title": "The server is on fire.",
    "service": {
      "id": "PWIXJZS",
      "type": "service_reference"
    },
    "incident_key": "baf7cf21b1da41b4b0221008339ff357"
  }
}
```

<!--
type: tab
title: Assignee Details
-->

```json
{
  "incident": {
    "title": "it's Friday :)",
    "service": {
      "id": "PW7YESS",
      "type": "service_reference"
    },
    "assignments": [
      {
        "assignee": {
          "id": "PZUVZZZ",
          "type": "user_reference"
        }
      }
    ]
  }
}
```

<!--
type: tab
title: Escalation Policy Details
-->

```json
{
  "incident": {
    "type": "incident",
    "title": "The server is on fire.",
    "service": {
      "id": "PWIXJZS",
      "type": "service_reference"
    },
    "escalation_policy": {
      "id": "PT20YPA",
      "type": "escalation_policy_reference"
    }
  }
}
```

<!-- type: tab-end -->

<!-- theme:warning -->
> Data accepted by the Incident Creation API and the Events APIs is different. The Events APIs accept arbitrary JSON objects, while the Incident creation API currently only supports string Incident bodies. For added customization of these fields, refer to the [Events v2 documentation](../../docs/events-API-v2/01-Overview.md).
</Alert>

## Use Case Examples

### Page a user

```json
{
  "incident": {
    "title": "Your desk is on fire!",
    "service": {
      "id": "PW7YESS",
      "type": "service_reference"
    },
    "assignments": [
      {
        "assignee": {
          "id": "PZUVZZZ",
          "type": "user_reference"
        }
      }
    ]
  }
}
```

### Page team via Escalation Policy

```json
{
  "incident": {
    "type": "incident",
    "title": "The server is on fire.",
    "service": {
      "id": "PWIXJZS",
      "type": "service_reference"
    },
    "escalation_policy": {
      "id": "PT20YPA",
      "type": "escalation_policy_reference"
    }
  }
}
```
