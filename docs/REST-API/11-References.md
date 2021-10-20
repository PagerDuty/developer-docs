---
tags: [rest-api]
---

# Resource References

A key component of the REST API is the use of `references` to represent a pointer to a resource. A reference consists of five fields:
- `id` (required, readOnly)
- `type` (required)
- `summary` (readOnly)
- `self` (readOnly)
- `html_url` (readOnly)

References are used anywhere that the identification of an object is important, but the object itself cannot be modified. In this regard, you can think of a reference like an alias — a different object type that represents a resource. They also provide an essential subset of information about the resource that tells a client how to uniquely identify it, provides some information about what it represents, and offers locations where more information can be found about it.

References will be used to represent relationships between resources. If a resource has a relationship with other types of entities, they may be displayed as part of the resource's schema and contain references to the resources of that type that make up the other end of the relationship. Modifying the reference or references in this field — identified only by `id` and `type` (see below) — will update the relationship between resources. Modifying any other fields of the reference has no effect on the resource it represents.

### `id` and `type`

Every PagerDuty resource can be uniquely identified by the combination of its `id` and `type`, ignoring the distinction between a given `type` and the corresponding `type_reference`. The `type` field indicates what kind of resource it is, which is [equivalent to specify its schema](../../docs/REST-API/10-Resource-Schemas.md)

### `summary`

The summary field contains a string that provides a brief, human-readable summary of what this particular resource instance represents. This is server-generated and cannot be specified by the client. It should be used whenever naming a resource in a client, such as in a title, list, or link.

### `self` and `html_url`

These fields provide the full [API `show` URL](../../docs/REST-API/05-Endpoints.md#resourcesid-show) at which the resource can be accessed in full form, and the full URL at which the resource can be viewed and manipulated in the PagerDuty web application, if applicable.

If the resource cannot be accessed directly in the API or in the web application, one or both of these fields may be `null`.

