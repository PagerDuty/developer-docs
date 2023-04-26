---
tags: [app-integration-development]
---

# OAuth 2.0: Authorization Code Grant Flow

<!-- theme:warning -->
> ### This flow is for server-side apps
> A client_secret should be treated as a password or private_key and not be stored in public code.
> If you're working on an a mobile app or in-browser app, please use the [PKCE flow](../../docs/app-integration-development/10-OAuth-2-PKCE.md).

[Create an app](../../docs/app-integration-development/03-Register-an-App.md) to get access to OAuth 2.0 credentials.

PagerDuty supports OAuth 2.0’s [Authorization Code Grant](https://tools.ietf.org/html/rfc6749#section-4.1) flow for third-party applications to obtain access tokens from PagerDuty and utilizes the following endpoints:

|||
|-|-|
|Authorization Endpoint|`https://identity.pagerduty.com/oauth/authorize`|
|Token Endpoint        |`https://identity.pagerduty.com/oauth/token`|


The following parameters will also be used in your requests or returned in the response:

|Parameter|Description|Required for token request|Required for authorization request|Required for refresh|
|-|-|:-:|:-:|:-:|
|`client_id`|An identifier issued when the app is created|✓|✓|✓|
|`client_secret`|A secret issued when the app is created.|✓| |✓|
|`code`|The authorization code issued upon a successful authorization request.|✓| | |
|`grant_type`|The OAuth 2.0 grant type. Value must be set to `authorization_code` or `refresh_token`|✓| |✓|
|`redirect_uri`|Registered with the app when OAuth 2.0 is added. PagerDuty will redirect here after a user grants or denies access to your app.|✓|✓| |
|`response_type`|Specifies the response type based on OAuth 2.0 flow. Value must be set to `code`.| |✓| |
|`scope`|Specifies the scope being requested, must match what is configured on the OAuth application. Should be either `read` or `write`.| |✓| |
|`access_token`|The token you will use to access the API after successful authorization.| | | |
|`refresh_token`|The token you will use to get a new access token after the current access token has expired.| | |✓|

## Obtaining an Access Token

Send a GET request to authorization endpoint with query parameters set for `client_id`, `redirect_uri` and `scope`, as defined in the app, and `response_type=code`

```
GET https://identity.pagerduty.com/oauth/authorize?client_id={CLIENT_ID}&redirect_uri={REDIRECT_URI}&scope={SCOPE}&response_type=code
```

### Authorized Requests

If the user authorizes the app, PagerDuty will redirect to the specified URI with the `code` (authorization code) in the URL:
```
{REDIRECT_URI}?code={AUTHORIZATION_CODE}
```

### Denied Requests

If the user denies authorization, PagerDuty will redirect to the specified URI with `error` and `error_description` parameters:

```
{REDIRECT_URI}?error=access_denied&error_description=The+resource+owner+or+authorization+server+denied+the+request.
```

## Exchanging an Authorization Code

To exchange the authorization code for an access token, send a POST request to the token endpoint. The authorization code has a time to live of 30 seconds, and your POST request must be received within that time. The body of the request should include the following parameters: `client_id`, `client_secret`, `redirect_uri`, the `code` (authorization code) received from PagerDuty, and `grant_type=authorization_code`. The content type should be `application/x-form-urlencoded`.

```
curl -X POST https://identity.pagerduty.com/oauth/token \
  -d "grant_type=authorization_code" \
  -d "client_id={CLIENT_ID}" \
  -d "client_secret={CLIENT_SECRET}" \
  -d "redirect_uri={REDIRECT_URI}" \
  -d "code={CODE}"
```

Note: By default, curl uses a content type of `application/x-form-urlencoded`.

The access token will be included in a JSON response. You may also want to take note of the id token and the refresh token:

```json
{
  "client_info":"prefix_legacy_app",
  "id_token":"eyJraWQiOiIxNzg3MzQ1MDA4IiwieDV0IjoiX2Nxbk1aWlBBcEF0V3kyVm11T1Y4dUc5VHNvIiwiYWxnIjoiUlMyNTYifQ.eyJleHAiOjE2NjYxOTk2MDEsIm5iZiI6MTY2NjE5NjAwMSwianRpIjoiODM3ODE1YzAtZTVmMi00M2RhLWFiYWYtYTE0ZjdhYzQ3ODYxIiwiaXNzIjoiaHR0cHM6Ly9hcHAucGFnZXJkdXR5LmNvbS9nbG9iYWwvb2F1dGgvYW5vbnltb3VzIiwiYXVkIjpbImh0dHBzOi8vYXBpLnBhZ2VyZHV0eS5jb20iLCI4ZTkxNDZmZS02NjllLTRjNjctYmIzOC1kODJhODg5YjM2ZWYiXSwic3ViIjoiUElZS0RCTiIsImF1dGhfdGltZSI6MTY2NjE5NTk5NSwiaWF0IjoxNjY2MTk2MDAxLCJwdXJwb3NlIjoiaWQiLCJhdF9oYXNoIjoiNkVqM3dzQUpDa2RPLTVtOFNuU29oUSIsImFjciI6ImFjcjpodG1sLWZvcm06dW5pdGVkc3RhdGVzIiwiZGVsZWdhdGlvbl9pZCI6ImM5YzliYWU1LWVkNzktNDg2Ny04NDQ1LWRmY2FkYmMwMzdiNSIsInByb2R1Y3RfYWJpbGl0aWVzIjpbXSwiYWNjb3VudF9pZCI6IlBFNlJMUTQiLCJ1c2VyX2lkIjoiUElZS0RCTiIsImF6cCI6IjhlOTE0NmZlLTY2OWUtNGM2Ny1iYjM4LWQ4MmE4ODliMzZlZiIsImFtciI6ImFjcjpodG1sLWZvcm06dW5pdGVkc3RhdGVzIiwic3ViZG9tYWluIjoicGR0LWhhbm5lbGUiLCJyZWdpb24iOiJVbml0ZWRTdGF0ZXMiLCJzaWQiOiJLRHo5V2h0bndqWXFKRnEzIn0.qBr2vJG-BkO0zAovDjxSkaxrenqzZC5Mcpy8Li-J37hae44j68PeIEJxMaknNZ3tMOyVjsd8AknjBoW2OeOv6Zk3RQMJd2inXDR9lIkEEMMgZ6PHI_tv3sM-9O4NR9OS4iCUtFMXjv6Sc-Dq_snjaTBw6ZK7vSERYwn57xe99z9JsaDzuLRX3mYhxApEUphr8GSty3TfI-fH_WIbuQhDOa6z8nExcKQWpNX18OEhig9AY2B88P21oBtYR3CnfqcRVH5nIXjAlGvCo6bcPM8MSVAmxY0spDFRNqaKNnPx4WMW_PyU7UxdMEZsO1fDOkTkHaS15FyRCoz0qhk5E3cYkg",
  "token_type":"bearer",
  "access_token":"pdus+_0XBPWQQ_b2b2060b-e7af-44a1-8ddf-9c56fedd8d4f",
  "refresh_token":"pdus+_1XBPWQQ_f85dd2f1-b906-478a-a9e6-678952529e4e",
  "scope":"openid write",
  "expires_in":864000
}
```

Note however, that our access tokens do expire after a defined period of time -- so you may want to make sure that you implement OAuth refresh to prevent users needing to re-authorize your app. See more information about token expiries at the bottom of this page.

For additional information about the user, account, and PagerDuty service region where your app is now authorized, you can look at [cracking open our PagerDuty-signed ID token](../../docs/app-integration-development/11-OAuth-2-Id-token.md). For example, the `aud` field will help your integration with data residency and processing guarantees if you have customers located in Europe.

## Sample Code

|Languauge|GitHub Repository|
|-|-|
|Javascript / Node.js|[pagerduty-oauth-sample-node](https://github.com/PagerDuty/pagerduty-oauth-sample-node)|
|Python 3        |[pagerduty-oauth-sample-python](https://github.com/PagerDuty/pagerduty-oauth-sample-python)|

## Using an Access Token

Once obtained, access tokens can be used to make [REST API](https://api-reference.pagerduty.com/#!/API_Reference/get_api_reference) requests on behalf of the user.

When making an API request, include the version of the API in the `Accept` header. Access tokens must also be sent in the request as part of the `Authorization` header along with the `Bearer` token type, using this format:

```http
Authorization: Bearer pdus+_0XBPWQQ_b2b2060b-e7af-44a1-8ddf-9c56fedd8d4f
Accept: application/vnd.pagerduty+json;version=2
```

### API Call results in 403

This means that although the OAuth credentials are valid, the token does not have access to that particular resource. For example, if you have a token with the read scope and try to write to a resource, it will result in 403.

If you think you requested the correct scope and should have access to the resource, double check the `scope` field in the POST Token Endpoint response. If an invalid scope is requested, we currently do not return an error. Instead, we grant partial scopes which will only be the `openid` scope automatically attached to all tokens.

Valid Scopes:
- `Read` access should request the `read` scope (case sensitive)
- `Read/Write` should request the `write` scope (case sensitive)

## Getting a new Access Token with a Refresh Token

As mentioned, all of our current access tokens have an expiration date defined, so, it would be to your benefit to implement OAuth refresh to prevent your users from logging in unnecessarily.

Exchanging the refresh token for the access token is similar to using an authorization code: send a POST request to the token endpoint, but using the `refresh_token` grant type instead.

The body of the request should include the following parameters: `client_id`, `client_secret`, the `refresh_token` previously received from PagerDuty, and `grant_type=refresh_token`. The content type should still be `application/x-form-urlencoded`.

```
curl -X POST https://identity.pagerduty.com/oauth/token \
  -d "grant_type=refresh_token" \
  -d "client_id={CLIENT_ID}" \
  -d "client_secret={CLIENT_SECRET}" \
  -d "refresh_token=pdus+_1XBPWQQ_f85dd2f1-b906-478a-a9e6-678952529e4e"
```

A successful response will include a new access token and a new refresh token:

```json
{
    "token_type": "bearer",
    "access_token": "pdus+_0XBPWQQ_39435d07-9232-4bc2-9dc9-c4a8fccc94ad",
    "refresh_token": "pdus+_1XBPWQQ_4f0dd763-8665-4fd7-aab7-d9f98e0c151b",
    "scope": "openid write",
    "expires_in": 864000
}
```

## Token Expiries

We've noted above that access tokens have an expiry date -- so do refresh tokens, and so does the lifetime of the user's authorization.

All OAuth clients registered prior to October 30th 2022 will have the following settings:

 - access token expiry of one year
 - refresh token expiry of one year

After October 30th 2022, all newly registered OAuth clients will have the following settings:

 - access token expiry of 30 days
 - refresh token expiry of 210 days
 - rolling refresh window of 3 years

After April 30th 2023, we will apply the new expiry settings to all OAuth clients.

Once you have implemented the refresh token flow, this will allow your customers to use your OAuth app continuously for three years, as long as
they use it at least once every 30 days.
