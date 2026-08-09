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
Yes, apps can be created for either Private use (on your account only) or Public use which can be installed and used by other PagerDuty customers. See [Private Apps](02-Private-Apps.md) and [Publish an App](08-Publish-Your-App.md).

### Getting Started
1. Not a PagerDuty customer? [Sign up for a developer account to get access to PagerDuty](https://developer.pagerduty.com/sign-up/).
1. [Register your app and add functionality to it](04-Register-an-App.md)
1. [Publish an app (completely optional)](08-Publish-Your-App.md)

## App Functionality

There are two kinds of app functionality. An app can have either, or both.

### Events Integration

[Events Integration](05-Events-Integration.md) sends machine events **from** your tool **to** PagerDuty over the asynchronous [Events API v2](../../docs/events-API-v2/01-Overview.md). This is the best way for a monitoring tool to connect with PagerDuty, and it can trigger, acknowledge, and resolve incidents.

The Events API is asynchronous and designed for high volume. Events flow through PagerDuty's [Event Intelligence](https://www.pagerduty.com/platform/event-intelligence-and-automation/) features, which group, deduplicate, and triage them to reduce noise and speed up response.

It can also send [change events](../../docs/events-API-v2/03-Change-Events.md) — informational signals about recent changes such as code deploys and system config changes, which are displayed in PagerDuty rather than triggering an incident.

Two optional pieces come with it:

* The [Simple Install Flow](05-Events-Integration.md#simple-install-flow-optional-but-recommended) lets your users connect to PagerDuty from inside your tool, instead of copying and pasting integration keys. [See Demo](https://acme.pagerduty.dev)
* An [Event Transformer](05-Events-Integration.md#add-an-event-transformer) converts a payload PagerDuty does not understand natively into the Events API v2 format, without you hosting anything.

### OAuth Integration

[OAuth](06-OAuth-Functionality.md) connects your app to the [REST API](https://api-reference.pagerduty.com/), which manages PagerDuty resources — users, teams, services, schedules, escalation policies, and so on. This is what you want for an app that administers PagerDuty or reads data out of it, such as a chat tool integration.

The REST API can also trigger and manage incidents, through the [/incidents endpoint](https://api-reference.pagerduty.com/#!/Incidents/post_incidents). Use it when you specifically intend to create one incident — typically in response to a human action such as opening a ticket, clicking a button, or typing a chat command. It is synchronous, so you get a reference to the incident back, and it is rate limited to 1 incident per second. For machine events at volume, use Events Integration instead.

There are two kinds of OAuth functionality:

* [Classic User OAuth](06-OAuth-Functionality.md#classic-user-oauth) always acts as a PagerDuty user, with a blanket `read` or `write` scope. It can be authorized by a user on any account, and can be published.
* [Scoped OAuth](02-Private-Apps.md) grants access per resource type and can act as the app itself rather than as a user. It only works on the account that created the app.

For chat integrations, see also [lita-pagerduty](https://github.com/PagerDuty/lita-pagerduty) and the [hubot library](https://github.com/hubot-scripts/hubot-pager-me).

## Other ways to integrate with PagerDuty

These are not app functionality — you configure them outside of app registration — but they are often part of an integration.

|    |    |
|--- |--- |
|Webhooks|PagerDuty will send messages **to** your tool when certain activities occur. For example: when an incident is triggered, when a note is added to an incident, etc). <br/><br/>There are 2 ways to create a webhook extension: <ul><li>Over the REST API with the [/extensions endpoint](https://api-reference.pagerduty.com/#!/Extensions/post_extensions) </li><li>[In the PagerDuty web interface](https://support.pagerduty.com/docs/webhooks)</li></ul>|
|Add-ons|[Add-ons](https://support.pagerduty.com/docs/extensions-add-ons#section-add-ons) are iframes which can be placed on a PagerDuty incident (web or mobile) or as a full page on the web.<br/><br/>Two ways to create add-ons:<ul><li>[In the PagerDuty user interface](https://support.pagerduty.com/docs/extensions-add-ons#section-add-ons)</li><li>Using the [/addons endpoint](https://api-reference.pagerduty.com/#!/Add-ons/post_addons) on the REST API</li></ul>|
|REST API Token|If you need account-level access to a PagerDuty account, you can [request a token from PagerDuty admin users](https://support.pagerduty.com/docs/generating-api-keys#section-rest-api-keys).|
