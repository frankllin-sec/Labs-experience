# 🛡️ SOC Fundamentals - TryHackMe

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-SOC%20Fundamentals-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

---

## 📌 About This Lab

This lab simulates a real **SOC Analyst Tier 1** workflow — receiving a security alert, investigating SIEM logs, applying the 5 Ws framework, (Who, what, where, when, why) and classifying the alert as True Positive or False Positive.

The scenario involved a port scanning activity detected on an internal host. My job was to analyze the logs, correlate the events, and document my findings professionally.

---
<a href="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Fundamentals-tryhackme/soc1.jpg" target="_blank">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Fundamentals-tryhackme/soc1.jpg" width="700"/>
</a>

## 🔍 Investigation — Port Scan Alert

### Scenario

> You are the **Level 1 Analyst** of your organization's SOC team. You receive an alert that a port scanning activity has been observed on one of the hosts in the network. You have access to the SIEM solution, where you can see all the associated logs for this alert.
>
> **Note:** The vulnerability assessment team notified the SOC team that they were running a port scan activity inside the network from the host: `10.0.0.8`

---

### Analysis

| # | Question | Answer |
|---|----------|--------|
| **What** | Activity that triggered the alert? | Port Scan |
| **When** | Time of the activity? | June 12, 2024 — 17:24 |

<a href="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Fundamentals-tryhackme/soc2.jpg" target="_blank">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Fundamentals-tryhackme/soc2.jpg" width="650"/>
</a>

| # | Question | Answer |
|---|----------|--------|
| **Where** | Destination host IP? | `10.0.0.3` (JOE PC) |
| **Who** | Source host name? | NESSUS (`10.0.0.8`) |
<a href="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Fundamentals-tryhackme/soc-fund4.jpg" target="_blank">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Fundamentals-tryhackme/soc-fund4.jpg" width="650"/>
</a>


| # | Question | Answer |
|---|----------|--------|
| **Why** | Reason for the activity? | ✅ **Intended** |


 **Why  Detailed Justification**

The activity was classified as **Intended**. The vulnerability assessment team had previously notified the SOC team of an authorized port scan being conducted from host `10.0.0.8` (NESSUS). The logs confirm this activity, as NESSUS systematically probed multiple ports on the destination host JOE PC (`10.0.0.3`), which is consistent with a scheduled internal vulnerability assessment.
<a href="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Fundamentals-tryhackme/soc-fund5.jpg" target="_blank">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Fundamentals-tryhackme/soc-fund5.jpg" width="650"/>
</a>


### 🔗 Additional Investigation — Response Traffic

**Has any response been sent back to the port scanner IP? → Yes**

One log entry showed reversed traffic direction, confirming a response was sent:

<a href="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Fundamentals-tryhackme/soc-fund6.jpg" target="_blank">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Fundamentals-tryhackme/soc-fund6.jpg" width="650"/>
</a>


**Port 22(SSH) on JOE PC was open and responsive.** This means JOE PC replied to the scanner, confirming SSH is active and reachable on the internal network.

---

## 📊 Final Result

| Metric | Result |
|---|---|
| 🔔 Alert Classification | False Positive |
| 🔍 Ports Identified | 22, 25, 53, 80, 143, 443 |
| ⚠️ Notable Finding | SSH (port 22) open and responsive on JOE PC |
| 🛠️ Tool Identified | NESSUS — Vulnerability Scanner |

---

## 🧠 What I Learned

### Technical Skills
- How to structure a SOC investigation
- Recognizing **NESSUS** as a legitimate vulnerability scanning tool.
- Reading SIEM logs to identify traffic direction and distinguish scan requests from responses
- Understanding **port 22 (SSH)** as a potential attack surface when left open unnecessarily

### Analyst Mindset
- Always cross-reference alert context with SIEM logs before classifying
- A response from the destination host doesn't mean malicious activity — but it does mean an open port worth reporting
- The **vulnerability assessment team's prior notification** was the key context that turned this from a suspicious alert into a confirmed false positive

---

## 💬 Honest Self-Assessment

**What went well:**
Identifying NESSUS as a known vulnerability scanner came naturally. Correlating the reversed traffic log to confirm SSH was open and responsive felt like real analyst work, not just answering questions, but actually reading the data.

**What I need to improve:**
Writing the justification for the "Why" field took the most time. Structuring the reasoning clearly and concisely is something I'm actively working on. The more writeups I complete, the more natural this process becomes.

---
---

---

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>

<p align="center">Thank you for visiting! 🙏</p>

<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
