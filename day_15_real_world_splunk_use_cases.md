# Day 15 — Real-World Splunk Use Cases & Complete Reference

## Introduction
This final lesson brings everything together — complete, production-ready SPL use cases across security, IT operations, and business analytics, plus a comprehensive reference for daily Splunk work.

---

## Real-World SOC Use Cases

### Use Case 1: Ransomware Detection
**Scenario:** Detect mass file encryption activity on endpoints.

```spl
index=winevent sourcetype=wineventlog EventCode=4663
  Object_Type=File Access_Mask=0x2 OR Access_Mask=0x6
| stats count AS file_modifications by host, Account_Name
| where file_modifications > 500
| eval risk = "CRITICAL - Possible Ransomware"
| table _time, host, Account_Name, file_modifications, risk
```

**Explanation:** Ransomware modifies/encrypts hundreds or thousands of files in minutes. EventCode 4663 tracks file access operations. A host modifying 500+ files in a short window is highly suspicious.

---

### Use Case 2: Pass-the-Hash Detection
**Scenario:** Detect lateral movement using stolen NTLM hashes.

```spl
index=winevent sourcetype=wineventlog EventCode=4624
  Logon_Type=3 Authentication_Package=NTLM
| where NOT (src_ip="127.0.0.1" OR src_ip="::1")
| stats count, dc(dest_host) AS targets by user, src_ip
| where targets > 3
| eval alert = "Possible Pass-the-Hash - lateral movement to " + tostring(targets) + " hosts"
```

**Explanation:** NTLM Type 3 logins (network logons) from a single source to many different hosts in a short time suggests hash reuse for lateral movement.

---

### Use Case 3: Insider Threat — Unusual Data Access
**Scenario:** Detect an employee accessing data outside their normal pattern.

```spl
index=file_access
| stats count AS daily_count by user, _time
| timechart span=1d avg(daily_count) AS avg_access by user
| eventstats avg(avg_access) AS baseline stdev(avg_access) AS std_dev by user
| where avg_access > (baseline + 3 * std_dev)
| eval alert = "Access anomaly: " + tostring(round(avg_access,0)) + " (baseline: " + tostring(round(baseline,0)) + ")"
```

**Explanation:** Computes per-user baseline and flags days where access was more than 3 standard deviations above their norm — statistical anomaly detection.

---

### Use Case 4: PowerShell Empire / C2 Detection
**Scenario:** Detect obfuscated PowerShell downloads used by C2 frameworks.

```spl
index=winevent sourcetype=wineventlog EventCode=4688
| where Process_Name LIKE "%powershell%"
| rex field=Process_Command_Line
    "(?i)(?P<suspicious_pattern>-enc\s|iex\s|invoke-expression|downloadstring|webclient|hidden|bypass|nop\s)"
| stats count AS hits, values(suspicious_pattern) AS patterns by host, Creator_Process_Name
| where hits > 0
| sort -hits
```

---

### Use Case 5: DNS Exfiltration
**Scenario:** Detect data exfiltration via DNS tunnelling.

```spl
index=dns
| eval subdomain_len = len(query) - len(replace(query, ".", "")) - 1
| eval subdomain_label_len = if(match(query, "^[^.]{30,}\."), len(mvindex(split(query, "."), 0)), 0)
| stats count AS query_count, avg(subdomain_label_len) AS avg_label_len,
        dc(query) AS unique_queries by src_ip, answer
| where avg_label_len > 30 AND unique_queries > 100
| eval alert = "Possible DNS Tunnelling - " + tostring(unique_queries) + " unique queries"
| sort -unique_queries
```

---

### Use Case 6: Cloud (AWS) Threat Detection

#### Root Account Usage (Should Almost Never Happen)
```spl
index=aws sourcetype=aws:cloudtrail userIdentity.type=Root
| table _time, eventName, sourceIPAddress, userAgent, errorCode
| sort -_time
```

#### AWS Credential Exfiltration via Instance Metadata Service
```spl
index=aws sourcetype=aws:cloudtrail
  eventSource=sts.amazonaws.com eventName=AssumeRole
| where userIdentity.type != "AssumeRole"
| stats count by userIdentity.principalId, sourceIPAddress, requestParameters.roleArn
| sort -count
```

---

## Real-World IT Ops Use Cases

### Use Case 7: Proactive Disk Space Alert
```spl
index=os sourcetype=df_output
| stats latest(pct_used) AS current_pct,
        min(pct_used) AS min_7d by mount, host
| eval growth_7d = current_pct - min_7d
| eval days_to_full = if(growth_7d > 0, (100 - current_pct) / (growth_7d / 7), 999)
| where days_to_full < 14
| eval alert = host + ":" + mount + " will fill in " + tostring(round(days_to_full,1)) + " days"
| table host, mount, current_pct, days_to_full, alert
| sort days_to_full
```

