# Day 8 — Splunk for Security (SIEM Use Cases)

## Introduction
Splunk is widely deployed as a **SIEM (Security Information and Event Management)** platform. This lesson covers core security operations use cases — the searches and workflows SOC analysts use daily.

---

## What Is a SIEM?

A SIEM:
- **Collects** security events from across the environment
- **Correlates** events across multiple sources to detect threats
- **Alerts** on suspicious activity
- **Enables investigation** of incidents
- **Supports compliance** through log retention and reporting

Splunk acts as a SIEM through:
- Native data collection (forwarders, HEC, APIs)
- Powerful SPL correlation searches
- **Splunk Enterprise Security (ES)** — premium SIEM application built on Splunk

---

## Core Security Log Sources

| Source | Data Provided |
|--------|--------------|
| **Windows Event Logs** | Logins, account changes, process creation, policy changes |
| **Linux /var/log/secure** | SSH logins, sudo commands, PAM authentication |
| **Firewall Logs** | Allowed/denied connections, ports, IPs |
| **VPN Logs** | Remote access authentication |
| **Proxy/Web Gateway Logs** | Web requests, categorised URLs, blocked sites |
| **DNS Logs** | Domain queries — C2 detection, data exfiltration |
| **Active Directory** | User/group changes, GPO modifications |
| **EDR/AV Logs** | Malware detections, process trees, file events |
| **Cloud Logs** | AWS CloudTrail, Azure AD, GCP audit logs |
| **Email Security** | Phishing detections, attachment analysis |

---

## Key Windows Event IDs for Security

| Event ID | Description | Security Relevance |
|----------|-------------|-------------------|
| **4624** | Successful logon | Baseline login activity |
| **4625** | Failed logon | Brute force indicator |
| **4648** | Logon with explicit credentials | Pass-the-hash, lateral movement |
| **4672** | Special privileges assigned to new logon | Admin login |
| **4688** | New process created | Malware execution |
| **4698** | Scheduled task created | Persistence |
| **4719** | Audit policy change | Attacker covering tracks |
| **4720** | User account created | Backdoor account |
| **4722** | User account enabled | Dormant account activated |
| **4723/4724** | Password change/reset | Credential manipulation |
| **4726** | User account deleted | Covering tracks |
| **4768/4769** | Kerberos TGT/TGS request | Kerberoasting, Golden Ticket |
| **4776** | NTLM authentication | Pass-the-Hash |
| **7045** | Service installed | Malware persistence |

---

## Essential Security SPL Searches

### Brute Force Detection
```spl
index=winevent sourcetype=wineventlog EventCode=4625
| stats count AS failed_logins by src_ip, user
| where failed_logins > 10
| sort -failed_logins
```

### Successful Login After Multiple Failures (Credential Stuffing)
```spl
index=winevent sourcetype=wineventlog
| where EventCode=4625 OR EventCode=4624
| stats count(eval(EventCode=4625)) AS failures,
        count(eval(EventCode=4624)) AS successes by user, src_ip
| where failures > 5 AND successes > 0
| eval risk = "High - Possible credential stuffing"
```

### Privilege Escalation (Admin Rights Assigned)
```spl
index=winevent sourcetype=wineventlog EventCode=4672
| stats count by user, Logon_Type, Workstation_Name
| where NOT user IN ("SYSTEM","LOCAL SERVICE","NETWORK SERVICE")
```

### New Admin Account Created
```spl
index=winevent sourcetype=wineventlog EventCode=4720
| stats count by user, Account_Name, Caller_User_Name
| table _time, Caller_User_Name, Account_Name
```

### Scheduled Task Persistence
```spl
index=winevent sourcetype=wineventlog EventCode=4698
| stats count by user, Task_Name, Task_Content
| where NOT user IN ("SYSTEM","NETWORK SERVICE")
```

