# Webhook Notifications

```json
{
"reason":"dissatisfactoryUserFeedback","timestamp":1471354044616,"appId":"xxxxx","triggerType":"interval",
"actualValue":5,"threshold":0.2,"fractionalValue":0.3333333333333333,
"period":{"start":1458125913802,"end":1458129519080},
"url":"https:\/\/dashboard.callstats.io\/search?feedback12=1",
"text":"33% of the conferences had dissatisfactory feedback in the last hour i.e., 5 of 15 total conferences with appID xxxxx. The current trigger threshold is 20%."
}
```

```json
{
"reason":"failedConferences","timestamp":1471354046133,"appId":"xxxxx","triggerType":"interval",
"actualValue":7,"threshold":0.1,"fractionalValue":0.11290322580645161,
"period":{"start":1458125913802,"end":1458129519080},
"url":"https:\/\/dashboard.callstats.io\/search?failureGroup=totalFailure",
"text":"11% of the conferences failed in the last hour i.e., 7 of 62 total conferences with appID xxxxx. The current trigger threshold is 10%."
}
```
### JSON encoding

callstats.io sends periodic notifications (JSON encoded messages) to webhook URLs. The notification messages are sent when either a single event occurs (__individual notification__) or a metric exceeds a threshold value in a defined interval (__interval notification__), in the message these are identified by the `triggerType`. Details of the notification message structure are below: 

  Keys | Description
----------- | ----------
`reason`  | camelCase string identifying the notification.
`timestamp` | Unix timestamp in seconds when the message was sent.
`appId` | The application identifier corresponding to the message.
`triggerType` | One of `interval` or `individual`.
`actualValue` | The actual measured value of the metric when the message was triggered.
`threshold` | The threshold over which the notification is triggered. For individual metrics, this value is 1.
`fractionalValue` | The ratio of the actualValue of a certain metric and the total measurement points of that metric. e.g., failed/total conferences.
`period` | Object containing start and end timestamps.
`url` | The search URL that will find the conference on callstats.io dashboard.
`text` | Notification in english text.

### Keys for the Notification reason

  Keys | Description
----------- | ----------
failedConferences | ratio of failed conferences exceeded threshold
dissatisfactoryUserFeedback | ratio of user feedback with dissatisfactory rating exceeded threshold
