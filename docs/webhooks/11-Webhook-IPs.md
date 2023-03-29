---
tags: [webhooks]
---
> Notice: This safelist of IP addresses will take effect May 5th, 2022 - 14:00 UTC

# Webhook IPs

This page lists the IP addresses that PagerDuty uses to send webhooks in each of our service regions.

The full list of IP addresses that our webhook notifications may come from is:

### US Service Region

download: [plain](https://developer.pagerduty.com/ip-safelists/webhooks-us-service-region) | [json](https://developer.pagerduty.com/ip-safelists/webhooks-us-service-region-json)

    44.242.69.192
    52.89.71.166
    54.213.187.133
    35.86.21.47
    52.88.94.18
    44.238.89.29
    54.241.68.46
    54.176.72.216
    54.177.81.67
    13.56.49.27
    34.210.57.30
    34.210.242.134
    52.34.208.156


### EU Service Region

download: [plain](https://developer.pagerduty.com/ip-safelists/webhooks-eu-service-region) | [json](https://developer.pagerduty.com/ip-safelists/webhooks-eu-service-region-json)

    18.192.91.93
    18.158.120.237
    18.194.177.30
    18.158.199.216
    3.126.25.87
    18.197.187.16
    54.76.3.62
    54.170.2.90
    52.213.188.110
    54.195.179.238
    34.250.91.200
    54.76.225.71


Until May 5th, 2022 - the [`/webhook_ips`](https://support.pagerduty.com/docs/safelist-ips#webhooks) programmatic endpoint is the authoritative source.

After May 5th, 2022 - the [`/webhook_ips`](https://support.pagerduty.com/docs/safelist-ips#webhooks) endpoint will no longer be updated, and this page should be used for all safelist updates.

If you want total continuity in your firewall, you will need to safelist both sets of ip addresses some time before May 5th, 2022.
