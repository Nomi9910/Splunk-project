# Day 12 — Data Models & Common Information Model (CIM)

## Introduction
The **Common Information Model (CIM)** is Splunk's standardised framework for normalising data from different sources into a consistent structure. Data models and the CIM are foundational for Splunk Enterprise Security, accelerated search, and cross-source correlation.

---

## What Is the Common Information Model (CIM)?

The CIM is a set of standardised **field names and data models** that define how security and IT data should be structured regardless of the source.

**Without CIM:**
- Cisco firewall logs call the source IP: `src_ip`
- Palo Alto calls it: `srcip`
- Check Point calls it: `orig`
- Each vendor's field name is different

**With CIM:**
- All sources normalise to `src` (the CIM field name for source IP)
- Searches work the same way across ALL sources
- Dashboard panels work with any vendor's data

---

## CIM Data Models

The CIM defines **data models** — structured schema for different data categories:

| Data Model | Content |
|-----------|---------|
| **Authentication** | Login/logout events, success/failure |
| **Network Traffic** | Firewall/router traffic flows |
| **Web** | Web proxy and server access events |
| **Email** | Email traffic, spam, phishing |
| **Endpoint** | Process, file, registry, network events on hosts |
| **Intrusion Detection** | IDS/IPS alerts |
| **Malware** | Antivirus and malware detection events |
| **Vulnerability** | Vulnerability scan results |
| **Network Sessions** | VPN and network session events |
| **DNS** | Domain Name System queries and responses |
| **Certificate** | SSL/TLS certificate events |
| **Change** | Configuration change events |
| **Alerts** | Security alert data |

---

## CIM Fields — Key Examples

### Authentication Data Model Fields
| CIM Field | Description | Example Value |
|-----------|-------------|--------------|
| `action` | What happened | `success`, `failure` |
| `user` | Username | `alice` |
| `src` | Source IP | `192.168.1.100` |
| `dest` | Destination | `auth-server-01` |
| `app` | Application | `ssh`, `windows` |
| `authentication_method` | Auth type | `kerberos`, `ntlm` |

### Network Traffic Data Model Fields
| CIM Field | Description |
|-----------|-------------|
| `src` | Source IP |
| `dest` | Destination IP |
| `src_port` | Source port |
| `dest_port` | Destination port |
| `transport` | TCP, UDP |
| `bytes_in` | Bytes received |
| `bytes_out` | Bytes sent |
| `action` | `allowed`, `blocked` |

---

## Technology Add-Ons (TAs)

**Technology Add-Ons (TAs)** are Splunk apps that:
1. Collect data from a specific vendor/technology
2. Parse and normalise the data to CIM field names
3. Assign correct source types and tags

Popular TAs from Splunkbase:
- **Splunk Add-on for Microsoft Windows** — normalises Windows Event Logs
- **Splunk Add-on for Unix/Linux** — normalises Linux syslog, ps, df
- **Palo Alto Networks Add-on** — normalises PAN-OS firewall logs
- **Splunk Add-on for AWS** — CloudTrail, CloudWatch, VPC Flow Logs
- **Microsoft Azure Add-on** — Azure AD, Activity Logs

### How a TA Normalises Data

```ini
# Palo Alto TA - props.conf excerpt
[pan:traffic]
# CIM Network Traffic normalisation
FIELDALIAS-src = src_ip AS src
FIELDALIAS-dest = dest_ip AS dest
FIELDALIAS-bytes_in = bytes_received AS bytes_in
FIELDALIAS-bytes_out = bytes_sent AS bytes_out
EVAL-action = if(action="allow", "allowed", "blocked")

# Tag the source type for data model compliance
[pan:traffic]
TAGS = network communicate
```

---

## Data Models in Splunk

A **data model** is a hierarchical schema that defines:
- The object types in your data
- The fields each object type should have
- Which events belong to each object (via constraints)

### Data Model Structure
```
Data Model: Authentication
└── Root Object: Authentication
    ├── Constraint: sourcetype=wineventlog OR sourcetype=linux_secure
    ├── Fields: user, src, dest, action, app
    └── Child Objects:
        ├── Successful_Authentication (action=success)
        └── Failed_Authentication (action=failure)
```

---

## Data Model Acceleration

For large datasets, data model acceleration pre-computes summary data, making searches **10–100x faster**.

```
Settings → Data Models → [Select Model] → Edit → Acceleration
→ Enable Acceleration
→ Summary Range: 1 month
→ Cron: 0 */1 * * * (run every hour)
```

Once accelerated, use `tstats` with the data model for ultra-fast queries:

```spl
# Fast authentication query using accelerated data model
| tstats count WHERE index=security
  BY Authentication.user, Authentication.action, Authentication.src
  FROM datamodel=Authentication
| rename Authentication.* AS *
| where action="failure"
| sort -count
```

---

## Pivot Interface

The **Pivot** tool provides a drag-and-drop interface to explore data models without writing SPL — great for non-technical users:

```
Settings → Data Models → [Select Model] → Pivot
→ Drag fields to Filters, Rows, Columns
→ Select visualisation type
→ Save as report or dashboard panel
```

---

## CIM Compliance Search

Check if your data is properly normalised to CIM:

```spl
# Check Authentication data model compliance
| datamodel Authentication Authentication search
| stats count by sourcetype, source
| sort -count

# Verify CIM tags are applied correctly
index=security
| tags outputfield=tags
| where like(tags, "%authentication%")
| stats count by sourcetype, tags
```

---

## CIM Tagging

Tags link raw events to data model objects:

```ini
# tags.conf — tag events so they match data model constraints
[eventtype=windows_authentication]
authentication = enabled
```

```spl
# Verify tags in search
index=winevent | tags outputfield=event_tags | table sourcetype, event_tags
```

---

## Practice Questions

1. What is the primary purpose of the Splunk Common Information Model (CIM)?
   - A) Encrypt log data  B) Normalise field names across different data sources for consistent searches ✓  C) Accelerate search performance  D) Define user roles and permissions

2. Which type of Splunk add-on collects, parses, and normalises vendor-specific data to CIM field names?
   - A) Premium App  B) Core App  C) Technology Add-on (TA) ✓  D) Deployment App

3. A security analyst uses `| tstats count FROM datamodel=Authentication`. What must be enabled for this to work efficiently?
   - A) Distributed search  B) Data model acceleration ✓  C) Field extractions  D) Summary indexing

4. In the CIM Authentication data model, what field name is used for the source IP of an authentication event?
   - A) `src_ip`  B) `source`  C) `clientip`  D) `src` ✓

5. Which tool in Splunk allows non-technical users to explore data models visually without writing SPL?
   - A) Search & Reporting  B) Field Extractor  C) Pivot ✓  D) Dashboard Studio
