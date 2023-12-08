---
tags: [rest-api]
---

# REST API Rate Limits

<!-- theme: info -->
> This page describes the rate limits for the PagerDuty REST API. The rate limits for the 
> PagerDuty Events API are described [here](../../docs/events-API-v2/05-Rate-Limits.md).

## What are the limits?

The PagerDuty REST API uses rate limits to provide a consistent experience for all of our customers. The limits depend on the type of authorization token being used. Each PagerDuty App has its own set of limits that are separate from both API keys and other apps.

### Rate limits
The limits described in this section apply to every request made to the PagerDuty REST API.

**Account API Keys**

Each Account API Key is allowed 960 requests per minute.

**User API Keys**

User API Keys act as a PagerDuty User and are limited as such. Each user is allowed 960 requests per minute across all of the user's API keys.

**PagerDuty App - App OAuth Tokens**

When a PagerDuty App [uses an app token](../../docs/app-integration-development/08-OAuth-Functionality.md) it is acting as the PagerDuty App. Each PagerDuty App is allowed 960 requests per minute against each PagerDuty Account it is authorized to access.

**PagerDuty App - User OAuth Tokens**

When a PagerDuty App [uses a user token](../../docs/app-integration-development/08-OAuth-Functionality.md) it is acting as the PagerDuty User. Each PagerDuty App is allowed 960 requests per minute per User it is authorized to act as.

### Operation specific limits
In a few exceptional circumstances, individual REST API operations may apply additional limits. If an endpoint applies additional limits; it will be noted in the [REST API](/api-reference/) documentation for that endpoint. The `ratelimit-` response headers described later in this document will provide information to the client about the current status of the limit and when it resets.

## Working with limits
How can your application gracefully interact with these limits? This section describes how our API informs client applications about limiting and covers some best practices.

### Rate limit headers
Requests that have been evaluated for rate limiting will include the following HTTP response headers:

|Header Name|Description|
|-|-|
|`ratelimit-limit`|The allowed number of requests for the current time window.|
|`ratelimit-remaining`|The remaining number requests in the current time window before limiting occurs.|
|`ratelimit-reset`|The number of seconds until the limit is reset.|

Note that when working with API endpoints that have operation specific limits; these headers will represent the limit that is closest to being reached.

### Reaching the limit
In the event that your application reaches a limit: the HTTP response code will be `429`, the rate limit headers will be present, and the response body will indicate that limiting has occurred:

```bash
HTTP/2 429 Too Many Requests

date: Tue, 03 Oct 2023 16:46:39 GMT
ratelimit-limit: 960
ratelimit-remaining: 0
ratelimit-reset: 20

{
  "error": {
    "message": "Rate Limit Exceeded",
    "code": 2020
  }
}
```

Note that there may be exceptional scenarios where an application receives a `429` response code _without_ the rate limit headers or the response body above.

### Best practices

#### Ensure your application is ready to handle being rate limited
Update your application to slow down the rate of requests after receiving a 429 response code. For fine-grained control your application can make use of the `ratelimit-reset` header to wait the appropriate number of seconds before retrying.

#### Avoid hitting rate limits
These guidelines will help your application avoid hitting rate limits:
* Make requests serially for a given authentication token.
* Avoid excessive polling for changes whenever possible. Make use of available [Webhooks](../../docs/webhooks/01-Overview.md) to receive push updates to data.

#### One "bot user" per application deployment
If you use PagerDuty Users / User API Keys as "bot users" for applications; create a separate bot user for each deployment of the application (e.g. "Acme Production Bot User", "Acme Test Bot User", etc.). This will keep the limits for each deployment of the application separate.
