---
tags: [rest-api]
---

# Alert Grouping Settings API - FAQ 


### What is the maximum number of services for a Global Alert Grouping setting?

With Global Alert Grouping - Content Based, there will be a **limit of 250 services maximum per alert grouping setting.**


### What is the difference between the [Services API] endpoints where I set the alert grouping parameters for a given service vs. the [Alert Grouping Settings API] endpoints?

- Existing `/services` endpoints only allow you to define Single Service Alert Grouping.
- The new `/alert_grouping_settings` endpoints allow you to define Single or Global Content Based Alert Grouping.
- Single Service Time and Single Intelligent Grouping types can still be set using the `/services` endpoints.


### Can I use only the new [Alert Grouping Settings API] endpoints?

Later in this year 2024, you will be able to use the `/alert_grouping_settings` endpoints **exclusively** to set **any** alert grouping types, including: 
- Create or update Single or Multi-Service (Global) Alert Grouping settings.
- Get and list Alert Grouping settings for all your services.


### What happens if I use the [Services API] endpoint with `alert_grouping_parameters` field populated for a service that is contained in a Global Alert Grouping setting (update a service that is part of a Global Alert Grouping setting)?

- That service will be removed from the Global Alert Grouping setting and updated with the new specific settings if you provide any update in the `alert_grouping_parameters` field in the `/services` endpoint.
- For example, if you have Service A, Service B, and Service C as part of one Global Alert Grouping setting, then use the `/services` endpoint to set Service A to with `type="intelligent"`, this will remove Service A from the Global Alert Grouping setting. Service B and Service C will remain in the Global Alert Grouping setting unchanged. 
    - Similarly, if you have only 2 services in your Global Alert Grouping setting (Service A and Service B), then update Service A with `aggregate="any"` (or **any** other alert grouping setting) via the `/services` endpoint, this will create a new single-service alert grouping setting for Service A, and Service B will maintain the prior alert grouping settings.
- Due to this, please make sure to exclude the `alert_group_parameters` field from any routine Terraform updates sent to `/services`. This will prevent unintended modifications to services configured under a Global Alert Grouping setting.

[Services API]: https://developer.pagerduty.com/api-reference/7062f2631b397-create-a-service
[Alert Grouping Settings API]: https://developer.pagerduty.com/api-reference/587edbc8ff416-create-an-alert-grouping-setting