---
tags: [rest-api]
---

# Rate Limiting

## What are our limits?
Our rate limits via the Events v2 API, there is a limit of approximately 120 calls/minute per integration key. The limit is calculated over a 60 second window looking back from the current time.

Our rate limits on our REST API is 900 events/min across an entire organization.

## The difference between an API call limit and an API rate limit
  A **call limit** is the number of times you can invoke an API within a certain time period. It is often a limit imposed purely by a SaaS vendor's business choice (pricing/packaging) rather than for technical reasons of capacity or fairness.

 A **rate limit** is one imposed by a vendor for reasons of fairness, so that one customer doesn’t overwhelm the vendor’s infrastructure with events or deny service to other customers.


## What happens when a customer reaches the Events API rate limit?
The response to the API call will say `429 - Request Limit Exceeded` and PagerDuty will not ingest the event.

## What are possible workarounds to the Events API Rate Limit?
 - *Retrying requests* - Simply retrying the request a moment later can help alleviate rate limiting. We suggest retrying 3 times with each request being 30 seconds apart.
- *Integrations* / *Service* - The rate limit applies per integration key.  An easy solution is to increase the number of integration keys used on a service (fan out).
- *Services* - Another solution involves splitting a service into two or more.  This includes the added benefit of being more specific with service definitions.
- In rare cases, engineering can raise the rate limit for Enterprise customers.  This is rarely done, but please reach out to your account manager for more details.
 
