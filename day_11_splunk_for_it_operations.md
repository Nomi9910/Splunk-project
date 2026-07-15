# Day 11 — Splunk for IT Operations (ITOA)

## Introduction
Beyond security, Splunk is heavily used for **IT Operations Analytics (ITOA)** — monitoring infrastructure health, troubleshooting application issues, tracking deployments, and ensuring service availability. This lesson covers key IT Ops use cases and SPL patterns.

---

## IT Operations Use Cases

| Use Case | Description |
|----------|-------------|
| **Infrastructure Monitoring** | Server health, CPU/memory/disk, OS errors |
| **Application Performance Monitoring (APM)** | Response times, error rates, slow queries |
| **Log Analytics** | Centralise and search all application logs |
| **Deployment Tracking** | Monitor deployments, detect regressions |
| **Capacity Planning** | Trend usage over time, forecast growth |
| **SLA/Uptime Reporting** | Measure availability, error rates |
| **Change Management** | Correlate config changes with incidents |

---

## Infrastructure Monitoring

### Server Health Checks
```spl
# CPU usage across all servers (requires OS metrics)
index=os_metrics metric_name=cpu.usage_user
| timechart span=5m avg(metric_value) BY host

# Memory usage by server
index=os_metrics metric_name=mem.used_percent
| stats avg(metric_value) AS avg_mem_pct by host
| where avg_mem_pct > 85
| sort -avg_mem_pct

# Disk space running low
index=os sourcetype=df_output
| eval pct_used = (used / total) * 100
| where pct_used > 90
| stats max(pct_used) AS max_pct by host, mount
| sort -max_pct
```

### Detecting Server Crashes / Reboots
```spl
# Find unexpected reboots (Linux)
index=linux sourcetype=syslog "reboot" OR "shutdown" OR "kernel: Linux"
| stats count by host, _time
| sort -_time

# Windows unexpected shutdown (Event ID 6008)
index=winevent EventCode=6008
| table _time, host, Message
```

### Process Monitoring
```spl
# Find high-CPU processes
index=os sourcetype=ps_output
| stats avg(pct_cpu) AS avg_cpu by process, host
| where avg_cpu > 50
| sort -avg_cpu

# Find processes that disappeared (service crash detection)
index=os sourcetype=ps_output
| stats latest(_time) AS last_seen by process, host
| where relative_time(now(), "-10m") > last_seen
| eval missing_since = strftime(last_seen, "%Y-%m-%d %H:%M:%S")
```

---

## Application Performance Monitoring

### Response Time Tracking
```spl
# Average response time per endpoint
index=web sourcetype=access_combined
| stats avg(response_time_ms) AS avg_rt, 
        perc95(response_time_ms) AS p95_rt,
        max(response_time_ms) AS max_rt
  by uri_path
| sort -avg_rt
| head 20

# Response time trend over 24 hours
index=web
| timechart span=5m avg(response_time_ms) AS avg_response_ms

# Detect slow responses (> 2 seconds)
index=web response_time_ms > 2000
| stats count AS slow_requests by uri_path, host
| sort -slow_requests
```

### Error Rate Monitoring
```spl
# Error rate per service
index=web
| stats count AS total,
        count(eval(status>=500)) AS server_errors,
        count(eval(status>=400 AND status<500)) AS client_errors
  by host
| eval error_rate = round((server_errors / total) * 100, 2)
| sort -error_rate

# SLA: Percentage of successful requests
index=web
| stats count AS total, count(eval(status<400)) AS success by host
| eval availability_pct = round((success / total) * 100, 2)
| where availability_pct < 99.9
```

### Application Exception Tracking
```spl
# Count and trend exceptions
index=app level=ERROR OR level=CRITICAL
| rex "(?P<exception_type>[A-Z][a-zA-Z]+Exception|[A-Z][a-zA-Z]+Error)"
| stats count by exception_type
| sort -count

# Find new exceptions (not seen before)
index=app level=ERROR
| stats earliest(_time) AS first_seen, count by exception_type
| where relative_time(now(), "-1h") < first_seen
| eval first_seen_human = strftime(first_seen, "%H:%M:%S")
```

