---
tags: [webhooks]
---

# Verifying Signatures

<!-- theme: warning -->
### V3 Webhooks are in Early Access
> Webhook signatures are part of V3 webhooks which are still under development.  The API specification is subject to change until they are generally released.

PagerDuty's [v3 webhooks](../../docs/webhooks/01-v3-Overview.md) are sent with a signature the destination server can use to verify that the event came from the PagerDuty platform and not a third party or malicious system.  It is _strongly recommended_ that webhook consumers verify these signatures before processing each event.

## Obtaining the Secret

When a webhook subscription is created, a strong unique secret is generated and provided in the response.  This secret is used to sign webhook payloads sent as part of this subscription.  The signature is then delivered along with each payload under the `X-PagerDuty-Signature` header.

## The Signature

The `X-PagerDuty-Signature` header included with each webhook event delivery contains one or more signatures.  Multiple signatures may be present to allow for a zero-downtime secret rotation.

The current signature version is `v1` and is computed as an [HMAC](https://en.wikipedia.org/wiki/HMAC) of the payload body using a [SHA-256](https://en.wikipedia.org/wiki/SHA-2) hash function.  This is performed for each signing secret and the results are concatenated using comma separation.  An example signature is shown below:

```
X-PagerDuty-Signature:
v1=f03de6f61df6e454f3620c4d6aca17ad072d3f8bbb2760eac3b2ad391b5e8073,
v1=130dcacb53a94d983a37cf2acba98e805a1c37185309ba56fdcccbcf00d6dd8b
```
(Note that the actual header value is sent as a single string without any new lines.)

## Verifying the Signature

A psuedo-algorithm to verify the signature of a webhook delivery is described below.

**Step 1**: *Extract the signature(s) from the request*

* Extract the signature string from the `X-PagerDuty-Signature` header on the request.
* Split the signatures which are separated by the `,` character.
* Select only signatures which are version `v1` and remove the `v1=` prefix.

**Step 2**: *Compute the expected valid signature*

Using the received JSON payload (entire request body):
* Compute the SHA-256 HMAC using the shared secret as the key.
* Take the [Base16](https://en.wikipedia.org/wiki/Hexadecimal) encoding of the result to obtain a value we can compare.

**Step 3**: *Compare the signatures*

Compare the expected signature obtained in Step 2 with the provided signatures from Step 1.

If at least one of the signatures matches, the webhook should be considered a trusted and authentic request from PagerDuty that can be processed by the server.

<!-- theme: info -->
> Note: When comparing signatures, be sure to use a constant-time string comparison to protect against timing attacks.

## Examples of webhooks signing

<!--
type: tab
title: Python
-->

```python
import hmac
import hashlib
import binascii

class PagerDutyVerifier:
  def __init__(self, key, version):
    self.key = key
    self.version = version

  def verify(self, payload, signatures):
    byte_key = self.key.encode('ASCII')
    signature = hmac.new(byte_key, payload.encode(), hashlib.sha256).hexdigest()
    signatureWithVersion = self.version + "=" + signature
    signatureList = signatures.split(",")

    if signatureWithVersion in signatureList:
      return True
    else:
      return False


pagerdutyVerifier = PagerDutyVerifier(key, version)
pagerdutyVerifier.verify(payload, signatures)
```

<!--
type: tab
title: JavaScript
-->

```javascript
const crypto = require('crypto')
module.exports = class PagerDutyVerifier {
	constructor(key, version) {
		this.key = key
		this.version = version
	}

	verify(payload, signatures) {
		var signature = crypto
	    	.createHmac('sha256', key)
	    	.update(payload)
	    	.digest('hex')

    var signatureWithVersion = version + "=" + signature
    var signatureList =  signatures.split(",")

    if (signatureList.indexOf(signatureWithVersion) > -1 ) {
      return true
		} else {
		  return false
		}
	}
 }

var PagerDutyVerifier = require('./pagerduty_verifier.js')
var verifier = new PagerDutyVerifier(key, version)
verifier.verify(payload, signatures)
```

<!-- type: tab-end -->
