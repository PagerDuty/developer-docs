---
tags: [rest-api]
---

# REST API v2 Overview

<!-- theme: warning -->
> ### Importing Pagerduty OpenAPI Specs into Postamn
>
> * The "Run in Postman" Button was removed in favor of this more concrete workflow
> * See Postman's [documentation](https://learning.postman.com/docs/integrations/available-integrations/working-with-openAPI/) on integrating Postman with OpenAPI
> * Use [this URL](https://raw.githubusercontent.com/PagerDuty/api-schema/main/reference/REST/openapiv3.json) in the OpenAPI import tool


The REST API provides a way for third parties to connect to a PagerDuty account and access or manipulate configuration data on that account. It is not for connecting your monitoring tools to send events to PagerDuty; for that, use the [Events API](../../docs/events-API-v2/01-Overview.md).

### What and Where
This API is designed around RESTful principles. It's based on managing [resources](../../docs/REST-API/10-Resource-Schemas.md) via the HTTP methods `GET`, `POST`, `PUT`, and `DELETE`.

All requests to the REST API are made to the same host:

```
api.pagerduty.com
```

Based on the [authentication](../../docs/REST-API/02-Authentication.md) that you provide, you'll receive data for the associated account.

### How
Only JSON is supported in the REST API. Other content types are currently not supported and may produce unpredictable results. Request bodies should supply JSON data and responses will come back as JSON.

Consult the topics on the left to learn more about the conventions of the REST API.

If you're comfortable with the layout and structure of the API, or want to get started right away, head over to the [API Reference](https://api-reference.pagerduty.com/). The reference documentation contains a comprehensive set of REST API [endpoints](../../docs/REST-API/05-Endpoints.md), parameters, and responses. You'll be able to find the resources you need and try making requests right from your browser!

### HTTP Request Headers
The following headers should be set as applicable when making HTTP requests to the REST API:

* `Accept`: to optionally specify a different API version than the version of the API token; see [Versioning](../../docs/REST-API/03-Versioning.md).
* `Authorization`: required for all requests; see [Authentication](../../docs/REST-API/02-Authentication.md).
* `Content-Type`: required when making a `POST` or `PUT` request. The MIME media type should be `application/json`.
* `From`: the email address of the user to record as having taken the action. Should be used when [creating a user](https://api-reference.pagerduty.com/#!/Users/post_users) or when performing [Incident Creation](../../docs/REST-API/15-Incident-Creation-API.md) in the REST API.

### TLS
Connecting to the PagerDuty REST API requires using [TLS (Transport Layer Security)](https://en.wikipedia.org/wiki/Transport_Layer_Security).

Currently, our REST API supports TLS protocol versions 1.2.

Our server certificate is signed by DigiCert. Client systems will need to have the **DigiCert Global Root CA** certificate in their local trust store (this is often already the case). CA certificates, if needed, can be [obtained from DigiCert](https://www.digicert.com/digicert-root-certificates.htm).