---

## Deployment and Change Tracking

### Correlate Deployments with Errors
```spl
# Show deployment events + error spikes on same timeline
(index=cicd sourcetype=jenkins) OR (index=web status>=500)
| eval event_type = if(index="cicd", "Deployment", "Error")
| timechart span=5m count BY event_type
```

### Detect Performance Regression After Deployment
```spl
# Compare response time before and after deployment
index=web
| eval period = if(_time < strptime("2025-06-25 14:00:00", "%Y-%m-%d %H:%M:%S"), "before", "after")
| stats avg(response_time_ms) AS avg_rt, count by period
```

---

## Capacity Planning

### Disk Growth Trend
```spl
index=os sourcetype=df_output
| timechart span=1d avg(pct_used) BY mount
```

### Projecting When Disk Will Be Full
```spl
index=os sourcetype=df_output
| stats avg(pct_used) AS current_pct, 
        (max(pct_used) - min(pct_used)) / 30 AS daily_growth_rate
  by mount, host
| eval days_until_full = (100 - current_pct) / daily_growth_rate
| where days_until_full < 30
| sort days_until_full
```

---

## Log Analytics Patterns

### Find Root Cause: Correlate Events Around Incident Time
```spl
# Show all events across systems ±5 minutes around an incident
(index=web OR index=linux OR index=db OR index=network)
earliest="2025-06-25T14:20:00" latest="2025-06-25T14:30:00"
| sort _time
| table _time, index, host, sourcetype, _raw
```

### Service Dependency Mapping
```spl
# Which services call which services?
index=app sourcetype=access_log
| rex "calling service=(?P<downstream>\S+)"
| stats count by host, downstream
| rename host AS upstream
```

---

## SLA Reporting

```spl
# Monthly availability report
index=web
| timechart span=1d 
    eval(round(count(eval(status<500)) / count() * 100, 3)) AS availability_pct

# Downtime events (service unavailable)
index=app level=CRITICAL "service unavailable"
| transaction host maxspan=30m
| eval downtime_minutes = round(duration / 60, 1)
| stats sum(downtime_minutes) AS total_downtime_minutes by host
```

---

## Splunk ITSI (IT Service Intelligence)

**Splunk ITSI** is a premium app for IT Ops teams providing:
- **Service monitoring** with glass tables (visual service maps)
- **Episode management** (intelligent alert grouping)
- **KPI monitoring** with multi-dimensional anomaly detection
- **Predictive analytics** using ML-based baseline models
- **Service health scoring** (0–100 score per service)

---

## Practice Questions

1. A DevOps team deployed a new version at 2pm. Error rates spiked at 2:05pm. Which approach in Splunk BEST helps confirm the correlation?
   - A) Create a lookup with deployment times  B) Plot deployments and error events together on a timechart ✓  C) Check the _internal index  D) Use transaction to group errors

2. Which SPL function returns the 95th percentile of response times?
   - A) `max(response_time)`  B) `avg(response_time)`  C) `perc95(response_time)` ✓  D) `stdev(response_time)`

3. A company wants to forecast when their disk will reach 100% capacity based on daily growth trends. This is an example of which Splunk use case?
   - A) Security monitoring  B) Application troubleshooting  C) Capacity planning ✓  D) Compliance reporting

4. Which Splunk product provides intelligent alert grouping, service health scores, and glass table visualisations for IT operations?
   - A) Splunk ES  B) Splunk SOAR  C) Splunk ITSI ✓  D) Splunk Observability Cloud

5. An analyst wants to understand the sequence of events across multiple systems 5 minutes before a production outage. Which SPL approach is BEST?
   - A) Use `stats` to aggregate events  B) Search with a narrow time range across all indexes and sort by `_time` ✓  C) Use `transaction` to group all events  D) Search only the main index
