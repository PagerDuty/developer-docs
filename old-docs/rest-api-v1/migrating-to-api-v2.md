---
title: "Migrate to REST API v2"
excerpt: "Migrating your existing REST API v1 client to API v2"
---
Migrating your existing PagerDuty [REST API v1](https://v1.developer.pagerduty.com/) tools and integrations to REST API v2 will let you leverage greatly increased consistency to simplify your code. It will also allow your integration to take advantage of new v2-only features and functionality coming to the API in the future.

A number of things have changed between REST API v1 and REST API v2. While this document covers the majority of the high-level, API-wide changes you need to be aware of, changes in specific [endpoints](#section-endpoints) are not covered comprehensively. You should consult the [API Reference](page:api-reference) for complete details of how REST API v2 endpoints behave.

Patterns followed in REST API v2 were typically present in REST API v1, but may have been followed inconsistently across different endpoints. Review your code to determine where a pattern is already being followed and where it needs to be updated.
[block:callout]
{
  "type": "info",
  "title": "REST API Only",
  "body": "This guide applies only to requests made against the [REST API](doc:rest-api). For information on Events API v2, see [Events API v2 Overview](/docs/events-api-v2/overview/)."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Versioned Accept Header"
}
[/block]
To make your code talk with REST API v2 instead of v1, you'll need to add the [versioned PagerDuty `Accept` header](doc:versioning#section-header-based-versioning) to any API requests being made by your code:

```
Accept: application/vnd.pagerduty+json;version=2
```

How you do this depends on the library you use to make HTTP requests, but is simple and straightforward in most libraries.
[block:api-header]
{
  "type": "basic",
  "title": "Base URL"
}
[/block]
Base paths have changed in REST API v2 from `https://<account-subdomain>.pagerduty.com/api/v1/` to simply `https://api.pagerduty.com/`. There's no more need to configure and store the account subdomain in order to make an API request — all you need is the authentication token.
[block:api-header]
{
  "type": "basic",
  "title": "HTTPS"
}
[/block]
In REST API v1, any requests made over HTTP (without SSL/TLS) would be redirected to the equivalent HTTPS endpoint.

To prevent accidentally sending sensitive information over the Internet, **REST API v2 requests require HTTPS**. Connections made over HTTP will be refused.
[block:api-header]
{
  "type": "basic",
  "title": "Authentication"
}
[/block]
The format of the API Token `Authorization` header in v1 remains unchanged. Authentication is still required to access any REST API endpoint.

See [Authentication](doc:authentication) for the full details on authenticating to the REST API.
[block:api-header]
{
  "type": "basic",
  "title": "Rate Limiting"
}
[/block]
Rate limited requests will now return a [`429 Too many requests`](http://tools.ietf.org/html/rfc6585#section-4) error code instead of [`403 Forbidden`](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.4). This allows your client to distinguish between requests that fail due to insufficient permissions and requests that fail because your client is making requests too frequently.

With this change, a client *should not* retry a request that returns a `403 Forbidden` response code. These requests will only succeed if the client acquires a different key with increased permission.

See [Rate Limiting](doc:rate-limiting) for more details on API rate limiting.
[block:api-header]
{
  "type": "basic",
  "title": "Pagination"
}
[/block]
Instead of the `total` resource count field being returned by default, a new boolean field `more` will indicate whether there are more resources to be requested. You can still request the total count by passing in the parameter `total=true` as part of the query string.
[block:api-header]
{
  "type": "basic",
  "title": "Filtering"
}
[/block]
For endpoints that allow filtering by resource `id`s, the parameter will only accept an array of `id`s and the name will be the singular form of the resource to filter on followed by `_ids`.

Previously, filters may have been specified as a comma-separated list, e.g.
```
/api/v1/incidents?service=PAGRDTY,PGRDUTY
```
In REST API v2, the parameter will instead look like
```
/incidents?service_ids[]=PAGRDTY&service_ids[]=PGRDUTY
```
[block:api-header]
{
  "type": "basic",
  "title": "Includes"
}
[/block]
All [parameters passed to the `include` array](doc:includes)  in the query string are the pluralized version of the resources to be included. Resources that are not `include`d will be presented as [resource references](doc:references).

For instance, where you may have previously used `?include[]=channel`, you should now use `?include[]=channels`.
[block:api-header]
{
  "type": "basic",
  "title": "Deleted References"
}
[/block]
Nested resources that have been deleted will still be returned as part of the object, but be displayed as [resource references](doc:references), even when [included](doc:includes). These references will have a `null` value for both `self` and `html_url` as they are not accessible directly in either the web app or the API.
[block:api-header]
{
  "type": "basic",
  "title": "Wrapped Entities"
}
[/block]
All request and response bodies contain a root-level key with the singular or plural name of the resource. In REST API v1, some endpoints would accept the resource's fields without this wrapping key; in v2, these requests are invalid.
[block:code]
{
  "codes": [
    {
      "code": "{\n  \"escalation_policy\": {\n    \"name\": \"Operations\",\n    …\n  }\n}",
      "language": "json",
      "name": "Valid wrapped request in v2"
    },
    {
      "code": "{\n  \"name\": \"Operations\",\n  …\n}",
      "language": "json",
      "name": "Unwrapped request in v1 (invalid in v2)"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Resource References"
}
[/block]
In REST API v1, relationships between resources were often specified with a `<resource>_id` field, such as `team_id` or `escalation_policy_id`. In REST API v2, all relationships are represented with [resource references](doc:references), which are objects that consist of an `id` field and `type` indicating the type of the object. Since every resource is also a valid resource reference, this allows you to compose request bodies by assembling entire resources together, rather than extracting the `id` field from a given resource.
[block:code]
{
  "codes": [
    {
      "code": "{\n  \"service\": {\n    …\n    \"escalation_policy\": {\n      \"id\": \"PASDF12\",\n      \"type\": \"escalation_policy\"\n    },\n  \t…\n  }\n}",
      "language": "json",
      "name": "Valid resource reference in v2"
    },
    {
      "code": "{\n  \"service\": {\n    …\n    \"escalation_policy_id\": \"PASDF12\",\n  \t…\n  }\n}",
      "language": "json",
      "name": "_id field in v1 (invalid in v2)"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Requester"
}
[/block]
In REST API v1, several endpoints provided a `requester_id` parameter, used to indicate what user the API client was acting on behalf of. This enabled an API client to acknowledge an incident on behalf of a user, invite a user on another user's behalf, etc.

In REST API v2, this functionality is available using the [HTTP `From:` header](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.22). Set this header with the email address of the user to act on behalf of.

This "requester" functionality has also been limited to endpoints where it provides significant value. Consult the [REST API Reference](doc:api-reference) to see if a particular endpoint accepts a `From:` header.
[block:api-header]
{
  "type": "basic",
  "title": "Endpoints"
}
[/block]
There are a handful of larger changes to endpoints that you should be aware of when migrating.

- The new `/oncalls` endpoint consolidates the functionality previously provided by several different endpoints, which are no longer available:
  - `/users/on_call`
  - `/users/:id/on_call`
  - `/escalation_policies/on_call`
  - `/schedules/:id/entries`
- The `/alerts` endpoint has been changed to `/notifications`, and the resources within are referred to as "notifications". This helps distinguish between *notifications*, which are messages that PagerDuty sends to on-call users via SMS, push notification, phone call, and email, and *alerts*, which are messages generated by monitoring tools.
- The `/users/:id/contact_methods` and `/users/:id/notification_rules` endpoints are no longer distinct endpoints. Read and manipulate a user's contact methods and notification rules by fetching and modifying `User` resources directly.
- `/services/:id/disable` and `/services/:id/enable` have been removed. Instead, use `PUT /services/:id` to update the service's `status` to `disabled` or `active`.
