---
tags: [webhooks]
---

# Early Access Webhook Events

<!-- theme: warning -->
> ### Early Access
>
> The features described on this page are in an Early Access state and are subject to change. Your PagerDuty Account may
> require a feature flag before this functionality is available to you. Please reach out to us if you have any questions or
> need support.

Early Access webhook events are provided here for informational purposes, but are subject to change in availability
(may be removed) and design (contents and shape of response may change). They can be created via the
 `/webhook_subscriptions` REST API endpoint.

## Action Invocation Events

The early access events are:

* `incident.action_invocation.updated`
* `incident.action_invocation.created`
* `incident.action_invocation.terminated`

The webhooks look like:

```json
{
  "id": "01CELD87NX07KAIBNCNNEKGT21",
  "event_type": "incident.action_invocation.created",
  "resource_type": "incident",
  "occurred_at": "2021-11-10T15:25:44Z",
  "agent": {
    "html_url": "https://acme.pagerduty.com/users/PIV35H0",
    "id": "PIV35H0",
    "self": "https://api.pagerduty.com/users/PIV35H0",
    "summary": "User 1822776",
    "type": "user_reference"
  },
  "client": null,
  "data": {
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
}

```

## Incident Workflow Events

The early access events are:

* `incident.workflow.started`
* `incident.workflow.completed`

The webhooks look like:

```json
{
  "id": "unique_event_id",
  "event_type": "incident.workflow.started",
  "resource_type": "incident",
  "occurred_at": "2023-01-05T19:35:03.642Z",
  "agent": {
    "html_url": "https://acme.pagerduty.com/users/PDJKATD",
    "id": "PDJKATD",
    "self": "https://api.pagerduty.com/users/PDJKATD",
    "summary": "User 22",
    "type": "user_reference"
  },
  "client": null,
  "data": {
    "id": "P3SNKQS",
    "type": "incident_workflow_instance",
    "summary": "A Workflow Instance Name",
    "incident_workflow": {
      "html_url": "https://acme.pagerduty.com/incident_workflows/PSFEVL7",
      "id": "PSFEVL7",
      "self": "https://api.pagerduty.com/incident_workflows/PSFEVL7",
      "summary": "A Workflow Name",
      "type": "incident_workflow_reference"
    },
    "workflow_trigger": {
      "html_url": null,
      "id": "4ad696eb-bb48-422a-8bd0-6efad6befa29",
      "self": "https://api.pagerduty.com/incident_workflows/triggers/4ad696eb-bb48-422a-8bd0-6efad6befa29",
      "summary": "Trigger Name",
      "type": "workflow_trigger_reference"
    },
    "incident": {
      "html_url": "https://acme.pagerduty.com/incidents/PBAZLIU",
      "id": "PBAZLIU",
      "self": "https://api.pagerduty.com/incidents/PBAZLIU",
      "summary": "A little bump in the road",
      "type": "incident_reference"
    },
    "service": {
      "html_url": "https://acme.pagerduty.com/services/PF9KMXH",
      "id": "PF9KMXH",
      "self": "https://api.pagerduty.com/services/PF9KMXH",
      "summary": "A service",
      "type": "service_reference"
    }
  }
}
```
