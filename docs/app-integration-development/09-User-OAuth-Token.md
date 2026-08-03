---
tags: [app-integration-development]
---

# Obtaining a User OAuth Token

## About User OAuth Tokens

A user OAuth token lets your app act as a PagerDuty User: access is the intersection of the scopes granted to your app and the permissions of the user who authorized it.

Both [Classic User OAuth and Scoped OAuth](08-OAuth-Functionality.md) obtain user tokens with the same flow — OAuth 2.0's [Authorization Code Grant](https://tools.ietf.org/html/rfc6749#section-4.1), using the [PKCE](https://oauth.net/2/pkce/) extension. The app sends the user to PagerDuty to log in and consent, PagerDuty redirects back with a short-lived authorization code, and the app exchanges that code for an access token.

Before proceeding you should [register a PagerDuty App](03-Register-an-App.md) with Scoped OAuth or Classic User OAuth functionality to obtain the `client_id`, `client_secret`, and scopes.

![Authorize an OAuth application](../../assets/images/oauth-authorize.png)

The flow uses the following endpoints:

|||
|-|-|
|Authorization Endpoint|`https://identity.pagerduty.com/oauth/authorize`|
|Token Endpoint        |`https://identity.pagerduty.com/oauth/token`|

## Confidential and Non-Confidential Clients

How you configure the flow depends on whether your app can keep a `client_secret` secret:

* A **confidential client** runs somewhere you control end to end — a server-side web app, a backend service, a CLI that reads credentials from a secure store. It authenticates to the Token Endpoint with its `client_secret`.
* A **non-confidential client** cannot protect a secret because its code and configuration are distributed to users — a single page app running in the browser, or a native mobile or desktop app. It authenticates to the Token Endpoint with PKCE and no `client_secret`.

Which one you can build depends on the OAuth functionality your app uses:

| |Classic User OAuth|Scoped OAuth|
|-|-|-|
|**Confidential client**|Supported. Send `client_secret` on the token request. PKCE is optional, and recommended.|Supported. Send `client_secret` on the token request. **PKCE is required.**|
|**Non-confidential client**|Supported. Omit `client_secret` on the token request. **PKCE is required.**|Not supported — Scoped OAuth apps must be able to secure a `client_secret`.|

In short: **Scoped OAuth apps must always use PKCE**, and any app that calls the Token Endpoint **without** a `client_secret` must use PKCE. Classic User OAuth confidential clients are the only case where PKCE is optional, and we recommend using it there too.

<!-- theme:warning -->
> A `client_secret` should be treated as a password and stored securely. Never send it as a query parameter or over a non-HTTPS connection, and never ship it in client-side code. If your app runs in a browser or on a mobile device, build it as a non-confidential client with Classic User OAuth.

### What is PKCE?

PKCE (pronounced "pixie") stands for Proof Key for Code Exchange. It lets a client-side JavaScript app, native mobile app, or server-side web app ensure that its authorization code cannot be intercepted and exchanged for a token by someone who does not hold the proof key.

Your app generates a one-time random `code_verifier`, sends the SHA-256 hash of it (the `code_challenge`) on the authorization request, and sends the original `code_verifier` on the token request. PagerDuty issues a token only if the two match. See [Generating a code verifier and challenge](#generating-a-code-verifier-and-challenge) for a JavaScript implementation.

## Initiating the Access Grant : Leg 1 of 3

Send a GET request to the Authorization Endpoint. The user will be required to a) log in with their credentials and b) authorize the permissions your app is requesting.

Parameter               | Description | Required
----------------------- | ----------- | :-:
`client_id`             | An identifier issued when the app is created. | ✓
`redirect_uri`          | Registered with the app when OAuth functionality is added. PagerDuty will redirect here after a user grants or denies access to your app. | ✓
`response_type`         | Specifies the response type based on OAuth 2.0 flow.<br/>Value must be set to `code`. | ✓
`scope`                 | Specifies the scope being requested, which must match [or be a subset of] what is configured for the OAuth functionality. Should be either `read` or `write` for Classic User OAuth, or a space separated set of scopes for Scoped OAuth. | ✓
`code_challenge`        | Base64 URL encoded (without padding) SHA-256 digest of your app's one-time random `code_verifier`. | When using PKCE
`code_challenge_method` | Specifies that we are using the PKCE SHA-256 signature.<br/>Value must be set to `S256`. | When using PKCE

