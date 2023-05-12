---
tags: [app-integration-development]
---

# Act as User Tokens via PKCE

## About Acting as a User With PKCE

To act as a PagerDuty User in an application where the OAuth client cannot be completely secured [a public client], the PKCE flow should be used.

PKCE (Proof Key for Credential Exchange) is recommended for all Single Page Apps and mobile apps and can also be used with a server side flow. PagerDuty also supports OAuth 2.0’s [Authorization Code Grant](https://oauth.net/2/grant-types/authorization-code/) with [PKCE](https://oauth.net/2/pkce/) extension flow for all third-party applications.

Before proceeding you should [register a PagerDuty App](03-Register-an-App.md) with Scoped OAuth or Classic User OAuth functionality to obtain the `client_id`, `client_secret` [required for Scoped OAuth] and scopes.

The following endpoints conform to the OAuth 2.0 protocol for Authorization Code Grant with PKCE:

|||
|-|-|
|Authorization Endpoint|`https://identity.pagerduty.com/oauth/authorize`|
|Token Endpoint        |`https://identity.pagerduty.com/oauth/token`|

### What is PKCE?

PKCE (pronounced pixie) stands for Proof Key for Code Exchange and allows a client side javascript, native mobile app, or server side web app to ensure that its code grant cannot be intercepted and exchanged for a token without knowing the Proof Key.

<!-- theme: info -->
> ### PKCE Code Sample For PagerDuty
> [Sample Application in Javascript](https://github.com/PagerDuty/pagerduty-bulk-user-mgr-sample)

## Initiating the Access Grant : Leg 1 of 3

All of these parameters are required to initiate an access grant to the Authorization Endpoint:


Required Parameters     | Description
----------------------- | ------------
`client_id`             | An identifier issued when the app is created
`redirect_uri`          | Registered with the app when OAuth 2.0 is added. PagerDuty will redirect here after a user grants or denies access to your app.
`response_type`         | Specifies the response type based on OAuth 2.0 flow.<br/> Value must be set to `code`.
`scope`                 | Specifies the scope being requested, must match [or be a subset of] what is configured for the OAuth functionality. Should be either `read` or `write` for Classic User OAuth or a space separated set of scopes for Scoped OAuth. 
`code_challenge`        | Base64 URL Encoded (without padding) string containing the SHA-256 digested form of the clients one-time random 128 byte verifier (also in Base64URLEncoded form without padding). See Javascript PKCE Example Algorithm below.
`code_challenge_method` | Specifies that we are using PKCE SHA-256 Signature.<br/> Value must be set to `S256`.

<!-- theme:warning -->
> Never send your `client_secret` as a query parameter or over a non-https connection.

The flow is initiated by sending a GET request to the Authorization Endpoint with query parameters set for `client_id`, `redirect_uri`, `scope`, `response_type=code`, as well as the PKCE extension fields (`code_challenge` and `code_challenge_method`)

The user will be required to a) login with credentials and b) authorize the permissions requested by the client application.
```
GET https://identity.pagerduty.com/oauth/authorize?client_id={CLIENT_ID}&redirect_uri={REDIRECT_URI}&scope={SCOPE}&response_type=code&code_challenge_method=S256&code_challenge
```

## Obtaining an Access Grant : Leg 2 of 3

Upon initiating an access grant there are three possibilities:

### #1 User Cannot Log In (Flow Stopped)

The flow ends without a valid user credential.

### #2 User Logged In and Denied Permission to Client Application (Flow Stopped) ###

The flow ends with access denied. PagerDuty will redirect to the specified URI with `error` and `error_description` parameters:

```http
{REDIRECT_URI}?error=access_denied&error_description=The+resource+owner+or+authorization+server+denied+the+request.
```

### #3 User Logged In and Approves Client Application Permission (Success) ###

If the user authorizes the client application, PagerDuty will redirect to the specified URI with the `code` (authorization code) in the URL:
```
{REDIRECT_URI}?code={AUTHORIZATION_CODE}&subdomain={ACCOUNT_SUBDOMAIN}
```
The authorization code is valid for 30 seconds.

