---
title: "REST API v2 Introduction FAQ"
excerpt: ""
---
##What are the changes in REST API v2? 

REST API v2 enables developers and non-developers to extend the PagerDuty platform with ease. Additionally, customers will have access to far more comprehensive documentation including reliable client libraries/SDKs, so they can deliver with more agility and performance when building new PagerDuty tools against the REST API.

##What are the benefits of the newer REST API? Why should I switch?

REST API v2 features include the following:

  * Our documentation now allows you to try the API inside your browser
  * The API calls no longer need redundant information like your subdomain or requester_id
  * Consistent naming and access of fields and parameterization
  * Improvements to multiple endpoints for greater speed and performance 
  * New API endpoints
  * An ["Account Abilities" API](https://api-reference.pagerduty.com/#!/Abilities/get_abilities) that allows apps (like the PagerDuty mobile app) to feature gate abilities by account
  * Create or leverage out of the box [add-ons](https://api-reference.pagerduty.com/#!/Add-ons/get_addons) to extend functionality for your users within the PagerDuty experience
  * An [on-calls endpoint](https://api-reference.pagerduty.com/#!/On-Calls/get_oncalls) that makes it easy to get information about who is on call for what and when

The benefits of migrating over to REST API v2 include improved consistency and ease of development, more endpoints and potential add-ons (both pre-built and custom/edge cases), as well as new and improved supporting documentation, best practices and SDKs. 

We strongly encourage you to migrate to v2, as all new features moving forward will only be available on v2, and will not be available on the previous version.

##What support can I expect from PagerDuty in making the switch?

Customers can expect full support for requests related to this migration — please direct all inquiries to [support@pagerduty.com](mailto:support@pagerduty.com). We have created a detailed [migration guide](doc:migrating-to-api-v2) to take you through the step-by-step process for migration, and our client libraries/SDKs include code samples to help streamline that process. 

Our support team has [updated documentation](https://support.pagerduty.com/docs/apis) and is ready to assist with any questions or issues associated with the migration. You can also find [resources for leveraging third party tools for v2](doc:tools-projects).

We look forward to partnering with you to help you with your migration.

##When does the support for the previous API version end?

REST API v1 is entering a decommissioning period on April 24. [Read our FAQ on the decommissioning here](doc:v1-rest-api-decommissioning-faq).

As of July 6, 2016, no new features will be released for v1. 

##What will happen to integrations?

Most integrations, which use the *[Events API](doc:trigger-events)* as distinct from the REST API, will not be affected. Support will end only for REST API v1, and not for the Events API.

##Who is eligible for REST API v2?

All accounts with access to the REST API will have access to v2.

##Who do I contact for more information? 

Please contact our support team at [support@pagerduty.com](mailto:support@pagerduty.com) if you have any questions.