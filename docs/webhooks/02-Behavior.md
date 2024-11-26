---
tags: [webhooks]
---

# Behavior

<!-- theme: warning -->
> ### Note
> Webhook behavior described on this page may change in the future. If you encounter unexpected behavior, please check this page for documentation updates.

## Setting Up Webhooks

[See our Knowledge Base article](https://support.pagerduty.com/docs/webhooks) for information on setting up webhooks on your account. This article contains technical documentation on our webhook delivery behavior.

## Ports and Connections

Webhooks can be sent to any publicly accessible web server, on any port, with or without encryption.

### Webhook Ports

If you specify `http://` in your Endpoint URL, we will initiate a standard HTTP connection on port 80. Likewise, entering `https://` means we will attempt to make a secure connection on port 443. To override the default port, add a colon (:) after the host address with the desired port number.

**Example:** `https://app.acmemonitoring.com:8443/pagerduty.php`

## Error Handling and Retries

### Timeouts

PagerDuty expects a 2xx response within 5 seconds for Generic Webhooks and within 16 seconds for webhooks generated from Custom Incident Actions.

### Temporary Errors and Retries

PagerDuty will retry sending a webhook in these cases:
  * No response / timeout
  * 5XX (server error) response
  * 429 (rate limit error) response
  * Connection cannot be established (except for most TLS errors)
  * TLS certificate expired errors
  * DNS errors / host name cannot be resolved

Webhooks will be periodically retried for up to 48 hours. While attempting to retry a failing webhook, subsequent webhooks for this extension or subscription and resource ID will be queued for delivery. After 48 hours, the delivery will be considered a failure and the webhook will be dropped.

### Permanent Errors

PagerDuty will drop a webhook *without* a retry in these non-transient cases:
  * 4XX response from the server (except for 429)
  * TLS errors when establishing a connection (except for expired certificates)

## Temporary Disablement

After 3 consecutive webhooks are dropped due to delivery failure, whether from permanent errors or temporary errors that exhausted their retries, the extension or subscription will be disabled for 24 hours and any other webhooks in its queue will be dropped. While an extension or subscription is disabled, no new webhooks will be enqueued for delivery. These webhooks will be dropped.

### Re-enable An Extension or Subscription

When an extension is disabled, it will appear this way on a Service’s integrations tab (under Extensions) or on the Configuration > Extensions page in the PagerDuty web app and can be manually re-enabled there.

![Screenshot of interface for Re-enabling webhook extensions in the PagerDuty web interface](../../assets/images/webhook-disabled.jpg)

When a subscription is disabled, it will be tagged as "Needs Attention" on the webhooks dashboard. It can be easily re-enabled by clicking the "Enable" button on the subscription's settings page, or via the "Enable a webhook subscription" REST API endpoint.

![Screenshot of interface for Re-enabling webhook subscriptions in the PagerDuty web interface](../../assets/images/webhook-subscription-disabled.png)

## Security

PagerDuty supports multiple methods for securing your webhooks.

### Mutual TLS Authentication (Recommended)

We send a client TLS certificate with webhooks on request. You can use this to verify that the webhook was sent from PagerDuty. If you specify an HTTPS endpoint, we will also verify your server certificate before sending webhooks to that endpoint.
<br/>

[Learn how to secure PagerDuty's webhooks using TLS](../../docs/webhooks/03-Mutual-TLS.md)

### Webhook Signature Validation

PagerDuty's [v3 webhooks](../../docs/webhooks/01-Overview.md) are sent with a signature the destination server can use to verify that the event came from the PagerDuty platform and not a third party or malicious system.  It is _strongly recommended_ that webhook consumers verify these signatures before processing each event.
<br/>

[Learn how to verify a PagerDuty webhook signature](../../docs/webhooks/04-Signatures.md)

### IP Safelists

You can configure your network or server to only accept inbound connections from certain IP addresses. See our [Webhook IPs page](../../docs/webhooks/11-Webhook-IPs.md) for information on obtaining the list of IPs webhooks may come from.  (Previously called IP whitelisting, [see why we changed the name here.](https://twitter.com/secnerdette/status/898314208097427457))

### Basic Authentication

If your web server uses basic authentication, you can add the username and password to the URL before the host address. Special characters such as @ in the username or password should be percent-encoded. We recommend sending these requests over `HTTPS` to protect credentials in the URL.

**Examples:**
`https://username:password@app.acmemonitoring.com`
`https://username:long%20password@example.com`

## Ordering

PagerDuty sends webhooks for a given extension and incident combination in the order that they were generated.

## At-Least-Once Delivery

When a webhook endpoint is behaving as specified in this documentation, PagerDuty delivers webhooks at least once. This means that duplicate webhooks may be delivered to the target endpoint in certain circumstances.

Customers wishing to de-duplicate webhooks may do so by using the `X-Webhook-Id` header provided with each webhook request. The value of this header is unique to the webhook but is repeated for each delivery attempt, so it may be used to ignore subsequent delivery attempts after an initial success.

## Batching

PagerDuty does not currently send batches that contain more than one webhook. [V1](../../docs/webhooks/10-V1-Overview.md) and [V2 webhook payloads](../../docs/webhooks/09-V2-Overview.md) contain a `messages` array, however, each event should only contain a single webhook event. [V3 webhook payloads](../../docs/webhooks/01-Overview.md) will only ever contain a single webhook event by design.

## Processing Webhooks

We recommend these best practices when processing webhooks:
  * Return a `202 Accepted` once you receive a payload and then process the batch of webhooks. Asynchronous processing will help prevent the connection from timing out.
  * Catch and handle errors when processing a webhook. This will isolate problems with a specific webhook and allow you to continue processing the rest of the batch.

## Size Limit

Webhook size is affected by a number of factors, including the size of the incoming event, the number of responders on an incident, the number of log entries for an incident, etc. PagerDuty guarantees delivery and ordering of webhooks up to 55 KB (56320 bytes) in size.

In the very rare case that a generated webhook is larger than 55 KB, PagerDuty will attempt to omit event details in the webhook to decrease its size. The fields affected by this omission are present on the first `log_entry` in the `channel` object: `details`, `cef_details.details`, and `body`. The `body` field is only present on events received by email, and the `cef_details.details` field is only present on [PD-CEF](https://support.pagerduty.com/docs/pd-cef) events.

When event details are omitted, this field is replaced with a message noting the omission. In addition, `channel` fields `details_omitted`, `cef_details.details_omitted`, and `body_omitted` will be updated, as appropriate, from `false` to `true` to indicate which fields have been changed.

In the extremely rare case that omitting event details does not bring the webhook under 55 KB, but the webhook is still less than 256 KB, it will be delivered on a best-effort basis. This means that the webhook may not be delivered, or it may be delivered out of order.

All webhooks with a final size greater than 256 KB are always dropped.
