---
tags: [webhooks]
---

# Public Certificates

PagerDuty Webhooks provide client certificates when requested by a server (Mutual TLS).  

It is our recommendation that customers configure their servers to trust the root certificate listed below. Customers choosing to rely on the PagerDuty client certificate are responsible for rotating to the new certificates at the appropriate time in order to avoid interrupted connectivity. PagerDuty webhook client certificates are rotated yearly. The next certificate rotation will occur on **December 9th, 2025** for both the US and EU service regions. The current and upcoming certificates are provided below.

#### Client Certificates
Certificate | Common Name | Start Date | End Date | Download
---------|----------|---------|---------|---------
 Upcoming US region certificate | webhooks.pagerduty.com | December 9th, 2025 | October 2026 | [link](https://developer.pagerduty.com/certificates/2026_webhooks_pagerduty_com.pem)
 Upcoming EU region certificate | webhooks.eu.pagerduty.com | December 9th, 2025 | October 2026 | [link](https://developer.pagerduty.com/certificates/2026_webhooks_eu_pagerduty_com.pem)
 Current US region certificate | webhooks.pagerduty.com | December 18th, 2024 | December 2025 | [link](https://developer.pagerduty.com/certificates/2025_webhooks_pagerduty_com.pem)
 Current EU region certificate | webhooks.eu.pagerduty.com | December 18th, 2024 | December 2025 | [link](https://developer.pagerduty.com/certificates/2025_webhooks_eu_pagerduty_com.pem)

#### Issuer Certificates
Certificate Type | Common Name | Valid Until | Download
---------|----------|---------|---------
 Intermediate Certificate | DigiCert Global G2 TLS RSA SHA256 2020 CA1 | March 29th, 2031 | [link](https://cacerts.digicert.com/DigiCertGlobalG2TLSRSASHA2562020CA1-1.crt.pem)
 Root Certificate | DigiCert Global Root G2 | January 15th, 2038 | [link](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem)
