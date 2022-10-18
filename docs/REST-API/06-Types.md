---
tags: [rest-api]
---

# Types

<!-- theme:warning -->
> #### The null value
> Unless otherwise noted, any field in the PagerDuty API can contain the value `null`. This represents the absence of a value for that field. For example, if a resource does not have a `description`, it may return `"description": null`.
> Fields that are `required` will never be `null`.

### ID

IDs are represented in the PagerDuty API as strings.

All IDs will be contained within an `id` key.

These IDs are not globally unique, but will be unique across a given endpoint. For example, a schedule and a service may both have the id `PSWK4Q7`, but no two schedules will have the same id.

<!-- theme:info -->
> An `id` field is never `null`.

### UUID

UUID fields are designated with the presence of `uuid` in their key name, and contain string ID values.

UUIDs can be considered unique across PagerDuty. That is, no two resources of any type will share the same UUID.

<!-- theme:info -->
> A `uuid` field is never `null`.

### String

A standard JSON string. Strings in the REST API use the UTF-8 character set.

### Integer

Integer types are a number without a fractional or decimal component.

### Boolean

A boolean has only two possible states: true and false.

In responses, booleans are always represented by native JSON booleans — `true` or `false` without quotes.

In query strings, booleans can also be represented by string values. `"1"` and `"true"` are acceptable to represent a truthy value, and `"0"` and `"false"` are acceptable to represent a falsy value.

<!-- theme:info -->
> A boolean field is never `null`.

### Array

A standard JSON array. Arrays may contain any number of values, from `0` to `n`, unless otherwise specified.

The primitive data type of the values within an array will always be consistent. That is, while a single array may contain objects representing both Users and Schedules, it will never contain both [objects](#object) and [integers](#integer).

<!-- theme:info -->
> An array field is never `null`.
> If there are no values for the associated field, the value will be an empty array (`[]`).

### Object

A standard JSON object. Objects consist of string keys paired with values that may be of any type.

### DateTime

All dates and times must be represented in the [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) date/time format. The time element is optional.

Example dates in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format:

| Date and Time | ISO 8601 Representation |
|:--------------|:------------------------|
| May 6th, 2011 at 5pm UTC | 2011-05-06T17:00Z |
| May 6th, 2011 at 3:30am PDT | 2011-05-06T03:30-07 |
| May 6th, 2011 at midnight (time is optional) | 2011-05-06 |


### Time Zone

A string representing a recognized time zone. Time zones are specified in the format of the [IANA time zone database](http://www.iana.org/time-zones). [See Wikipedia for a list of common time zone identifiers](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List).

### URL

A URL is a string that conforms to the [RFC 3986 syntax](https://tools.ietf.org/html/rfc3986). URLs are fully-qualified URIs that [provide a means of locating the resource](https://tools.ietf.org/html/rfc3986#section-1.1.3), and will never have only a subset of the URL, such as just a hostname or path.

A number of URL validation libraries are available; any that conform to the spec should be able to correctly validate a URL field.

