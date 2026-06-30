---
layout: default
title: Grafana Alerting & Automation Lab
---

# Grafana Alerting & Automation Lab

This lab walks you through three core alerting and automation scenarios using the Field Eng OTel environment. Complete each exercise in order. Each exercise has a reference ID that corresponds to a lab outcome.

---

## CO-006 — Alert Trigger & Escalation Policy Routing

In this exercise you will create an alert rule that fires on a threshold breach and verify it routes to the correct team via a notification policy.

**Objectives**
- Create an alert rule with a static threshold condition
- Configure a notification policy that routes to a specific team contact point
- Confirm the alert fires and is received by the correct team

### Step 1 — Create the alert rule

1. Navigate to **Alerting → Alert rules** and click **New alert rule**.
2. Select a Prometheus datasource and enter a query that will breach a known threshold — for example, a request error rate metric.
3. Under **Set alert conditions**, add a **Threshold** expression. Set the condition to fire when the value exceeds your chosen threshold (e.g. `> 0.05` for a 5% error rate).
4. Set the **Evaluation group** interval to `1m` and pending period to `0s` so it fires immediately.
5. Name the rule clearly, e.g. `High Error Rate - CO-006`.

### Step 2 — Add labels for routing

1. Under **Labels**, add a label that matches your notification policy route.
   Example: `team=platform` or `severity=critical`.
2. This label is what the notification policy uses to route to the correct contact point.

### Step 3 — Verify the notification policy route

1. Navigate to **Alerting → Notification policies**.
2. Confirm a policy exists (or create one) matching your label (e.g. `team=platform`) and pointing to your team's contact point.
3. If no policy exists, click **New nested policy**, set the matcher, and select the appropriate contact point.

### Step 4 — Trigger the alert

1. If the environment is not already generating traffic above your threshold, temporarily lower the threshold value to guarantee a breach.
2. Wait for the next evaluation cycle (up to 1 minute).
3. Navigate to **Alerting → Alert rules** and confirm the rule transitions to **Firing** state.

### Verification

- Alert rule shows **Firing** state in the alert rules list
- **Alerting → Notification history** shows the alert was dispatched to the correct contact point
- The correct team/channel received the notification

---

## CO-019 — Maintenance Window & Alert Suppression

In this exercise you will create a maintenance window (silence) to suppress alerts during a defined period and confirm suppression on the performance timeline.

**Objectives**
- Create a silence covering one or more firing alert rules
- Confirm alerts are suppressed during the window
- Verify the suppression period is visible on the System Overview performance timeline

### Step 1 — Create a silence (maintenance window)

1. Navigate to **Alerting → Silences** and click **Add silence**.
2. Set the **Start** and **End** time to cover a window of at least 10 minutes from now.
3. Under **Matchers**, add a matcher that covers your CO-006 alert rule — for example:
   `alertname=High Error Rate - CO-006` or match on the `team` label set earlier.
4. Add a comment: `Planned maintenance window - CO-019 lab exercise`.
5. Click **Submit** to activate the silence.

### Step 2 — Confirm suppression

1. Navigate to **Alerting → Alert rules**.
2. Confirm the previously firing alert now shows a **Suppressed** badge.
3. Check **Notification history** — no new notifications should be dispatched after silence creation.

### Step 3 — Confirm on the performance timeline

1. Open the **System Overview** dashboard in this environment.
2. Locate the performance timeline or annotations panel.
3. The silence period should be visible as a maintenance annotation on the timeline.

### Step 4 — Expire the silence

1. Navigate back to **Alerting → Silences**.
2. Click **Expire** on your silence to end the maintenance window early.
3. Confirm the alert returns to **Firing** state, demonstrating suppression is no longer active.

### Verification

- Silence is listed as **Active** during the window
- Alert shows **Suppressed** state during the window
- Performance timeline on the System Overview dashboard reflects the maintenance window
- Alert returns to **Firing** after the silence expires

---

## CO-017 — Auto-Healing Webhook Trigger

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
