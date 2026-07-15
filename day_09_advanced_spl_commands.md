# Day 9 — Advanced SPL Commands

## Introduction
This lesson covers powerful SPL commands that take your searches to the next level — correlation, time-based analysis, subsearches, data manipulation, and performance optimisation.

---

## transaction — Group Related Events

`transaction` groups related events into a single event (a "transaction") based on a shared field value or time boundaries.

```spl
# Group login/logout events per user into sessions
index=auth
| transaction user startswith="Login" endswith="Logout" maxspan=8h

# Group all events per session ID within 5 minutes
index=web
| transaction sessionid maxspan=5m maxpause=30s

# Output:
# - duration: how long the transaction lasted
# - eventcount: how many events in the transaction
# - All raw events merged into _raw
```

**Use cases:**
- Session analysis (login → activity → logout)
- Multi-step attack detection (scan → exploit → persistence)
- Call centre workflows (call start → action → resolution)

---

## join — Combine Two Search Results

`join` merges results from a subsearch with the main search on a common field — like SQL JOIN.

```spl
# Inner join: only events that match in both
index=web
| join src_ip [
    search index=threat_intel | table ip, threat_type, severity
  ]
| table _time, src_ip, uri_path, threat_type, severity

# Left join: all web events, threat info where available
index=web
| join type=left src_ip [
    search index=threat_intel | table ip, threat_type
  ]
```

**Warning:** `join` is resource-intensive. Prefer `lookup` for static data and `append + stats` for large datasets.

---

## append — Combine Two Separate Searches

`append` adds results from a subsearch to the bottom of the main search results:

```spl
# Combine web and API error events
index=web status>=400 | table _time, host, status, uri
| append [
    search index=api status=error | table _time, host, status, endpoint
  ]
| sort -_time
```

---

## appendcols — Add Columns from a Second Search

`appendcols` adds columns (fields) from a second search to the right of the main results:

```spl
index=web | stats count AS web_errors by host
| appendcols [
    search index=api | stats count AS api_errors by host
  ]
```

---

## Subsearches

A **subsearch** is a search within a search (in square brackets). The inner search runs first, and its results are used to filter or enrich the outer search.

```spl
# Find web activity from IPs that had failed logins
index=web
| where src_ip IN [
    search index=security EventCode=4625 | stats count by src_ip | where count>5 | fields src_ip
  ]

# Find users who logged in from flagged IPs
index=auth
| where src_ip IN [
    search index=threat_intel threat_level=high | fields ip
  ]
```

**Performance tip:** Subsearches are limited to 10,000 results by default. Use `lookup` for large datasets instead.

---

## makeresults — Create Synthetic Events (Testing)

Useful for testing SPL logic without real data:

```spl
# Create a synthetic event
| makeresults
| eval host="test-server", status=404, bytes=1234

# Create multiple synthetic events
| makeresults count=5
| eval host="host-" + tostring(random() % 3), status=if(random()%2=0,200,404)
```

---

## map — Apply a Search per Event

`map` runs a search for EACH event in the results (powerful but slow — use sparingly):

```spl
index=security
| top 5 src_ip
| map search="search index=web src_ip=$src_ip$ | stats count by uri_path"
```

---

## mvexpand, mvjoin, and multival Fields

**Multivalue fields** contain multiple values in one field.

```spl
# Split a multivalue field into separate events
index=web
| stats values(uri_path) AS all_paths by src_ip
| mvexpand all_paths              # One event per URI path per src_ip

# Join multivalue field into a delimited string
| eval path_list = mvjoin(all_paths, ", ")

# Count values in a multivalue field
| eval path_count = mvcount(all_paths)

# Filter by multivalue field membership
| where mvfind(all_paths, "/admin") >= 0
```

---

## tstats — Ultra-Fast Statistics on Indexed Fields

`tstats` performs statistics on **indexed fields only** (metadata: host, source, sourcetype, _time, and indexed extracted fields). It bypasses the normal search pipeline for extreme speed.

```spl
# Count events per sourcetype per day (very fast)
| tstats count BY sourcetype span=1d

# Count events per host without reading raw events
| tstats count WHERE index=web BY host, sourcetype

# Statistics from data model (used with accelerated data models)
| tstats sum(Web.bytes) AS total_bytes BY Web.src, Web.uri_path
  FROM datamodel=Web WHERE Web.status>=400
```

---

## eval Functions Reference

```spl
# String functions
| eval upper_host = upper(host)
| eval lower_status = lower(status)
| eval trimmed = trim(field, " ")
| eval length = len(uri_path)
| eval sub = substr(uri_path, 1, 10)      # First 10 chars
| eval replaced = replace(uri_path, "/old/", "/new/")

# Math functions
| eval kb = bytes / 1024
| eval rounded = round(bytes/1024, 2)
| eval abs_val = abs(delta)
| eval log_val = log(bytes, 10)           # Log base 10
| eval power = pow(2, 10)                 # 2^10 = 1024

# Time functions
| eval epoch = _time
| eval human_time = strftime(_time, "%Y-%m-%d %H:%M:%S")
| eval from_string = strptime("2025-06-25 14:30:00", "%Y-%m-%d %H:%M:%S")
| eval hour_of_day = strftime(_time, "%H")
| eval day_of_week = strftime(_time, "%A")

# Conditional functions
| eval result = if(status=200, "OK", "ERROR")
| eval category = case(
    status < 300, "Success",
    status < 400, "Redirect",
    status < 500, "Client Error",
    true(), "Server Error"
  )
| eval has_error = if(match(uri_path, "(?i)error"), 1, 0)
| eval coalesced = coalesce(user, username, "anonymous")

# Type functions
| eval num = tonumber(bytes_str)
| eval str = tostring(status)
| eval is_num = isnull(tonumber(field))
| eval null_check = isnull(username)
```

---

## where vs search (Filtering After Pipe)

```spl
# search command — uses Splunk's boolean syntax (fast)
| search status=404 host=web*

# where command — uses SPL expression syntax (more flexible)
| where status=404 AND match(host, "web.*")
| where len(uri_path) > 100
| where _time > relative_time(now(), "-1h")
| where isnotnull(username)
```

---

## Performance Optimisation Tips

```
✔ Use index= and sourcetype= at the start (reduces data early)
✔ Use time ranges — smallest possible window
✔ Use fields command early to limit field extraction
✔ Use tstats for pure count/sum queries on metadata
✔ Use data model acceleration for repeated heavy queries
✔ Avoid wildcards at the start of field values (*term)
✔ Use lookup instead of subsearches for large reference datasets
✔ Schedule expensive reports during off-peak hours
✔ Use summary indexing for long-term trend data
```

---

## Practice Questions

1. Which SPL command groups related events (like login → logout) into single events based on shared field values?
   - A) `stats`  B) `join`  C) `transaction` ✓  D) `append`

2. Which command creates synthetic test events in a search without needing real indexed data?
   - A) `inputlookup`  B) `makeresults` ✓  C) `append`  D) `mvcombine`

3. Which SPL command runs statistics directly against indexed metadata fields for extremely fast results?
   - A) `stats`  B) `tstats` ✓  C) `eventstats`  D) `streamstats`

4. A search returns a multivalue field with multiple URI paths. Which command splits this into separate events?
   - A) `mvjoin`  B) `mvexpand` ✓  C) `mvsplit`  D) `expand`

5. Which eval function returns the first non-null value from a list of fields?
   - A) `if`  B) `case`  C) `coalesce` ✓  D) `isnull`
