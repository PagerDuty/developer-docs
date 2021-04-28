---
tags: [rest-api]
---

# Pagination

When you retrieve a collection over the API, the results are returned in portions, also known as pages.

<!-- theme:info -->
> Currently, most endpoints use classic pagination. However, a growing list of [endpoints support cursor-based pagination.](#endpoints-supporting-cursor-based-pagination)

## Classic pagination

Many [index endpoints](../../docs/rest-api/05-Endpoints.md#resources-index) employ the classic pagination approach. Initially, only a certain number of results will be returned at a time. The default is 25. You can override this by passing a `limit` parameter to set the maximum number of results, but cannot exceed 100. Specifying a number for `offset` sets the starting point for the result set, allowing you to fetch subsequent resources that are not in the initial set of results.

Every response from endpoints employing this pagination approach contains additional fields, as seen in the example below:

### Pagination response fields
```json
{
  "limit": 25, // echoes the 'limit' value that was used
  "offset": 0, // echoes the 'offset' value that was used
  "more": true, // indicates if there are more resources available than were returned
  "total": 44 // the total number of resources matching the query
}
```

The `more` field will always be `true` or `false`, indicating whether there are more resources matching the query than present in the response (because of the request's `limit`). If `true`, you may want to make subsequent requests with a different `offset` until `more` is `false` in order to retrieve all of the resources.

The `total` field is `null` by default. This behavior will provide the fastest response times when the total number of records is not important. If you need to know how many records match the query, pass a `total=true` parameter as part of the request and this field will be populated.

<!-- theme:warning -->
> Maximum upper limit of classic pagination
> The REST API permits retrieving a maximum of 10000 records via pagination. That is to say, the sum of the `offset` and `limit` parameters cannot exceed 10000, or the REST API will respond with status 400.
> If the `total` parameter is greater than 10000, then not all records in the set can be retrieved. As an alternative, one should constrain the results to under 10000 by [using a filter](../../docs/rest-api/07-Filtering.md) and then adjust the filter to encompass more of the target set to be retrieved.

## Cursor-based pagination

Responses from PagerDuty APIs employing cursor-based pagination will include the following fields:

### Pagination response fields
```json
{
  "limit": 25, //  The minimum of the 'limit' parameter used in the request or the maximum request size of the API
  "next_cursor": "UExIMUhLVg==" // An opaque string used to fetch the next set of results or 'null' if no additional results are available
}
```

The next set of results can be obtained by providing the `next_cursor` value from the previous request in a `cursor` query parameter on the subsequent request. Excluding the `cursor` query parameter will return results from the beginning of the list.

When a response returns a `null` value for `next_cursor` then the end of the list has been reached.

### Endpoints supporting cursor-based pagination
* [/audit/records](https://developer.pagerduty.com/api-reference/reference/REST/openapiv3.json/paths/~1audit~1records/get)
