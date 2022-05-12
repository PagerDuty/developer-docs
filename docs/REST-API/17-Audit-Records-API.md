---
tags: [rest-api]
---

# Audit Records API

## Overview

Audit records are produced when configuration changes are made to certain resources in PagerDuty (see the [list of resource types](#resources-tracked) below). Over time, more resources will be tracked. You can use the Audit API to:
* Proactively monitor your PagerDuty instance for problems or to adhere to best practices
* Periodically send audit records to a security information and event management (SIEM) tool for querying and alerting
* Determine if a PagerDuty configuration change led to a problem during incident response
* Write a script to generate compliance reports
* Track when users are created / deleted

## Resources Tracked

Resource Type       | Associated Resources                | First Records Produced
------------------- | ----------------------------------- | -----------------------
Escalation policies | Teams                               | December 2, 2020
Schedules           | Teams                               | December 2, 2020
Services            | Integrations, Teams                 | December 2, 2020
Teams               | Team membership changes             | December 2, 2020
Users               | Contact methods, Notification rules | December 2, 2020

More resource types may be added to this list over time. If you don't see a resource type you're interested in, please [contact our Support Team](mailto:support@pagerduty.com) or your PagerDuty rep to let them know.

## API Endpoints

<!-- theme:info -->
> The `/audit/records` endpoint is only available to customers with certain pricing plans. The API returns a `HTTP 402 - Payment Required` error when an account does not have access to this feature. Please see [PagerDuty pricing](https://www.pagerduty.com/pricing/) for more information about which plans include the Admin Audit API.

Endpoint                                   |  Permissions Required  | Example Usage
------------------------------------------ | ---------------------- | -------------
**Admin (account-wide)**<br/>Access all audit records<br/><br/> [`/audit/records/` View API Reference](/api-reference/reference/REST/openapiv3.json/paths/~1audit~1records/get) | Admin (global API token or user / OAuth token for an admin user) | Show records of all users deleted between May and July
**Resource-level**<br/>Access audit records of changes made to a single resource<br/><br/>`/{resource_type}/{id}/audit/records/` | Any API token that has Read access to the resource | Show records of changes made to Ellen's user profile

## API Access
The Audit API is part of PagerDuty's REST API. Please see documentation for the [REST API](../../docs/REST-API/01-Overview.md) for more information about authentication, pagination, rate limiting, etc.

## Request Format & Errors

Please see the [API Reference](/api-reference/reference/REST/openapiv3.json/paths/~1audit~1records/get) for detailed information about the request format (including filtering query parameters), the response format, and possible error codes. You can also make a request directly from the API Reference.

The Admin audit endpoint is under **Audit** in the side navigation. Resource-level endpoints are labeled **List audit records for a...** under each resource type. For example, you can [view the audit records for a single service](/api-reference/reference/REST/openapiv3.json/paths/~1services~1%7Bid%7D~1audit~1records/get).

## Record Format
Each record describes an `ACTION` executed on an aggregate (identified by the aggregate "root_resource") by `ACTOR`s using a `METHOD`.

### Record Skeleton

```json {.line-numbers}
{
   "records":[
      {
         "action":"update",                     # <-- The action (from the aggregate point of view)
         "actors": [
            {
              "id": "USERID",
              "type": "user_reference"
            }
          ],
          "method": {
            "type": "method_type"
          },
         "details":{
            // ...
            // Change details
            // ...
            "resource":{                        # <-- The resource in the aggregate the was acted upon
               "id":"ROOT_OR_NON_ROOT_RESOURCE",
               "type":"resource_reference"
            }
         },
         "root_resource":{                      # <-- Aggregate root resource
            "id":"ROOTRESOURCE",
            "type":"resource_reference"
         }
      }
   ]
}
```
- The details of the action are captured in the `details` object. Changes can be made to the root resource or any other resource in the aggregate.
The `details` object provides the `resource` reference that identifies the resource that changes were made to.
If the change was made to the root resource itself, the `details.resource` reference will be the same as the `root_resource` reference.

<!-- theme:info -->
> Actions can be split across multiple records, but records produced by a single API request will share the `execution_context.request_id`

### Change Details
The record `details` object describes the changes made to the resource. It may describe changes made to the resource's `fields`, `references` or both.

**Example: Setting the `urgency` of a notification rule to `high`**
```json
{
   "records":[
      {
         <...>
         "details":{
            "fields":[
               {
                  "name":"urgency",
                  "value":"high"
               }
            ],
            "resource":{
               "id":"P4SBFXI",
               "type":"notification_rule_reference"
            }
         },
         <...>
      }
   ]
}
```

**Example: Adding a user to a team's `members` collection**
```json
{
   "records":[
      {
         <...>
         "details":{
            "references":[
               {
                  "added":[
                     {
                        "id":"PKWWAM2",
                        "type":"user_reference"
                     }
                  ],
                  "name":"members"
               }
            ],
            "resource":{
               "id":"PGVLPJ5",
               "type":"team_reference"
            }
         },
         <...>
      }
   ]
}
```

<!-- theme:info -->
> A `details` object might have multiple objects in the `fields` and `references` array. All of them apply to the resource identified by the `resource` reference.

## Data Retention

Audit records are accessible for up to 12 months after they are created. You may fetch and store records if you wish to access them after 12 months.


<!-- theme:warning -->
> Note that PagerDuty began producing audit records on December 2, 2020, so no audit records exist before this time. See the [Resources Tracked](#resources-tracked) table to see when the first audit records were produced for a given resource type.