### Use Case 8: Application Error Spike Detection
```spl
index=app level=ERROR
| timechart span=5m count AS error_count
| streamstats window=12 avg(error_count) AS rolling_avg stdev(error_count) AS rolling_std
| eval threshold = rolling_avg + (2 * rolling_std)
| where error_count > threshold AND error_count > 10
| eval spike_pct = round((error_count - rolling_avg) / rolling_avg * 100, 0)
| table _time, error_count, rolling_avg, threshold, spike_pct
```

---

## Quick Reference: SPL Command Cheatsheet

### Most Used Commands
```spl
# Basic filter
index=X sourcetype=Y host=Z earliest=-1h

# Count
| stats count by field
| stats count, avg(bytes), max(bytes) by host

# Time series
| timechart span=5m count by status

# Top values
| top 10 field
| rare 5 field

# Sort and limit
| sort -count
| head 20

# Create/modify fields
| eval result = if(status>=400, "error", "ok")
| eval kb = bytes / 1024

# Regex extraction
| rex "user=(?P<username>\w+)"

# Lookup enrichment
| lookup user_info username OUTPUT department

# Table output
| table _time host user status uri_path
```

### Time Modifiers Reference
```
-60m      = last 60 minutes
-24h      = last 24 hours
-7d       = last 7 days
-30d      = last 30 days
@h        = snap to current hour
@d        = snap to start of today
@w0       = snap to Sunday
earliest="06/25/2025:00:00:00" latest="06/25/2025:23:59:59"
```

### Key Metadata Fields
```
_time        event timestamp (epoch)
host         machine that sent the data
source       file path / input name
sourcetype   data format category
index        storage index name
_raw         original raw event text
```

---

## Splunk Deployment Checklist

```
PRE-DEPLOYMENT:
□ Define data sources and ingestion method (UF, HEC, syslog)
□ Plan index structure (separate indexes by team/data type/retention)
□ Define roles and access controls
□ Size hardware (CPU, RAM, storage) based on daily ingest volume
□ Plan for high availability (indexer clustering, SHC)

DATA ONBOARDING:
□ Install appropriate Technology Add-ons (TAs) on search head and indexers
□ Configure props.conf for custom source types
□ Validate field extractions with `| fields *`
□ Verify CIM compliance for security data sources
□ Set data retention policies per index

OPERATIONS:
□ Set up monitoring dashboards for Splunk health
□ Configure licence usage alerts at 80% daily volume
□ Schedule heavy searches during off-peak hours
□ Implement search concurrency limits
□ Back up configuration files and saved searches regularly
□ Document all custom configs in version control (Git)

SECURITY:
□ Enable SSL for Splunk web and forwarder-to-indexer communication
□ Implement LDAP/SSO for user authentication
□ Enable audit logging of all searches
□ Configure role-based access — users only see their data
□ Regularly review and rotate service account credentials
```

---

## Complete Key Terms Reference

| Term | Definition |
|------|-----------|
| **SPL** | Search Processing Language — Splunk's query language |
| **Index** | Storage repository for events in Splunk |
| **Source Type** | Data format category that tells Splunk how to parse data |
| **Universal Forwarder** | Lightweight agent that collects and forwards data |
| **HEC** | HTTP Event Collector — REST API for ingesting data |
| **CIM** | Common Information Model — standardised field names |
| **TA** | Technology Add-on — vendor data collection + CIM normalisation |
| **Data Model** | Schema defining structure of a data category |
| **Acceleration** | Pre-computed summary of data model for fast searches |
| **Notable Event** | Splunk ES term for a correlated security finding |
| **Macro** | Reusable SPL code snippet called with backticks |
| **Event Type** | Saved search condition used to categorise events |
| **Workflow Action** | Custom right-click action on a field value |
| **Lookup** | Reference table used to enrich search results |
| **Throttle** | Setting that limits how often an alert fires |
| **Bucket** | Physical storage unit within an index |
| **tstats** | Fast SPL command operating on indexed metadata only |
| **transaction** | SPL command grouping related events into one record |
| **rex** | SPL command for field extraction using regex |
| **SHC** | Search Head Cluster — HA for the search tier |
| **ES** | Splunk Enterprise Security — premium SIEM app |
| **ITSI** | IT Service Intelligence — premium IT Ops app |
| **SOAR** | Security Orchestration, Automation & Response |

---

## Congratulations — Course Complete! 🎉

**You've covered:**
- Splunk architecture, components, and data pipeline
- Data ingestion: forwarders, HEC, monitor inputs, indexes
- SPL basics to advanced: search, filtering, stats, charts, macros
- Field extractions and lookups for data enrichment
- Dashboards, visualisations, alerts, and reports
- Security use cases: brute force, lateral movement, C2, exfiltration
- IT operations: performance monitoring, capacity planning
- Administration: roles, config files, deployment server
- Data models and CIM for cross-source correlation
- Apps ecosystem and Splunkbase
- Certification paths and exam preparation
- Real-world production-ready use cases

**Next steps:**
1. Enrol in **Splunk Fundamentals 1** (free at education.splunk.com)
2. Set up a **Splunk developer trial** and practice hands-on
3. Complete **Boss of the SOC (BOTS)** challenges
4. Take the **SPLK-1001 Core Certified User** exam