## Exchanging Auth Code For Access Token : Leg 3 of 3

All of these parameters are required to get an access token from the Token Endpoint:

Parameter       | Description
--------------- | -----------
`client_id`     | An identifier issued when the app is created.
`client_secret` | [Scoped OAuth only] A secret issued when the app is created.
`code_verifier` | Original one-time, random 128 byte verifier (also in Base64URLEncoded without padding) used to generate the code_challenge for the authorization code request.
`code`          | The authorization code issued upon a successful authorization request.
`redirect_uri`  | Registered with the app when OAuth 2.0 is added.
`grant_type`    | Value must be set to `authorization_code`.


To exchange the authorization code for an access token, send a POST request to the Token Endpoint.

The authorization code has a time to live of 30 seconds, and your POST request must be received within that time.

The body of the request should include the following parameters: `client_id`,  `redirect_uri`, the `code` (authorization code) received from PagerDuty, `grant_type=authorization_code`, and finally the `code_verifier` that was generated to create the code_challenge originally. The content-type should be `application/x-form-urlencoded`.

```bash
curl -X POST https://identity.pagerduty.com/oauth/token \
  --header "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=authorization_code"
  -d "client_id={CLIENT_ID}"
  -d "redirect_uri={REDIRECT_URI}"
  -d "code={CODE}"
  -d "code_verifier={CODE_VERIFIER}"
```

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

Note that our access tokens do expire after a defined period of time -- so you may want to make sure that you implement OAuth refresh to prevent users needing to re-authorize your app. See more information about token expiries at the bottom of this page.

For additional information about the user, account, and PagerDuty service region where your app is now authorized, you can look at [cracking open our PagerDuty-signed ID token](../../docs/app-integration-development/11-OAuth-2-Id-token.md). For example, the `aud` field will help your integration with data residency and processing guarantees if you have customers located in Europe.


## Using an Access Token

