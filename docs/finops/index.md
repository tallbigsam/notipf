---
title: FinOps
permalink: /finops/
---
# FinOps

# Grafana Cloud Billing Lab  

> **Duration:** ~20 minutes | **Level:** Beginner | **Prereq:** Grafana Cloud access (Editor role for Task 5)  

Explore your AWS billing data, understand dashboard filters, identify cost drivers, and configure a proactive spend alert.  

---  

## Task 1 — Find your current monthly spend  

**Navigate to:** Cloud Provider Observability → AWS → Billing  
`/a/grafana-csp-app/aws/dashboards/billing`  

1. In the left-hand navigation expand **Cloud Provider Observability**, select **AWS**, then **Billing**.  
2. Look at the top section of the dashboard — a summary panel shows the estimated monthly spend. This is calculated from AWS Cost and Usage Report data ingested into your Grafana Cloud metrics stack.  
3. Note the time picker in the top-right. The billing dashboard aggregates spend up to the most recently ingested data point.  

> ❓ **Answer this:** What is the current estimated monthly spend shown on the billing dashboard?  

---  

## Task 2 — Identify the Account ID filter  

Dashboard filters (called **variables** in Grafana) appear as dropdowns at the top of the dashboard. They scope all panels simultaneously without editing any queries.  

1. At the top of the Billing dashboard, locate the **Account** filter dropdown.  
2. The value shown is your AWS Account ID — a 12-digit number that uniquely identifies your AWS account (or management/payer account if you use AWS Organizations).  
3. If you manage multiple AWS accounts (e.g. dev, staging, production), each has its own Account ID and appears as a separate option, letting you isolate spend per account.  

> ❓ **Answer this:** What Account ID is shown in the Account filter? What do you think would happen if you selected _All_ from the dropdown?  

---  

## Task 3 — Explore all available filters  

The Billing dashboard exposes several filters that work together. Each one narrows all panels simultaneously.  

| Filter | What it controls | Example values |  
|---|---|---|  
| **Datasource** | Selects which Prometheus datasource is queried. Useful if you have multiple Grafana Cloud stacks. | `grafanacloud-dfcb24-prom` |  
| **Account** | Scopes all panels to a single AWS Account ID. Essential in multi-account AWS Organizations setups. | `008971678742` |  
| **Region** | Filters spend to a specific AWS region. Helps identify regional cost concentration. | `us-east-1`, `eu-west-1` |  
| **Service Name** | Narrows panels to a single AWS service. Useful for per-service cost audits. | `AmazonEC2`, `AmazonS3` |  

> ❓ **Answer this:** Try changing the Region filter to a specific region. Do the totals change? What does this tell you about where your costs are concentrated?  

---  

## Task 4 — Find the highest spend AWS service  

The billing dashboard includes a breakdown of spend by AWS service — one of the most useful views for cost optimisation.  

1. Scroll down to find the panel showing cost broken down by AWS service (titled **Cost by Service** or displayed as a table/bar chart).  
2. Services are listed with their estimated monthly cost. Common top spenders: EC2 (compute), S3 (storage), RDS (databases), Data Transfer.  
3. Use the **Service Name** filter at the top to isolate a single service and see its trend over time.  
4. Hover over a data point in any time-series panel to see the exact cost. Click and drag to zoom into a narrower time window.  

> ❓ **Answer this:** Which AWS service is your highest line-item cost? What percentage of total monthly spend does it represent?  

---  

## Task 5 — Create a spend threshold alert using Grafana Alerting  

AWS billing data is ingested as Prometheus metrics into your Grafana Cloud stack. This means you can alert on it using standard Grafana alert rules, exactly as you would for any other infrastructure metric.  

The metric you will use is:  

```  
`aws_billing_estimated_charges_average`  
```  

It has labels including `account_id`, `dimension_ServiceName`, and `dimension_Currency`. To alert on **total** spend across all services, you sum across all `dimension_ServiceName` values.  

---  

### Step 1 — Create a contact point  

A contact point tells Grafana where to send alert notifications (email, Slack, PagerDuty, etc.).  

**Navigate to:** Alerting → Contact points  
`/alerting/notifications`  

