# Day 7 — Alerts & Reports

## Introduction
Alerts notify you when something important happens in your data — without you having to manually run searches. Reports schedule searches to run automatically and deliver results. Together, they form the proactive monitoring layer of Splunk.

---

## Reports

### What Is a Report?
A **saved search** that can be:
- Run on demand
- Scheduled to run automatically
- Added to dashboards
- Sent via email
- Used as a data source for alerts

### Creating a Report
```
1. Run a search in Search & Reporting
2. Click "Save As" → Report
3. Configure:
   - Title and description
   - Time range
   - Schedule (optional)
   - Permissions
4. Save
```

### Scheduling a Report
```
Reports → [Select Report] → Edit → Schedule
→ Schedule type: Run on Cron Schedule or Every [interval]
→ Cron expression: 0 8 * * 1-5  (weekdays at 8am)
→ Time range: Last 24 hours
→ Action: Send Email / Log Event / etc.
```

---

## Alerts

### What Is an Alert?
An alert monitors a search condition and triggers actions when that condition is met.

### Alert Types

| Type | Description |
|------|-------------|
| **Scheduled Alert** | Runs search on a schedule; triggers if condition met |
| **Real-Time Alert** | Continuously monitors; triggers as events arrive |

---

## Alert Trigger Conditions

| Condition | Description |
|-----------|-------------|
| **Number of Results** | Trigger if result count `>`, `<`, `=` a threshold |
| **Number of Hosts** | Trigger based on unique host count |
| **Number of Sources** | Trigger based on unique source count |
| **Custom Condition** | Trigger based on a field value in results |
| **Always** | Trigger every time the search runs |

---

## Alert Actions

When an alert fires, it can perform one or more actions:

| Action | Description |
|--------|-------------|
| **Send Email** | Notify on-call team, SOC, or management |
| **Run a Script** | Execute a custom script for remediation |
| **Webhook** | Send HTTP POST to external system (PagerDuty, Slack, JIRA) |
| **Add to Triggered Alerts** | Log in Splunk's alert list |
| **Create a Ticket (via SOAR)** | Integration with ticketing systems |
| **Log Event** | Write event to a Splunk index |
| **Output to Lookup** | Write matching events to a lookup file |

---

## Creating an Alert

### Method 1: From a Search
```
1. Run your SPL search and verify it catches the right events
2. Click "Save As" → Alert
3. Configure:
   - Title: "Failed SSH Logins Spike"
   - Alert type: Scheduled
   - Schedule: Every 5 minutes
   - Trigger condition: Number of results > 10
   - Action: Send Email → security-team@company.com
4. Save
```

### Method 2: From Alerts Menu
```
Alerts → Create Alert
→ Define search, schedule, condition, and actions
```

---

## Real-World Alert Examples

### Failed Login Brute Force
```spl
index=linux sourcetype=linux_secure "Failed password"
| stats count by src_ip
| where count > 20
```
**Condition:** Number of results > 0 (any host with 20+ failures)
**Schedule:** Every 5 minutes
**Action:** Send email + webhook to PagerDuty

### Critical Error Rate Spike
```spl
index=app level=CRITICAL earliest=-5m
| stats count AS critical_errors
| where critical_errors > 50
```
**Condition:** Number of results > 0
**Schedule:** Every 5 minutes
**Action:** Send email to dev-oncall@company.com

### New Admin Account Created
```spl
index=winevent sourcetype=wineventlog EventCode=4720
| stats count by user, Account_Name
```
**Condition:** Number of results > 0
**Schedule:** Every 1 minute (near real-time)
**Action:** Send email + log to security_events index

### Data Exfiltration Detection
```spl
index=proxy
| stats sum(bytes_out) AS total_out by src_ip
| where total_out > 1073741824
```
**Condition:** Number of results > 0 (any IP sending >1GB)
**Schedule:** Every 15 minutes
**Action:** Webhook to SOAR playbook

---

## Alert Throttling

Prevent alert fatigue by controlling how often the same alert fires:

```
Alert Settings → Throttle
→ Suppress trigger for: 60 minutes
→ Suppress by field: src_ip (one alert per IP per hour)
```

Without throttling, a brute-force alert could fire every 5 minutes for hours, flooding your inbox.

---

## Triggered Alerts Dashboard

View all recently triggered alerts:
```
Alerts → Triggered Alerts
→ See: Alert name, count, time triggered, actions taken
```

---

## Alert Severity Levels

| Level | When to Use |
|-------|-------------|
| **Critical** | Immediate action required (active breach, service down) |
| **High** | Urgent but not immediate (suspicious activity) |
| **Medium** | Investigate soon (policy violations, anomalies) |
| **Low** | Informational, investigate when time permits |
| **Info** | Logging only, no action needed |

---

## Best Practices for Alerts

```
✔ Test the SPL search thoroughly before saving as alert
✔ Set appropriate throttle to prevent alert fatigue
✔ Use meaningful alert titles and descriptions
✔ Include relevant context in email notifications (top events, link to search)
✔ Tune alert thresholds based on historical baseline
✔ Document expected actions in alert description (runbook link)
✔ Review and clean up unused alerts regularly
✔ Route alerts to appropriate teams (security vs ops vs dev)
✔ Use SOAR integration for automated response to common alert types
✔ Monitor alert trigger history to detect threshold drift
```

---

## SPL for Alert Tuning (Baselining)

Before setting an alert threshold, understand your baseline:

```spl
# See normal login failure counts per 5-minute window over last 7 days
index=linux sourcetype=linux_secure "Failed password"
| bucket span=5m _time
| stats count by _time
| stats avg(count) AS avg_count, max(count) AS max_count, stdev(count) AS std_dev

# Set alert threshold at: avg + (3 * std_dev)
```

---

## Practice Questions

1. Which alert type continuously monitors a search and fires as matching events arrive?
   - A) Scheduled Alert  B) Real-Time Alert ✓  C) Triggered Alert  D) Custom Alert

2. A security team receives 200 email alerts in one hour for the same brute-force event. What setting should they configure?
   - A) Disable the alert  B) Alert Throttling ✓  C) Increase the threshold  D) Change to real-time alert

3. Which alert action sends an HTTP POST request to an external tool like PagerDuty or Slack?
   - A) Run Script  B) Log Event  C) Webhook ✓  D) Send Email

4. An analyst wants to be notified whenever a search returns MORE than 5 results. Which trigger condition should they use?
   - A) Always  B) Number of Hosts > 5  C) Number of Results > 5 ✓  D) Custom Condition

5. Which Splunk feature allows an analyst to view all recently fired alerts in one place?
   - A) Reports Page  B) Triggered Alerts Dashboard ✓  C) Alert Actions  D) Dashboard Drilldown
