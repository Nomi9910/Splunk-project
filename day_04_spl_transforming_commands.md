# Day 4 — SPL: Transforming Commands (stats, chart, timechart, top, rare)

## Introduction
**Transforming commands** convert search results into statistical summaries and aggregations. They are the most powerful analytical tools in SPL — used in almost every dashboard, report, and alert.

---

## stats — Statistical Aggregation

The `stats` command is the most important transforming command. It calculates aggregate statistics and groups results.

### Syntax
```spl
| stats FUNCTION(field) [AS alias] [BY field1, field2]
```

### Common stats Functions

| Function | Description | Example |
|----------|-------------|---------|
| `count` | Count events | `stats count` |
| `count(field)` | Count non-null field values | `stats count(status)` |
| `dc(field)` | Distinct count (unique values) | `stats dc(src_ip)` |
| `sum(field)` | Sum of numeric field | `stats sum(bytes)` |
| `avg(field)` | Average | `stats avg(response_time)` |
| `min(field)` | Minimum value | `stats min(bytes)` |
| `max(field)` | Maximum value | `stats max(bytes)` |
| `stdev(field)` | Standard deviation | `stats stdev(response_time)` |
| `values(field)` | List all unique values | `stats values(status)` |
| `list(field)` | List all values (including dupes) | `stats list(uri_path)` |
| `first(field)` | First value seen | `stats first(status)` |
| `last(field)` | Last value seen | `stats last(status)` |
| `range(field)` | Max minus min | `stats range(bytes)` |
| `median(field)` | Median value | `stats median(response_time)` |
| `perc95(field)` | 95th percentile | `stats perc95(response_time)` |

### stats Examples

```spl
# Count total web events
index=web | stats count

# Count events per HTTP status
index=web | stats count by status

# Multiple functions at once
index=web
| stats count, avg(bytes) AS avg_bytes, max(bytes) AS max_bytes by status

# Count unique source IPs
index=web | stats dc(src_ip) AS unique_visitors

# Sum bandwidth per host
index=web | stats sum(bytes) AS total_bytes by host

# Complex: top users by failed logins
index=security sourcetype=wineventlog EventCode=4625
| stats count AS failed_logins by user
| sort -failed_logins
| head 10
```

---

## chart — Two-Dimensional Statistics

`chart` creates a table with one field across rows and another across columns — ideal for multi-series charts in dashboards.

```spl
# Syntax
| chart FUNCTION(field) BY row_field [OVER col_field]

# HTTP methods per status code
index=web
| chart count BY status OVER method

# Average response time per host over status
index=web
| chart avg(response_time) BY host OVER status
```

---

## timechart — Time-Series Statistics

`timechart` groups events into time buckets — perfect for trend graphs over time.

```spl
# Syntax
| timechart [span=TIME] FUNCTION(field) [BY split_field]

# Events per minute
index=web | timechart count

# Events per hour
index=web | timechart span=1h count

# HTTP errors per day
index=web status>=400 | timechart span=1d count

# Requests per minute split by status
index=web | timechart span=1m count BY status

# Average response time per hour by host
index=web | timechart span=1h avg(response_time) BY host
```

**Span options:** `1s`, `1m`, `5m`, `1h`, `1d`, `1w`, `1mon`

---

## top — Most Common Values

`top` shows the most frequently occurring values for a field — faster than `stats count | sort` for simple use cases.

```spl
# Top 10 source IPs
index=web | top src_ip

# Top 5 URI paths with percentage
index=web | top 5 uri_path

# Top 10 per host
index=web | top 10 uri_path by host

# Top 10 with custom limit
index=web | top limit=20 status
```

Output includes: count and percent columns automatically.

---

## rare — Least Common Values

`rare` is the opposite of `top` — shows the least frequently occurring values.

```spl
# Find rare user agents (could indicate bots or attackers)
index=web | rare useragent

# Rare source IPs
index=web | rare 5 src_ip

# Rarely accessed endpoints (could be forgotten/forgotten attack surfaces)
index=web | rare 10 uri_path
```

---

## eventstats — Add Stats Without Collapsing Events

Unlike `stats` which returns one row per group, `eventstats` adds calculated values as new fields to EACH event.

```spl
# Add average bytes to each event without collapsing
index=web
| eventstats avg(bytes) AS avg_bytes
| where bytes > avg_bytes * 2        # Filter events with 2x above-average bytes
```

---

## streamstats — Running Calculations

Calculates running totals, cumulative statistics, moving averages.

```spl
# Running total of bytes
index=web
| sort _time
| streamstats sum(bytes) AS running_total

# Moving average (window of 5 events)
index=web
| sort _time
| streamstats window=5 avg(bytes) AS moving_avg_bytes
```

---

## Chaining Transforming Commands

```spl
# Find hosts generating most errors above average
index=web status>=400
| stats count AS error_count by host
| eventstats avg(error_count) AS avg_errors
| where error_count > avg_errors
| sort -error_count

# Web traffic dashboard query: hourly request rate split by status class
index=web
| eval status_class = case(
    status<200, "1xx Informational",
    status<300, "2xx Success",
    status<400, "3xx Redirect",
    status<500, "4xx Client Error",
    true(), "5xx Server Error"
  )
| timechart span=1h count BY status_class
```

---

## Practice Questions

1. Which SPL command is used to count the number of events grouped by a specific field?
   - A) `top`  B) `chart`  C) `stats count by` ✓  D) `timechart`

2. A SOC analyst wants to see the number of security events over the past 24 hours plotted on a timeline. Which command should they use?
   - A) `stats`  B) `chart`  C) `top`  D) `timechart` ✓

3. Which function in `stats` counts the number of UNIQUE values in a field?
   - A) `count`  B) `values`  C) `dc` ✓  D) `list`

4. Which command shows the most frequently occurring values of a field?
   - A) `rare`  B) `stats`  C) `top` ✓  D) `head`

5. What is the key difference between `stats` and `eventstats`?
   - A) `stats` supports more functions  B) `eventstats` does not aggregate; it adds stats as fields to each event without collapsing ✓  C) `eventstats` is faster  D) `stats` requires a BY clause