Once obtained, access tokens can be used to make [REST API](https://api-reference.pagerduty.com) requests on behalf of the user.

When making an API request, include the version of the API in the `Accept` header. Access tokens must also be sent in the request as part of the `Authorization` header along with the `Bearer` token type, using this format:

```http
Authorization: Bearer pdus+_0XBPWQQ_39435d07-9232-4bc2-9dc9-c4a8fccc94ad
Accept: application/vnd.pagerduty+json;version=2
```

### Troubleshooting: API Call results in 403

This means that although the OAuth credentials are valid, the token does not have access to that particular resource. For example, if you have a token with the read scope and try to write to a resource, it will result in 403.

If you think you requested the correct scope and should have access to the resource, double check the `scope` field in the POST Token Endpoint response. If an invalid scope is requested, we currently do not return an error. Instead, we grant partial scopes which will only be the `openid` scope automatically attached to all tokens.

Valid Scopes:
- `Read` access should request the `read` scope (case sensitive)
- `Read/Write` should request the `write` scope (case sensitive)

## Getting a new Access Token with a Refresh Token

<!-- theme:warning -->
> ### Securing credentials for public clients
> Note that we do not recommend storing OAuth client secrets for a public OAuth client in a browser or mobile app, although it is required to provide your credentials when implementing OAuth refresh (including the client secret).
> Long-term, we would currently recommend using a [Backend-for-Frontend](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-browser-based-apps#section-6.2) to store your OAuth client credentials securely.

As mentioned, all of our current access tokens have an expiration date defined, so, it would be to your benefit to implement OAuth refresh to prevent your users from logging in unnecessarily.

Exchanging the refresh token for the access token is similar to using an authorization code: send a POST request to the token endpoint, but using the `refresh_token` grant type instead.

The body of the request should include the following parameters: `client_id`, `client_secret`, the `refresh_token` previously received from PagerDuty, and `grant_type=refresh_token`. The content type should still be `application/x-form-urlencoded`.

```bash
curl -X POST https://identity.pagerduty.com/oauth/token \
  --header "Content-Type: application/x-www-form-urlencoded" \
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

## Sample Code

### Javascript Application
[https://github.com/PagerDuty/pagerduty-bulk-user-mgr-sample](https://github.com/PagerDuty/pagerduty-bulk-user-mgr-sample)

### Javascript PKCE Example Algorithm

Since Javascript has a non-binary-unicode compatible Base64 operation, we provide a Javascript PKCE generation mechanism here:

```javascript
/*
 *    // Example of using this function:
 *
 *   A generateCodePackagePromise().then(function(package) {
 *     //
 *     // These two fields are for leg1
 *     //
 *     package.code_challenge;
 *     package.code_challenge_method;
 *     //
 *     // This will need to be stored in sessionStore
 *     // such that it is avalible when the page returns via redirect (leg2)
 *     //
 *     package.code_verifer;
 *    })
 *
 */
 function generateCodePackagePromise() {
    var base64Url = function(buffer) {
          /*
          |*|
          |*|  Base64 / binary data / UTF-8 strings utilities (#1)
          |*|  Based on Code From:
          |*|  https://developer.mozilla.org/en-US/docs/Web/API/WindowBase64/Base64_encoding_and_decoding
          |*|
          |*|  Author: madmurphy
          |*|
          */

        var uint6ToB64 = function(nUint6) {

            return nUint6 < 26 ?
                nUint6 + 65 :
                nUint6 < 52 ?
                nUint6 + 71 :
                nUint6 < 62 ?
                nUint6 - 4 :
                nUint6 === 62 ?
                43 :
                nUint6 === 63 ?
                47 :
                65;

        }

        var base64EncArr = function(aBytes) {

            var eqLen = (3 - (aBytes.length % 3)) % 3,
                sB64Enc = "";

            for (var nMod3, nLen = aBytes.length, nUint24 = 0, nIdx = 0; nIdx < nLen; nIdx++) {
                nMod3 = nIdx % 3;
                /* Uncomment the following line in order to split the output in lines 76-character long: */
                /*
                  if (nIdx > 0 && (nIdx * 4 / 3) % 76 === 0) { sB64Enc += "r"; }
                */
                nUint24 |= aBytes[nIdx] << (16 >>> nMod3 & 24);
                if (nMod3 === 2 || aBytes.length - nIdx === 1) {
                    sB64Enc += String.fromCharCode(uint6ToB64(nUint24 >>> 18 & 63), uint6ToB64(nUint24 >>> 12 & 63), uint6ToB64(nUint24 >>> 6 & 63), uint6ToB64(nUint24 & 63));
                    nUint24 = 0;
                }
            }

            return eqLen === 0 ?
                sB64Enc :
                sB64Enc.substring(0, sB64Enc.length - eqLen) + (eqLen === 1 ? "=" : "==");

        };


        var base64 = base64EncArr(new Uint8Array(buffer));
        var base64url_no_padding = base64.replace(/+/g, '-')
              .replace(///g, '_')
              .replace(/=/g, '');

        return base64url_no_padding;
    }
      var gen128ishVerifier = function() {
         // account for the overhead of going to base64
         var bytes = Math.floor(128  / 1.37);  
         var array = new Uint8Array(bytes); //
         // note: there was a bug where getRandomValues was assumed
         // to modify the reference to the array and not return
         // a value
         array = window.crypto.getRandomValues(array);
         return base64Url(array.buffer);
    };

    var code_verifier = gen128x8bitNonce();

    /*
       Note : We consider the code_verifier used for the purpose
       of generating a hash to be in the base64_url_no_padding format
       Since any random sequence of 1024bits can be mapped into a base64
       encoding and the standard recommends the code_verifier be specified
       in terms of the characters that base64_url_no_padding is made up
       of - the most effective way to create a code_verifier is by not
       generating a random sequence of the base64 characters but simply
       generate a 1024bit number sequence and convert it to base64 string.
    */
    var base64_verifier = base64Url(code_verifier.buffer);
    var encoder = new TextEncoder();
    var base64_arraybuffer = encoder.encode(base64_verifier);

    return crypto.subtle.digest("SHA-256", base64_arraybuffer).then(function(code_challenge){

        return {
            code_verifer: base64_verifier,
            code_challenge: base64Url(code_challenge),
            code_challenge_method: "S256"
        }
    });
  }

```
