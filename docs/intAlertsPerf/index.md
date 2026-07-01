---
layout: default
title: Grafana Alerting & Automation Lab
---

# Grafana Alerting & Automation Lab

This lab walks you through three core alerting and automation scenarios using the Field Eng OTel environment. Complete each exercise in order. Each exercise has a reference ID that corresponds to a lab outcome.

---

## CO-006 — Alert Trigger & Escalation Policy Routing

In this exercise you will create an SLO with alert rules, that fires on a threshold breach and verify it routes to the correct team via a notification policy.

**Objectives**
- Create an SLO with a metric query
- Configure a notification policy that routes to a specific team contact point
- Confirm the alert fires and is received by the correct team

## Think about your team_name value, ensure its unique, normally doing something like ${yourname}team ensure's it's unique. e.g. samredteam.

### Step 1 - Create a new contact point
1. Navigate to "Alerts & IRM" -> Alerting.
2. Click "Manage contact points"
3. Create a new contact point.
4. Name it "{$yourname} integration"
5. Under "Integration" select "Grafana IRM"
6. Choose "New Integration"
7. Name the integration "{$Yourname}Integration"
8. Click "Save contact point"

### Create a notification policy
1. Head to "Alerts & IRM -> Alerting -> Notification configuration"
2. Name your policy "{$yourname}-notification-policy"
3. Set the contact point to your integration which you created previously
4. Create the policy
5. Create a route for your new policy "+ Add Route"
6. Set the label to "team_name", leave operator to "=", set the value to the teamname you used on your alerts for your SLO's, set the contact point to the integration which you created.
7. Click "Add Route"

