---
tags: [webhooks]
---

# v1 Overview

import Alert from 'react-bootstrap/Alert'
import Tabs from 'react-bootstrap/Tabs'
import Tab from 'react-bootstrap/Tab'
import TabContainer from 'react-bootstrap/TabContainer'


Webhooks let you receive HTTP callbacks when interesting events happen within your PagerDuty account. Details surrounding the interesting event will be sent via HTTP POST to a URL that you specify.

PagerDuty currently supports incident-based webhooks. After adding a webhook URL to a PagerDuty service, the triggering of new incidents on that service will cause outgoing webhook messages to be sent to that URL. In addition, certain interesting changes to an incident's state will cause other types of incident webhook messages to be sent. Generally, any change to the `status` or `assignees` of an incident will cause an outgoing message to be sent.

### Webhook Payload

<!-- theme: info -->
### Upgrade to v2 Webhooks</Alert.Heading>
> Looking for webhooks on notes, or more details in your webhook payload? Add a new __Generic V2 Webhook__ extension to receive the [v2 webhooks payload](../../docs/webhooks/01-Webhooks-v2-Overview.md).

Webhook recipients receive a payload containing a single `messages` array that may contain multiple `message` elements if webhook firing actions occurred in quick succession. Each `message` in the array consists of four fields:

<!-- TODO link to types page -->

Field Name   | Type     | Description
------------ | -------- | ------------
`id`         | UUID     | Uniquely identifies this outgoing webhook message; can be used for idempotency when processing the messages.
`type`       | String   | The webhook message type (see below).
`created_on` | DateTime | The date/time when the incident changed state.
`data`       | Object   | The incident details at the time of the state change.


### Webhook Incident Details

For incident webhooks, the `data` field contains a representation of the incident associated with the action that caused this webhook message.

