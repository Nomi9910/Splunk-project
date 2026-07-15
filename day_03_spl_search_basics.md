# Day 3 — SPL: Search Processing Language Basics

## Introduction
**SPL (Search Processing Language)** is Splunk's powerful query language. Everything in Splunk — searches, reports, dashboards, alerts — is built on SPL. This lesson covers the fundamentals every Splunk user must know.

---

## The Search Bar

When you open Splunk Search & Reporting, you see the search bar at the top. SPL queries are typed here and executed against your indexed data.

```
index=web | stats count by status | sort -count
```

**Search anatomy:**
```
[Search Terms]  |  [Command 1]  |  [Command 2]  |  [Command N]
```
- Start with search terms to filter events
- Use `|` (pipe) to pass results to commands
- Chain commands left to right

---

## Time Range

Every search in Splunk requires a time range. Always limit your time range — searching all time on large indexes is very slow.

```
# In the UI: Use the time picker (Last 24 hours, Last 7 days, etc.)

# In SPL:
index=web earliest=-24h latest=now
index=web earliest=-1h@h latest=@h    # Last complete hour
index=web earliest="06/25/2025:00:00:00" latest="06/25/2025:23:59:59"
```

**Time modifiers:**
| Modifier | Meaning |
|----------|---------|
| `-15m` | 15 minutes ago |
| `-1h` | 1 hour ago |
| `-7d` | 7 days ago |
| `-1d@d` | Start of yesterday |
| `@h` | Snap to current hour |
| `@d` | Snap to start of day |
| `@w0` | Snap to Sunday |
| `@mon` | Snap to start of month |

---

## Basic Search Syntax

### Keyword Search
```spl
error
"connection refused"          # Exact phrase in quotes
error OR warning              # Boolean OR
error NOT timeout             # Exclude keyword
error AND (timeout OR refused)# Grouped boolean
```

### Field-Based Search
```spl
status=404                    # Field equals value
status!=200                   # Field not equal
bytes>10000                   # Numeric comparison
bytes>=1000 bytes<=5000       # Range
host="web-server-01"          # String field
host=web*                     # Wildcard
sourcetype=access_combined    # Search specific sourcetype
index=security                # Search specific index
```

### Combining Filters
```spl
index=web status=404 method=GET
index=web status>=500 host=web*
index=security sourcetype=wineventlog EventCode=4625
```

---

## The Search Pipeline

SPL processes data through a pipeline using the `|` operator:

```spl
index=web status=404
| stats count by uri_path
| sort -count
| head 10
```

Reading this left to right:
1. Get all 404 events from the web index
2. Count how many times each URI path appeared
3. Sort by count, highest first
4. Show only the top 10 results

---

## Essential SPL Commands

### fields — Select or Remove Fields
```spl
index=web
| fields status method uri_path bytes        # Keep only these fields

index=web
| fields - _raw _indextime                   # Remove these fields (- prefix)
```

### rename — Rename a Field
```spl
index=web
| rename status AS "HTTP Status" uri_path AS "URL"
```

### table — Display Specific Fields as a Table
```spl
index=web
| table _time host status uri_path bytes     # Show as clean table
```

### dedup — Remove Duplicate Events
```spl
index=web
| dedup src_ip                               # Keep first event per unique src_ip

index=web
| dedup 3 uri_path                           # Keep up to 3 events per uri_path
```

### sort — Sort Results
```spl
index=web
| sort status                                # Ascending (A-Z, 0-9)
| sort -status                               # Descending (Z-A, 9-0)
| sort -count +host                          # Multiple fields
```

### head and tail — Limit Results
```spl
index=web | head 20                          # First 20 results
index=web | tail 5                           # Last 5 results
```

### where — Filter Results After Search
```spl
index=web
| stats count by host
| where count > 100                          # Only hosts with >100 events
```

### eval — Create or Modify Fields
```spl
index=web
| eval response_kb = bytes / 1024            # Create new calculated field
| eval status_label = if(status=200, "OK", "ERROR")  # Conditional field
| eval full_url = host + uri_path            # String concatenation
```

### rex — Extract Fields Using Regex
```spl
index=web
| rex field=_raw "user=(?P<username>\w+)"   # Extract 'username' field

index=web
| rex "(?P<ip_addr>\d+\.\d+\.\d+\.\d+)"    # Extract IP from raw event
```

---

## Boolean and Comparison Operators

| Operator | Use | Example |
|----------|-----|---------|
| `AND` | Both conditions true | `status=404 AND method=GET` |
| `OR` | Either condition true | `status=404 OR status=500` |
| `NOT` | Negate condition | `NOT status=200` |
| `=` | Equal | `host="web-01"` |
| `!=` | Not equal | `status!=200` |
| `<` `>` | Less/greater than | `bytes>10000` |
| `<=` `>=` | Less/greater or equal | `bytes>=1024` |

---

## Wildcards

```spl
host=web*           # Matches web-01, web-02, webserver, etc.
sourcetype=*log*    # Any sourcetype containing "log"
uri_path=/api/*     # Any path starting with /api/
```

**Note:** Wildcards at the START of a value (`*error`) are very slow — avoid them.

---

## Search Best Practices

```
✔ Always scope your search with index= and sourcetype=
✔ Use specific time ranges — never "All time" on production
✔ Filter early (before the pipe) to reduce data volume
✔ Use fields command to limit field extraction overhead
✔ Test searches on short time ranges before widening
✔ Use | head 10 during development to see sample data
✔ Avoid leading wildcards (*keyword) — use keyword* instead
✔ Comment complex SPL with triple backtick notes
```

---

## Practice SPL Queries

```spl
# Find all failed SSH logins
index=linux sourcetype=linux_secure "Failed password" | table _time host user

# Top 10 most accessed URLs
index=web sourcetype=access_combined
| stats count by uri_path
| sort -count
| head 10

# Find HTTP errors in last hour
index=web status>=400 earliest=-1h
| table _time host status uri_path

# Count events per host
index=* | stats count by host | sort -count

# Find unique source IPs hitting a 404
index=web status=404
| stats dc(src_ip) as unique_ips by uri_path
| sort -unique_ips
```

---

## Practice Questions

1. What character is used to separate SPL commands in a pipeline?
   - A) `/`  B) `>`  C) `|` ✓  D) `&`

2. Which SPL command creates new calculated fields from existing data?
   - A) `fields`  B) `rex`  C) `rename`  D) `eval` ✓

3. Which time modifier would you use to search the last 6 hours?
   - A) `-6m`  B) `-6h` ✓  C) `@6h`  D) `-360`

4. A search returns too many results. Which command keeps only the top 5 rows?
   - A) `tail 5`  B) `limit 5`  C) `head 5` ✓  D) `top 5`

5. Which command removes duplicate events based on a specific field?
   - A) `unique`  B) `dedup` ✓  C) `distinct`  D) `fields -`
