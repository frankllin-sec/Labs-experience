# 🚨 SOC Alert Triage — TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Alert%20Triage-blue?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-green?style=for-the-badge"/>
</p>

---

## 📌 About This Lab

This lab covers the full **alert triage lifecycle** — one of the most essential skills for any SOC L1 analyst. Knowing how to properly handle an alert ultimately decides whether a security breach is detected and prevented, or missed and devastating.

The lab provided a live TryHackMe SIEM dashboard with 5 real alerts across different severity levels. The task was to triage each alert following proper SOC methodology — assign, investigate, classify, comment, and close.

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **Alert Triage** | The process of reviewing, prioritizing, investigating, and resolving security alerts |
| **True Positive** | A real threat confirmed by investigation |
| **False Positive** | A benign event incorrectly flagged as a threat |
| **Severity Levels** | Critical → High → Medium → Low — determines investigation priority |
| **Analyst Comment** | Written documentation of the investigation rationale and verdict |
| **Alert Lifecycle** | New → Awaiting Action → In Progress → Closed |

---

## 🛠️ Alert Prioritization Methodology

When picking alerts to triage, the correct SOC approach is:

1. **Filter** — Only take new, unassigned alerts not already being investigated
2. **Sort by Severity** — Critical first, then High, Medium, Low
3. **Sort by Time** — Oldest first within the same severity level — older breaches may already be deeper into the kill chain

---

## 🔍 Alert Triage — SIEM Dashboard Investigation

### Dashboard Overview

5 alerts were present in the SIEM dashboard on **March 21st 2025**, ranging from Critical to Low severity across multiple hosts and users.

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/SOC-Alert-Triage/Screenshots/alert_triage1.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Alert-Triage/Screenshots/alert_triage1.jpg" width="600"/>
  </a>
</p>

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/SOC-Alert-Triage/Screenshots/alert_triage3.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Alert-Triage/Screenshots/alert_triage3.jpg" width="600"/>
  </a>
</p>

---

### ❓ Alert 1 — Double-Extension File Creation | Severity: HIGH

**Host:** `LPT-HR-009` | **User:** `S.Conway` | **Process:** `chrome.exe`

**Investigation:** User downloaded `cats2025.mp4.exe` from `freecatvideoshd.monster` — a suspicious `.monster` TLD domain commonly associated with malicious infrastructure. The double-extension filename is consistent with **T1036.007 (Masquerading — Double File Extension)**, a technique used to trick users into executing malicious files disguised as media. File MD5 `14d8486f3f63875ef93cfd240c5dc10b` was flagged as malicious by 50/70 vendors on VirusTotal. Execution has not yet been confirmed.

**Verdict:** ✅ True Positive — Malicious file download confirmed, execution pending.

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/SOC-Alert-Triage/Screenshots/alert_triage6.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Alert-Triage/Screenshots/alert_triage6.jpg" width="600"/>
  </a>
</p>

---

### ❓ Alert 2 — Potential Data Exfiltration | Severity: CRITICAL

**Source IP:** `192.168.45.66` | **Network:** `UK04/MEETINGROOM` | **Destination:** `*.zoom.us`

**Investigation:** 5.8 GB sent and 5.2 GB received to `*.zoom.us`. Source IP `192.168.45.66` returned clean on AbuseIPDB and VirusTotal. Destination `*.zoom.us` is a trusted video conferencing platform. Sent/Received data volumes are nearly equal — consistent with active video conferencing, not exfiltration.

**Verdict:** ❌ False Positive — Normal Zoom video conferencing traffic from a meeting room.

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/SOC-Alert-Triage/Screenshots/alert_triage2.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Alert-Triage/Screenshots/alert_triage2.jpg" width="600"/>
  </a>
</p>

---

### ❓ Alert 3 — Download from GitHub Repository | Severity: LOW

**User:** `G.Chandler` | **Host:** `LPT-IT-063` | **Network:** `VPN/DEVELOPERS`

**Investigation:** User accessed `https://github.com/facebook/react` — the official React repository maintained by Facebook. Source network `VPN/DEVELOPERS` is a trusted segment for IT and development staff. No indicators of malicious content, exploit code, or suspicious scripts in the accessed URL.

**Verdict:** ❌ False Positive — Expected developer activity from a trusted network segment.

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/SOC-Alert-Triage/Screenshots/alert_triage4.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Alert-Triage/Screenshots/alert_triage4.jpg" width="600"/>
  </a>
</p>

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/SOC-Alert-Triage/Screenshots/alert_triage5.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Alert-Triage/Screenshots/alert_triage5.jpg" width="600"/>
  </a>
</p>

---

### ❓ Alert 4 — Unusual VPN Login Location | Severity: MEDIUM

**Analyst:** T.Ross (L1) | **Verdict:** False Positive

**Investigation:** Login location anomaly was determined to be a legitimate user accessing via VPN from an unusual but authorized location. No indicators of compromise identified.

**Verdict:** ❌ False Positive — Legitimate VPN login confirmed.

---

### ❓ Alert 5 — Bruteforce Attack from External | Severity: MEDIUM

**Analyst:** J.Adams (L2) | **Verdict:** True Positive

**Investigation:** External brute-force attack confirmed against an internet-facing system. Attack originated from external IP with multiple failed authentication attempts in a short timeframe. Escalated to L2 for containment and response.

**Verdict:** ✅ True Positive — Escalated to L2 for containment and response.

---

## 🗺️ MITRE ATT&CK Mapping

| Alert | Tactic | Technique |
|---|---|---|
| Double-Extension File Creation | Defense Evasion (TA0005) | T1036.007 — Masquerading: Double File Extension |
| Potential Data Exfiltration | Exfiltration (TA0010) | T1048 — Exfiltration Over Alternative Protocol |
| Download from GitHub Repository | Resource Development (TA0042) | T1583 — Acquire Infrastructure |
| Bruteforce Attack from External | Credential Access (TA0006) | T1110 — Brute Force |

---

## 🧠 What I Learned

### Technical Skills
- How to navigate a SIEM dashboard and apply correct alert prioritization logic
- How to distinguish True Positives from False Positives using contextual investigation
- How to correlate source IP, destination, network segment, and file reputation to reach a verdict
- How double-extension file masquerading works as a real-world evasion technique
- How to use VirusTotal and AbuseIPDB inline during triage to enrich alert context

### Analyst Mindset
- Severity does not always equal threat — a Critical alert can be a False Positive if context supports it
- Always check the source network segment — traffic from `VPN/DEVELOPERS` behaves differently than traffic from `HR`
- Equal sent/received data volumes are a strong indicator of video conferencing, not exfiltration
- The analyst comment is the most important output of triage — it must justify the verdict clearly

---

## 💬 Honest Self-Assessment

**What went well:**
Identifying the alerts and reaching the correct verdict felt natural once I understood the triage methodology. Connecting the file hash to VirusTotal results and the domain reputation to the verdict made the investigation feel like real SOC work.

**What I need to improve:**
Writing professional analyst comments independently. At this stage, my biggest challenge was not identifying the threat — it was articulating the investigation findings in clear, professional English. I used AI assistance to help structure and improve the grammar of my analyst comments. With more practice and repetition, I expect this to become more natural over time.

---

## 📁 Repository Structure

```
SOC-Alert-Triage/
├── README.md
└── Screenshots/
```

---

## 🖼️ Screenshots

> Screenshots of the SIEM dashboard, alert details, and analyst comments are available in the [`Screenshots`](./Screenshots/) folder.

---

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
