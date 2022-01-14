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
|Authorization Endpoint|`https://app.pagerduty.com/oauth/authorize`|
|Token Endpoint        |`https://app.pagerduty.com/oauth/token`|


The following parameters will also be used in your requests or returned in the response:

|Parameter|Description|Required for token request|Required for authorization request|
|-|-|:-:|:-:|
|`client_id`|An identifier issued when the app is created|✓|✓|
|`client_secret`|A secret issued when the app is created.|✓||
|`code`|The authorization code issued upon a successful authorization request.|✓||
|`grant_type`|The OAuth 2.0 grant type. Value must be set to `authorization_code`|✓||
|`redirect_uri`|Registered with the app when OAuth 2.0 is added. PagerDuty will redirect here after a user grants or denies access to your app.|✓|✓|
|`response_type`|Specifies the response type based on OAuth 2.0 flow. Value must be set to `code`.||✓|
|`scope`|Specifies the scope being requested, must match what is configured on the OAuth application.||✓|
|`subdomain`|The subdomain of the user authorizing the app.|||

## Obtaining an Access Token

Send a GET request to authorization endpoint with query parameters set for `client_id`, `redirect_uri` and `scope`, as defined in the app, and `response_type=code`

```
GET https://app.pagerduty.com/oauth/authorize?client_id={CLIENT_ID}&redirect_uri={REDIRECT_URI}&scope={SCOPE}&response_type=code
```

### Authorized Requests

If the user authorizes the app, PagerDuty will redirect to the specified URI with the `code` (authorization code) in the URL:
```
{REDIRECT_URI}?code={AUTHORIZATION_CODE}&subdomain={ACCOUNT_SUBDOMAIN}
```

### Denied Requests

If the user denies authorization, PagerDuty will redirect to the specified URI with `error` and `error_description` parameters:

```
{REDIRECT_URI}?error=access_denied&error_description=The+resource+owner+or+authorization+server+denied+the+request.
```

## Exchanging Authorization Code

To exchange the authorization code for an access token, send a POST request to the token endpoint. The authorization code has a time to live of 30 seconds, and your POST request must be received within that time. The body of the request should include the following parameters: `client_id`, `client_secret`, `redirect_uri`, the `code` (authorization code) received from PagerDuty, and `grant_type=authorization_code`. The content type should be `application/x-form-urlencoded`.

```
curl -X POST https://app.pagerduty.com/oauth/token \
  -d "grant_type=authorization_code" \
  -d "client_id={CLIENT_ID}" \
  -d "client_secret={CLIENT_SECRET}" \
  -d "redirect_uri={REDIRECT_URI}" \
  -d "code={CODE}"
```

Note: By default, curl uses a content type of `application/x-form-urlencoded`. 

The access token will be included in a JSON response:

```json
{
  "access_token":"9937611c354d287d3ff509afdde5b1d6d500c73a67387d666ca1e8e3d502d516",
  "token_type":"bearer",
  "scope":"user"
}
```

## Sample Code

|Languauge|GitHub Repository|
|-|-|
|Javascript / Node.js|[pagerduty-oauth-sample-node](https://github.com/PagerDuty/pagerduty-oauth-sample-node)|
|Python 3        |[pagerduty-oauth-sample-python](https://github.com/PagerDuty/pagerduty-oauth-sample-python)|

## Using an Access Token

Once obtained, access tokens can be used to make [REST API](https://api-reference.pagerduty.com/#!/API_Reference/get_api_reference) requests on behalf of the user.

When making an API request, include the version of the API in the `Accept` header. Access tokens must also be sent in the request as part of the `Authorization` header along with the `Bearer` token type, using this format:

```http
Authorization: Bearer 9937611c354d287d3ff509afdde5b1d6d500c73a67387d666ca1e8e3d502d516
Accept: application/vnd.pagerduty+json;version=2
```

The token can be used continuously to make requests until the user or app owner revokes it.