### Process Creation — Suspicious PowerShell
```spl
index=winevent sourcetype=wineventlog EventCode=4688
| where Process_Name LIKE "%powershell%"
| rex field=Process_Command_Line "(?i)(?P<suspicious_flag>-enc|-encode|-executionpolicy bypass|-nop|-w hidden)"
| where isnotnull(suspicious_flag)
| table _time, Computer, Creator_Process_Name, Process_Name, Process_Command_Line
```

### DNS-Based C2 Detection
```spl
index=dns
| rex field=query "^(?P<subdomain_count>[^.]+\.[^.]+\.[^.]+\.[^.]+)"
| where len(query) > 50
| stats count by query, src_ip
| where count > 5
| sort -count
```

### Data Exfiltration — Large Outbound Transfers
```spl
index=proxy
| stats sum(bytes_out) AS total_out by src_ip, dest_host
| where total_out > 100000000
| eval total_out_mb = round(total_out/1024/1024, 2)
| sort -total_out_mb
| table src_ip, dest_host, total_out_mb
```

### Lateral Movement — RDP Connections
```spl
index=winevent sourcetype=wineventlog EventCode=4624 Logon_Type=10
| stats count by src_ip, dest_host, user
| where src_ip != dest_host
| sort -count
```

### After-Hours Activity
```spl
index=winevent sourcetype=wineventlog EventCode=4624
| eval hour = strftime(_time, "%H")
| where hour < 6 OR hour >= 20
| stats count by user, src_ip, Workstation_Name
| sort -count
```

---

## Splunk Enterprise Security (ES)

**Splunk ES** is a premium security-focused app built on Splunk Enterprise:

### Key ES Features

| Feature | Description |
|---------|-------------|
| **Notable Events** | Correlated findings from security detections |
| **Risk Score System** | Assign risk scores to users and systems |
| **Threat Intelligence** | Import and correlate IoCs (IPs, domains, hashes) |
| **Asset & Identity Management** | Asset database for context enrichment |
| **Incident Review** | Queue for analysts to work through notable events |
| **Security Posture Dashboard** | Real-time SOC overview |
| **Correlation Searches** | Pre-built + custom threat detections |

### ES Correlation Search Example
```spl
# Notable event: Impossible travel
index=authentication action=success
| iplocation src_ip
| stats earliest(_time) AS first_seen, 
        latest(_time) AS last_seen,
        values(Country) AS countries by user
| where mvcount(countries) > 1
| eval alert = "Impossible Travel - User logged in from multiple countries"
```

---

## Splunk Security Essentials (SSE)

Free app providing:
- 300+ security searches mapped to MITRE ATT&CK
- Pre-built detections for common attack scenarios
- Guidance for implementing detections

---

## MITRE ATT&CK Mapping in Splunk

```spl
# Tag searches with ATT&CK technique IDs
# Example: T1059.001 PowerShell execution
index=winevent EventCode=4688 Process_Name="*powershell*"
| eval technique = "T1059.001"
| eval tactic = "Execution"
```

Splunk ES includes ATT&CK mapping in correlation searches out of the box.

---

## Practice Questions

1. Which Windows Event ID indicates a successful user logon?
   - A) 4625  B) 4624 ✓  C) 4688  D) 4698

2. A SOC analyst finds multiple failed logins (EventCode 4625) followed by a successful login (EventCode 4624) from the same IP. What does this likely indicate?
   - A) Normal user activity  B) Software error  C) Credential stuffing or brute force ✓  D) Password reset

3. Which Splunk product provides a full SIEM experience with notable events, risk scoring, and threat intelligence?
   - A) Splunk Cloud  B) Splunk SOAR  C) Splunk Enterprise Security ✓  D) Splunk Observability

4. Which Windows Event ID tracks the creation of a new scheduled task (persistence mechanism)?
   - A) 4720  B) 4688  C) 4698 ✓  D) 4624

5. A large volume of DNS queries with very long subdomain names to a recently registered domain is indicative of:
   - A) Normal CDN behaviour  B) DNS tunnelling for C2 communication ✓  C) Load balancing  D) Certificate validation
