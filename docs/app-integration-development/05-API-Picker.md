---
tags: [app-integration-development]
---

# API Picker

PagerDuty's Developer Platform has a lot to offer! We want to make it easy to find the right tool for the job. Browse below

| What do you want to do? | PagerDuty API or Platform Feature|
|----|----|
|**Trigger incidents in PagerDuty** (machine events or signals)<br/><br/>If you're integrating a monitoring tool with PagerDuty, this is for you!| Events API - [Read this guide to publish an integration with us](../..//docs/app-integration-development/06-Events-Integration.md) or get started on an internal integration.<br/><br/>The Events API is asynchronous, designed for high volume, and connected to PagerDuty Event Intelligence which does grouping, deduplication, and more to help reduce noise and speed up response times. |
|**Send a Change Event** - displays updates and changes in PagerDuty<br/><br/>This enables you to send informational events about recent changes such as code deploys and system config changes from any system that can make an outbound HTTP connection.|[Change Events API](../../docs/events-API-v2/trigger-events/) - /change/enqueue endpoint<br/><br/>The Change Events endpoint is asynchronous.|
|**Trigger incidents in PagerDuty** (human activity)<br/><br/>If a human action is required to trigger the incident (for instance: opening a ticket, clicking a button, typing a chat command), this is the best way to create an incident in PagerDuty.|[ REST API - /incidents endpoint](https://api-reference.pagerduty.com/#!/Incidents/post_incidents)<br/><br/>This endpoint is synchronous (you will receive a reference to the incident created in the response) and rate limited to 1 incident per second.|
|**Connect PagerDuty to a chat tool** - connect PagerDuty to other software systems|REST API via [OAuth](../../docs/app-integration-development/08-OAuth-Functionality.md)<br/><br/>See also: [lita-pagerduty](https://github.com/PagerDuty/lita-pagerduty), [hubot library](https://github.com/hubot-scripts/hubot-pager-me)|
|**Place an iframe in PagerDuty** - display information like runbooks, internal tools or dashboards, etc|[Addons](https://support.pagerduty.com/docs/extensions-add-ons#section-add-ons) are iframes which can be placed on a PagerDuty incident (web or mobile) or as a full page on the web.<br/><br/>Two ways to create addons:<ul><li>[In the PagerDuty user interface](https://support.pagerduty.com/docs/extensions-add-ons#section-add-ons)</li><li>Using the [/addons endpoint](https://api-reference.pagerduty.com/#!/Add-ons/post_addons)on the REST API</li></ul>|

