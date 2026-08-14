---
tags: [app-integration-development]
---

# Private Apps

## What is a private app?

A private app is a PagerDuty App that is only used on the account that created it. You register it on your own account, and it works there and nowhere else.

Private apps are not reviewed or published by PagerDuty. There is no distribution form to complete and no listing for other customers to discover — you build the app, add functionality to it, and start using it.

An app is private by default: its **App Type** is Private when you register it, and stays that way unless you change it and submit it for [review](08-Publish-Your-App.md). For REST API access, **Scoped OAuth** can only be granted on the account that registered the app, so a Scoped OAuth app is always private — the option to change its App Type is disabled.

<!-- theme:info -->
> ### Classic User OAuth does not respect the App Type
> [Classic User OAuth](06-OAuth-Functionality.md#classic-user-oauth), the legacy OAuth functionality, ignores the Private App Type entirely. As soon as you register a Classic User OAuth client, a user on **any** PagerDuty account can authorize it and the app can act on their behalf — whether or not the app is marked Private.
>
> If you want OAuth functionality on an app that stays on your own account, use [Scoped OAuth](#scoped-oauth). If you want an app that works across accounts, Classic User OAuth is how you build it.

A private app is not limited to REST API access: it can also have [Events Integration functionality](05-Events-Integration.md), which is a common way to reuse a single [Event Transformer](05-Events-Integration.md#add-an-event-transformer) across all of your services. An app with Events Integration functionality and no OAuth functionality can go either way — it stays private unless you submit it for [publication](08-Publish-Your-App.md) and it is approved.

## Why build a private app?

**Scoped access to the REST API for internal tooling.** Scoped OAuth grants access per resource type rather than handing over everything the authorizing user can reach. A script that closes stale incidents needs `incidents.write` and nothing else. See [Scoped OAuth](#scoped-oauth) below.

**Acting as the app rather than as a person.** A private app can obtain an app token with no user involved, which suits server-to-server work like responding to a webhook or running a scheduled job. It removes the need to maintain a service or bot user account just to hold credentials, and the integration keeps working when people leave the team.

**Writing one Event Transformer and reusing it everywhere.** This is a common reason to register a private app with [Events Integration functionality](05-Events-Integration.md). If some system sends you a payload that PagerDuty does not understand natively, you can write a single [Event Transformer](05-Events-Integration.md#add-an-event-transformer) on the app that converts that payload into the Events API v2 format — and then add that one integration to every PagerDuty service that receives the payload. The transform logic lives in one place instead of being duplicated per service, and editing it on the app updates the behavior for all of them.

## Scoped OAuth

Scoped OAuth is the REST API functionality behind a private app. Two things distinguish it from Classic User OAuth.

**Its scopes apply to individual resource types.** Rather than a single `read` or `write` scope covering everything the authorizing user can reach, Scoped OAuth grants access per resource type: `incidents.read` reads incidents and nothing else, while `incidents.write` can create, update, or delete an incident without being able to read existing ones. A different resource type is a separate grant entirely — an app that also needs to read services must be given `services.read`. This lets you grant an app only the access it actually needs.

**It can act as the app itself, not just as a user.** A Scoped OAuth app can obtain an app token through the OAuth 2.0 client credentials flow, with no user involved.

An app with Scoped OAuth can obtain an app token, a user token, or both. The access a token has depends on its type:

* With an **app token**, access is limited only by the scopes granted to the app. If the app has the `incidents.read` scope, it can read all incidents on the account.
* With a **user token**, access is the intersection of the app's scopes and the user's permissions. If an app has the `incidents.read` scope and is acting as user Pagey, it can only read incidents that Pagey has permission to see.

### Client requirements

<!-- theme:warning -->
> ### Scoped OAuth apps must secure a `client_secret`
> Scoped OAuth only supports confidential clients — apps that run somewhere you control end to end, such as a server-side web app or a backend service. A single page app running in the browser or a native mobile app cannot protect a `client_secret` and cannot use Scoped OAuth. Use [Classic User OAuth](06-OAuth-Functionality.md#classic-user-oauth) for those.

Two rules follow from this, and both apply to every Scoped OAuth app:

* The `client_secret` must be sent on the token request.
* **PKCE is required** when obtaining a user token. Unlike Classic User OAuth, where PKCE is recommended but optional for confidential clients, a Scoped OAuth app must always use it.

To obtain a **user token**, follow [Obtaining a User OAuth Token](06-OAuth-Functionality.md#obtaining-a-user-oauth-token) — the flow is the same one Classic User OAuth uses, with the requirements above. To obtain an **app token**, see below.

## Obtaining an App OAuth Token

To act on a PagerDuty Account as a PagerDuty App, you must obtain an app token for that account. Applications are implicitly granted access to the account that created them.

Before proceeding you should [register a PagerDuty App](04-Register-an-App.md) with Scoped OAuth functionality to obtain the `client_id`, `client_secret`, and scopes.

An app token is obtained by making an OAuth 2.0 client credentials request. Send a `POST` request to `https://identity.pagerduty.com/oauth/token` with a `Content-Type` of `application/x-www-form-urlencoded` and the following form parameters:

|Parameter|Description|
|-|-|
|`grant_type`|The OAuth 2.0 grant type. Value must be set to `client_credentials`|
|`client_id`|An identifier issued when the Scoped OAuth client was added to a PagerDuty App|
|`client_secret`|A secret issued when the Scoped OAuth client was added to a PagerDuty App|
|`scope`|A space separated list of scopes available to the client. Must contain the `as_account-` scope that specifies the PagerDuty Account the token is being requested for using a `{REGION}.{SUBDOMAIN}` format. Currently accepted service regions are `us` or `eu`.|

The following curl request could be used to obtain an access token for the `companysubdomain` account in the `us` service region:
```bash
curl -i --request POST \
  https://identity.pagerduty.com/oauth/token \
  --header "Content-Type: application/x-www-form-urlencoded" \
  --data-urlencode "grant_type=client_credentials" \
  --data-urlencode "client_id={CLIENT_ID}" \
  --data-urlencode "client_secret={CLIENT_SECRET}" \
  --data-urlencode "scope=as_account-us.companysubdomain incidents.read services.read"
```

The JSON response includes the access token, the scopes that were actually issued to the token, and the number of seconds before the token expires.

```json
{
  "access_token": "pdus+_0XBPWQQ_dfd3c718-4a46-400d-a8ec-45bab1fd417e",
  "scope": "as_account-us.companysubdomain incidents.read services.read",
  "token_type": "bearer",
  "expires_in": 86400
}
```

In the client credentials OAuth flow, it is not necessary to refresh an access token. When a particular access token expires, the application should simply request a new token.

### Using an Access Token

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

## Token Lifetimes

All Scoped OAuth clients have the following settings:

 - access token expiry of 1 day
 - refresh token expiry of 30 days
 - rolling refresh window of 1 year

Refresh tokens only apply to user tokens. When an app token expires, the application can simply request a new token.

Scoped OAuth applications that implement the refresh token flow for user tokens will allow your customers to use your application continuously for one year, as long as they use it at least once every 30 days.

See [Token Lifetimes](06-OAuth-Functionality.md#token-lifetimes) for the Classic User OAuth expiries, and [Revoking Tokens](06-OAuth-Functionality.md#revoking-tokens) if you believe your app's tokens or `client_secret` have been compromised.
