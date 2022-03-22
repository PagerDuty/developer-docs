---
tags: [app-integration-development]
---

# OAuth 2.0 Functionality

## What is OAuth 2.0 functionality?
OAuth 2 allows your app to connect to our [REST API](/api-reference/) as a PagerDuty user (not full account access) to administer PagerDuty or get data (create an on-call schedule, get a list of team members, etc).

## Why should I use OAuth 2.0?
With OAuth 2.0, you can present a user with a prompt to log in with their PagerDuty account and authorize your app to access their PagerDuty data.

This is a simple and seamless process for the user and more secure because as an app developer you can:
* Limit your access to read-only
* Scope your access to the permissions of a specific user
* Allow PagerDuty users to monitor and revoke access to your app at any time
* Eliminate copying and pasting API tokens which could lead to the token falling into the wrong hands
![Authorize an OAuth application](../../assets/images/oauth-authorize.png)

## Add OAuth 2.0 functionality to your app
1. [Create an app in PagerDuty](../../docs/app-integration-development/03-Register-an-App.md)

2. In the **Functionality** section, click **Add** on the OAuth 2.0 card.
![Add OAuth Functionality](../../assets/images/add-oauth-functionality.png)

3. On the next page, enter a **Redirect URL**. PagerDuty will only redirect users to a URL saved to your OAuth configuration. Click **Save**. You can edit or add redirect URLs later.
![Specify a redirect URL](../../assets/images/specify-redirect-url.png)

4. You will be shown a modal that displays the **Client ID** and the **Client Secret**. The Client Secret will only be displayed once for security purposes, after which you will have to delete the OAuth client and create a new one if the Client Secret is compromised or otherwise lost. The Client Secret should be stored securely and must not be shared publicly -- if you are using OAuth in a frontend application we recommend the PKCE flow instead (see section on Choosing an OAuth Flow below). The Client ID is public and will be used to identify the app when it authenticates with PagerDuty.

5. Under **Set Permission Scopes**, select an option from the drop-down. By default, the app does not have any permissions set. There are two scope options: **Read** or **Read/Write**. These scopes are tied to the user’s permissions. Authenticated users will only be able to read and write to objects that they have access to.

6. It is recommended to **Add a message to users** to let them know what data the app will access and how the app will utilize that data.

![Select OAuth Scopes](../../assets/images/select-oauth-scopes.png)

Congratulations! OAuth 2.0 is successfully configured for the app. Now you can move on to implementing one of the authorization flows below.

## Implementing OAuth / Choosing An OAuth Flow

