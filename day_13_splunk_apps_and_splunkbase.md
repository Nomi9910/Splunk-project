# Day 13 — Splunk Apps, Splunkbase & the Ecosystem

## Introduction
Splunk's power extends far beyond its core through a rich ecosystem of **apps and add-ons** available on **Splunkbase** — Splunk's app marketplace. This lesson covers the app ecosystem, major apps, and how to build and distribute custom apps.

---

## What Is a Splunk App?

A **Splunk App** is a packaged collection of:
- Dashboards and views
- Saved searches and alerts
- Reports and macros
- Data models and lookups
- Configuration files

Apps are self-contained — they can be installed, shared, and managed independently. Every deployment includes the default **Search & Reporting** app, but you add specialised apps for specific use cases.

---

## Splunkbase — The App Marketplace

**Splunkbase** (splunkbase.splunk.com) is Splunk's official marketplace with:
- 2,000+ free and paid apps and add-ons
- Vendor-certified and community-contributed apps
- Apps categorised by use case, industry, and technology

### App Types on Splunkbase

| Type | Purpose |
|------|---------|
| **Technology Add-on (TA)** | Data collection + CIM normalisation for specific vendors |
| **App** | Full use-case solution with dashboards + searches |
| **Visualisation** | Custom chart types (leaflet maps, timeline, sankey) |
| **Command** | Custom SPL commands (e.g., ML toolkit) |

---

## Essential Splunk Apps

### Security Apps

| App | Purpose |
|-----|---------|
| **Splunk Enterprise Security (ES)** | Enterprise SIEM — notable events, threat intel, correlation |
| **Splunk Security Essentials (SSE)** | Free — 300+ pre-built detections mapped to MITRE ATT&CK |
| **Splunk SOAR** | Security orchestration, automation, and response |
| **MITRE ATT&CK App for Splunk** | Visualise detections against the ATT&CK matrix |
| **Splunk Threat Intelligence Management** | IoC management and enrichment |
| **Splunk UBA (User Behaviour Analytics)** | Detect insider threats and compromised accounts with ML |

### IT Operations Apps

| App | Purpose |
|-----|---------|
| **Splunk ITSI** | IT Service Intelligence — service health, KPI monitoring |
| **Splunk Observability Cloud** | APM, infrastructure monitoring, log analytics |
| **IT Essentials Work** | Free IT operations starter app |
| **Splunk App for Infrastructure** | Infrastructure monitoring dashboards |
| **Splunk Add-on for VMware** | Monitor VMware vSphere infrastructure |

### DevOps / Cloud Apps

| App | Purpose |
|-----|---------|
| **Splunk Add-on for AWS** | CloudTrail, S3, EC2, VPC Flow Logs |
| **Splunk Add-on for Microsoft Azure** | Azure AD, Activity Logs, diagnostics |
| **Splunk Add-on for GCP** | Google Cloud audit logs, pub/sub |
| **Splunk App for Jenkins** | CI/CD pipeline monitoring |
| **Splunk Add-on for Kubernetes** | Container and pod monitoring |

### Data Collection TAs

| TA | Vendor |
|----|--------|
| Splunk Add-on for Microsoft Windows | Windows Event Logs |
| Splunk Add-on for Unix and Linux | Linux syslog, ps, df, netstat |
| Splunk Add-on for Palo Alto Networks | PAN-OS firewall, Cortex XDR |
| Splunk Add-on for Cisco ASA | Cisco firewall logs |
| Splunk Add-on for Fortinet | FortiGate firewall |
| Splunk Add-on for Crowdstrike | EDR telemetry |
| Splunk Add-on for SentinelOne | EDR telemetry |
| Splunk Add-on for Microsoft Defender | Windows Defender events |
| Splunk Add-on for Office 365 | Email, Teams, SharePoint activity |

---

## Building a Custom Splunk App

