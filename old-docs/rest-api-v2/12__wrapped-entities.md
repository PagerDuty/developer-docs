---
title: "Wrapped Entities"
excerpt: ""
---
All resources represented in the REST API are *wrapped* with a root key in API requests and responses. This root key is either the *singular* name of the resource and has an <Link to="/docs/rest-api-v2/types/#object">object</Link> value, or it's the *pluralized* name of a collection of resources and has an <Link to="/docs/rest-api-v2/types/#array">array</Link>.

When submitting a request body to a <Link to="/docs/rest-api-v2/endpoints#resources-create">create</Link> or an <Link to="/docs/rest-api-v2/endpoints#resourcesid-update-single">update</Link> endpoint, the data you're providing about the resource needs to be wrapped in this same manner. Responses to any type of <Link to="/docs/rest-api-v2/endpoints/">endpoint</Link> will reflect it as well.
