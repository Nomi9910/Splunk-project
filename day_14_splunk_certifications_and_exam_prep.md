# Day 14 — Splunk Certifications & Exam Prep

## Introduction
This lesson is your complete guide to Splunk certifications — what they cover, how to prepare, and the key concepts most likely to appear on each exam.

---

## Splunk Certification Path

```
┌──────────────────────────────────────────────────────────┐
│           SPLUNK CERTIFICATION LADDER                    │
├──────────────────────────────────────────────────────────┤
│  BEGINNER                                                │
│  ├── Splunk Core Certified User (SCCU)                   │
│  │   Basic searching, dashboards, alerts                 │
│                                                          │
│  INTERMEDIATE                                            │
│  ├── Splunk Core Certified Power User (SCCPU)            │
│  │   Advanced SPL, data models, macros, workflow actions │
│                                                          │
│  ADVANCED                                                │
│  ├── Splunk Core Certified Advanced Power User           │
│  │   Complex SPL, summary indexing, acceleration         │
│                                                          │
│  ADMIN                                                   │
│  ├── Splunk Enterprise Certified Admin (SECA)            │
│  │   Installation, config files, clustering, indexing   │
│                                                          │
│  ARCHITECT                                               │
│  ├── Splunk Enterprise Certified Architect (SECA2)       │
│  │   Large-scale deployment design                       │
│                                                          │
│  SECURITY SPECIALIST                                     │
│  ├── Splunk Certified Cybersecurity Defense Analyst      │
│  │   SOC operations, ES, threat detection, investigation │
│  └── Splunk Certified SOAR Automation Developer          │
│      Playbook development, automation                    │
└──────────────────────────────────────────────────────────┘
```

---

## Splunk Core Certified User (SCCU)

**Exam:** SPLK-1001  
**Prerequisites:** None  
**Cost:** ~$130 USD  
**Time:** 60 minutes, 65 questions  
**Passing score:** 68%  

### Topics Covered

| Domain | Weight |
|--------|--------|
| Splunk Basics | 10% |
| Basic Searching | 25% |
| Using Fields | 15% |
| Search Results Visualisation | 15% |
| Alerts and Reports | 20% |
| Statistical Commands | 15% |

### Key Topics to Master

**Searching:**
- Search syntax: field=value, wildcards, boolean operators
- Time modifiers: `-1h`, `-7d`, `@d`, `@h`
- `index=`, `sourcetype=`, `host=` filters
- `| head`, `| tail`, `| sort`, `| table`, `| fields`

**Statistical Commands:**
- `stats count by`, `stats avg()`, `stats sum()`, `stats dc()`
- `timechart span=1h count`
- `top 10 field`
- `rare 5 field`

**Fields:**
- Metadata fields: `_time`, `host`, `source`, `sourcetype`, `index`
- Field extraction with `rex`
- Renaming fields with `rename`

**Dashboards:**
- Create dashboards from search results
- Visualisation types: line, bar, pie, single value, table
- Dashboard tokens and time range pickers

**Alerts:**
- Alert types: Scheduled vs Real-Time
- Trigger conditions
- Alert actions: email, webhook, script

---

## Splunk Core Certified Power User (SCCPU)

**Exam:** SPLK-2001  
**Prerequisites:** Core Certified User (recommended)  
**Cost:** ~$130 USD  
**Time:** 60 minutes, 65 questions  
**Passing score:** 70%  

### Key Additional Topics

**Advanced SPL:**
- `transaction` command with `startswith`, `endswith`, `maxspan`
- `join`, `append`, `appendcols`
- Subsearches
- `eventstats`, `streamstats`
- `mvexpand`, `mvjoin`, `mvcount`
- Advanced `eval` functions

**Field Extractions:**
- `rex` command: named capture groups `(?P<name>pattern)`
- `props.conf` EXTRACT- stanza
- Field aliases with `FIELDALIAS-`
- Calculated fields with `EVAL-`

**Data Models:**
- Creating and editing data models
- Data model acceleration
- Using `tstats` with data models
- Pivot interface

**Macros and Event Types:**
- Creating macros with arguments
- Event types and tagging
- Workflow actions

**Lookups:**
- CSV lookups: `inputlookup`, `outputlookup`, `lookup`
- Automatic lookups via props.conf
- Lookup with CIDR ranges
- KV Store lookups

---

## Splunk Enterprise Certified Admin (SECA)