1. Click **+ Add contact point**.  
2. Give it a name, e.g. `AWS Billing Alerts`.  
3. Select an integration type — for example **Email**:  
   - Add one or more recipient addresses under _Addresses_  
4. Click **Test** to send a test notification and confirm delivery.  
5. Click **Save contact point**.  

---  

### Step 2 — Create the alert rule  

**Navigate to:** Alerting → Alert rules → New alert rule  
`/alerting/new`  

#### A — Set the query  

1. Under **Define query and alert condition**, select your Prometheus datasource (e.g. `grafanacloud-dfcb24-prom`).  
2. Switch to **Code** mode and enter the following PromQL:  

```promql  
sum(`aws_billing_estimated_charges_average`{`dimension_Currency`="USD"})  
```  

This sums estimated charges across all AWS services for USD. Set the query type to **Instant**.  

3. Click **Run queries** to confirm a value is returned — it should reflect your current estimated spend in dollars.  

#### B — Set the threshold condition  

4. Under **Expressions**, you will see a default **Reduce** expression (B) and **Threshold** expression (C).  
   - **B (Reduce):** Function = `Last`, Input = `A`  
   - **C (Threshold):** set condition to `IS ABOVE` → `1200`  
5. Set **C** as the alert condition.  

#### C — Configure evaluation  

6. Under **Alert evaluation**, set:  
   - **Evaluation group:** create a new group, e.g. `billing-alerts`  
   - **Evaluation interval:** `10m` (AWS billing metrics update infrequently, so 10–60 min is appropriate)  
   - **Pending period:** `0s` (fire immediately when threshold is crossed)  

#### D — Add details  

7. Under **Add details for your alert rule**:  
   - **Rule name:** `AWS Monthly Spend Over $1200`  
   - Add an annotation **Summary:** `AWS estimated monthly spend has exceeded $1,200 (current: {{ $values.B.Value | printf "%.2f" }})`  

#### E — Set notifications  

8. Under **Configure notifications**, choose **Select contact point** and pick the `AWS Billing Alerts` contact point you created in Step 1.  

9. Click **Save rule and exit**.  

---  

### Verify the alert  

Navigate to **Alerting → Alert rules** and find your new rule. It will show as:  
- **Normal** — spend is below $1,200  
- **Firing** — spend has exceeded $1,200  

> ❓ **Answer this:** What state is your alert currently in? Based on Task 1, does this match your actual spend?  

> 💡 **Pro tip:** To alert per-service rather than total spend, add a `by (dimension_ServiceName)` clause:  
> ```promql  
> sum by (`dimension_ServiceName`) (`aws_billing_estimated_charges_average`{`dimension_Currency`="USD"})  
> ```  
> This fires a separate alert instance for each service that exceeds the threshold.  

---  

## Bonus Task — Explore anomaly and attribution alerts  

**Navigate to:** `/a/grafana-cmab-app/usage-alerts` → **+ New alert**  

The Usage Alerts page supports two additional alert types:  

- **Anomaly alerts** — fire when usage departs significantly from a 7-day rolling average. Useful for catching unexpected spikes (e.g. a runaway Lambda function) even when absolute spend is below a fixed threshold.  
- **Attribution alerts** — scope spend tracking to a specific label value (e.g. `team=payments` or `environment=production`). Requires consistent AWS resource tagging.  

Explore the alert type options in the form — you don't need to save this step.  

> 💬 **Discuss:** In your environment, which alert type would be most useful — threshold, anomaly, or attribution? What resource tags would you need in AWS for attribution alerts to work effectively?  

---  

## Reference links  

- [AWS Billing dashboard](/a/grafana-csp-app/aws/dashboards/billing)  
- [Cost Management and Billing app](/a/grafana-cmab-app/usage)  
- [Usage Alerts](/a/grafana-cmab-app/usage-alerts)  
- [Grafana docs: Usage cost alerts](https://grafana.com/docs/grafana-cloud/cost-management-and-billing/usage-cost-alerts/)  

---  

*Well done! You now know how to read current cloud spend, interpret dashboard filters, identify cost drivers, and configure proactive alerting — key skills for cloud cost observability.*  
