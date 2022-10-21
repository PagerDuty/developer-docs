# The PagerDuty OpenID ID Token

PagerDuty uses ID Tokens [as defined by the OpenID specification][openid] to securely provide additional details (also called claims) associated with your access token, such as the account subdomain and service region. These tokens are structured as [JSON Web Tokens (JWT)](https://datatracker.ietf.org/doc/html/rfc7519). The token is encoded and signed by PagerDuty.

  [openid]: https://openid.net/specs/openid-connect-core-1_0.html#IDToken


 URL Name           | URL  | Description
----------------|------|------------
 Public Key URL | https://identity.pagerduty.com/global/oauth/anonymous/jwks | Use the public key to verify that the token is unmodified and sent by PagerDuty
 Open ID Configuration | https://identity.pagerduty.com/global/oauth/anonymous/.well-known/openid-configuration | Additional details on OpenID features supported by PagerDuty

The ID token will be returned with the access token on a successful call to `oauth/token`:

```
eyJraWQiOiIxNzg3MzQ1MDA4IiwieDV0IjoiX2Nxbk1aWlBBcEF0V3kyVm11T1Y4dUc5VHNvIiwiYWxnIjoiUlMyNTYifQ.eyJleHAiOjE2NDYwODYyNjAsIm5iZiI6MTY0NjA4MjY2MCwianRpIjoiYTlmZDQzYTQtYjAzNy00ZWViLTk4YjAtNDA1NDJlM2I5OGZmIiwiaXNzIjoiaHR0cHM6Ly9hcHAucGQtc3RhZ2luZy5jb20vZ2xvYmFsL29hdXRoL2Fub255bW91cyIsImF1ZCI6WyJodHRwczovL2FwaS5wZC1zdGFnaW5nLmNvbSIsIjkwYjE1M2JmLWNmZWItNDgzNy04YjZhLTg2NGI3ZmY4NTY1NiJdLCJzdWIiOiJmdGFudGF3aUBwYWdlcmR1dHkuY29tIiwiYXV0aF90aW1lIjoxNjQ2MDgyNjQ5LCJpYXQiOjE2NDYwODI2NjAsInB1cnBvc2UiOiJpZCIsImF0X2hhc2giOiJMTXRGS01MWWVGZXppTnR4OVBKV0hBIiwiYWNyIjoiYWNyOmh0bWwtZm9ybTp1bml0ZWRzdGF0ZXMiLCJkZWxlZ2F0aW9uX2lkIjoiNjU2NjAyN2QtNzcwNy00MjhmLTg3NTItM2VmOTcxYjc4ZDFhIiwiYWNjb3VudF9pZCI6IlA5SkVITUsiLCJ1c2VyX2lkIjoiUDhPR01JSyIsImF6cCI6IjkwYjE1M2JmLWNmZWItNDgzNy04YjZhLTg2NGI3ZmY4NTY1NiIsImFtciI6ImFjcjpodG1sLWZvcm06dW5pdGVkc3RhdGVzIiwic3ViZG9tYWluIjoicGR0LWZhcmVzIiwicmVnaW9uIjoiVW5pdGVkU3RhdGVzIiwic2lkIjoiTGR1Mm03RnFuUDdjSDZHbCJ9.uhbXVou8raKUtx56D8pGmWn3VyX8X1ZhadKYAwftcnc5DNHvEXco8MJKle8w5a1v1f9l881eGHLCsrRUb-B0AOHsWVF0EJTGOhWKgVpx9_SrsGXa7qyVlS3fBh-Gh2IrvDHTBWfe2bQ2g_qEfvCneIBIaELVdOeCysxMShygYgd7iOWwM6m3KGNrtCM4RK0JqYSdTRFpXP-OCBVy6JenJqK3maevffi0-7Z0Q0XFuMIwRR-M3A90Wt9349AhwXNK2kL7mvI3ZltnuomoTcB6rRMUylTp7YXyFjhc4nwA8ZlBw6T8SYPvjXy7iRE0ud4TxEh0J_bfs0gOfqgvKY47aQ
```

The main parts of the ID token are the following:

  Name            | Description
----------------- | -----------
`Header`          | Contains token metadata such as the type of token and hash used
`Payload`         | Contains the user, authorization, and claims
`Signature`       | Used to verify the token and check for tampering

Each part of the token is encoded using the algorithm specified in the header, split by `.`. Every part is structured in the JSON format.

## Header

The header includes three fields. The algorithm used to encrypt and decrypt the token is a variant of SHA-256 called RS256, and is declared as the first part of the header. The next part of the header is the Key ID, which is later used for verification. The last part is the certificate (x5t) which is base64url encoded. Putting these parts together, the header is formed.

The main purpose of the header is to determine a decoding algorithm to use for the rest of the token and what key is used for validation. All valid header parameters can be found [here](https://datatracker.ietf.org/doc/html/rfc7515#section-4.1), but the most common ones are the following:

  Claim           | Description
----------------- | -----------
`alg`             | Algorithm used for signature of the token
`kid`             | At a given point, PagerDuty may support multiple signing keys. This id identifies which signing key was used for this token.
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

## Payload

The payload contains claims about the access token, which are statements about the user along with any additional data needed. PagerDuty uses two types of claims:

  Type            | Description
----------------- | -----------
Registered Claims | Predefined recommended claims that are reserved by [the JWT standard](https://datatracker.ietf.org/doc/html/rfc7519#section-4.1). The most common ones used are `iss` (issuer), `aud` (audience), `sub` (subject), `exp` (expiration time). Some of the less common ones are `nbf` (not before), `iat` (issued at), and `jti` (JWT ID).
Private Claims    | PagerDuty includes a number of private claims related to the user and account for convenience.

The claims included in the PagerDuty ID Token are described here in detail:

 Claim           |      Description   
---------------- | ----------------------------------
 `exp`           |  Expiry time, in a Unix timestamp
 `nbf`           |  Unix timestamp that identifies the time before the ID Token must NOT be accepted (not before)
 `jti`           |  Unique JWT ID
 `iss`           |  The principal/issuer that issued the ID Token. This should always be `https://identity.pagerduty.com/global/oauth/anonymous`.
 `aud`           |  Intended reciever of the token (audience). For PagerDuty ID Tokens, this will include a URL specific to the account's service region, either `https://api.pagerduty.com` or `https://api.eu.pagerduty.com`, as well as the client ID.
 `sub`           |  Subject of the ID Token. For PagerDuty ID tokens, this can be used to uniquely identify the user that authorized the oauth client.
 `auth_time`     |  Time when authentication occurred, in a Unix timestamp
 `iat`           |  Time when the ID Token was issued, in a Unix timestamp
 `purpose`       |  Purpose of the JWT. For PagerDuty ID tokens, this is always identification.
 `at_hash`       |  ID Token hash value
 `acr`           |  The authentication context class reference, which gives context about how the user authenticated the oauth client.
 `delegation_id` |  A delegation represents that authorization was granted by a user to your oauth client, similar to a session. It will be consistent across the history of all access tokens and refresh tokens for that authorization.
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
  "iss": "https://identity.pagerduty.com/global/oauth/anonymous",
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

## Signature

The signature is used to verify that the message still has its original integrity and hasn't been tampered with and is sent from a trusted issuer.

You can find the public key that PagerDuty uses to sign the token at the following URL: https://identity.pagerduty.com/global/oauth/anonymous/jwks Please note however that the signing key could be rotated at any time, so we would recommend fetching the signing key from PagerDuty on each use instead of storing it locally.

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

```javascript
function verifyAndExtractIdTokenPayload(oauthTokenResponseJson) {
  const jwt = require('jsonwebtoken');
  const jwksClient = require('jwks-rsa');
  const https = require('https');

  const { idToken, domain, jwtIssuer } = oauthTokenResponseJson;

  return new Promise((resolve, reject) => {
    https.get('https://identity.pagerduty.com/global/oauth/anonymous/jwks', (jwksResponse) => {
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
