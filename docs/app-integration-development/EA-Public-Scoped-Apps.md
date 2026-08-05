---
tags: [app-integration-development]
---

# Public Scoped Apps

<!-- theme: warning -->

> ### Early Access
>
> The features described on this page are in an Early Access state and are subject to change. Please reach out to
> us if you have any questions or need support.

Normally, a PagerDuty App with [Scoped OAuth functionality](08-OAuth-Functionality.md#scoped-oauth) is limited to a
single account — the one that created it. Public Scoped Apps extend Scoped OAuth so that a single app can be used
across many PagerDuty accounts, as [Classic User OAuth](08-OAuth-Functionality.md#classic-user-oauth) apps can be
today.

## User tokens only

A Public Scoped App can only obtain **user tokens** for other accounts, by taking a user on that account through the
[user token flow](09-User-OAuth-Token.md) to get their authorization and consent.

The [client credentials flow](12-App-OAuth-Token.md), which issues an **app token** with no user involved, is not
available across accounts. An app token can still only be issued for the account that created the app.

So when your app acts on another account, its access is the intersection of the scopes granted to the app and the
permissions of the authorizing user, as described for user tokens under
[Scoped OAuth](08-OAuth-Functionality.md#scoped-oauth). Any part of your app that relies on app-token access will only
work on your own account.

The requirements of the user token flow are unchanged: a Scoped OAuth app is always a
[confidential client](08-OAuth-Functionality.md#confidential-vs-non-confidential-clients-and-pkce), so it must send its
`client_secret` on the token request and must use PKCE. Being usable by many accounts does not make it safe to
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
[user token flow](09-User-OAuth-Token.md) as normal.
