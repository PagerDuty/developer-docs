---
tags: [rest-api]
---

# Filtering

Each endpoint will have specific parameters that constrain the result set. These may take one of two forms:

### Scalar filters

These parameters have a single key and a single value. The behavior of a scalar filter may vary based on the parameter and endpoint; consult the reference documentation for more details.

### Set filters

These parameters contain a set of possible values that can be matched against in order to satisfy the query. The name of the parameter will be pluralized and end in two square brackets (`[]`). Multiple values should be specified in the query string by repeating the same query parameter with a different value.

### Composition

The result set displayed in the response will be the intersection of all given filter criteria; that is, only resources satisfying all of the given parameters are shown.

Set filters (see above) only require one possible value to be matched within that filter, but at least one match must be found in each distinct set filter in order to match the query.

