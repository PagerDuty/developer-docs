---
tags: [app-integration-development]
---

# OAuth 2.0: PKCE Flow

[Create an app](../../docs/app-integration-development/03-Register-an-App.md) to get access to OAuth 2.0 credentials and access PagerDuty's REST API on behalf of a user.

PKCE (Proof Key for Credential Exchange) is recommended for all Single Page Apps and mobile apps and can also be used with a server side flow. PagerDuty also supports OAuth 2.0’s [Authorization Code Grant](https://oauth.net/2/grant-types/authorization-code/) with [PKCE](https://oauth.net/2/pkce/) extension flow for all third-party applications.

### Endpoints for Authorization Code Grant with PKCE:
The following endpoints conform to the OAuth 2.0 protocol for Authorization Code Grant with PKCE.

|||
|-|-|
|Authorization Endpoint|`https://app.pagerduty.com/oauth/authorize`|
|Token Endpoint        |`https://app.pagerduty.com/oauth/token`|

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
`code_challenge`        | Base64 URL Encoded (without padding) string containing the SHA-256 digested form of the clients one-time random 128 byte verifier (also in Base64URLEncoded form without padding). See Javascript PKCE Example Algorithm below.
`code_challenge_method` | Specifies that we are using PKCE SHA-256 Signature.<br/> Value must be set to `S256`.

The flow is initiated by sending a GET request to the Authorization Endpoint with query parameters set for `client_id`, `redirect_uri`, `response_type=code` , as well as the PKCE extension fields (`code_challenge` and `code_challenge_method`)

The user will be required to a) login with credentials and b) authorize the permissions requested by the client application.
```
GET https://app.pagerduty.com/oauth/authorize?client_id={CLIENT_ID}&redirect_uri={REDIRECT_URI}&response_type=code&code_challenge_method=S256&code_challenge
```

## Obtaining an Access Grant : Leg 2 of 3

Upon initiating an access grant there are three possibilities:

### #1 User Cannot Log In (Flow Stopped)

The flow ends without a valid user credential.

### #2 User Logged In and Denied Permission to Client Application (Flow Stopped) ###

The flow ends with access denied. PagerDuty will redirect to the specified URI with `error` and `error_description` parameters:

```http
{REDIRECT_URI}?error=access_denied&error_description=The+resource+owner+or+authorization+server+denied+the+request.&subdomain={ACCOUNT_SUBDOMAIN}
```

### #3 User Logged In and Approves Client Application Permission (Success) ###

If the user authorizes the client application, PagerDuty will redirect to the specified URI with the `code` (authorization code) in the URL:
```
{REDIRECT_URI}?code={AUTHORIZATION_CODE}&subdomain={ACCOUNT_SUBDOMAIN}
```
The authorization code is valid for 10 minutes.

## Exchanging Auth Code For Access Token : Leg 3 of 3

All of these parameters are required to get an access token from the Token Endpoint:

Parameter       | Description
--------------- | -----------
`client_id`     | An identifier issued when the app is created.
`code_verifier` | Original one-time, random 128 byte verifier (also in Base64URLEncoded without padding) used to generate the code_challenge for the authorization code request.
`code`          | The authorization code issued upon a successful authorization request.
`redirect_uri`  | Registered with the app when OAuth 2.0 is added.
`grant_type`    | Value must be set to `authorization_code`.


To exchange the authorization code for an access token, send a POST request to the Token Endpoint.

The authorization code has a time to live of 10 minutes, and your POST request must be received within that time.

Additionally, specify the following query parameters when making the request: `client_id`,  `redirect_uri`, the `code` (authorization code) received from PagerDuty, `grant_type=authorization_code`, and finally the `code_verifier` that was generated to create the code_challenge originally.

```
curl -X POST https://app.pagerduty.com/oauth/token \
  -d "grant_type=authorization_code"
  -d "client_id={CLIENT_ID}"
  -d "redirect_uri={REDIRECT_URI}"
  -d "code={CODE}"
  -d "code_verifier={CODE_VERIFIER}"
```

The access token will be included in a JSON response:

```json
{
  "access_token":"9937611c354d287d3ff509afdde5b1d6d500c73a67387d666ca1e8e3d502d516",
  "token_type":"bearer",
  "scope":"user"
}
```
## Using an Access Token

Once obtained, access tokens can be used to make [REST API](https://api-reference.pagerduty.com) requests on behalf of the user.

When making an API request, include the version of the API in the `Accept` header. Access tokens must also be sent in the request as part of the `Authorization` header along with the `Bearer` token type, using this format:

```http
Authorization: Bearer 9937611c354d287d3ff509afdde5b1d6d500c73a67387d666ca1e8e3d502d516
Accept: application/vnd.pagerduty+json;version=2
```

The token can be used continuously to make requests until the user or app owner revokes it.

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
