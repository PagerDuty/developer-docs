---
tags: [rest-api]
---

# Errors

If an API request is successful, PagerDuty will respond to it with HTTP status code 2XX and the resources that match the query.

If the API request is unsuccessful, PagerDuty will respond with a corresponding HTTP error status code and the response body will contain PagerDuty error details.

The response body (with error details) will have the following properties:

### Error Response

```json
{
    "error": {
        "message": "Error Message String",
        "code": 2001,
        "errors": [
            "human-readable error details",
            "more details"
        ]
    }
}
```

### HTTP Responses

The HTTP response can be used to determine if the request was successful, and if not, whether the request should be retried.

| Code | HTTP Definition | Description |
|:--------------|:------------------|:------|
| 200 | OK | The request was successful |
| 201 | Created | The request was successful and a new resource was created. |
| 204 | No Content | The request was successful. No content is returned. |
| 400 | Bad Request | Caller provided invalid arguments. Please review the response for error details. Retrying with the same arguments will *not* work. |
| 401 | Unauthorized | Caller did not supply credentials or did not provide the correct credentials. See [Authentication](../../docs/REST-API/02-Authentication.md). |
| 402 | Payment Required | The PagerDuty account does not have access to one or more abilities needed to complete this request. Use the [Account Abilities API](https://api-reference.pagerduty.com/#!/Abilities/get_abilities) to determine which abilities the account supports. |
| 403 | Forbidden | Caller is not authorized to view the requested resource. |
| 404 | Not Found | The requested resource was not found. |
| 408 | Request Timeout | The request took too long to process. Please try again. |
| 429 | Too Many Requests | The caller is making requests too frequently. Wait a moment before making further requests and make them at a reduced rate. See [Rate Limiting](../../docs/REST-API/04-Rate-Limiting.md). |
| 500 | Internal Server Error | Internal error occurred while processing request, likely due to transient issues. Please try again. **Tip:** We recommend adding retry logic to your client to automatically try your request again later. |

### PagerDuty Error Codes

In the case of an error, the PagerDuty error code can give further details on the nature of the error.

This list of error codes is not exhaustive, but is representative of the kinds of errors you will see in API responses.

| Code | Message |
|:-----|:--------|
| 1001 | Incident Already Resolved |
| 1002 | Incident Already Acknowledged |
| 1003 | Invalid Status Provided |
| 1004 | Invalid ID Provided |
| 1005 | Data Updated Since Last Request |
| 1006 | Cannot Escalate |
| 1007 | Assigned To User Not Found |
| 1008 | Requester User Not Found |
| 1009 | Error Parsing before_time Parameter |
| 1010 | Before Time or Before Incident Number Not Found |
| 1011 | Cannot process reassign action |
| 1012 | Incident is not Acknowledged |
| 1013 | Snooze Duration is Invalid |
| 1014 | Escalation Policy Not Found |
| 1015 | Escalation Policy Has No User On Call |
| 1018 | Requester not authorized to perform action |
| 1019 | Incident Already Merged |
| 1020 | Invalid Priority |
| 2000 | An internal error has occurred. PagerDuty team has been notified. |
| 2001 | <ul><li>Cannot Delete Only Rule For Escalation Policy</li><li>Cannot create more than 50 users at one time</li><li>Escalation Policy has already been taken</li><li>Incident could not be created</li><li>Invalid Input Provided</li><li>Users must be an array</li></ul> |
| 2002 | Arguments Caused Error |
| 2003 | Missing Arguments |
| 2004 | Invalid time values in 'since' and/or 'until' parameters |
| 2005 | Invalid Query Date Range |
| 2006 | Authentication failed |
| 2007 | Account Not Found |
| 2008 | Account Locked |
| 2009 | Only HTTPS Allowed For This Call |
| 2010 | Access Denied |
| 2011 | You must specify a requester_id to perform this action |
| 2012 | Your account is expired and cannot use the API |
| 2013 | User not found |
| 2015 | Invalid operation |
| 2016 | The request took too long to process |
| 2100 | Not Found |
| 3001 | Invalid Schedule |
| 3004 | <ul><li>Schedule Not Found</li><li>Unknown Schedule</li></ul> |
| 4004 | <ul><li>Invalid Override</li><li>Override Not Found</li></ul> |
| 5001 | <ul><li>Affiliate Partner Not Found</li><li>Color Not Found</li><li> Escalation Policy Not Found</li><li>Escalation Rule Not Found</li><li>Incident Not Found</li><li>Log Entry Not Found</li><li>Maintenance Window Not Found</li><li>User Next On-call Not Found</li><li>Vendor Not Found</li><li>Webhook Not Found</li></ul> |
| 5002 | <ul><li>Service Not Found</li><li>Simple Log Entry Limit Reached</li></ul> |
| 5003 | Cannot cancel a maintenance window from the past |
| 6001 | Team Not Found |
| 6002 | Team has Incidents but Reassignment Team is not valid |
| 6003 | Team has Incidents but Reassignment Team is not found |
| 6004 | Team could not be deleted because there are too many incidents, please contact support. |
| 6006 | Team has existing associations |
| 6007 | The request failed to complete |
| 7001 | saml is not enabled |
| 7002 | SAMLResponse is not provided |
| 7003 | failed to parse SAMLResponse |
| 7004 | SAMLResponse is invalid |
| 7005 | unable to find user by name_id |
| 7006 | invalid name_id found in SAMLResponse |
| 7007 | unexpected validation error |
| 8001 | <ul><li>google auth is not enabled</li><li>pre_auth.authorizable? returned false</li></ul> |
| 8002 | redirect_uri is invalid |
| 8004 | unable to find account |
| 8005 | unable to find user by name_id |
| 8006 | user has denied google auth |
| 8007 | unable to log in using google auth |
| 8008 | user has tried to access an invalid host domain |