### App Directory Structure
```
$SPLUNK_HOME/etc/apps/my_app/
├── default/
│   ├── app.conf          ← App metadata and configuration
│   ├── inputs.conf       ← Data inputs
│   ├── savedsearches.conf← Saved searches, reports, alerts
│   ├── transforms.conf   ← Lookup definitions
│   └── props.conf        ← Parsing rules
├── local/
│   └── [override files]  ← Local customisations
├── lookups/
│   └── my_lookup.csv     ← Lookup data files
├── metadata/
│   └── default.meta      ← Permissions
└── appserver/
    └── static/           ← Custom images, CSS, JS
```

### app.conf
```ini
[launcher]
author = Your Name
description = My Custom Security Dashboard App
version = 1.0.0

[ui]
is_visible = 1
label = My Security App

[package]
id = my_security_app
```

### Creating the App via Splunk UI
```
Manage Apps → Create App
→ Name: My Security App
→ Folder name: my_security_app
→ Template: barebones (or sample_app)
→ Upload asset: app icon
→ Create
```

---

## Macros — Reusable SPL Snippets

**Search macros** define reusable SPL code that can be inserted into searches:

```ini
# macros.conf
[failed_logins(1)]
args = threshold
definition = index=security sourcetype=wineventlog EventCode=4625 \
  | stats count by user, src_ip \
  | where count > $threshold$
iseval = 0
```

**Usage in search:**
```spl
# Use the macro with argument 10
`failed_logins(10)`

# Macros are called with backticks
`index_web`         # calls macro named index_web
`common_filters`    # inserts the macro definition inline
```

---

## Event Types — Categorising Events

**Event types** save complex search conditions as reusable named categories:

```ini
# eventtypes.conf
[failed_login]
search = index=security (EventCode=4625 OR "Failed password")

[successful_login]
search = index=security (EventCode=4624 OR "Accepted password")
```

```spl
# Use event types in searches
eventtype=failed_login | stats count by user

# Event types appear as a field in search results
index=security | table _time, eventtype, user, src_ip
```

---

## Workflow Actions — Right-Click Context Menus

Workflow actions add custom right-click options in search results:

**Example: "Lookup IP in VirusTotal"**
```ini
# workflow_actions.conf
[lookup_virustotal]
display_location = both
fields = src_ip
label = Lookup in VirusTotal
link.uri = https://www.virustotal.com/gui/ip-address/$src_ip$
link.target = _blank
type = link
```

Now right-clicking any `src_ip` field shows "Lookup in VirusTotal" option.

---

## Apps vs Add-ons: Key Differences

| Feature | App | Add-on (TA) |
|---------|-----|-------------|
| Dashboards | ✔ | ✗ |
| Saved Searches | ✔ | ✗ |
| Data Collection | Optional | ✔ |
| CIM Normalisation | Optional | ✔ |
| Visible in UI | ✔ | Usually not |
| Purpose | End-user analytics | Data ingestion + parsing |

Often used together: TA collects and normalises data → App provides dashboards and analysis.

---

## Practice Questions

1. Where is the official Splunk marketplace where you can download apps and add-ons?
   - A) apps.splunk.com  B) Splunkbase ✓  C) Splunk Hub  D) Splunk Store

2. A security team installs the Splunk Add-on for Palo Alto Networks. What does this TA primarily provide?
   - A) Security dashboards and alert correlations  B) Data collection and CIM normalisation for PAN-OS logs ✓  C) A SIEM interface  D) User behaviour analytics

3. Which file in a Splunk app defines reusable SPL code snippets called with backticks?
   - A) eventtypes.conf  B) props.conf  C) savedsearches.conf  D) macros.conf ✓

4. A Splunk admin right-clicks a source IP in search results and sees a "Lookup in Threat Intel" option. This is configured using:
   - A) Macros  B) Workflow Actions ✓  C) Drilldowns  D) Event Types

5. An analyst uses `eventtype=failed_login` in a search. What is an event type?
   - A) A type of log format  B) A CIM field  C) A saved complex search condition used as a reusable category ✓  D) A workflow action
