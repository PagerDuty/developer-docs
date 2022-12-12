---
tags: [app-integration-development]
---

# Scoped OAuth

<!-- theme: warning -->
> ### Early Access
>
> The features described on this page are in an Early Access state and are subject to change. Your PagerDuty Account may
> require a feature flag before this functionality is available to you. Please reach out to us if you have any questions or
> need support.

## Register an App
Scoped OAuth clients allow your application to act on a PagerDuty Account as a PagerDuty App. Your application's access to the PagerDuty Account is controlled by the scopes it is granted. Before you start building, you first need to [register a PagerDuty App](03-Register-an-App.md), then [add Scoped OAuth functionality](04-App-Functionality.md). This is done via the Developer Mode UI in your PagerDuty Account.

The `client_id`, `client_secret`, and all selected scopes will be used to obtain an access token.

## Obtaining an Access Token

A scoped access token is obtained by making a client credentials request to the token endpoint.

|Parameter|Description|
|-|-|
|`grant_type`|The OAuth 2.0 grant type. Value must be set to `client_credentials`|
|`client_id`|An identifier issued when the client was added to a PagerDuty App|
|`client_secret`|A secret issued when the client was added to a PagerDuty App|
|`scope`|A space separated list of scopes available to the client. Must contain the `as_account-` scope that specifies the PagerDuty Account the token is being requested for using a `{REGION}.{SUBDOMAIN}` format. Currently accepted region: `us`|


```bash
curl -i --request POST \
  https://identity.pagerduty.com/oauth/token \
  --header "Content-Type: application/x-www-form-urlencoded" \
  --data-urlencode "grant_type=client_credentials" \
  --data-urlencode "client_id={CLIENT_ID}" \
  --data-urlencode "client_secret={CLIENT_SECRET}" \
  --data-urlencode "scope=as_account-us.companysubdomain incidents.read services.read"
```

The access token will be included in a JSON response along with the scopes that were actually issued to the token.

```json
{
  "access_token": "pdus+_0XBPWQQ_dfd3c718-4a46-400d-a8ec-45bab1fd417e",
  "scope": "as_account-us.companysubdomain incidents.read services.read",
  "token_type": "bearer",
  "expires_in": 2592000
}
```

The token is valid for the number of seconds specified `expires_in` in the response.

## Using an Access Token

The access token can be used to access the [REST API](https://developer.pagerduty.com/api-reference/) as a PagerDuty App.

When making an API request, include the version of the API in the `Accept` header. Access tokens must also be sent in the request as part of the `Authorization` header along with the `Bearer` token type, using this format:

```http
Authorization: Bearer pdus+_0XBPWQQ_dfd3c718-4a46-400d-a8ec-45bab1fd417e
Accept: application/vnd.pagerduty+json;version=2
```

A `403 - Forbidden` response will be returned if the token does not contain the scope required to access a particular API endpoint
or the API endpoint does not yet support API Scopes. When the token expires a `401 - Unauthorized` response will be returned
and a new token must be obtained.

## Which APIs support Scoped OAuth?

Each endpoint that supports Scoped OAuth indicates the required scope in its description in our [API Reference](https://developer.pagerduty.com/api-reference/). For example, the list incidents endpoint requires the `incidents.read` scope.
![List incidents required scope](../../assets/images/list-incidents-required-scope.png)
