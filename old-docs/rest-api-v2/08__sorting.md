---
title: "Sorting"
excerpt: ""
---
Sorting refers to having the server sort the results by a field or combination of fields and returning the result set to the client in sorted order. It applies only to <Link to="docs/rest-api-v2/endpoints/">index endpoints</Link>.

This is especially important when <Link to="docs/rest-api-v2/pagination/">paginating</Link>, as not all results may be available in the first response.

Sorting is specified by the client using a `sort_by` parameter, which uses comma-separated values. Multiple sort values are separated by commas.
Each sort value consists of a field name that is eligible for sorting and an optional direction specifier. The direction specifier can be `:asc` or `:desc` and is appended to the name of the field. If no direction is specified, an ascending sort is assumed. If multiple values are provided, results will be sorted by the first value, using the second and subsequent values only to break ties with the previous value.

All <Link to="docs/rest-api-v2/endpoints/">index endpoints</Link> endpoints have a consistent default sort. Resources will be returned in the same order across requests even if no `sort_by` parameter is specified.