There two options for implementing PagerDuty OAuth in your app. [PKCE (Proof Key for Code Exchange](../../docs/app-integration-development/10-OAuth-2-PKCE.md) is recommended and should work for all apps. The [Authorization Code Grant](../../docs/app-integration-development/09-OAuth-2-Auth-Code-Grant.md) Flow is also supported, but is only recommended for server-side applications where you have control over the entire environment.


| Choose A Flow For Your App:   |      Server-side App*      |  Client-side App** |
|:---------------------------------------------------------------------------------------|:-----|:----|
| PKCE - Proof Key for Code Exchange **(Recommended)** |  Yes | Yes |
| Authorization Code Grant |  Yes | No  |


**Client-side App* - an app which runs in the browser or a native mobile app

**Server-side App* - an app running on a server which can securely store secrets

## Cracking Open the PagerDuty ID Token

PagerDuty uses ID Tokens [as defined by the OpenID specification][openid] to securely provide additional details (also called claims) associated with your access token, such as the account subdomain and service region. These tokens are structured as [Json Web Tokens (JWT)](https://datatracker.ietf.org/doc/html/rfc7519). The token is encoded and signed by PagerDuty. 

  [openid]: https://openid.net/specs/openid-connect-core-1_0.html#IDToken

The following is a sample of a full ID token:

```
eyJraWQiOiIxNzg3MzQ1MDA4IiwieDV0IjoiX2Nxbk1aWlBBcEF0V3kyVm11T1Y4dUc5VHNvIiwiYWxnIjoiUlMyNTYifQ.eyJleHAiOjE2NDYwODYyNjAsIm5iZiI6MTY0NjA4MjY2MCwianRpIjoiYTlmZDQzYTQtYjAzNy00ZWViLTk4YjAtNDA1NDJlM2I5OGZmIiwiaXNzIjoiaHR0cHM6Ly9hcHAucGQtc3RhZ2luZy5jb20vZ2xvYmFsL29hdXRoL2Fub255bW91cyIsImF1ZCI6WyJodHRwczovL2FwaS5wZC1zdGFnaW5nLmNvbSIsIjkwYjE1M2JmLWNmZWItNDgzNy04YjZhLTg2NGI3ZmY4NTY1NiJdLCJzdWIiOiJmdGFudGF3aUBwYWdlcmR1dHkuY29tIiwiYXV0aF90aW1lIjoxNjQ2MDgyNjQ5LCJpYXQiOjE2NDYwODI2NjAsInB1cnBvc2UiOiJpZCIsImF0X2hhc2giOiJMTXRGS01MWWVGZXppTnR4OVBKV0hBIiwiYWNyIjoiYWNyOmh0bWwtZm9ybTp1bml0ZWRzdGF0ZXMiLCJkZWxlZ2F0aW9uX2lkIjoiNjU2NjAyN2QtNzcwNy00MjhmLTg3NTItM2VmOTcxYjc4ZDFhIiwiYWNjb3VudF9pZCI6IlA5SkVITUsiLCJ1c2VyX2lkIjoiUDhPR01JSyIsImF6cCI6IjkwYjE1M2JmLWNmZWItNDgzNy04YjZhLTg2NGI3ZmY4NTY1NiIsImFtciI6ImFjcjpodG1sLWZvcm06dW5pdGVkc3RhdGVzIiwic3ViZG9tYWluIjoicGR0LWZhcmVzIiwicmVnaW9uIjoiVW5pdGVkU3RhdGVzIiwic2lkIjoiTGR1Mm03RnFuUDdjSDZHbCJ9.uhbXVou8raKUtx56D8pGmWn3VyX8X1ZhadKYAwftcnc5DNHvEXco8MJKle8w5a1v1f9l881eGHLCsrRUb-B0AOHsWVF0EJTGOhWKgVpx9_SrsGXa7qyVlS3fBh-Gh2IrvDHTBWfe2bQ2g_qEfvCneIBIaELVdOeCysxMShygYgd7iOWwM6m3KGNrtCM4RK0JqYSdTRFpXP-OCBVy6JenJqK3maevffi0-7Z0Q0XFuMIwRR-M3A90Wt9349AhwXNK2kL7mvI3ZltnuomoTcB6rRMUylTp7YXyFjhc4nwA8ZlBw6T8SYPvjXy7iRE0ud4TxEh0J_bfs0gOfqgvKY47aQ
```

The main parts of the ID token are the following.

  Name            | Description
----------------- | -----------
`Header`          | Contains token metadata such as the type of token and hash used
`Payload`         | Contains the user, authorization, and claims
`Signature`       | Used to verify the token and check for tampering

Each part of the token is encoded using the algorithm specified in the header, split by `.`. Every part is structured in the JSON format.

### Header

The header includes three fields. The algorithm used to encrypt and decrypt the token is a variant of SHA-256 called RS256, and is declared as the first part of the header. The next part of the header is the Key ID, which is later used for verification. The last part is the certificate (x5t) which is base64url encoded. Putting these parts together, the header is formed.

The main purpose of the header is to determine a decoding algorithm to use for the rest of the token and what key is used for validation. All valid header parameters can be found [here](https://datatracker.ietf.org/doc/html/rfc7515#section-4.1), but the most common ones are the following.

  Claim           | Description
----------------- | -----------
`alg`             | Algorithm used for signature of the token
`kid`             | Key ID found on the JWKS endpoint of the issuer
`x5t`             | Fingerprint of the certificate used

In the sample ID token above, the header portion is the first section:

```eyJraWQiOiIxNzg3MzQ1MDA4IiwieDV0IjoiX2Nxbk1aWlBBcEF0V3kyVm11T1Y4dUc5VHNvIiwiYWxnIjoiUlMyNTYifQ```

After being base64 decoded, the sample JSON header looks something like this:

``` json
{
  "alg": "RS256",
  "kid": "1787345008",
  "x5t": "_cqnMZZPApAtWy2VmuOV8uG9Tso"
}
```

### Payload

The payload contains claims about the access token, which are statements about the user along with any additional data needed. PagerDuty uses two types of claims:

  Type            | Description
----------------- | -----------
Registered Claims | Predefined recommended claims that are reserved by [the JWT standard](https://datatracker.ietf.org/doc/html/rfc7519#section-4.1). The most common ones used are `iss` (issuer), `aud` (audience), `sub` (subject), `exp` (expiration time). Some of the less common ones are `nbf` (not before), `iat` (issued at), and `jti` (JWT ID).
Private Claims    | PagerDuty includes a number of private claims related to the user and account for convenience.

The claims included in the PagerDuty ID Token are described here in detail:

 Claim           |      Description   
---------------- | ----------------------------------
 `exp`           |  Expiry time, in a Unix timestamp
 `nbf`           |  Unix timestamp that identifies the time before the ID Token must NOT be accpeted (not before)
 `jti`           |  Unique JWT ID
 `iss`           |  The principal/issuer that issued the ID Token. This should always be `https://app.pagerduty.com/global/oauth/anonymous`.
 `aud`           |  Intended reciever of the token (audience). For PagerDuty ID Tokens, this will include a URL specific to the account's service region, either `https://api.pagerduty.com` or `https://api.eu.pagerduty.com`, as well as the client ID.
 `sub`           |  Subject of the ID Token. For PagerDuty ID tokens, this is the user's email address that authenticated the oauth client.
 `auth_time`     |  Time when authentication occurred, in a Unix timestamp
 `iat`           |  Time when the ID Token was issued, in a Unix timestamp
 `purpose`       |  Purpose of the JWT. For PagerDuty ID tokens, this is always identification.
 `at_hash`       |  ID Token hash value
 `acr`           |  The authentication context class reference, which gives context about how the user authenticated the oauth client.
 `delegation_id` |  ID Token delegation that tracks permissions of OAuth clients, and the history of access tokens and refresh tokens used.
 `account_id`    |  PagerDuty account ID linked to the token
 `user_id`       |  PagerDuty user ID linked to the token 
 `azp`           |  The authorized party that was issued the ID Token, which is the client ID
 `amr`           |  Authentication method references. Similar to the `acr` field, this gives context about how the user authenticated the oauth client.
 `subdomain`     |  PagerDuty subdomain linked to the ID token 
 `region`        |  User and account service region
 `sid`           |  Session ID

Here is a sample decrypted payload for a PagerDuty ID token:

```json
{
  "exp": 1646086260,
  "nbf": 1646082660,
  "jti": "a9fd43a4-b037-4eeb-98b0-40542e3b98ff",
  "iss": "https://app.pagerduty.com/global/oauth/anonymous",
  "aud": [
    "https://api.pagerduty.com",
    "90b153bf-cfeb-4837-8b6a-864b7ff85656"
  ],
  "sub": "example@pagerduty.com",
  "auth_time": 1646082649,
  "iat": 1646082660,
  "purpose": "id",
  "at_hash": "LMtFKMLYeFeziNtx9PJWHA",
  "acr": "acr:html-form:unitedstates",
  "delegation_id": "6566027d-7707-428f-8752-3ef971b78d1a",
  "account_id": "P9JEHMK",
  "user_id": "P8OGMIK",
  "azp": "90b153bf-cfeb-4837-8b6a-864b7ff85656",
  "amr": "acr:html-form:unitedstates",
  "subdomain": "pdt-sample",
  "region": "UnitedStates",
  "sid": "Ldu2m7FqnP7cH6Gl"
}
```

### Signature

The signature is used to verify that the message still has its original integrity and hasn't been tampered with and is sent from a trusted issuer.

You can find the public key that PagerDuty uses to sign the token at the following URL: https://app.pagerduty.com/global/oauth/anonymous/jwks Please note however that the signing key could be rotated at any time, so we would recommend fetching the signing key from PagerDuty on each use instead of storing it locally.

This signature is created using the hash algorithm from the header, and to verify a token, you must do the following:

* Retrieve the algorithm used to hash the token from the header
* Use the `x5t` or `kid` parameter from the header to retrieve the public key
* Seperate the signature from the message
* Convert the remaining parts to an ASCII array
* Decode the signature using Base64Url
* Use the decoded signature to validate the ASCII array generated previously

## Decoding the full ID token with Node and Javascript

Once an ID token has been received from PagerDuty, it can be decrypted. The following code sample uses [jsonwebtoken], [jwks-rsa], and 
[node https] to fetch the public PagerDuty key, verify the signature on the ID Token, and returns a Promise that contains the extracted
payload details.

  [jsonwebtoken]: https://www.npmjs.com/package/jsonwebtoken
  [jwks-rsa]: https://www.npmjs.com/package/jwks-rsa
  [node https]: https://nodejs.org/api/https.html

```
function verifyAndExtractIdTokenPayload(oauthTokenResponseJson) {
  const jwt = require('jsonwebtoken');
  const jwksClient = require('jwks-rsa');
  const https = require('https');

  const { idToken, domain, jwtIssuer } = oauthTokenResponseJson;

  return new Promise((resolve, reject) => {
    https.get('https://app.pagerduty.com/global/oauth/anonymous/jwks', (jwksResponse) => {
      let jwksData = '';

      jwksResponse.on('data', (chunk) => {
        jwksData += chunk;
      });

      jwksResponse.on('end', () => {
        const client = jwksClient({
          jwksUri: JSON.parse(jwksData).jwks_uri
        });

        jwt.verify(
          idToken,
          async (header, cb) => {
            const key = await client.getSigningKey(header.kid);
            const signingKey = key.getPublicKey();
            cb(null, signingKey);
          },
          { algorithms: ['RS256'], issuer: jwtIssuer },
          (err, decodedToken) => {
            if (err) {
              reject(`ID Token signature validation failed due to ${err}`);
            } else {
              resolve(decodedToken);
            }
          }
        );
      }).on('error', (err) => {
        reject(`Fetching the PagerDuty public key failed due to ${err}`);
      });
    });
  };
}

```

## Removing OAuth 2.0 Functionality

See [Removing Functionality From Your App](../../docs/app-integration-development/04-App-Functionality.md#removing-functionality-from-your-app)