**Exam:** SPLK-2003  
**Cost:** ~$130 USD  
**Time:** 60 minutes, 65 questions  
**Passing score:** 66%  

### Key Topics

| Domain | Key Points |
|--------|-----------|
| **Architecture** | Components (forwarder, indexer, search head), clustering |
| **Installation** | Install Enterprise, configure limits, directories |
| **Getting Data In** | inputs.conf, forwarder management, HEC |
| **Index Management** | indexes.conf, retention, bucket management |
| **Users and Roles** | RBAC, custom roles, LDAP integration |
| **Config Files** | precedence, props.conf, transforms.conf, outputs.conf |
| **Deployment Server** | Server classes, apps, forwarder groups |
| **Search Performance** | Scheduling, concurrent searches, limits.conf |

---

## Key Concepts Summary for All Exams

### SPL Command Categories

| Category | Commands |
|----------|---------|
| **Filtering** | `search`, `where`, `head`, `tail`, `dedup` |
| **Reporting** | `stats`, `chart`, `timechart`, `top`, `rare`, `addtotals` |
| **Modifying** | `eval`, `rex`, `rename`, `fields`, `sort` |
| **Combining** | `append`, `join`, `appendcols`, `transaction` |
| **Lookup** | `lookup`, `inputlookup`, `outputlookup` |
| **Display** | `table`, `format` |

### Configuration File Quick Reference

| File | Stanza | Key Setting |
|------|--------|------------|
| inputs.conf | `[monitor://path]` | `sourcetype`, `index` |
| outputs.conf | `[tcpout]` | `server`, `group` |
| props.conf | `[sourcetype]` | `EXTRACT-`, `TIME_FORMAT`, `FIELDALIAS-` |
| transforms.conf | `[lookup_name]` | `filename`, `fields_list` |
| indexes.conf | `[index_name]` | `frozenTimePeriodInSecs`, `maxTotalDataSizeMB` |
| savedsearches.conf | `[alert_name]` | `search`, `alert.condition`, `cron_schedule` |

### Metadata Fields Always Available
```
_time     → event timestamp
host      → machine that sent the data
source    → file path or input
sourcetype→ data format type
index     → storage index
_raw      → original event text
```

---

## Top 20 Must-Know SPL Commands

```spl
stats, timechart, chart      # Aggregation / reporting
eval                          # Create/modify fields
rex                           # Extract fields with regex
sort, head, tail              # Ordering and limiting
fields, table                 # Field selection
where, search                 # Filtering
dedup                         # Remove duplicates
top, rare                     # Frequency analysis
lookup, inputlookup, outputlookup  # Lookup operations
transaction                   # Group related events
join, append                  # Combine searches
rename                        # Rename fields
bucket                        # Time bucketing
tstats                        # Fast metadata stats
makeresults                   # Create test events
mvexpand                      # Expand multivalue fields
streamstats, eventstats       # Running/inline stats
```

---

## Study Plan Recommendations

```
Week 1 — Foundations (Days 1-5)
  □ Complete Splunk Fundamentals 1 (free on Splunk Education)
  □ Set up Splunk free trial or developer instance
  □ Practice basic searches daily on sample data
  □ Understand all metadata fields

Week 2 — Intermediate (Days 6-10)
  □ Complete Splunk Fundamentals 2
  □ Build 3 dashboards from scratch
  □ Create alerts with different trigger conditions
  □ Practice stats/timechart/chart commands

Week 3 — Advanced (Days 11-14)
  □ Learn props.conf and transforms.conf
  □ Understand data models and CIM
  □ Practice advanced SPL: transaction, join, subsearch, tstats
  □ Take practice exams from Splunk Education

Week 4 — Exam Prep
  □ Complete Splunk practice exam (80%+ before sitting real exam)
  □ Review any weak areas from practice results
  □ Review official exam objectives
  □ Schedule and sit the exam
```

---

## Free Splunk Learning Resources

| Resource | URL |
|----------|-----|
| Splunk Education | education.splunk.com (free courses) |
| Splunk Fundamentals 1 & 2 | Free self-paced on education.splunk.com |
| Splunk Documentation | docs.splunk.com |
| Splunk Developer Trial | splunk.com/en_us/download.html (90-day trial) |
| Splunk Community | community.splunk.com |
| Splunk YouTube | youtube.com/splunk |
| Boss of the SOC (BOTS) | Hands-on CTF challenges with Splunk |
| TryHackMe Splunk rooms | tryhackme.com |
