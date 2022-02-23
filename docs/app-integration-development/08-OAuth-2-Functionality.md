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

## Cracking Open an ID Token

Pagerduty uses ID tokens to fetch access tokens, which are structured as [Json Web Tokens (JWT)](https://datatracker.ietf.org/doc/html/rfc7519). The token is encoded and signed, and contains claims from the OpenID Connect Provider used. 

The main parts of the ID token are the following.

  Name            | Description
----------------- | -----------
`Header`          | Contains token metadata such as the type of token and hash used
`Payload`         | Contains user, authorization, and claims
`Signature`       | Used to verify the token and check for tampering

Each part of the token is encoded using the algorithm specified in the header, split by `.`. Every part is structured in the JSON format.

### Header
The main purpose of the header is to determine a decoding algorithm to use for the rest of the token and what key is used for validation. All valid header parameters can be found [here](https://datatracker.ietf.org/doc/html/rfc7515#section-4.1), but the most common ones are the following.

  Claim           | Description
----------------- | -----------
`kid`             | Key ID found on the JWKS endpoint of the issuer
`x5t`             | Fingerprint of the certificate used
`alg`             | Algoirthm used for signature of the token

### Payload
The payload contains all claims, which are statements about the user along with any additional data needed. There are three types of claims, and are the following.

  Type            | Description
----------------- | -----------
Registered Claims | Predefined recommended claims that are inoperable. The most common ones used are `iss` (issuer), `aud` (audience), `sub` (subject), `exp` (expiration time). Some of the less common ones are `nbf` (not before), `iat` (issued at), and `jti` (JWT ID)
Public Claims     | Claims created and defined by users which are publicly available, typically on the [JSON Web Token Registry](https://www.iana.org/assignments/jwt/jwt.xhtml)
Private Claims    | Claims created by two parties that are not registered or public

Each payload is Base64Url encoded.

### Signature
The signature is used to verify that the message still has its original integrity and hasn't been tampered with and is sent from a trusted issuer, and will also contain a private key if the token has one.

This signature is created using the hash algorithm from the header, and to verify a token, you must do the following;

* Retrieve the algorithm used to hash the token from the header
* Use the `x5t` or `kid` parameter from the header to retrieve the public key
* Seperate the signature from the message
* Convert the remaining parts to an ASCII array
* Decode the signature using Base65Url
* Use the decoded signature to validate the ASCII array generated previously

The contents are now validated, and can be further inspected to ensure the message is correct. The following claims can be inspected and verified;

 Claim       |      Description   
-------------------------------------------------
 `acr`       |  Authentication method used should match `acr_values` request parameter
 `iss`       |  Identifier of the token sender, and must be a plain https url
 `sub`       |  Identifier of the user
 `aud`       |  Expected reciever of the token which must contain a client identifier
 `iat`       |  Time the ID Token was issued
 `auth_time` |  Time the user was last logged in without SSO
 `jti`       |  Unique token identifier is what was expected 
 `nonce`     |  The nonce if passed into the token

## Full ID Token Example



## Removing OAuth 2.0 Functionality
See [Removing Functionality From Your App](../../docs/app-integration-development/04-App-Functionality.md#removing-functionality-from-your-app)
