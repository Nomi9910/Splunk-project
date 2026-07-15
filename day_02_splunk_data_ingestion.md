# Day 2 — Splunk Data Ingestion & Index Management

## Introduction
Before you can search data in Splunk, you need to get data IN. Understanding how Splunk ingests, parses, and stores data is foundational to everything else. This lesson covers inputs, source types, indexes, and the data pipeline in detail.

---

## Data Input Methods

Splunk accepts data from virtually any source through several input methods:

### 1. Files and Directories (Monitor Inputs)
```
Settings → Data Inputs → Files & Directories
```
- Monitor a log file or directory for new/changed data
- Splunk tails files like `tail -f` — picks up new lines as they arrive
- Common for: Apache logs, application logs, system logs

```bash
# In inputs.conf:
[monitor:///var/log/apache2/access.log]
sourcetype = access_combined
index = web

[monitor:///var/log/]
sourcetype = syslog
index = linux_logs
```

### 2. Network Inputs (TCP/UDP)
- Receive data over a network port
- Common for: syslog, network devices, firewalls

```bash
# inputs.conf - listen on UDP 514 for syslog
[udp://514]
sourcetype = syslog
index = network

# inputs.conf - listen on TCP 9997 (Splunk-to-Splunk)
[splunktcp://9997]
```

### 3. Universal Forwarder (Most Common)
- Install UF agent on each server
- UF monitors files/directories and forwards to indexer
- Lightweight — minimal resource impact
- Encrypted communication (SSL) to indexer on port 9997

### 4. HTTP Event Collector (HEC)
- REST API endpoint for sending events over HTTP/HTTPS
- Token-based authentication
- Used by: applications, cloud services, scripted inputs

```bash
# Send event via HEC
curl -k https://splunk-server:8088/services/collector/event \
  -H "Authorization: Splunk YOUR-TOKEN-HERE" \
  -d '{"event": "Hello, Splunk!", "sourcetype": "manual"}'
```

### 5. Scripted Inputs
- Run a script on a schedule to collect data
- Custom data sources: APIs, databases, proprietary formats

### 6. Modular Inputs
- Pre-built packaged inputs (via Add-ons from Splunkbase)
- Examples: AWS inputs, Azure inputs, Office 365, Sysmon

---

## Source Types

A **source type** tells Splunk how to parse and interpret incoming data.

| Source Type | Data It Handles |
|------------|----------------|
| `access_combined` | Apache/Nginx web server access logs |
| `syslog` | Linux/Unix syslog format |
| `wineventlog` | Windows Event Logs |
| `json` | JSON-formatted data |
| `csv` | Comma-separated values |
| `xmlwineventlog` | Windows Event Log in XML format |
| `linux_secure` | /var/log/secure (auth events) |
| `aws:cloudtrail` | AWS CloudTrail API logs |
| `pan:traffic` | Palo Alto firewall traffic logs |

### Source Type Determines:
- How events are broken up (line-by-line or multi-line)
- Where timestamps are found and what format they use
- Which fields are automatically extracted (field extractions)

---

## Indexes

An **index** is where Splunk stores your data — like a database table.

### Default Indexes

| Index | Purpose |
|-------|---------|
| `main` | Default index for all data |
| `_internal` | Splunk's own operational logs |
| `_audit` | Audit trail of Splunk activities |
| `summary` | Summary index for accelerated searches |

### Creating Custom Indexes
Organising data into separate indexes allows:
- Role-based access control (users see only their indexes)
- Different retention policies per index
- Better search performance (scope searches to specific indexes)

```bash
# In indexes.conf:
[security_logs]
homePath = $SPLUNK_DB/security_logs/db
coldPath = $SPLUNK_DB/security_logs/colddb
thawedPath = $SPLUNK_DB/security_logs/thaweddb
maxTotalDataSizeMB = 500000
frozenTimePeriodInSecs = 7776000  # 90 days retention
```

### Index Data Storage (Hot/Warm/Cold/Frozen)

```
HOT    → Current data, searchable, stored on fast SSD
WARM   → Older data, searchable, can move to slower disk
COLD   → Older still, searchable, cheaper storage
FROZEN → Archived, not searchable (can be thawed if needed)
```

---

## The Data Pipeline (Detailed)

```
┌─────────────────────────────────────────────────────────────────┐
│                     SPLUNK DATA PIPELINE                        │
├──────────┬────────────┬──────────────┬──────────────────────────┤
│  INPUT   │  PARSING   │   INDEXING   │        SEARCHING         │
├──────────┼────────────┼──────────────┼──────────────────────────┤
│• Collect │• Break into│• Compress    │• Run SPL queries         │
│  raw data│  events    │  and store   │• Apply field extractions │
│• Apply   │• Extract   │• Build TSIDX │• Return results to user  │
│  input   │  timestamp │  (time-series│                          │
│  settings│• Apply     │  index) files│                          │
│          │  source    │              │                          │
│          │  type      │              │                          │
└──────────┴────────────┴──────────────┴──────────────────────────┘
```

---

## Metadata Fields (Automatically Added)

Every event indexed in Splunk automatically gets these metadata fields:

| Field | Description | Example |
|-------|-------------|---------|
| `_time` | Timestamp of the event | `2025-06-25 14:23:45` |
| `host` | Hostname that generated the event | `web-server-01` |
| `source` | File path or input source | `/var/log/apache2/access.log` |
| `sourcetype` | Data format category | `access_combined` |
| `index` | Which index stored the event | `web` |

These fields are **always available** in searches without configuration.

---

## Props.conf and Transforms.conf

**props.conf** — Controls how Splunk processes data at parse time:
- Event breaking (line merging)
- Timestamp extraction and format
- Field extractions via regex
- Source type configuration

**transforms.conf** — Used with props.conf for:
- Lookup tables
- Field aliasing
- Source type routing

```ini
# props.conf example: custom timestamp format
[my_app_logs]
TIME_FORMAT = %Y-%m-%dT%H:%M:%S
TIME_PREFIX = timestamp=
MAX_TIMESTAMP_LOOKAHEAD = 30
```

---

## Data Ingestion Best Practices

```
✔ Use Universal Forwarders — don't send raw data directly
✔ Create separate indexes per data type/department
✔ Set retention policies per index based on compliance needs
✔ Always specify sourcetype — don't rely on auto-detection
✔ Use HEC for application and cloud data ingestion
✔ Monitor ingestion volumes to manage licence usage
✔ Test data ingestion with small samples first
✔ Use Add-ons (TAs) from Splunkbase for vendor data
```

---

## Practice Questions

1. Which input method provides a REST API for sending events to Splunk over HTTP?
   - A) Monitor inputs  B) Scripted inputs  C) HTTP Event Collector ✓  D) TCP/UDP inputs

2. What does the `sourcetype` field tell Splunk?
   - A) Which user sent the data  B) How to parse and interpret the incoming data ✓  C) Which indexer stored the data  D) The retention period

3. In which storage tier is the most recently indexed data kept?
   - A) Cold  B) Frozen  C) Warm  D) Hot ✓

4. Which Splunk component is installed on a source machine to collect and forward data?
   - A) Search Head  B) Indexer  C) Universal Forwarder ✓  D) Deployment Server

5. Which metadata field contains the timestamp of an indexed event?
   - A) `time`  B) `timestamp`  C) `_time` ✓  D) `event_time`
