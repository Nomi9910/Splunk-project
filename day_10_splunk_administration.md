# Day 10 — Splunk Administration

## Introduction
This lesson covers essential Splunk administration tasks — user management, roles, configuration files, deployment management, and performance tuning. Understanding these is required for the Splunk Admin certification and for any analyst working in an enterprise environment.

---

## Users and Roles

### Role-Based Access Control (RBAC)
Splunk uses **roles** to control what users can see and do.

### Built-In Roles

| Role | Capabilities |
|------|-------------|
| **admin** | Full access — all data, all settings, all capabilities |
| **power** | Create alerts, reports, dashboards; share searches |
| **user** | Run searches on assigned indexes, create private dashboards |
| **can_delete** | Allows use of `delete` command (delete specific events) |

### Creating Custom Roles
```
Settings → Access Controls → Roles → New Role
→ Name: soc_analyst
→ Indexes: security, winevent, network (assign specific indexes)
→ Inherited roles: user
→ Capabilities: schedule_search, create_dashboard, use_splunk_search_jobs_for_workload_management
```

### User Management
```
Settings → Access Controls → Users → New User
→ Username, email, password
→ Assign roles: soc_analyst, user
→ Default app: Splunk Enterprise Security
```

---

## Configuration Files

Splunk is configured through `.conf` files in `$SPLUNK_HOME/etc/system/local/` and app directories.

### Key Configuration Files

| File | Purpose |
|------|---------|
| `inputs.conf` | Define data inputs (monitor, network, HEC) |
| `outputs.conf` | Configure forwarding (forwarder → indexer) |
| `indexes.conf` | Define and configure data indexes |
| `props.conf` | Data parsing: sourcetype settings, field extractions |
| `transforms.conf` | Lookup definitions, field transformations |
| `savedsearches.conf` | Saved searches, alerts, reports |
| `authentication.conf` | Authentication methods (LDAP, SAML) |
| `authorize.conf` | Role capabilities and permissions |
| `server.conf` | Splunk server settings (licensing, clustering) |
| `web.conf` | Web UI settings (port, SSL) |
| `limits.conf` | Search limits and resource constraints |

### Configuration File Precedence (Lowest to Highest)
```
$SPLUNK_HOME/etc/system/default/  (default, never edit)
$SPLUNK_HOME/etc/system/local/    (system-level overrides)
$SPLUNK_HOME/etc/apps/<appname>/  (app-level settings)
```

Higher precedence overrides lower. Always put customisations in `/local/`, never edit `/default/`.

---

## Deployment Server

Manages configuration deployment to Universal Forwarders at scale.

```
Deployment Server → Deployment Apps → Deployed to Forwarder Groups
```

### Deployment Apps
Each app contains configuration files (inputs.conf, outputs.conf) for specific use cases:
- `uf_outputs` — tells forwarders which indexers to send data to
- `uf_linux_inputs` — monitors specific log files on Linux servers
- `uf_windows_inputs` — monitors Windows Event Logs

### Forwarder Client Setup
```bash
# On the Universal Forwarder:
$SPLUNK_HOME/bin/splunk set deploy-poll splunk-deployment-server:8089
$SPLUNK_HOME/bin/splunk restart
```

### Server Classes
Group forwarders by role for targeted app deployment:
```
Settings → Forwarder Management → Server Classes
→ Class: Windows_Servers (filter: OS=Windows)
   → Deployed apps: uf_outputs, uf_windows_inputs

→ Class: Linux_Web_Servers (filter: hostname=web*)
   → Deployed apps: uf_outputs, uf_linux_inputs, uf_apache_inputs
```

---

## Indexer Clustering (High Availability)

**Indexer Cluster** provides replication and fault tolerance:

| Component | Role |
|-----------|------|
| **Cluster Master (Manager)** | Coordinates indexers, maintains cluster state |
| **Peer Nodes** | Indexers that store replicated data |
| **Replication Factor** | How many copies of data to keep (default: 3) |
| **Search Factor** | How many searchable copies to keep (default: 2) |

**Smart Store** — offloads cold data to S3/blob storage while keeping hot/warm locally.

---

## Search Head Clustering (SHC)

Provides high availability for the search tier:
- Multiple search heads operate as a cluster
- Jobs distributed across search heads
- Shared configurations, scheduled searches, alerts
- **Captain** coordinates job distribution

---

## Index Management

### Checking Index Volume
```spl
| dbinspect index=security
| stats sum(sizeOnDiskMB) AS total_mb

| rest /services/data/indexes
| table title, totalEventCount, currentDBSizeMB, maxTotalDataSizeMB
```

### Bucket Management
```bash
# List buckets in an index
$SPLUNK_HOME/bin/splunk list index security

# Move cold buckets to archive (freeze)
$SPLUNK_HOME/bin/splunk offline --enforce-counts
```

### Data Retention
```ini
# indexes.conf
[security]
frozenTimePeriodInSecs = 7776000   # 90 days
maxTotalDataSizeMB = 500000        # 500 GB max
```

---

## Splunk Licence Management

### Licence Types
| Type | Daily Volume |
|------|-------------|
| **Enterprise Trial** | 500 MB/day for 60 days |
| **Enterprise Licence** | As purchased |
| **Free** | 500 MB/day, no auth, 1 user |
| **Dev/Test** | 10 GB/day for testing |

### Monitoring Licence Usage
```spl
# Check daily licence usage
index=_internal source=*license_usage.log type=Usage
| timechart span=1d sum(b) AS bytes_indexed

# Licence usage by index
index=_internal source=*license_usage.log type=Usage
| stats sum(b) AS bytes_indexed by idx
| sort -bytes_indexed
```

---

## Splunk Health and Performance Monitoring

### Internal Logs
```spl
# Splunk search performance
index=_internal sourcetype=splunkd component=SearchScheduler
| stats avg(run_time) AS avg_runtime by search_id
| sort -avg_runtime

# Indexing rate
index=_internal sourcetype=splunkd component=Metrics name=index_thruput
| timechart avg(kbps) AS avg_kbps

# Search errors
index=_internal sourcetype=splunkd log_level=ERROR
| stats count by component, message
| sort -count
```

---

## Practice Questions

1. Which built-in Splunk role provides full access to all data and all system settings?
   - A) power  B) user  C) admin ✓  D) can_delete

2. Which configuration file defines how Splunk parses incoming data and extracts fields for a specific source type?
   - A) inputs.conf  B) transforms.conf  C) props.conf ✓  D) outputs.conf

3. Which Splunk component is responsible for deploying configuration apps to Universal Forwarders?
   - A) Cluster Master  B) Deployment Server ✓  C) Search Head  D) Heavy Forwarder

4. Where should custom Splunk configuration file changes always be placed?
   - A) $SPLUNK_HOME/etc/system/default/  B) $SPLUNK_HOME/etc/system/local/ ✓  C) $SPLUNK_HOME/bin/  D) $SPLUNK_HOME/var/

5. Which index contains Splunk's own internal operational logs?
   - A) main  B) summary  C) _audit  D) _internal ✓