With PKCE — always for Scoped OAuth, and recommended for Classic User OAuth:

```
GET https://identity.pagerduty.com/oauth/authorize?client_id={CLIENT_ID}&redirect_uri={REDIRECT_URI}&scope={SCOPE}&response_type=code&code_challenge={CODE_CHALLENGE}&code_challenge_method=S256
```

Without PKCE — Classic User OAuth confidential clients only:

```
GET https://identity.pagerduty.com/oauth/authorize?client_id={CLIENT_ID}&redirect_uri={REDIRECT_URI}&scope={SCOPE}&response_type=code
```

## Obtaining an Access Grant : Leg 2 of 3

Upon initiating an access grant there are three possibilities:

### #1 User Cannot Log In (Flow Stopped)

The flow ends without a valid user credential.

### #2 User Logged In and Denied Permission to Client Application (Flow Stopped)

The flow ends with access denied. PagerDuty will redirect to the specified URI with `error` and `error_description` parameters:

```
{REDIRECT_URI}?error=access_denied&error_description=The+resource+owner+or+authorization+server+denied+the+request.
```

### #3 User Logged In and Approves Client Application Permission (Success)

If the user authorizes the app, PagerDuty will redirect to the specified URI with the `code` (authorization code) in the URL:

```
{REDIRECT_URI}?code={AUTHORIZATION_CODE}
```

The authorization code is valid for 30 seconds.

## Exchanging Auth Code For Access Token : Leg 3 of 3

To exchange the authorization code for an access token, send a POST request to the Token Endpoint. The authorization code has a time to live of 30 seconds, and your POST request must be received within that time. The content type should be `application/x-www-form-urlencoded`.

Parameter       | Description | Required
--------------- | ----------- | :-:
`grant_type`    | Value must be set to `authorization_code`. | ✓
`client_id`     | An identifier issued when the app is created. | ✓
`code`          | The authorization code issued upon a successful authorization request. | ✓
`redirect_uri`  | Registered with the app when OAuth functionality is added. Must match the `redirect_uri` used on the authorization request. | ✓
`client_secret` | A secret issued when the app is created. | Confidential clients
`code_verifier` | The original one-time random verifier used to generate the `code_challenge` on the authorization request. | When using PKCE

### Confidential clients

Scoped OAuth apps send both a `client_secret` and a `code_verifier`:

```bash
curl -X POST https://identity.pagerduty.com/oauth/token \
  --header "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=authorization_code" \
  -d "client_id={CLIENT_ID}" \
  -d "client_secret={CLIENT_SECRET}" \
  -d "redirect_uri={REDIRECT_URI}" \
  -d "code={CODE}" \
  -d "code_verifier={CODE_VERIFIER}"
```

Classic User OAuth confidential clients may omit `code_verifier` if they did not send a `code_challenge` on the authorization request, though we recommend using PKCE.

### Non-confidential clients

Classic User OAuth non-confidential clients omit the `client_secret`, and `code_verifier` is required:

```bash
curl -X POST https://identity.pagerduty.com/oauth/token \
  --header "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=authorization_code" \
  -d "client_id={CLIENT_ID}" \
  -d "redirect_uri={REDIRECT_URI}" \
  -d "code={CODE}" \
  -d "code_verifier={CODE_VERIFIER}"
```

### The token response

The access token will be included in a JSON response. You may also want to take note of the ID token and the refresh token:

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

