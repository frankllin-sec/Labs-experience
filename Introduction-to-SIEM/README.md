# Shield Introduction to SIEM - TryHackMe Lab

## Overview

This lab introduces the fundamentals of **Security Information and Event Management (SIEM)** — a core technology in any SOC environment. SIEM solutions collect logs from multiple sources, normalize them into a consistent format, correlate events, and trigger alerts based on detection rules.

---

## Objectives

- Understand what SIEM is and how it works
- Navigate a SIEM dashboard and interpret alerts
- Identify suspicious activity through event correlation
- Trace an alert back to the responsible user, host, and process

---

## Lab Summary

In this lab, a static SIEM dashboard was provided with sample events and pre-configured detection rules. Upon triggering the **Start Suspicious Activity** button, an alert fired — indicating that an event matched an existing detection rule.

The investigation involved:
1. Identifying the alert in the dashboard
2. Drilling down into the event details
3. Tracing the process, user, and hostname responsible

---

## Investigation Findings

| Field | Value |
|---|---|
| Process that caused the alert | `cudaMiner.exe` |
| User responsible | `chris` |
| Hostname of suspect | `HR_02` |
| Term that matched the rule | `miner` |

---

## Key Concepts Learned

- **SIEM** collects, normalizes, and correlates logs from multiple sources
- **Alerts** are triggered when events match pre-configured detection rules
- **Event drilldown** allows analysts to trace alerts back to specific users and processes
- **Cryptomining malware** (like `cudaMiner.exe`) is a common threat detected via process name rules
- Hostname and username correlation is essential for scoping an incident

---

## MITRE ATT&CK Mapping

| Field | Detail |
|---|---|
| Tactic | Resource Hijacking (TA0040) |
| Technique | T1496 - Resource Hijacking |
| Tool | cudaMiner.exe (cryptominer) |

---

## Repository Structure

```
Introduction-to-SIEM/
├── README.md          # Lab documentation
└── Screenshots/       # Evidence screenshots from the lab
```

---

## Screenshots

> Screenshots of the SIEM dashboard, alert details, and investigation steps are available in the [Screenshots](./Screenshots/) folder.

---

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>