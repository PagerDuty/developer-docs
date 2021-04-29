---
tags: [rest-api]
---

# Includes

Depending on your client's needs, you may want to have all of a resource's information available in the response of a related resource. Instead of just receiving the [references](../../docs/rest-api/11-References.md) (an abbreviated form of the related resource), including a type of resource will make all resources of that type contain their full set of fields in the response.

Includes are specified with the `include[]` parameter. The values allowed in an `include[]` parameter vary from endpoint to endpoint, but will always be the pluralized type of a resource or the pluralized name of a resource field representing the relationship between resources.

<!-- theme:info -->
> "include[]" is a multi-value parameter
> The `include[]` parameter may include more than one value. For instance:
> `GET /users?include[]=contact_methods&include[]=notification_rules`
> That will include both contact methods and notification rules in each of the first 25 users returned by the users index.

### Example

In the below example, the `/incidents` API is accessed with and without including `services`.
Note how including `services` causes a full `Service` [schema](../../docs/rest-api/10-Resource-Schemas.md) (with type `service`) to be displayed in the `service` field of the incident, while without the `include[]` the `service` field contains a `ServiceReference` schema (with type `service_reference`).

<!--
type: tab
title: without include[] values
-->

```json
// # https://api.pagerduty.com/incidents

{
  "incidents": [
    {
      "incident_number": 45938,
      "created_at": "2016-01-23T17:49:10Z",
      "status": "resolved",
      "pending_actions": [],
      "incident_key": "srv01/HTTP",
      "service": {
        "id": "P3U7V58",
        "type": "service_reference",
        "summary": "Website",
        "self": "https://webdemo.pagerduty.com/api/v1/services/P3U7V58",
        "html_url": "https://webdemo.pagerduty.com/services/P3U7V58"
      },
      "assignments": [],
      "acknowledgements": null,
      "last_status_change_at": "2016-01-23T21:49:10Z",
      "last_status_change_by": {
        "id": "P3U7V58",
        "type": "service_reference",
        "summary": "Website",
        "self": "https://webdemo.pagerduty.com/api/v1/services/P3U7V58",
        "html_url": "https://webdemo.pagerduty.com/services/P3U7V58"
      },
      "first_trigger_log_entry": {
        "id": "Q1869ITH09X2CV",
        "type": "trigger_log_entry_reference",
        "summary": "Triggered through the API",
        "self": "https://webdemo.pagerduty.com/api/v1/log_entries/Q1869ITH09X2CV",
        "html_url": "https://webdemo.pagerduty.com/incidents/P6X6LJU/log_entries/Q1869ITH09X2CV"
      },
      "escalation_policy": {
        "id": "PEEJXL1",
        "type": "escalation_policy_reference",
        "summary": "DevOps team",
        "self": "https://webdemo.pagerduty.com/api/v1/escalation_policies/PEEJXL1",
        "html_url": "https://webdemo.pagerduty.com/escalation_policies/PEEJXL1"
      },
      "teams": [
        {
          "id": "PNOIFIV",
          "type": "team_reference",
          "summary": "Ops Team",
          "self": "https://webdemo.pagerduty.com/api/v1/teams/PNOIFIV",
          "html_url": "https://webdemo.pagerduty.com/teams/PNOIFIV"
        }
      ],
      "urgency": "high",
      "id": "P6X6LJU",
      "type": "incident",
      "summary": "[#45938] The server is on fire.",
      "self": "https://webdemo.pagerduty.com/api/v1/incidents/P6X6LJU",
      "html_url": "https://webdemo.pagerduty.com/incidents/P6X6LJU"
    }
  ],
  "limit": 1,
  "offset": 0,
  "total": null,
  "more": true
}
```

<!--
type: tab
title: with included services
-->

