---
tags: [rest-api]
---

# Types

<!-- theme:warning -->

> #### The null value
>
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

| Date and Time                                | ISO 8601 Representation |
| :------------------------------------------- | :---------------------- |
| May 6th, 2011 at 5pm UTC                     | 2011-05-06T17:00Z       |
| May 6th, 2011 at 3:30am PDT                  | 2011-05-06T03:30-07     |
| May 6th, 2011 at midnight (time is optional) | 2011-05-06              |

### Time Zone

A string of a recognized time zone identifier. The acceptable identifiers and
their name/description are shown below.

| Time Zone Identifier           | Name/Description             |
| :----------------------------- | :--------------------------- |
| Africa/Algiers                 | West Central Africa          |
| Africa/Cairo                   | Cairo                        |
| Africa/Casablanca              | Casablanca                   |
| Africa/Harare                  | Harare                       |
| Africa/Johannesburg            | Pretoria                     |
| Africa/Monrovia                | Monrovia                     |
| Africa/Nairobi                 | Nairobi                      |
| America/Argentina/Buenos_Aires | Buenos Aires                 |
| America/Bogota                 | Bogota                       |
| America/Caracas                | Caracas                      |
| America/Chicago                | Central Time (US & Canada)   |
| America/Chihuahua              | Chihuahua                    |
| America/Denver                 | Mountain Time (US & Canada)  |
| America/Godthab                | Greenland                    |
| America/Guatemala              | Central America              |
| America/Guyana                 | Georgetown                   |
| America/Halifax                | Atlantic Time (Canada)       |
| America/Indiana/Indianapolis   | Indiana (East)               |
| America/Juneau                 | Alaska                       |
| America/La_Paz                 | La Paz                       |
| America/Lima                   | Lima                         |
| America/Lima                   | Quito                        |
| America/Los_Angeles            | Pacific Time (US & Canada)   |
| America/Mazatlan               | Mazatlan                     |
| America/Mexico_City            | Guadalajara                  |
| America/Mexico_City            | Mexico City                  |
| America/Monterrey              | Monterrey                    |
| America/Montevideo             | Montevideo                   |
| America/New_York               | Eastern Time (US & Canada)   |
| America/Phoenix                | Arizona                      |
| America/Puerto_Rico            | Puerto Rico                  |
| America/Regina                 | Saskatchewan                 |
| America/Santiago               | Santiago                     |
| America/Sao_Paulo              | Brasilia                     |
| America/St_Johns               | Newfoundland                 |
| America/Tijuana                | Tijuana                      |
| Asia/Almaty                    | Almaty                       |
| Asia/Baghdad                   | Baghdad                      |
| Asia/Baku                      | Baku                         |
| Asia/Bangkok                   | Bangkok                      |
| Asia/Bangkok                   | Hanoi                        |
| Asia/Chongqing                 | Chongqing                    |
| Asia/Colombo                   | Sri Jayawardenepura          |
| Asia/Dhaka                     | Astana                       |
| Asia/Dhaka                     | Dhaka                        |
| Asia/Hong_Kong                 | Hong Kong                    |
| Asia/Irkutsk                   | Irkutsk                      |
| Asia/Jakarta                   | Jakarta                      |
| Asia/Jerusalem                 | Jerusalem                    |
| Asia/Kabul                     | Kabul                        |
| Asia/Kamchatka                 | Kamchatka                    |
| Asia/Karachi                   | Islamabad                    |
| Asia/Karachi                   | Karachi                      |
| Asia/Kathmandu                 | Kathmandu                    |
| Asia/Kolkata                   | Chennai                      |
| Asia/Kolkata                   | Kolkata                      |
| Asia/Kolkata                   | Mumbai                       |
| Asia/Kolkata                   | New Delhi                    |
| Asia/Krasnoyarsk               | Krasnoyarsk                  |
| Asia/Kuala_Lumpur              | Kuala Lumpur                 |
| Asia/Kuwait                    | Kuwait                       |
| Asia/Magadan                   | Magadan                      |
| Asia/Muscat                    | Abu Dhabi                    |
| Asia/Muscat                    | Muscat                       |
| Asia/Novosibirsk               | Novosibirsk                  |
| Asia/Rangoon                   | Rangoon                      |
| Asia/Riyadh                    | Riyadh                       |
| Asia/Seoul                     | Seoul                        |
| Asia/Shanghai                  | Beijing                      |
| Asia/Singapore                 | Singapore                    |
| Asia/Srednekolymsk             | Srednekolymsk                |
| Asia/Taipei                    | Taipei                       |
| Asia/Tashkent                  | Tashkent                     |
| Asia/Tbilisi                   | Tbilisi                      |
| Asia/Tehran                    | Tehran                       |
| Asia/Tokyo                     | Osaka                        |
| Asia/Tokyo                     | Sapporo                      |
| Asia/Tokyo                     | Tokyo                        |
| Asia/Ulaanbaatar               | Ulaanbaatar                  |
| Asia/Urumqi                    | Urumqi                       |
| Asia/Vladivostok               | Vladivostok                  |
| Asia/Yakutsk                   | Yakutsk                      |
| Asia/Yekaterinburg             | Ekaterinburg                 |
| Asia/Yerevan                   | Yerevan                      |
| Atlantic/Azores                | Azores                       |
| Atlantic/Cape_Verde            | Cape Verde Is.               |
| Atlantic/South_Georgia         | Mid-Atlantic                 |
| Australia/Adelaide             | Adelaide                     |
| Australia/Brisbane             | Brisbane                     |
| Australia/Darwin               | Darwin                       |
| Australia/Hobart               | Hobart                       |
| Australia/Melbourne            | Canberra                     |
| Australia/Melbourne            | Melbourne                    |
| Australia/Perth                | Perth                        |
| Australia/Sydney               | Sydney                       |
| Etc/GMT+12                     | International Date Line West |
| Etc/UTC                        | UTC                          |
| Europe/Amsterdam               | Amsterdam                    |
| Europe/Athens                  | Athens                       |
| Europe/Belgrade                | Belgrade                     |
| Europe/Berlin                  | Berlin                       |
| Europe/Bratislava              | Bratislava                   |
| Europe/Brussels                | Brussels                     |
| Europe/Bucharest               | Bucharest                    |
| Europe/Budapest                | Budapest                     |
| Europe/Copenhagen              | Copenhagen                   |
| Europe/Dublin                  | Dublin                       |
| Europe/Helsinki                | Helsinki                     |
| Europe/Istanbul                | Istanbul                     |
| Europe/Kaliningrad             | Kaliningrad                  |
| Europe/Kiev                    | Kyiv                         |
| Europe/Lisbon                  | Lisbon                       |
| Europe/Ljubljana               | Ljubljana                    |
| Europe/London                  | Edinburgh                    |
| Europe/London                  | London                       |
| Europe/Madrid                  | Madrid                       |
| Europe/Minsk                   | Minsk                        |
| Europe/Moscow                  | Moscow                       |
| Europe/Moscow                  | St. Petersburg               |
| Europe/Paris                   | Paris                        |
| Europe/Prague                  | Prague                       |
| Europe/Riga                    | Riga                         |
| Europe/Rome                    | Rome                         |
| Europe/Samara                  | Samara                       |
| Europe/Sarajevo                | Sarajevo                     |
| Europe/Skopje                  | Skopje                       |
| Europe/Sofia                   | Sofia                        |
| Europe/Stockholm               | Stockholm                    |
| Europe/Tallinn                 | Tallinn                      |
| Europe/Vienna                  | Vienna                       |
| Europe/Vilnius                 | Vilnius                      |
| Europe/Volgograd               | Volgograd                    |
| Europe/Warsaw                  | Warsaw                       |
| Europe/Zagreb                  | Zagreb                       |
| Europe/Zurich                  | Bern                         |
| Europe/Zurich                  | Zurich                       |
| Pacific/Apia                   | Samoa                        |
| Pacific/Auckland               | Auckland                     |
| Pacific/Auckland               | Wellington                   |
| Pacific/Chatham                | Chatham Is.                  |
| Pacific/Fakaofo                | Tokelau Is.                  |
| Pacific/Fiji                   | Fiji                         |
| Pacific/Guadalcanal            | Solomon Is.                  |
| Pacific/Guam                   | Guam                         |
| Pacific/Honolulu               | Hawaii                       |
| Pacific/Majuro                 | Marshall Is.                 |
| Pacific/Midway                 | Midway Island                |
| Pacific/Noumea                 | New Caledonia                |
| Pacific/Pago_Pago              | American Samoa               |
| Pacific/Port_Moresby           | Port Moresby                 |
| Pacific/Tongatapu              | Nuku'alofa                   |

### URL

A URL is a string that conforms to the [RFC 3986 syntax](https://tools.ietf.org/html/rfc3986). URLs are fully-qualified URIs that [provide a means of locating the resource](https://tools.ietf.org/html/rfc3986#section-1.1.3), and will never have only a subset of the URL, such as just a hostname or path.

A number of URL validation libraries are available; any that conform to the spec should be able to correctly validate a URL field.
