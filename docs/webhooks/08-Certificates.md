---
tags: [webhooks]
---

# Public Certificates

PagerDuty Webhooks provide client certificates when requested by a server (Mutual TLS).  

It is our recommendation that customers configure their servers to trust the root certificate listed below. Customers choosing to rely on the PagerDuty leaf certificate are responsible for rotating to the [new certificate](#new-certificates) at the appropriate time in order to avoid interrupted connectivity.

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

PagerDuty webhook certificates are rotated yearly. The next certificate rotation will occur on January 5th, 2022 for both the US and EU service regions. The new certificates are provided below.

### US Service Region (webhooks.pagerduty.com)

#### PagerDuty Webhooks Certificate x.509 ([download](https://developer.pagerduty.com/certificates/2022_webhooks_pagerduty_com.pem))

* Current as of December 8th, 2021
* Expires on January 8th, 2023
* Common Name: _webhooks.pagerduty.com_

#### Intermediate Certificate ([download](https://cacerts.digicert.com/DigiCertTLSRSASHA2562020CA1-1.crt.pem))

* Expires on April 13, 2031
* Common Name: _DigiCert TLS RSA SHA256 2020 CA1_

#### Root Certificate ([download](https://cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem))

* Expires on November 10, 2031
* Common Name: _DigiCert Global Root CA_

### EU Service Region (webhooks.eu.pagerduty.com)

#### PagerDuty Webhooks EU Certificate x.509 ([download](https://developer.pagerduty.com/certificates/2022_webhooks_eu_pagerduty_com.pem))

* Current as of December 8th, 2021
* Expires on January 8th, 2023
* Common Name: _webhooks.eu.pagerduty.com_

#### Intermediate Certificate ([download](https://cacerts.digicert.com/DigiCertTLSRSASHA2562020CA1-1.crt.pem))

* Expires on April 13, 2031
* Common Name: _DigiCert TLS RSA SHA256 2020 CA1_

#### Root Certificate ([download](https://cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem))

* Expires on November 10, 2031
* Common Name: _DigiCert Global Root CA_
