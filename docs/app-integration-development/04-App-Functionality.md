---
tags: [app-integration-development]
---

# App Functionality

Once you have [registered your app](../../docs/app-integration-development/03-Register-an-App.md), add functionality to make it useful.

### PagerDuty App Functionality


| App Functionality   | Description|
|:--------------------|:----------------------------------------|
| [Events Integration](../../docs/app-integration-development/06-Events-Integration.md) |  Send machine events **from** your tool **to** PagerDuty over our asynchronous [Events API](../../docs/events-API-v2/01-Overview.md). This is the best way for monitoring tools to connect with PagerDuty in order to trigger incidents. |
| [Simple Install Flow](../../docs/app-integration-development/06-Events-Integration.md#simple-install-flow-optional-but-recommended)|  If you're using the Events API, your users can quickly connect to PagerDuty directly from your tool with the Simple Install Flow! [See Demo](https://acme.pagerduty.dev) |
| [Classic User OAuth](../../docs/app-integration-development/08-Classic-User-OAuth.md)|  Connect to our [REST API](https://api-reference.pagerduty.com/) as a PagerDuty user (not full account access) to administer PagerDuty or get data (create an on-call schedule, get a list of team members, etc). |

### More Ways To Connect With PagerDuty

These platform features are not available through PagerDuty's app configuration framework (yet!) but are still available for your app to use.

|    |    |
|--- |--- |
|REST API Token|If you need account-level access to a PagerDuty account, you can [request a token from PagerDuty admin users](https://support.pagerduty.com/docs/generating-api-keys#section-rest-api-keys).|
|Webhooks|PagerDuty will send messages **to** your tool when certain activities occur. For example: when an incident is triggered, when a note is added to an incident, etc). <br/><br/>There are 2 ways to create a webhook extension: <ul><li>Over the REST API with the [/extensions endpoint](https://api-reference.pagerduty.com/#!/Extensions/post_extensions) </li><li>[In the PagerDuty web interface](https://support.pagerduty.com/docs/webhooks)</li></ul>|


### Which app functionality is right for me?

Use our [API Picker](../../docs/app-integration-development/05-API-Picker.md) resource to choose the app functionality that's right for your needs.

### Adding functionality to your app

1. Go to the configuration page for your app. [See how to register or edit an app](../../docs/app-integration-development/03-Register-an-App.md)
2. Scroll down the the **Functionality** section of the page and click the **Add** button on the card which represents the functionality you would like to add. See the above section for more information on app functionality.
3. On the next page, you'll be asked for more information to configure that app functionality.
4. Click **Save** to save the configuration and add that functionality to your app.
5. Click the **Manage** button to make edits or remove functionality from an app
![Screenshot of app functionality management](../../assets/images/app_functionality.png)

### Removing functionality from your app

<!-- theme:warning -->
> This action cannot be undone. Be sure that app functionality is not needed before you remove it.

1. Go to your app's configuration page. [See how to register or edit an app](../../docs/app-integration-development/03-Register-an-App.md)
2. Scroll down the the **Functionality** section of the page and click the **Manage** button on the card which represents the functionality you would like to remove.
3. Scroll to the **Danger Zone** at the bottom of the page and click **Delete**
![Screenshot of delete functionality section](../../assets/images/delete_functionality.png)
4. Confirm that you would like to remove app functionality
![Screenshot of delete confirmation modal](../../assets/images/delete_confirm.png)