### Create a webhook
1. head over to webhook.site
2. Copy your unique URL
3. Go to "Alerts & IRM -> IRM -> Integrations", choose "Outgoing Webhooks"
4. Name the webhook
5. Apply some labels (we'll just use them as a test to show that we can send values in the payload).
6. Paste your webhook url into the webhookURL field (You'll need to click the pencil first).
7. Click Create.

### Create an escalation chain
1. Head to the integration you created under "Alerts & IRM -> IRM -> Integrations"
2. Click "Add Route"
3. Click "Template Matching"
4. Add:
```
{{ payload.commonLabels.team_name == "${YOURTEAMNAME}" }}
```
5. Click save
6. Click "Add an escalation chain"
7. Click New Escalation Chain
8. Name it and create
9. For the first step, add "Trigger Webhook"
10. Select the webhook we created earlier.
11. Add a Wait period, of your choosing, no lower than 5 mins
12. Add another step for your amusement.
13. Go back to your integration and apply the escalation chain to your route, you may need to press the refresh button, but it should then appear in the dropdown. You know it has linked when you have a button "Show escalation chain"

### Dropping a payload
1. Go to your integration
2. Find the "send demo alert" button and click it.
3. Add the following in each of the "labels" fields, there should be 3 within the alerts array. "team_name": "${yourteamname}", e.g. if my team_name was "samteam", it would be "team_name": "samteam".
4. Add the label to the "commonLabels" object, there should already be a team_name field there, just change it to be yours.
5. Click send alert - you should see a green success message in the top right.
6. Click the blue "alert groups" status, it should be to the left of your integration name, and be 1/1 if you've never had an alert sent there before.
7. You should see an instance of an alert saying "InstanceDown", click it.
8. In here - you should see the escalation timeline in the right, explaining the route your alert took, and in there, it should say you have fired the escalation chain, and then by proxy the webhook. 

## CO-019 — Maintenance Window & Alert Suppression

In this exercise you will create a maintenance window (silence) to suppress alerts during a defined period and confirm suppression on the performance timeline.

**Objectives**
- Create a silence covering one or more firing alert rules
- Confirm alerts are suppressed during the window
- Verify the suppression period is visible on the System Overview performance timeline

### Info — What you could do: Create a silence (maintenance window)
**Don't do this, we're instead going to do it through IRM, but this is a mechanism which you could perform**
1. Read the message above and make sure you understand you're not supposed to follow these steps.
2. Navigate to **Alerting → Silences** and click **Add silence**.
3. Set the **Start** and **End** time to cover a window of at least 10 minutes from now.
4. Under **Matchers**, add a matcher that covers your CO-006 alert rule — for example:
   `alertname=High Error Rate - CO-006` or match on the `team` label set earlier.
5. Add a comment: `Planned maintenance window - CO-019 lab exercise`.
6. Click **Submit** to activate the silence.

### Start Maintenance window on your integration
1. HEad to your integration: Alerts & IRM -> IRM -> Integrations"
2. Click the kebab menu, the three vertical dots, click "Start Maintenance".
3. Choose the behaviour you desire, but ensure that you understand its impact, the easiest one my mind can handle is "Silence escalations" as this allows us to see the alerts, but we know that the escalation chain doesn't fire because of the abscence of the call to the webhook.
4. Press start, you'll see that a maintenance window notice has appeared below your integration name. e.g. "1h 0m left"

### Verification
1. Fire a demo alert, ensure you can mimic the routing, just like in the steps in "Dropping a payload"
2. If you configured your maintenance window with "silence escalations", you should be able to head over to view the newly created alert, and will see that the alert has been routed correctly, but it has not haad its escalation chain triggered.

### Bonus: Create the SLO with alert rules

1. Create your SLO, navigate to "Alerts & IRM" -> SLO
2. Press Create SLO
3. Ensure your prom source is selected (by default it is), choose Ratio (it's the default).
4. Input your "success" query:
#### Good events — 200s completing under threshold (adjust le bucket to your target)
```promql  
traces_spanmetrics_latency_bucket{http_status_code="200", le="0.5"}
```
5. Input your "total" query:
#### Total events — all 200 responses
```promql  
traces_spanmetrics_latency_count{http_status_code="200"}
```
6. Press "Run queries" and ensure that your graph is populated.
7. Advance to Set target + error budget
8. Set your target to something unachievable, 99% or higher.
9. Fill out some serious values in the name and description.
10. Set the team_name value - to something you'll remember to then use as routing for the alert, maybe {$yourname}team
11. Click "Add SLO alert rules & assistant investigations"
12. Check "Add SLO alert rules and assistant investigations" - review the items, ensure "Enable Assistant investigations..." is disabled.
13. Ensure you add your team_name label and value to the alerts, without this, your alert will not route to your integration. 
14. Review the alerts you get created for you, for free!
15. Click "Review SLO"
16. Review until you feel you can review no more.
17. Click "Save SLO"

You'll now see the SLO creating, once complete, review the SLO by clicking the name, 

---

## END OF LAB ! 

---



## CO-017 — Auto-Healing Webhook Trigger

We've covered this by proxy with the configuration of the escalation chain within the alert, Here's a specific step to help you out in the future:

This exercise introduces the concept of auto-healing workflows triggered by an alert metric. Full implementation of the downstream remediation system is out of scope for this lab — the key outcome is understanding how Grafana fires an action to an external webhook when an alert condition is met.

> **ℹ️ Callout — CO-017**
>
> Grafana Alerting supports a **Webhook** contact point type. When an alert transitions to Firing, Grafana POSTs a structured JSON payload to any webhook URL — for example, an auto-remediation endpoint, a PagerDuty automation action, or a custom script runner.
>
> **To configure this in your environment:**
>
> 1. Navigate to **Alerting → Contact points** and create a new contact point of type **Webhook**.
> 2. Enter the target webhook URL (e.g. an Ansible AWX endpoint, cloud function, or remediation service).
> 3. Attach this contact point to a notification policy matching the alert you want to auto-heal.
> 4. When the alert fires, Grafana POSTs the alert payload to the webhook, triggering downstream remediation automatically.
>
> This pattern enables **closed-loop alerting** — detect a problem, notify the team, and simultaneously trigger an automated fix.

---

## Lab Summary

| ID | Exercise | Key Outcome |
|---|---|---|
| CO-006 | Alert trigger & escalation routing | Alert fires on threshold breach and routes to the correct team via notification policy |
| CO-019 | Maintenance window & suppression | Silence suppresses alerts during the defined window; confirmed on performance timeline |
| CO-017 | Auto-healing webhook trigger | Understand how Grafana fires a webhook action on alert metric to trigger auto-remediation |
