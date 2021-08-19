---
tags: [webhooks]
---

# Public Certificates

PagerDuty Webhooks provide client certificates when requested by a server (Mutual TLS).  

It is our recommendation that customers configure their servers to trust the root certificate listed below.  Customers choosing to rely on the PagerDuty leaf certificate are responsible for rotating to the [new certificate](#new-certificates) at the appropriate time in order to avoid interrupted connectivity.

## Current Certificates

### PagerDuty Webhooks Certificate x.509 ([download](https://developer.pagerduty.com/certificates/2021_webhooks_pagerduty_com.pem))

* Current as of December 8th, 2020
* Expires on January 8th, 2022
* Common Name: _webhooks.pagerduty.com_

### Intermediate Certificate ([download](https://cacerts.digicert.com/DigiCertTLSRSASHA2562020CA1.crt.pem))

* Expires on September 23, 2030
* Common Name: _DigiCert TLS RSA SHA256 2020 CA1_

### Root Certificate ([download](https://cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem))

* Expires on November 10, 2031
* Common Name: _DigiCert Global Root CA_

## New Certificates

PagerDuty Webhook certificates are rotated yearly.  The new certificate(s) will be made available on this page in the November 2021 - December 2021 time frame.
