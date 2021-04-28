---
tags: [rest-api]
---

# Versioning

The version of the API that responds to a given request is based on the authentication token provided as part of the request, but can be overridden with a header.

## Token-based versioning

By default, all new authentication tokens will be set to provide the latest version of the API. Existing authentication tokens will request the final released version of the API that was available at the time of their creation.

## Header-based versioning

If you're writing code that's designed to work with multiple authentication tokens or with a specific version of the API, the best way to ensure you get the right version is with [PagerDuty's `Accept` header](http://www.iana.org/assignments/media-types/application/vnd.pagerduty+json):

```http
Accept: application/vnd.pagerduty+json;version=2
```

Any request made using this header will respond with the specified API version, regardless of the authentication token used.

### Samples

Here's how you can set the `Accept` header in some popular HTTP libraries:

<!--
type: tab
title: Ruby
-->

```ruby
req = Net::HTTP::Get.new('https://api.pagerduty.com/users')
api_version = '2'
req.add_field('Accept', 'application/vnd.pagerduty+json;version=#{api_version}')
```

<!--
type: tab
title: JavaScript (Client-Side)
-->

```javascript
var api_version = "2";
$.ajax({
  url: "https://api.pagerduty.com/users",
  type: "GET",
  headers: {"Accept": "application/vnd.pagerduty+json;version=" + api_version}
});
```

<!--
type: tab
title: Python
-->

```python
api_version = '2'
requests.get('https://api.pagerduty.com/users', headers={'Accept': 'application/vnd.pagerduty+json;version=' + api_version})
```


<!-- type: tab-end -->
