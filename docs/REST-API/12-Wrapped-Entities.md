---
tags: [rest-api]
---

# Wrapped-Entities

All resources represented in the REST API are *wrapped* with a root key in API requests and responses. This root key is either the *singular* name of the resource and has an [object](../../docs/rest-api/06-Types.md#object) value, or it's the *pluralized* name of a collection of resources and has an [array](../../docs/rest-api/06-Types.md#array).

When submitting a request body to a [create](../../docs/rest-api/05-Endpoints.md#resource-create) or an [update](../../docs/rest-api/05-Endpoints.md#resourcesid-update-single) endpoint, the data you're providing about the resource needs to be wrapped in this same manner. Responses to any type of [endpoint](../../docs/rest-api/05-Endpoints.md) will reflect it as well.
