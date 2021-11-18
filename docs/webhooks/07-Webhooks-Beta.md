---
tags: [webhooks]
---

# Webhooks v3 Beta

import Alert from 'react-bootstrap/Alert'

<!-- theme: warning -->
### V3 Webhooks are in Early Access
The v3 Webhook features described on this page are still under development.  The features are subject to change at any time.  This page is for informational purposes only.

## Event Types

### incident.action_invocation.created
`data.type` is [`incident_action_invocation`](#incident_action_invocation)

_Brief description goes here_

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
