---
tags: [webhooks]
---

# Public Certificates

PagerDuty Webhooks provide client certificates when requested by a server (Mutual TLS).  

It is our recommendation that customers configure their servers to trust the root certificate listed below. Customers choosing to rely on the PagerDuty client certificate are responsible for rotating to the new certificates at the appropriate time in order to avoid interrupted connectivity. PagerDuty webhook certificates are rotated yearly. The most recent certificate rotation occurred on **January 4th 2024** for both the US and EU service regions. The latest valid certificates are provided below. 

## Current Certificates

#### US Service Region (webhooks.pagerduty.com)
| Certificate Type                                                                                                             | Common Name                      | Valid Until         |
|:-----------------------------------------------------------------------------------------------------------------------------|:---------------------------------|:--------------------|
| PagerDuty Webhooks Certificate ([download](https://developer.pagerduty.com/certificates/2024_webhooks_pagerduty_com.pem))    | webhooks.pagerduty.com           | January 3th, 2025   |
| Intermediate Certificate ([download](https://cacerts.digicert.com/DigiCertGlobalG2TLSRSASHA2562020CA1-1.crt.pem))    | DigiCert Global G2 TLS RSA SHA256 2020 CA1  | March 29th, 2031   |
| Root Certificate ([download](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem))    | DigiCert Global Root G2           | January 15th, 2038   |

#### EU Service Region (webhooks.eu.pagerduty.com)

| Certificate Type                                                                                                             | Common Name                      | Valid Until         |
|:-----------------------------------------------------------------------------------------------------------------------------|:---------------------------------|:--------------------|
| PagerDuty Webhooks Certificate ([download](https://developer.pagerduty.com/certificates/2024_webhooks_eu_pagerduty_com.pem)) | webhooks.eu.pagerduty.com        | January 3th, 2025   |
| Intermediate Certificate ([download](https://cacerts.digicert.com/DigiCertGlobalG2TLSRSASHA2562020CA1-1.crt.pem))    | DigiCert Global G2 TLS RSA SHA256 2020 CA1  | March 29th, 2031   |
| Root Certificate ([download](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem))    | DigiCert Global Root G2           | January 15th, 2038   |
