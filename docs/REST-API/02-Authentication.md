---
tags: [rest-api]
---

# Authentication

<!-- theme:warning -->
>All REST API requests must be made over HTTPS. Connections made using HTTP will be refused.

All REST API calls require authentication. In order to make successful requests to the REST API, you must provide a valid form of authorization.

### API Token Authentication
The PagerDuty REST API supports authenticating via an account or user API token. 

- Account API tokens have access to all of the data on an account, can either be granted read-only access or full access to read, write, update, and delete, and are tied to the account rather than the user that created them.  
- User API tokens are available for PagerDuty accounts with [Advanced Permissions](https://support.pagerduty.com/docs/advanced-permissions), and these user API tokens have access to all of the data that the associated user account has access to.

Only account administrators have the ability to [generate account API tokens](https://support.pagerduty.com/docs/using-the-api#section-generating-a-general-access-rest-api-key).

Tokens should be sent in the request as part of an `Authorization` header, using this format:

```
Authorization: Token token=y_NbAkKc66ryYTWUXYEu
```

Below is how to set the header in a few popular HTTP libraries. Note that this only sets the header; other code may be needed to create and process the request.

<!--
type: tab
title: JavaScript (Node.js)
-->

```javascript
/*
  This example uses PDJS for Node.js

  https://github.com/PagerDuty/pdjs
*/

import {api} from '@pagerduty/pdjs';

const pd = api({token: 'someToken1234567890'});

pd.get('/incidents')
  .then({data, resource, next} => console.log(data, resource, next))
  .catch(console.error);
```

<!--
type: tab
title: JavaScript (Client-side)
-->

```javascript
/*
  This example uses the PDJS library for client-side JavaScript.

  https://github.com/PagerDuty/pdjs
*/
PagerDuty.api({token: 'someToken1234567890', endpoint: '/incidents'})
  .then(response => console.log(response.data))
  .catch(console.error);
```

<!--
type: tab
title: Python
-->

```python
/*
  This example uses the "pagerduty" library for Python

  https://github.com/PagerDuty/python-pagerduty
*/

from pagerduty import RestApiV2Client
client = RestApiV2Client('your-token-here')
for user in client.iter_all('users'):
    print(user['id'], user['email'], user['name'])
```

<!--
type: tab
title: Go
-->

```go
authtoken := "your-access-token"
var opts pagerduty.ListUsersOptions
client := pagerduty.NewClient(authtoken)

if users, err := client.ListUsers(opts); err != nil {
  panic(err)
} else {
  for _, p := range users.Users {
    fmt.Println(p)
    fmt.Println()
  }
}
```

<!--
type: tab
title: Ruby
-->

```ruby
pagerduty_conn = Faraday.new(url: 'https://api.pagerduty.com') do |conn|
conn.headers['Authorization'] = "Token token=#{authorization_token}"
conn.headers['Accept'] = 'application/vnd.pagerduty+json;version=2'
conn.adapter :net_http
end

pagerduty_conn.get '/users'"
```

<!-- type: tab-end -->

Please note that this format must be followed precisely or you will receive a `401 Unauthorized` response.

<!-- theme:danger -->
> ### 401 Unauthorized
> Requests that cannot be authenticated will return a `401 Unauthorized` error response.
> If you are receiving a `401`, check:
> - is your `Authorization` header being sent?
> - is your `Authorization` header formatted properly?
> - are you using a valid, active API token?

