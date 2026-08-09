---
tags: [app-integration-development]
---

# App Overview

---

<!-- theme: info -->
> This section is intended for developers building an app with PagerDuty. For information on how to add or remove an existing app on your PagerDuty account, visit our [Knowledge Base](https://support.pagerduty.com/docs/apps)

#### What is an app?
An app allows customers to set up other web-enabled businesses to work with PagerDuty or to make PagerDuty more useful. Apps can send data to PagerDuty or receive data from PagerDuty.

#### Who builds apps?
Apps are built by PagerDuty, our partners, our customers, system integrators, and the open source community! Any PagerDuty customer or developer with a [PagerDuty developer account](03-Developer-Account.md) has access to PagerDuty app development tools.

#### Can my app be used by other PagerDuty accounts?
That depends on the functionality you choose. An app using Classic User OAuth can be authorized by a user on any account, and can optionally be [published](08-Publish-Your-App.md) for all PagerDuty customers to discover. An app using Scoped OAuth is a [private app](02-Private-Apps.md) — it only works on the account that created it.

### Getting Started
1. Not a PagerDuty customer? [Sign up for a developer account to get access to PagerDuty](https://developer.pagerduty.com/sign-up/).
1. [Register your app and add functionality to it](04-Register-an-App.md)
1. [Publish an app (completely optional)](08-Publish-Your-App.md)

## App Functionality

Once you have [registered your app](04-Register-an-App.md), add functionality to make it useful.

| App Functionality   | Description|
|:--------------------|:----------------------------------------|
| [Events Integration](05-Events-Integration.md) |  Send machine events **from** your tool **to** PagerDuty over our asynchronous [Events API](../../docs/events-API-v2/01-Overview.md). This is the best way for monitoring tools to connect with PagerDuty in order to trigger incidents. |
| [Simple Install Flow](05-Events-Integration.md#simple-install-flow-optional-but-recommended)|  If you're using the Events API, your users can quickly connect to PagerDuty directly from your tool with the Simple Install Flow! [See Demo](https://acme.pagerduty.dev) |
| [Classic User OAuth](06-OAuth-Functionality.md#classic-user-oauth)|  Connect to our [REST API](https://api-reference.pagerduty.com/) as a PagerDuty User to administer PagerDuty or access data (create an on-call schedule, get a list of team members, etc). |
| [Scoped OAuth](02-Private-Apps.md)|  Connect to our [REST API](https://api-reference.pagerduty.com/) as a PagerDuty App or a PagerDuty User with detailed scopes, on the account that created the app. |

See [Adding functionality to your app](04-Register-an-App.md#adding-functionality-to-your-app) for the steps, and [OAuth Functionality](06-OAuth-Functionality.md) for a more detailed description of the OAuth options.

### More ways to integrate with PagerDuty

These platform features are not available through PagerDuty's app configuration framework (yet!) but are still available for your app to use.

|    |    |
|--- |--- |
|REST API Token|If you need account-level access to a PagerDuty account, you can [request a token from PagerDuty admin users](https://support.pagerduty.com/docs/generating-api-keys#section-rest-api-keys).|
|Webhooks|PagerDuty will send messages **to** your tool when certain activities occur. For example: when an incident is triggered, when a note is added to an incident, etc). <br/><br/>There are 2 ways to create a webhook extension: <ul><li>Over the REST API with the [/extensions endpoint](https://api-reference.pagerduty.com/#!/Extensions/post_extensions) </li><li>[In the PagerDuty web interface](https://support.pagerduty.com/docs/webhooks)</li></ul>|

## Which functionality is right for me?

PagerDuty's Developer Platform has a lot to offer! We want to make it easy to find the right tool for the job. Browse below

| What do you want to do? | PagerDuty API or Platform Feature|
|----|----|
|**Trigger incidents in PagerDuty** (machine events or signals)<br/><br/>If you're integrating a monitoring tool with PagerDuty, this is for you!| Events API - [Read this guide to publish an integration with us](05-Events-Integration.md) or get started on an internal integration.<br/><br/>The Events API is asynchronous, designed for high volume, and connected to PagerDuty Event Intelligence which does grouping, deduplication, and more to help reduce noise and speed up response times. |
|**Send a Change Event** - displays updates and changes in PagerDuty<br/><br/>This enables you to send informational events about recent changes such as code deploys and system config changes from any system that can make an outbound HTTP connection.|[Change Events API](../../docs/events-API-v2/trigger-events/) - /change/enqueue endpoint<br/><br/>The Change Events endpoint is asynchronous.|
|**Trigger incidents in PagerDuty** (human activity)<br/><br/>If a human action is required to trigger the incident (for instance: opening a ticket, clicking a button, typing a chat command), this is the best way to create an incident in PagerDuty.|[ REST API - /incidents endpoint](https://api-reference.pagerduty.com/#!/Incidents/post_incidents)<br/><br/>This endpoint is synchronous (you will receive a reference to the incident created in the response) and rate limited to 1 incident per second.|
|**Connect PagerDuty to a chat tool** - connect PagerDuty to other software systems|REST API via [OAuth](06-OAuth-Functionality.md)<br/><br/>See also: [lita-pagerduty](https://github.com/PagerDuty/lita-pagerduty), [hubot library](https://github.com/hubot-scripts/hubot-pager-me)|
|**Place an iframe in PagerDuty** - display information like runbooks, internal tools or dashboards, etc|[Addons](https://support.pagerduty.com/docs/extensions-add-ons#section-add-ons) are iframes which can be placed on a PagerDuty incident (web or mobile) or as a full page on the web.<br/><br/>Two ways to create addons:<ul><li>[In the PagerDuty user interface](https://support.pagerduty.com/docs/extensions-add-ons#section-add-ons)</li><li>Using the [/addons endpoint](https://api-reference.pagerduty.com/#!/Add-ons/post_addons)on the REST API</li></ul>|
