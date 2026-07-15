# Day 6 — Dashboards & Visualisations

## Introduction
Dashboards transform SPL search results into visual displays — charts, tables, maps, and single-value panels — that stakeholders can monitor in real time. This lesson covers building, customising, and managing Splunk dashboards.

---

## Types of Visualisations

| Visualisation | Best For |
|--------------|---------|
| **Bar Chart** | Comparing values across categories |
| **Line Chart** | Trends over time (use with timechart) |
| **Area Chart** | Cumulative trends over time |
| **Pie Chart** | Proportions of a whole (limit to ≤5 slices) |
| **Column Chart** | Same as bar but vertical |
| **Single Value** | KPI panels — one number prominently displayed |
| **Table** | Detailed event data in rows/columns |
| **Scatter Plot** | Correlation between two numeric fields |
| **Bubble Chart** | Three-variable scatter with bubble size |
| **Heat Map** | Intensity of a value across two dimensions |
| **Choropleth Map** | Geographic regions coloured by value |
| **Cluster Map** | Points plotted on a map with lat/lon |
| **Radial Gauge** | Single value with threshold ranges |
| **Filler Gauge** | Single value filling a container |
| **Marker Gauge** | Single value on a scale |

---

## Creating a Dashboard

### From a Search Result
```
1. Run a search in Search & Reporting
2. In results panel: Visualisation tab → select chart type
3. Click "Save As" → Dashboard Panel
4. Choose existing dashboard or create new
5. Name the panel, select dashboard title
6. Save
```

### From Scratch
```
Dashboards → Create New Dashboard
→ Title, ID, permissions
→ Add Panel → choose visualisation type
→ Enter SPL query for each panel
→ Save
```

---

## Dashboard Panel Types

### Charts Panel
```spl
# Line chart: web requests per hour
index=web | timechart span=1h count

# Bar chart: top 10 error pages
index=web status>=400
| stats count by uri_path
| sort -count
| head 10
```

### Single Value Panel
```spl
# Total failed logins today
index=security sourcetype=wineventlog EventCode=4625 earliest=@d
| stats count AS "Failed Logins Today"

# Active users right now
index=web earliest=-15m
| stats dc(username) AS "Active Users"
```

### Table Panel
```spl
# Recent security events
index=security earliest=-1h
| table _time, host, user, action, result
| sort -_time
```

### Map Panel
```spl
# Geographic distribution of visitors
index=web
| iplocation src_ip
| geostats count by Country
```

---

## Dashboard XML (Simple XML)

Dashboards are stored as XML. Understanding the structure helps with advanced customisation:

```xml
<dashboard>
  <label>Web Traffic Monitor</label>

  <row>
    <panel>
      <title>Requests Per Hour</title>
      <chart>
        <search>
          <query>index=web | timechart span=1h count</query>
          <earliest>-24h</earliest>
          <latest>now</latest>
        </search>
        <option name="charting.chart">line</option>
        <option name="charting.legend.placement">bottom</option>
      </chart>
    </panel>

    <panel>
      <title>Total Requests Today</title>
      <single>
        <search>
          <query>index=web earliest=@d | stats count</query>
        </search>
        <option name="drilldown">none</option>
      </single>
    </panel>
  </row>

  <row>
    <panel>
      <title>Top 10 Error Pages</title>
      <chart>
        <search>
          <query>index=web status>=400 | stats count by uri_path | sort -count | head 10</query>
        </search>
        <option name="charting.chart">bar</option>
      </chart>
    </panel>
  </row>
</dashboard>
```

---

## Dashboard Inputs (Tokens / Filters)

Add interactive inputs to let users filter dashboards dynamically:

### Common Input Types
- **Time range picker** — let users select time window
- **Dropdown** — select from predefined options
- **Text input** — free-text filter
- **Radio button** — mutually exclusive choices
- **Checkbox** — multiple selection

### Token Example
```xml
<input type="dropdown" token="selected_host">
  <label>Select Host</label>
  <choice value="*">All Hosts</choice>
  <choice value="web-01">web-01</choice>
  <choice value="web-02">web-02</choice>
  <default>*</default>
</input>

<!-- Panel using the token -->
<query>index=web host=$selected_host$ | timechart span=1h count</query>
```

The `$selected_host$` is replaced with the dropdown selection at search time.

---

## Drilldowns

Drilldowns allow clicking on a panel element to run another search or navigate to a URL:

```xml
<drilldown>
  <link target="_blank">
    /app/search/search?q=index%3Dweb+uri_path%3D$click.value$
  </link>
</drilldown>
```

When a user clicks a bar in the chart, they're taken to a filtered search for that specific value.

---

## Dashboard Best Practices

```
✔ Name dashboards clearly — include purpose and data scope
✔ Use a consistent time range token across all panels
✔ Limit panels per dashboard — 6-8 is manageable
✔ Use Single Value panels for KPIs that need immediate attention
✔ Always set panel time ranges (don't rely on global default)
✔ Add descriptions to panels for non-technical viewers
✔ Schedule dashboard refreshes for live monitoring (auto-refresh)
✔ Use colour thresholds on single values to show red/yellow/green status
✔ Export dashboards as PDFs for executive reports
```

---

## Colour Thresholds on Single Value Panels

```xml
<single>
  <search>
    <query>index=web status>=400 earliest=-1h | stats count AS errors</query>
  </search>
  <option name="colorMode">block</option>
  <option name="rangeColors">["0x53A051","0xF8BE34","0xF1813F","0xDC4E41"]</option>
  <option name="rangeValues">[0,10,50,100]</option>
</single>
```
- 0–9 errors: Green
- 10–49: Yellow
- 50–99: Orange
- 100+: Red

---

## Sharing Dashboards

| Permission Level | Who Can See/Edit |
|-----------------|-----------------|
| **Private** | Only the creator |
| **App** | All users of the app |
| **Global** | All users of Splunk (across all apps) |

```
Dashboard → Edit Permissions → Set Read/Write access by role
```

---

## Practice Questions

1. Which visualisation type is BEST for showing trends over time?
   - A) Pie chart  B) Single value  C) Line chart ✓  D) Table

2. What are "tokens" used for in Splunk dashboards?
   - A) Authentication  B) Licensing  C) Dynamic input variables that filter dashboard panels ✓  D) Encryption keys

3. Which format is used to store Splunk dashboard definitions?
   - A) JSON  B) YAML  C) XML ✓  D) CSV

4. A security manager wants to see the current count of critical alerts in a prominent KPI display. Which panel type is BEST?
   - A) Table  B) Bar chart  C) Single value ✓  D) Line chart

5. What does a dashboard "drilldown" enable?
   - A) Scheduling the dashboard to run automatically  B) Clicking a panel element to navigate to a filtered search or URL ✓  C) Exporting data to CSV  D) Setting colour thresholds
