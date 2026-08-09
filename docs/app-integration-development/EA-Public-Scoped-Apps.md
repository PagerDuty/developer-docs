---
tags: [app-integration-development]
---

# Public Scoped Apps

<!-- theme: warning -->

> ### Early Access
>
> The features described on this page are in an Early Access state and are subject to change. Please reach out to
> us if you have any questions or need support.

An app that uses Scoped OAuth is normally a [private app](02-Private-Apps.md): it only works on the account that
created it. Public Scoped Apps extend Scoped OAuth so that a single app can be used across many PagerDuty accounts,
as [Classic User OAuth](06-OAuth-Functionality.md#classic-user-oauth) apps can be today.

## User tokens only

A Public Scoped App can only obtain **user tokens** for other accounts, by taking a user on that account through the
[user token flow](06-OAuth-Functionality.md#obtaining-a-user-oauth-token) to get their authorization and consent.

The [client credentials flow](02-Private-Apps.md#obtaining-an-app-oauth-token), which issues an **app token** with no
user involved, is not available across accounts. An app token can still only be issued for the account that created
the app.

So when your app acts on another account, its access is the intersection of the scopes granted to the app and the
permissions of the authorizing user, as described for user tokens under [Scoped OAuth](02-Private-Apps.md#scoped-oauth).
Any part of your app that relies on app-token access will only work on your own account.

The requirements of the user token flow are unchanged: a Scoped OAuth app is always a
[confidential client](06-OAuth-Functionality.md#confidential-vs-non-confidential-clients-and-pkce), so it must send
its `client_secret` on the token request and must use PKCE. Being usable by many accounts does not make it safe to
distribute the `client_secret` — a Public Scoped App still needs a server-side component that keeps the secret secure.

## Installation by an account admin

Before any user on another account can authorize your app, an admin on that account must install it. Until the app is
installed, authorization requests from users on that account will not succeed.

To have your app installed on an account, give an admin on that account the following installation link, replacing
`[app_id]` with the ID of your app:

```
https://app.pagerduty.com/oauth_apps/[app_id]
```

The admin will be shown the app and the scopes it is requesting, and can install it for their account:

<!-- TODO: screenshot of the admin installation page -->

Once the admin has completed the installation, users on that account can authorize your app through the
[user token flow](06-OAuth-Functionality.md#obtaining-a-user-oauth-token) as normal.

## How this differs from the rest of the docs

The rest of the developer documentation describes the platform as it behaves outside this Early Access program, so a
few statements elsewhere do not hold for participants. Specifically:

|Where|What it says|What applies to you|
|-|-|-|
|[Private Apps](02-Private-Apps.md)|A Scoped OAuth app "only works on the account that created it"|Your app can also obtain user tokens on any account that has installed it|
|[Private Apps](02-Private-Apps.md)|Private apps "are not reviewed or published by PagerDuty"|Still true. There is no listing for Public Scoped Apps — distribution is by installation link|
|[OAuth Functionality](06-OAuth-Functionality.md#classic-user-oauth)|Classic User OAuth is the option for apps used by many accounts|Scoped OAuth is now also an option, with the user-token limitation above|
|[App Overview](01-Overview.md#app-functionality)|Scoped OAuth connects to the REST API "on the account that created the app"|Extends to accounts that have installed your app, for user tokens only|

Everything else on those pages — scopes, the user token flow, PKCE and confidential client requirements, token
lifetimes, and the app token flow — applies to your app unchanged.
