---
tags: [app-integration-development]
---

# Events Integration Functionality

### What is Events Integration functionality?
Events Integration allows you to send machine events **from** your tool **to** PagerDuty over our asynchronous [Events API v2](../../docs/events-API-v2/01-Overview.md)

### Why should I use Events Integration?
This is the best way for monitoring tools to connect with PagerDuty in order to trigger incidents. You'll also be able to acknowledge and resolve incidents. Using the Events API means your app will work with PagerDuty's [Event Intelligence](https://www.pagerduty.com/platform/event-intelligence-and-automation/) features like intelligent grouping and triage.

## Add Events Integration to your app

1. First [create an app in PagerDuty](../../docs/app-integration-development/03-Register-an-App.md)

2. In the **Functionality** section, click **Add** next to Events Integration. Then click **Save** on the Events Integration config page.

![Screenshot of Addding Events Integration to app](../../assets/images/events_integration.png)

## Simple Install Flow (optional, but recommended)

The Simple Install Flow provides an interface for installing a PagerDuty integration directly from your application. Similar to OAuth!

**Why should I use this flow?**
* Better experience for users. They don't need to copy and paste keys from PagerDuty.
* Less work for you (interface is built and maintained by PagerDuty)

**See it in action here:** [acme.pagerduty.dev](https://acme.pagerduty.dev)

Follow these steps to leverage Simple Install Flow for your app:

1. Add a redirect URL for your application. This is where we will redirect the browser to once the user completes the install flow. If you would like to pass through additional GET parameters or a URL fragment when redirecting back, you should URL-encode the `redirect_uri` parameter in the Integration Setup URL.

![Screenshot of Simple Install Redirect URL collection](../../assets/images/simple-install-redirect-uri.png)

2. Implement a page in your app which will receive the request. You should be prepared to handle a request in the format below. Note: you will receive a collection of integration keys and should be prepared to handle more than one.

```http
GET http://<your app URL>?config=<encoded JSON>
```

|||
|-|-|
|`integration_key`|For use the with PagerDuty Events API|
|`name`           |Name of the object the integration key connects to|
|`id`             |Unique ID of the object the integration key connects to|
|`type`           |Type of object (service, global_event_rules, team_rules) Note: others may be added in the future|

**Decoded JSON will follow this format:**

```json
{
  "integration_keys": [
    {
      "integration_key": "key1",
      "name": "Super Cool Service",
      "id": "PD12345",
      "type": "service"
    },
    {
      "integration_key": "key2",
      "name": "Global Event Rules",
      "id": "PD6789A",
      "type": "global_rule_set"
    },
    {
      "integration_key": "key3",
      "name": "B Team's Rules",
      "id": "PDBCDEF",
      "type": "team_rule_set"
    }
  ],
  "account": {
    "subdomain": "dev-acme",
    "Name": "Acme Monitoring"
  }
}
```

3. Once you’ve saved, test out your flow using the **Integration Setup URL** on the page. If you would like to pass through additional GET parameters or a URL fragment, you should URL-encode the `redirect_uri` parameter in the Integration Setup URL.

![Screenshot of copy Simple Install URL For Your App](../../assets/images/copy-simple-install-url-for-app.png)

4. Present this link to users in your application at the right time.

## Add An Event Transformer

**What is an Event Transformer?**

An Event Transformer is an optional part of a PagerDuty app. It contains JavaScript code used to convert a payload sent to PagerDuty into the Events API v2 Common Event Format. Event Transformers are hosted and executed in PagerDuty.

**Why should I use an Event Transformer?**

Use an Event Transformer when a technical service you are connecting to PagerDuty is not capable of modifying its payload before they are sent to PagerDuty. An Event Transformer will allow you to connect it to PagerDuty without hosting an application or serverless function to transform the webhook payload.

### Set up an Event Transformer

1. Under **Transform Event Data** on the Events Integration functionality page, select Yes.

![Screenshot of dropdown to add an Event Transformer to an app](../../assets/images/add-event-transformer.png)

2. Event Transformers support JavaScript (ES6). Use the editor to modify the template to transform any webhook payload into the >Events API v2 format. <br/>[View the detailed guide for writing an app Event Transformer.](../../docs/app-integration-development/07-Writing-App-Event-Transformers.md)

3. After making necessary code changes, click **Save** at the bottom of the page to deploy the transform.

![Screenshot of Save button](../../assets/images/save-events-integration.png)

**Note:**
* The pre-populated template creates an event with the raw body of the POST request payload in custom details.
* You must **Save** to redeploy your transformer code.
* After saving, your transform may take several minutes to deploy. **During this time, events will not be processed.**

## Test your integration

Before submitting, make sure your integration is able to trigger, acknowledge, or resolve events in PagerDuty as you expect.

1. Follow the instructions [here](https://support.pagerduty.com/docs/services-and-integrations#add-integrations-to-an-existing-service) to add your integration to a service; you may wish to create a new service for this purpose.

2. Send test events to the integration key for your test service

![Screenshot of integration key and endpoint for test events](../../assets/images/test-service-details.png)

3. Click on Activity to view incidents or check your notifications to see if they look as you expect.

**Note:**
* You will need to re create your integration for it to pick up any new transform changes you make.
