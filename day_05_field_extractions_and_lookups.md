# Day 5 — Field Extractions & Lookups

## Introduction
Fields are the heart of Splunk searches. By default, Splunk automatically extracts many fields from common source types. This lesson teaches you how to extract custom fields and enrich your data with lookups.

---

## How Splunk Extracts Fields

Splunk extracts fields in two ways:

### 1. Index-Time Extraction (at Indexing)
- Performed once when data is indexed
- Stored with the event
- Fast at search time
- Used for: host, source, sourcetype, _time, and custom index-time fields

### 2. Search-Time Extraction (at Search)
- Applied to matching events when a search runs
- More flexible — no re-indexing needed to change
- Stored in props.conf and transforms.conf (or via Field Extractor UI)
- **Most custom field extractions are search-time**

---

## Automatic Field Extraction

For common source types, Splunk automatically extracts known fields:

```
Apache Access Log event:
192.168.1.100 - frank [25/Jun/2025:14:23:45] "GET /index.html HTTP/1.1" 200 2326

Auto-extracted fields:
  clientip = 192.168.1.100
  user = frank
  method = GET
  uri_path = /index.html
  status = 200
  bytes = 2326
```

---

## The Field Extractor (FX) — UI Method

The easiest way to create custom field extractions:

```
Settings → Fields → Field Extractions → Open Field Extractor
  1. Select your sourcetype
  2. Pick a sample event
  3. Highlight the value you want to extract
  4. Name the field
  5. Splunk auto-generates the regex
  6. Validate and save
```

---

## Manual Field Extraction with rex

Use `rex` in searches to extract fields using regular expressions (inline, no configuration needed):

```spl
# Extract a field named 'username' from a log event
index=app
| rex field=_raw "user=(?P<username>[a-zA-Z0-9_]+)"

# Extract IP address
index=web
| rex "(?P<client_ip>(?:\d{1,3}\.){3}\d{1,3})"

# Extract multiple fields
index=app
| rex "action=(?P<action>\w+)\s+resource=(?P<resource>\S+)"
```

**Named capture group syntax:** `(?P<fieldname>regex_pattern)`

---

## Persistent Field Extraction via props.conf

For permanent extractions (applied to all searches for a source type):

```ini
# props.conf - using EXTRACT-
[my_app_logs]
EXTRACT-username = user=(?P<username>[a-zA-Z0-9_]+)
EXTRACT-action = action=(?P<action>\w+)

# Using REPORT- (references transforms.conf)
[my_app_logs]
REPORT-user_fields = extract_user_and_action
```

```ini
# transforms.conf
[extract_user_and_action]
REGEX = user=(?P<username>[a-zA-Z0-9_]+)\s+action=(?P<action>\w+)
```

---

## Lookups — Enriching Data

**Lookups** add external reference data to search results — enriching events with information not in the original logs.

### Types of Lookups

| Type | Description |
|------|-------------|
| **CSV Lookup** | Static CSV file with reference data |
| **KV Store Lookup** | Key-value store for dynamic data |
| **External Lookup** | Python script queries external source |
| **Geospatial Lookup** | Map IPs to lat/lon coordinates |
| **CIDR Lookup** | Match IPs to network ranges |

---

### CSV Lookup Example

**Step 1: Create the CSV file** (`user_info.csv`):
```
username,department,location,role
alice,Security,London,analyst
bob,IT,New York,admin
charlie,HR,Sydney,manager
```

**Step 2: Upload to Splunk:**
```
Settings → Lookups → Lookup table files → Add New
→ Upload user_info.csv
→ Destination app: search
```

**Step 3: Define the lookup:**
```
Settings → Lookups → Lookup definitions → Add New
→ Name: user_info
→ File: user_info.csv
→ Lookup table fields: username
```

**Step 4: Use in a search:**
```spl
index=security
| lookup user_info username OUTPUT department, location, role
| table _time, username, department, location, action
```

---

### Automatic Lookups

Configure lookups to fire automatically on matching events — no need to add `| lookup` to every search:

```ini
# transforms.conf
[user_info]
filename = user_info.csv

# props.conf
[linux_secure]
LOOKUP-user_details = user_info username OUTPUT department location role
```

Now every linux_secure event automatically gets department, location, and role fields added.

---

### lookup with CIDR Ranges

```spl
# lookup that maps IP ranges to network names
# ip_ranges.csv:
# network_cidr,network_name,environment
# 10.0.0.0/8,Internal,Production
# 192.168.0.0/16,VPN,Corporate

| lookup ip_ranges network_cidr AS src_ip OUTPUT network_name, environment CIDR
```

---

## inputlookup — Search a Lookup Table Directly

```spl
# View the contents of a lookup table
| inputlookup user_info.csv

# Search within a lookup
| inputlookup user_info.csv WHERE department="Security"

# Join lookup results with search results
index=security
| stats count by username
| inputlookup append=true user_info.csv
```

---

## outputlookup — Save Search Results to a Lookup

```spl
# Save current search results as a lookup table
index=web status=404
| stats count by uri_path
| outputlookup blocked_paths.csv

# Update a lookup with new data
index=security
| stats dc(src_ip) AS unique_ips by user
| outputlookup update=true suspicious_users.csv
```

---

## Field Aliases

Map alternative field names to existing fields (without modifying data):

```ini
# props.conf
[access_combined]
FIELDALIAS-src = clientip AS src_ip, src AS src_ip
```

Now both `clientip` and `src` queries work as `src_ip` in searches.

---

## Calculated Fields (via eval)

Define commonly used calculated fields to be added at search time:

```ini
# props.conf
[access_combined]
EVAL-response_kb = bytes / 1024
EVAL-is_error = if(status>=400, "true", "false")
```

---

## Practice Questions

1. Which SPL command extracts fields using regular expressions within a search?
   - A) `eval`  B) `fields`  C) `rex` ✓  D) `extract`

2. What is the purpose of a Splunk lookup?
   - A) To delete duplicate events  B) To add reference data to events from external files ✓  C) To extract fields using regex  D) To create dashboards

3. Which command saves the results of a search to a CSV lookup file?
   - A) `inputlookup`  B) `writeLookup`  C) `outputlookup` ✓  D) `savelookup`

4. A search-time extraction defined in props.conf uses which prefix to define a regex extraction?
   - A) `SEARCH-`  B) `EXTRACT-` ✓  C) `FIELD-`  D) `REX-`

5. Which command allows you to directly search the contents of a lookup table?
   - A) `outputlookup`  B) `lookup`  C) `inputlookup` ✓  D) `readlookup`