Note that our access tokens do expire after a defined period of time — so you may want to make sure that you implement OAuth refresh to prevent users needing to re-authorize your app. See [Token Lifetimes](08-OAuth-Functionality.md#token-lifetimes) for the expiries that apply to each type of OAuth functionality.

For additional information about the user, account, and PagerDuty service region where your app is now authorized, you can look at [cracking open our PagerDuty-signed ID token](11-PagerDuty-OpenId-Token.md). For example, the `aud` field will help your integration with data residency and processing guarantees if you have customers located in Europe.

## Using an Access Token

Once obtained, access tokens can be used to make [REST API](https://api-reference.pagerduty.com) requests on behalf of the user.

When making an API request, include the version of the API in the `Accept` header. Access tokens must also be sent in the request as part of the `Authorization` header along with the `Bearer` token type, using this format:

```http
Authorization: Bearer pdus+_0XBPWQQ_b2b2060b-e7af-44a1-8ddf-9c56fedd8d4f
Accept: application/vnd.pagerduty+json;version=2
```

### Troubleshooting: API Call results in 403

This means that although the OAuth credentials are valid, the token does not have access to that particular resource. For example, if you have a token with the read scope and try to write to a resource, it will result in 403.

If you think you requested the correct scope and should have access to the resource, double check the `scope` field in the POST Token Endpoint response. If an invalid scope is requested, we currently do not return an error. Instead, we grant partial scopes which will only be the `openid` scope automatically attached to all tokens.

Valid scopes for Classic User OAuth:
- `Read` access should request the `read` scope (case sensitive)
- `Read/Write` should request the `write` scope (case sensitive)

For Scoped OAuth, request the specific scopes configured on your app. If a [REST API](/api-reference/) endpoint works with Scoped OAuth, the documentation for that endpoint will say "Scoped OAuth requires:" and list the required scope.

## Getting a new Access Token with a Refresh Token

<!-- theme:warning -->
> ### Securing credentials for non-confidential clients
> Note that we do not recommend storing OAuth client secrets in a browser or mobile app, although it is required to provide your credentials when implementing OAuth refresh (including the client secret).
> Long-term, we would currently recommend using a [Backend-for-Frontend](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-browser-based-apps#section-6.2) to store your OAuth client credentials securely.

As mentioned, all of our current access tokens have an expiration date defined, so it would be to your benefit to implement OAuth refresh to prevent your users from logging in unnecessarily.

Exchanging the refresh token for the access token is similar to using an authorization code: send a POST request to the Token Endpoint, but using the `refresh_token` grant type instead.

The body of the request should include the following parameters: `client_id`, `client_secret`, the `refresh_token` previously received from PagerDuty, and `grant_type=refresh_token`. The content type should still be `application/x-www-form-urlencoded`.

```bash
curl -X POST https://identity.pagerduty.com/oauth/token \
  --header "Content-Type: application/x-www-form-urlencoded" \
  --data-urlencode "grant_type=refresh_token" \
  --data-urlencode "client_id={CLIENT_ID}" \
  --data-urlencode "client_secret={CLIENT_SECRET}" \
  --data-urlencode "refresh_token=pdus+_1XBPWQQ_f85dd2f1-b906-478a-a9e6-678952529e4e"
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

## Sample Code

|Language|GitHub Repository|
|-|-|
|Javascript / Node.js|[pagerduty-oauth-sample-node](https://github.com/PagerDuty/pagerduty-oauth-sample-node)|
|Python 3|[pagerduty-oauth-sample-python](https://github.com/PagerDuty/pagerduty-oauth-sample-python)|
|Javascript, single page app with PKCE|[pagerduty-bulk-user-mgr-sample](https://github.com/PagerDuty/pagerduty-bulk-user-mgr-sample)|

### Generating a code verifier and challenge

The `code_verifier` is a high-entropy random string, and the `code_challenge` is the Base64 URL encoded (without padding) SHA-256 digest of it. Since JavaScript's built-in Base64 operations are not Base64 URL encoded, we provide a generation mechanism here:

```javascript
/*
 *   Example of using these functions:
 *
 *   const pkce = await generateCodePackage();
 *
 *   // These two fields are for leg 1:
 *   pkce.code_challenge;
 *   pkce.code_challenge_method;
 *
 *   // This will need to be stored in sessionStorage, such that it is
 *   // available when the page returns via redirect (leg 3):
 *   pkce.code_verifier;
 */
async function generateCodePackage() {
  // 96 random bytes encode to a 128 character Base64 URL string, the maximum
  // code_verifier length allowed by the PKCE specification.
  const code_verifier = base64UrlEncode(window.crypto.getRandomValues(new Uint8Array(96)));

  // The challenge is the SHA-256 digest of the ASCII bytes of the verifier.
  const digest = await window.crypto.subtle.digest(
    "SHA-256",
    new TextEncoder().encode(code_verifier)
  );

  return {
    code_verifier: code_verifier,
    code_challenge: base64UrlEncode(new Uint8Array(digest)),
    code_challenge_method: "S256",
  };
}

function base64UrlEncode(bytes) {
  return btoa(String.fromCharCode.apply(null, bytes))
    .replace(/\+/g, "-")
    .replace(/\//g, "_")
    .replace(/=+$/, "");
}
```
