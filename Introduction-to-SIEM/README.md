# 🛡️ Introduction to SIEM Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-SIEM%20Analysis-green?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Introduction-to-SIEM/Screenshots/introtosiem-intro.jpg" width="700"/>
</p>
---

## 📌 About This Lab

This lab introduces the fundamentals of **Security Information and Event Management (SIEM)** - a core technology in any SOC environment. SIEM solutions collect logs from multiple sources, normalize them into a consistent format, correlate events, and trigger alerts based on pre-configured detection rules.

A static SIEM dashboard was provided with sample events. Upon triggering the **Start Suspicious Activity** button, an alert fired - indicating that an event matched an existing detection rule. The goal was to investigate the alert and trace it back to the responsible process, user, and host.

---

## 🔍 Investigation - SIEM Dashboard Analysis

### Scenario

A suspicious activity alert was triggered in the SIEM dashboard. The task was to identify the process that caused the alert, find the responsible user, determine the hostname, and identify the rule term that matched the detection logic.

---

### ❓ Q1 - After clicking on the Start Suspicious Activity button, which process caused the alert?

**Method:** Navigated to the Alerts section of the SIEM dashboard. Clicked on the triggered alert to view its details. Identified the process name associated with the suspicious activity.

> **Answer:** `cudaMiner.exe`
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Introduction-to-SIEM/Screenshots/introtosiem-1.jpg" width="700"/>
</p>
---

### ❓ Q2 - Find the event that caused the alert and identify the user responsible for the process execution.

**Method:** Drilled down into the alert details → clicked on the event entry → inspected the user field to identify who executed the process.

> **Answer:** `chris`
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Introduction-to-SIEM/Screenshots/introtosiem-2-1.jpg" width="700"/>
</p>
---

### ❓ Q3 - What is the hostname of the suspect user?

**Method:** Examined the event details further and located the hostname field associated with the user `chris`.

> **Answer:** `HR_02`
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Introduction-to-SIEM/Screenshots/introtosiem-2.jpg" width="700"/>
</p>
---

### ❓ Q4 - Examine the rule and the suspicious process - which term matched the rule that caused the alert?

**Method:** Reviewed the detection rule that triggered the alert and compared it against the process name to identify the matching keyword.

> **Answer:** `miner`
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Introduction-to-SIEM/Screenshots/introtosiem-3.jpg" width="700"/>
</p>

---

## 🧠 What I Learned

### Technical Skills
- How to navigate a SIEM dashboard and interpret triggered alerts
- How detection rules work - keyword matching against process names
- How to drilldown from an alert to the raw event to identify user, host, and process
- Cryptomining malware (`cudaMiner.exe`) is a real-world threat caught by SIEM rules targeting keywords like `miner`

### Analyst Mindset
- Alerts are just the starting point - the real work is tracing them back to the source
- Always correlate **process + user + hostname** to scope an incident properly
- A process name containing `miner` on a corporate endpoint is an immediate red flag

---
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Introduction-to-SIEM/Screenshots/introtosiem-final.jpg" width="700"/>
</p>

## 💬 Honest Self-Assessment

**What went well:**
Understanding the alert-to-event drilldown flow felt natural. Connecting the process name `cudaMiner.exe` to the `miner` keyword rule made the detection logic click.

---

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