```json
// # https://api.pagerduty.com/incidents?include[]=services

{
  "incidents": [
    {
      "incident_number": 45938,
      "created_at": "2016-01-23T17:49:10Z",
      "status": "resolved",
      "pending_actions": [],
      "incident_key": "srv01/HTTP",
      "service": {
        "id": "P3U7V58",
        "name": "Website",
        "description": "Website Monitoring",
        "service_key": "73c3047621974a99a83e91babdd4a2ba",
        "auto_resolve_timeout": 14400,
        "acknowledgement_timeout": 1800,
        "created_at": "2015-07-27T18:16:20Z",
        "status": "critical",
        "last_incident_timestamp": "2016-02-23T17:19:38Z",
        "teams": [
          {
            "id": "PNOIFIV",
            "type": "team_reference",
            "summary": "Ops Team",
            "self": "https://webdemo.pagerduty.com/api/v1/teams/PNOIFIV",
            "html_url": "https://webdemo.pagerduty.com/teams/PNOIFIV"
          }
        ],
        "incident_urgency_rule": {
          "type": "constant",
          "urgency": "high"
        },
        "scheduled_actions": [],
        "support_hours": null,
        "integrations": [
          {
            "id": "P05SJ4C",
            "type": "generic_events_api_inbound_integration_reference",
            "summary": "Generic API",
            "self": "https://webdemo.pagerduty.com/api/v1/services/P3U7V58/integrations/P05SJ4C",
            "html_url": "https://webdemo.pagerduty.com/services/P3U7V58/integrations/P05SJ4C"
          }
        ],
        "escalation_policy": {
          "id": "PEEJXL1",
          "type": "escalation_policy_reference",
          "summary": "DevOps team",
          "self": "https://webdemo.pagerduty.com/api/v1/escalation_policies/PEEJXL1",
          "html_url": "https://webdemo.pagerduty.com/escalation_policies/PEEJXL1"
        },
        "type": "service",
        "summary": "Website",
        "self": "https://webdemo.pagerduty.com/api/v1/services/P3U7V58",
        "html_url": "https://webdemo.pagerduty.com/services/P3U7V58"
      },
      "assignments": [],
      "acknowledgements": null,
      "last_status_change_at": "2016-01-23T21:49:10Z",
      "last_status_change_by": {
        "id": "P3U7V58",
        "type": "service_reference",
        "summary": "Website",
        "self": "https://webdemo.pagerduty.com/api/v1/services/P3U7V58",
        "html_url": "https://webdemo.pagerduty.com/services/P3U7V58"
      },
      "first_trigger_log_entry": {
        "id": "Q1869ITH09X2CV",
        "type": "trigger_log_entry_reference",
        "summary": "Triggered through the API",
        "self": "https://webdemo.pagerduty.com/api/v1/log_entries/Q1869ITH09X2CV",
        "html_url": "https://webdemo.pagerduty.com/incidents/P6X6LJU/log_entries/Q1869ITH09X2CV"
      },
      "escalation_policy": {
        "id": "PEEJXL1",
        "type": "escalation_policy_reference",
        "summary": "DevOps team",
        "self": "https://webdemo.pagerduty.com/api/v1/escalation_policies/PEEJXL1",
        "html_url": "https://webdemo.pagerduty.com/escalation_policies/PEEJXL1"
      },
      "teams": [
        {
          "id": "PNOIFIV",
          "type": "team_reference",
          "summary": "Ops Team",
          "self": "https://webdemo.pagerduty.com/api/v1/teams/PNOIFIV",
          "html_url": "https://webdemo.pagerduty.com/teams/PNOIFIV"
        }
      ],
      "urgency": "high",
      "id": "P6X6LJU",
      "type": "incident",
      "summary": "[#45938] The server is on fire.",
      "self": "https://webdemo.pagerduty.com/api/v1/incidents/P6X6LJU",
      "html_url": "https://webdemo.pagerduty.com/incidents/P6X6LJU"
    }
  ],
  "limit": 1,
  "offset": 0,
  "total": null,
  "more": true
}
```

<!-- type: tab-end -->
