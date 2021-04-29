---
tags: [rest-api]
---

# Resource Schemas

All resources in the REST API have a single, canonical schema uniquely identified by the resource's `type`. Simply put, the same set of fields will always be present for a resource of a given `type`.

Every field within the schema adheres to a single [primitive types.](../../docs/rest-api/06-Types.md)

In some cases, a resource may not contain a value for a field. The field will still be present in the resource and the value will be `null`. If the field can contain multiple values but the resource does not have any values for that field, the field will still be present and the value will be an empty array (`[]`).

To promote forward compatibility, clients should expect and ignore any extra fields in the schema that they do not recognize. This also enables the use of generic schemas for specific types.

### Type specificity

While the value of a `type` field always provides the most specific information available about a resource's schema, there may be a more generic schema available that can also be used to understand the resource. More specific schemas are purely additive to the corresponding generic schema.

For instance, while there may be a schema corresponding to `cat_animal`, any `cat_animal` resource will also conform to the `animal` schema. A client concerned about `animal`s in general and not the specifics of `dog_animal` vs. `cat_animal` vs. `mouse_animal` can interpret every response from the `/animals` endpoint, or any `type` ending in `_animal` across the API as having a valid `animal` schema.

Similarly, *any* resource always conforms to a valid `reference` schema. If the client is only interested in displaying a list of resources by their [using a filter](../../docs/rest-api/11-References.md#summary), it does not need to know anything about the `type` of those resources as they can all be interpreted as valid `reference`s.
