---
tags: [webhooks]
---

# Webhooks Early Access

<!-- theme: warning -->
> **Early Access Webhooks v3 Features**
> 
> The items described on this page are still under development and are subject to change at any time. This page is for informational purposes only.

## Event Types

### incident.action_invocation.created
`data.type` is [`incident_action_invocation`](#incident_action_invocation)

A Rundeck Action has been newly invoked on an existing Incident. The resource representing this Invocation has a state of "created", indicating it exists, but no progress has been made on it as yet.

### incident.action_invocation.updated
`data.type` is [`incident_action_invocation`](#incident_action_invocation)

A Rundeck Action Invocation, running on an Incident, has had its lifecycle state updated. The Invocation is still considered active, and further lifecycle updates are expected.

### incident.action_invocation.terminated
`data.type` is [`incident_action_invocation`](#incident_action_invocation)

A Rundeck Action Invocation, running on an Incident, has had its lifecycle state updated to a terminal state. This means it has completed, either successfully or in error, and no further updates will be made.

## Event Data Types

### incident_action_invocation

```json
{
  "id": "01CELD6T9C2JS745I7CAK0LRRF",
  "self": "https://api.pagerduty.com/automation/invocations/01CELD6T9C2JS745I7CAK0LRRF",
  "html_url": "https://acme.pagerduty.com/rundeck-actions/actions/01CDYN0IRV4VG991K5FR73YNTW/invocations/01CELD6T9C2JS745I7CAK0LRRF/report",
  "incident": {
    "html_url": "https://acme.pagerduty.com/incidents/PBAZLIU",
    "id": "PBAZLIU",
    "self": "https://api.pagerduty.com/incidents/PBAZLIU",
    "summary": "An Incident",
    "type": "incident_reference"
  },
  "action": {
    "html_url": "https://acme.pagerduty.com/rundeck-actions/actions/01CDYN0IRV4VG991K5FR73YNTW",
    "id": "01CDYN0IRV4VG991K5FR73YNTW",
    "self": "https://api.pagerduty.com/automation/actions/01CDYN0IRV4VG991K5FR73YNTW",
    "summary": "A Helpful Action",
    "type": "action_reference"
  },
  "state": "created",
  "type": "incident_action_invocation"
}
```
