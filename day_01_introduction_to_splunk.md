# Day 1 — Introduction to Splunk

## What Is Splunk?

**Splunk** is an industry-leading data platform that collects, indexes, and searches machine-generated data in real time. It transforms raw logs, metrics, and events from any source into operational intelligence — enabling security monitoring, IT operations, business analytics, and compliance.

> **In one sentence:** Splunk is a search engine for machine data.

---

## Why Splunk?

Modern organisations generate enormous volumes of machine data from:
- Web servers, application logs
- Security devices (firewalls, IDS/IPS)
- Network devices (routers, switches)
- Operating systems (Windows Event Logs, syslog)
- Cloud platforms (AWS CloudTrail, Azure Monitor)
- IoT sensors, SCADA systems
- Databases and middleware

Without a tool like Splunk, finding patterns, troubleshooting failures, or detecting threats in this data is nearly impossible.

**Splunk solves this by:**
- Collecting data from any source, in any format
- Indexing it so it is instantly searchable
- Providing powerful search and analytics
- Visualising results in dashboards
- Alerting on anomalies in real time

---

## Splunk Use Cases

| Domain | Use Case |
|--------|---------|
| **Security (SIEM)** | Detect threats, correlate security events, investigate incidents |
| **IT Operations** | Troubleshoot infrastructure issues, monitor application health |
| **DevOps** | Monitor deployments, track errors, trace microservices |
| **Business Analytics** | Analyse customer behaviour, track KPIs |
| **Compliance** | Audit trails, regulatory reporting, log retention |
| **Network Operations** | Monitor bandwidth, detect outages, analyse flows |

---

## Splunk Deployment Options

### 1. Splunk Enterprise
- On-premises or cloud-hosted installation
- Full control over data and configuration
- Used by large enterprises with specific compliance or data requirements

### 2. Splunk Cloud Platform
- Fully managed SaaS by Splunk
- No infrastructure to maintain
- Fast deployment, automatic upgrades

### 3. Splunk Free
- Single-user version, limited to 500 MB/day indexing
- Good for learning and small environments

### 4. Splunk SOAR (Security Orchestration, Automation & Response)
- Automates security workflows and incident response

### 5. Splunk Observability Cloud
- APM (Application Performance Monitoring)
- Infrastructure monitoring, log management for DevOps

---

## Splunk Architecture Overview

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Data Sources│────▶│   Forwarder   │────▶│   Indexer    │
│  (logs,events│     │ (collects &   │     │ (stores &    │
│   metrics)   │     │  forwards)    │     │  indexes)    │
└──────────────┘     └──────────────┘     └──────┬───────┘
                                                  │
                                    ┌─────────────▼──────────┐
                                    │      Search Head        │
                                    │  (search, dashboards,   │
                                    │   alerts, reports)      │
                                    └────────────────────────┘
```

### Core Components

| Component | Role |
|-----------|------|
| **Forwarder** | Lightweight agent that collects and sends data to indexers |
| **Indexer** | Receives, parses, and stores data in indexes |
| **Search Head** | User-facing interface for running searches, dashboards, alerts |
| **Deployment Server** | Manages configuration of forwarders at scale |
| **Cluster Master** | Manages indexer clusters (high availability) |
| **License Master** | Controls licensing across the deployment |

### Forwarder Types

| Type | Description |
|------|-------------|
| **Universal Forwarder (UF)** | Lightweight, forwards raw data only — most common |
| **Heavy Forwarder (HF)** | Full Splunk instance, can parse/filter data before forwarding |

---

## The Splunk Data Pipeline

```
Input → Parsing → Indexing → Searching
```

| Stage | What Happens |
|-------|-------------|
| **Input** | Data collected from sources (files, network, APIs) |
| **Parsing** | Data broken into events, timestamps extracted, fields identified |
| **Indexing** | Events stored with metadata in compressed index files |
| **Searching** | SPL queries executed against indexed data |

---

## Splunk Web Interface Overview

When you log into Splunk, you see:

| Section | Description |
|---------|-------------|
| **Home** | App launcher and summary |
| **Search & Reporting** | Main search interface — where you run SPL queries |
| **Dashboards** | View and build visualisations |
| **Alerts** | Set up and manage alert conditions |
| **Reports** | Saved searches run on a schedule |
| **Settings** | Configure indexes, inputs, users, roles |

---

## Key Terminology

| Term | Definition |
|------|-----------|
| **Event** | A single record of data (one log line) |
| **Index** | A repository where data is stored (like a database table) |
| **Source** | The file or input path where data came from |
| **Source Type** | The data format category (syslog, access_combined, wineventlog) |
| **Host** | The machine that generated the data |
| **Field** | A key-value pair extracted from an event |
| **SPL** | Search Processing Language — Splunk's query language |
| **App** | A packaged set of dashboards, searches, and configs for a use case |
| **Add-on (TA)** | Technology Add-on — provides data ingestion and parsing for a specific source |

---

## Splunk Certifications (Learning Path)

| Cert | Level |
|------|-------|
| **Splunk Core Certified User** | Beginner — basic searching and dashboards |
| **Splunk Core Certified Power User** | Intermediate — advanced SPL, data models |
| **Splunk Core Certified Advanced Power User** | Advanced — complex searches, acceleration |
| **Splunk Enterprise Certified Admin** | Admin — installation, configuration, management |
| **Splunk Enterprise Certified Architect** | Expert — large-scale deployments |
| **Splunk Certified Cybersecurity Defense Analyst** | Security — SOC analyst skills |

---

## Practice Questions

1. What is the primary function of a Splunk indexer?
   - A) Display dashboards  B) Collect data from sources  C) Store and index incoming data ✓  D) Run SPL searches

2. Which forwarder type is lightweight and only forwards raw data?
   - A) Heavy Forwarder  B) Universal Forwarder ✓  C) Search Head  D) Deployment Server

3. What does SPL stand for?
   - A) Splunk Processing Log  B) Search Processing Language ✓  C) System Protocol Layer  D) Search Program Logic

4. Which Splunk component provides the user interface for running searches and viewing dashboards?
   - A) Indexer  B) Forwarder  C) Search Head ✓  D) Deployment Server
