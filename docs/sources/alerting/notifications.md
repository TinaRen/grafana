+++
title = "Alerting Notifications"
description = "Alerting Notifications Guide"
keywords = ["Grafana", "alerting", "guide", "notifications"]
type = "docs"
[menu.docs]
name = "Notifications"
parent = "alerting"
weight = 2
+++


# Alert Notifications

{{< imgbox max-width="40%" img="/img/docs/v4/alert_notifications_menu.png" caption="Alerting notifications" >}}

> Alerting is only available in Grafana v4.0 and above.

When an alert changes state it sends out notifications. Each alert rule can have
multiple notifications. But in order to add a notification to an alert rule you first need
to add and configure a notification channel. This is done from the Alerting/Notifications page.

## Notification Setup

On the notifications list page hit the `New Notification` button to go to the page where you
can configure and set up a new notification.

You specify name, type, and type-specific options. You can also test the notification to make
sure the channel credentials and delivery settings are working.

### Send on all alerts

When checked this option will make this notification channel used for all alert rules, existing and new.
Default notification channels are sent in addition to any channels selected directly on an alert rule.

## Rule messages and notification routing

Notification channels are shared objects. The incident-specific text lives on the alert rule itself:
open the graph panel's **Alert** tab, select the **Notifications** sub-tab, and enter a **Message** for
that rule. Use the message for runbook links, ownership details, dashboards to inspect, or remediation
steps.

Example message:

```text
High checkout latency.
Runbook: https://example.com/runbooks/checkout-latency
Check the load balancer and webfront error rate before scaling workers.
```

The message is sent with the alert notification, but each notification type formats it differently.
For example, webhook notifications include a `message` property in the JSON body when the alert rule
message is not empty, and Slack notifications append the rule message to the Slack attachment text
when the rule is not returning to `OK`. Do not put secrets in alert rule messages; they can be sent to
every default notification channel and every channel selected on the rule.

The **Send Test** button on a notification channel sends a synthetic test alert. Use it to verify the
channel configuration, but not to validate a specific alert rule's query, conditions, or rule message.
Use the alert rule **Test Rule** action for condition troubleshooting.

## Supported notification types

Grafana ships with a set of notification types. More will be added in future releases.

### Email

To enable email notification you have to set up [SMTP settings](/installation/configuration/#smtp)
in the Grafana config. Email notification will upload an image of the alert graph to an
external image destination if available or fallback on attaching the image in the email.

### Slack

{{< imgbox max-width="40%" img="/img/docs/v4/slack_notification.png" caption="Alerting Slack Notification" >}}

To set up slack you need to configure an incoming webhook url at slack. You can follow their guide for how
to do that https://api.slack.com/incoming-webhooks If you want to include screenshots of the firing alerts
in the slack messages you have to configure the [external image destination](#external-image-store) in Grafana.

Setting | Description
---------- | -----------
Recipient | allows you to override the slack recipient.
Mention | make it possible to include a mention in the slack notification sent by Grafana. Ex @here or @channel

### Webhook

The webhook notification is a simple way to send information about a state change over HTTP to a custom endpoint.
Using this notification you could integrate Grafana into any system you choose, by yourself.

Example json body:
```json
{
  "title": "My alert",
  "ruleId": 1,
  "ruleName": "Load peaking!",
  "ruleUrl": "http://url.to.grafana/db/dashboard/my_dashboard?panelId=2",
  "state": "Alerting",
  "imageUrl": "http://s3.image.url",
  "message": "Load is peaking. Make sure the traffic is real and spin up more webfronts",
  "evalMatches": [
    {
      "metric": "requests",
      "tags": {},
      "value": 122
    }
  ]
}
```

### PagerDuty

To set up PagerDuty, all you have to do is to provide an api key.

Setting | Description
---------- | -----------
Integration Key | Integration key for pagerduty.
Auto resolve incidents | Resolve incidents in pagerduty once the alert goes back to ok


# Enable images in notifications {#external-image-store}

Grafana can render the panel associated with the alert rule and include that in the notification. Some types
of notifications require that this image be publicly accessible (Slack for example). In order to support
images in notifications like Slack Grafana can upload the image to an image store. It currently supports
Amazon S3 for this and Webdav. So to set that up you need to configure the
[external image uploader](/installation/configuration/#external-image-storage) in your grafana-server ini
config file.

This is an optional requirement, you can get slack and email notifications without setting this up.
Each notification channel has an **Include image** option. If at least one channel used by an alert rule
has this option enabled, Grafana renders the panel image before sending notifications. When an external
image store is configured, channels receive a public image URL. Without an external image store, channels
that require a public URL will still send the notification, but without a hosted image URL; email can use
the rendered image as an attachment.

# Configure the link back to Grafana from alert notifications

All alert notifications contain a link back to the triggered alert in the Grafana instance.
This URL is based on the [root_url](/installation/configuration/#root_url) setting in Grafana.
Set `root_url` to the same public URL that users open in their browser, especially when Grafana runs
behind a reverse proxy or on a sub path.