The fields below can be expected to be found in the incident data. Refer to the [webhook examples](#examples) to see what the data looks like in practice.

Field Name                 | Type     | Description
-------------------------- | -------- | -----------
`id`                       | String   | The incident's id, which can be used to find it in the REST API.
`incident_number`          | Integer  | The number of the incident. This is unique across the account.
`created_on`               | DateTime | The date/time the incident was first triggered.
`status`                   | String   | The current status of the incident.
`html_url`                 | URL      |
`service`                  | Object   | A representation of the PagerDuty service that the incident belongs to. See the webhook examples for what the `service` object looks like.
`assigned_to_user`         | Object   | _Deprecated_. Use `assigned_to` instead, as the incident can have multiple assignees.
`assigned_to`              | Array    | A list of users assigned to the incident at the time of the webhook action. Each entry in the array indicates when the user was assigned.
`trigger_summary_data`     | Object   | Contains details about the event that triggered the incident.
`trigger_details_html_url` | URL      |
`last_status_change_on`    | DateTime | The time at which the status of the incident last changed.
`last_status_change_by`    | Object   | A representation of the PagerDuty user that is responsible for the incident's last status change.
`number_of_escalations`    | Integer  | Number of times the incident has been escalated.
`urgency`                  | String   | The incident's urgency.


### Webhook Types

A webhook's `type` field indicates what action caused it to be sent.

Type                     | Description
------------------------ | ------------
`incident.trigger`       | Sent when an incident is newly created/triggered.
`incident.acknowledge`   | Sent when an incident is acknowledged by a user.
`incident.unacknowledge` | Sent when an incident is unacknowledged due to its acknowledgement timing out.
`incident.resolve`       | Sent when an incident has been resolved.
`incident.assign`        | Sent when an incident has been assigned to another user. Often occurs in concert with an `acknowledge`.
`incident.escalate`      | Sent when an incident has been escalated to another user in the same escalation chain.
`incident.delegate`      | Sent when an incident has been reassigned to another escalation policy.


### Examples

<!--
type: tab
title: incident.trigger
-->

```json
{
  "messages": [
    {
      "type": "incident.trigger",
      "data": {
        "incident": {
          "id": "PRORDTY",
          "incident_number": 2126,
          "created_on": "2016-02-22T21:02:55Z",
          "status": "triggered",
          "pending_actions": [
            {
              "type": "escalate",
              "at": "2016-02-22T13:07:55-08:00"
            },
            {
              "type": "resolve",
              "at": "2016-02-22T17:02:55-08:00"
            }
          ],
          "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY",
          "incident_key": "17a02d0d370d4add8e53132199614121",
          "service": {
            "id": "PDS1SN6",
            "name": "Production XDB Cluster",
            "html_url": "https://webdemo.pagerduty.com/services/PDS1SN6",
            "deleted_at": null,
            "description": "Primary production datastore."
          },
          "escalation_policy": {
            "id": "P5ARF12",
            "name": "Database Team",
            "deleted_at": null
          },
          "assigned_to_user": {
            "id": "P553OPV",
            "name": "Laura Haley",
            "email": "laura.haley@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/P553OPV"
          },
          "trigger_summary_data": {
            "subject": "CPU Load High on xdb_production_echo"
          },
          "trigger_details_html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY/log_entries/Q2AIXW2ZIMCI4P",
          "trigger_type": "web_trigger",
          "last_status_change_on": "2016-02-22T21:02:55Z",
          "last_status_change_by": null,
          "number_of_escalations": 0,
          "assigned_to": [
            {
              "at": "2016-02-22T21:02:55Z",
              "object": {
                "id": "P553OPV",
                "name": "Laura Haley",
                "email": "laura.haley@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/P553OPV",
                "type": "user"
              }
            }
          ],
          "urgency": "high"
        }
      },
      "id": "a52d3f80-d9a7-11e5-8db3-22000ad5aec9",
      "created_on": "2016-02-22T21:02:55Z"
    }
  ]
}
```

<!--
type: tab
title: incident.acknowledge
-->

```json
{
  "messages": [
    {
      "type": "incident.acknowledge",
      "data": {
        "incident": {
          "id": "PRORDTY",
          "incident_number": 2126,
          "created_on": "2016-02-22T21:02:55Z",
          "status": "acknowledged",
          "pending_actions": [
            {
              "type": "unacknowledge",
              "at": "2016-02-22T13:23:44-08:00"
            },
            {
              "type": "resolve",
              "at": "2016-02-22T17:02:55-08:00"
            }
          ],
          "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY",
          "incident_key": "17a02d0d370d4add8e53132199614121",
          "service": {
            "id": "PDS1SN6",
            "name": "Production XDB Cluster",
            "html_url": "https://webdemo.pagerduty.com/services/PDS1SN6",
            "deleted_at": null,
            "description": "Primary production datastore."
          },
          "escalation_policy": {
            "id": "P5ARF12",
            "name": "Database Team",
            "deleted_at": null
          },
          "assigned_to_user": {
            "id": "PFBSJ2Z",
            "name": "Wiley Jacobson",
            "email": "wiley.jacobson@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z"
          },
          "trigger_summary_data": {
            "subject": "CPU Load High on xdb_production_echo"
          },
          "trigger_details_html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY/log_entries/Q2AIXW2ZIMCI4P",
          "trigger_type": "web_trigger",
          "last_status_change_on": "2016-02-22T21:13:44Z",
          "last_status_change_by": {
            "id": "PFBSJ2Z",
            "name": "Wiley Jacobson",
            "email": "wiley.jacobson@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z"
          },
          "number_of_escalations": 0,
          "assigned_to": [
            {
              "at": "2016-02-22T21:03:21Z",
              "object": {
                "id": "PFBSJ2Z",
                "name": "Wiley Jacobson",
                "email": "wiley.jacobson@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z",
                "type": "user"
              }
            },
            {
              "at": "2016-02-22T21:13:21Z",
              "object": {
                "id": "P553OPV",
                "name": "Laura Haley",
                "email": "laura.haley@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/P553OPV",
                "type": "user"
              }
            }
          ],
          "acknowledgers": [
            {
              "at": "2016-02-22T21:13:44Z",
              "object": {
                "id": "PFBSJ2Z",
                "name": "Wiley Jacobson",
                "email": "wiley.jacobson@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z",
                "type": "user"
              }
            }
          ],
          "urgency": "high"
        }
      },
      "id": "286890b0-d9a9-11e5-bbf4-22000ae21df7",
      "created_on": "2016-02-22T21:13:44Z"
    }
  ]
}
```

<!--
type: tab
title: incident.unacknowledge
-->

```json
{
  "messages": [
    {
      "type": "incident.unacknowledge",
      "data": {
        "incident": {
          "id": "PRORDTY",
          "incident_number": 2126,
          "created_on": "2016-02-22T21:02:55Z",
          "status": "triggered",
          "pending_actions": [
            {
              "type": "resolve",
              "at": "2016-02-22T17:02:55-08:00"
            }
          ],
          "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY",
          "incident_key": "17a02d0d370d4add8e53132199614121",
          "service": {
            "id": "PDS1SN6",
            "name": "Production XDB Cluster",
            "html_url": "https://webdemo.pagerduty.com/services/PDS1SN6",
            "deleted_at": null,
            "description": "Primary production datastore."
          },
          "escalation_policy": {
            "id": "P5ARF12",
            "name": "Database Team",
            "deleted_at": null
          },
          "assigned_to_user": {
            "id": "PFBSJ2Z",
            "name": "Wiley Jacobson",
            "email": "wiley.jacobson@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z"
          },
          "trigger_summary_data": {
            "subject": "CPU Load High on xdb_production_echo"
          },
          "trigger_details_html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY/log_entries/Q2AIXW2ZIMCI4P",
          "trigger_type": "web_trigger",
          "last_status_change_on": "2016-02-22T21:13:21Z",
          "last_status_change_by": null,
          "number_of_escalations": 0,
          "assigned_to": [
            {
              "at": "2016-02-22T21:03:21Z",
              "object": {
                "id": "PFBSJ2Z",
                "name": "Wiley Jacobson",
                "email": "wiley.jacobson@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z",
                "type": "user"
              }
            },
            {
              "at": "2016-02-22T21:13:21Z",
              "object": {
                "id": "P553OPV",
                "name": "Laura Haley",
                "email": "laura.haley@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/P553OPV",
                "type": "user"
              }
            }
          ],
          "urgency": "high"
        }
      },
      "id": "1acf45c0-d9a9-11e5-9ad4-22000a1798ef",
      "created_on": "2016-02-22T21:13:21Z"
    }
  ]
}
```

<!--
type: tab
title: incident.resolve
-->

```json
{
  "messages": [
    {
      "type": "incident.resolve",
      "data": {
        "incident": {
          "id": "PRORDTY",
          "incident_number": 2126,
          "created_on": "2016-02-22T21:02:55Z",
          "status": "resolved",
          "pending_actions": [],
          "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY",
          "incident_key": "17a02d0d370d4add8e53132199614121",
          "service": {
            "id": "PDS1SN6",
            "name": "Production XDB Cluster",
            "html_url": "https://webdemo.pagerduty.com/services/PDS1SN6",
            "deleted_at": null,
            "description": "Primary production datastore."
          },
          "escalation_policy": {
            "id": "P55N6KH",
            "name": "Database Executive",
            "deleted_at": null
          },
          "assigned_to_user": null,
          "trigger_summary_data": {
            "subject": "CPU Load High on xdb_production_echo"
          },
          "trigger_details_html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY/log_entries/Q2AIXW2ZIMCI4P",
          "trigger_type": "web_trigger",
          "last_status_change_on": "2016-02-22T21:38:30Z",
          "last_status_change_by": {
            "id": "PFBSJ2Z",
            "name": "Wiley Jacobson",
            "email": "wiley.jacobson@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z"
          },
          "number_of_escalations": 4,
          "resolved_by_user": {
            "id": "PFBSJ2Z",
            "name": "Wiley Jacobson",
            "email": "wiley.jacobson@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z"
          },
          "assigned_to": [],
          "urgency": "high"
        }
      },
      "id": "9dd14c90-d9ac-11e5-9ad4-22000a1798ef",
      "created_on": "2016-02-22T21:38:30Z"
    }
  ]
}
```

<!--
type: tab
title: incident.assign
-->

```json
{
  "messages": [
    {
      "type": "incident.assign",
      "data": {
        "incident": {
          "id": "PRORDTY",
          "incident_number": 2126,
          "created_on": "2016-02-22T21:02:55Z",
          "status": "acknowledged",
          "pending_actions": [
            {
              "type": "unacknowledge",
              "at": "2016-02-22T21:13:21Z"
            },
            {
              "type": "resolve",
              "at": "2016-02-23T01:02:55Z"
            }
          ],
          "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY",
          "incident_key": "17a02d0d370d4add8e53132199614121",
          "service": {
            "id": "PDS1SN6",
            "name": "Production XDB Cluster",
            "html_url": "https://webdemo.pagerduty.com/services/PDS1SN6",
            "deleted_at": null,
            "description": "Primary production datastore."
          },
          "escalation_policy": {
            "id": "P5ARF12",
            "name": "Database Team",
            "deleted_at": null
          },
          "assigned_to_user": {
            "id": "PFBSJ2Z",
            "name": "Wiley Jacobson",
            "email": "wiley.jacobson@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z"
          },
          "trigger_summary_data": {
            "subject": "CPU Load High on xdb_production_echo"
          },
          "trigger_details_html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY/log_entries/Q2AIXW2ZIMCI4P",
          "trigger_type": "web_trigger",
          "last_status_change_on": "2016-02-22T21:03:21Z",
          "last_status_change_by": {
            "id": "PFBSJ2Z",
            "name": "Wiley Jacobson",
            "email": "wiley.jacobson@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z"
          },
          "number_of_escalations": 0,
          "assigned_to": [
            {
              "at": "2016-02-22T21:02:55Z",
              "object": {
                "id": "P553OPV",
                "name": "Laura Haley",
                "email": "laura.haley@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/P553OPV",
                "type": "user"
              }
            },
            {
              "at": "2016-02-22T21:03:21Z",
              "object": {
                "id": "PFBSJ2Z",
                "name": "Wiley Jacobson",
                "email": "wiley.jacobson@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z",
                "type": "user"
              }
            }
          ],
          "acknowledgers": [
            {
              "at": "2016-02-22T21:03:21Z",
              "object": {
                "id": "PFBSJ2Z",
                "name": "Wiley Jacobson",
                "email": "wiley.jacobson@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z",
                "type": "user"
              }
            }
          ],
          "urgency": "high"
        }
      },
      "id": "b4dbd5e0-d9a7-11e5-9ad4-22000a1798ef",
      "created_on": "2016-02-22T21:03:21Z"
    }
  ]
}
```

<!--
type: tab
title: incident.escalate
-->

```json
{
  "messages": [
    {
      "type": "incident.escalate",
      "data": {
        "incident": {
          "id": "PRORDTY",
          "incident_number": 2126,
          "created_on": "2016-02-22T21:02:55Z",
          "status": "triggered",
          "pending_actions": [
            {
              "type": "resolve",
              "at": "2016-02-22T17:02:55-08:00"
            }
          ],
          "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY",
          "incident_key": "17a02d0d370d4add8e53132199614121",
          "service": {
            "id": "PDS1SN6",
            "name": "Production XDB Cluster",
            "html_url": "https://webdemo.pagerduty.com/services/PDS1SN6",
            "deleted_at": null,
            "description": "Primary production datastore."
          },
          "escalation_policy": {
            "id": "P5ARF12",
            "name": "Database Team",
            "deleted_at": null
          },
          "assigned_to_user": {
            "id": "PGLPTJ3",
            "name": "Nina Gulgowski",
            "email": "nina.gulgowski@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/PGLPTJ3"
          },
          "trigger_summary_data": {
            "subject": "CPU Load High on xdb_production_echo"
          },
          "trigger_details_html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY/log_entries/Q2AIXW2ZIMCI4P",
          "trigger_type": "web_trigger",
          "last_status_change_on": "2016-02-22T21:28:45Z",
          "last_status_change_by": null,
          "number_of_escalations": 1,
          "assigned_to": [
            {
              "at": "2016-02-22T21:28:45Z",
              "object": {
                "id": "PGLPTJ3",
                "name": "Nina Gulgowski",
                "email": "nina.gulgowski@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/PGLPTJ3",
                "type": "user"
              }
            }
          ],
          "urgency": "high"
        }
      },
      "id": "41163e80-d9ab-11e5-bbf4-22000ae21df7",
      "created_on": "2016-02-22T21:28:45Z"
    }
  ]
}
```

<!--
type: tab
title: incident.delegate
-->

```json
{
  "messages": [
    {
      "type": "incident.delegate",
      "data": {
        "incident": {
          "id": "PRORDTY",
          "incident_number": 2126,
          "created_on": "2016-02-22T21:02:55Z",
          "status": "triggered",
          "pending_actions": [
            {
              "type": "resolve",
              "at": "2016-02-23T01:37:49Z"
            }
          ],
          "html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY",
          "incident_key": "17a02d0d370d4add8e53132199614121",
          "service": {
            "id": "PDS1SN6",
            "name": "Production XDB Cluster",
            "html_url": "https://webdemo.pagerduty.com/services/PDS1SN6",
            "deleted_at": null,
            "description": "Primary production datastore."
          },
          "escalation_policy": {
            "id": "P55N6KH",
            "name": "Default",
            "deleted_at": null
          },
          "assigned_to_user": {
            "id": "PFBSJ2Z",
            "name": "Wiley Jacobson",
            "email": "wiley.jacobson@example.com",
            "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z"
          },
          "trigger_summary_data": {
            "subject": "CPU Load High on xdb_production_echo"
          },
          "trigger_details_html_url": "https://webdemo.pagerduty.com/incidents/PRORDTY/log_entries/Q2AIXW2ZIMCI4P",
          "trigger_type": "web_trigger",
          "last_status_change_on": "2016-02-22T21:37:49Z",
          "last_status_change_by": null,
          "number_of_escalations": 4,
          "assigned_to": [
            {
              "at": "2016-02-22T21:37:49Z",
              "object": {
                "id": "PFBSJ2Z",
                "name": "Wiley Jacobson",
                "email": "wiley.jacobson@example.com",
                "html_url": "https://webdemo.pagerduty.com/users/PFBSJ2Z",
                "type": "user"
              }
            }
          ],
          "urgency": "high"
        }
      },
      "id": "85973630-d9ac-11e5-bbf4-22000ae21df7",
      "created_on": "2016-02-22T21:37:49Z"
    }
  ]
}
```

<!-- type: tab-end -->
