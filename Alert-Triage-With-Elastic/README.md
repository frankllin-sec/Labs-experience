# 🛡️ Alert Triage With Elastic: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Alert%20Triage-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Elastic/Screenshots/ainicio.jpg" width="700"/>
</p>

**Room link:** [tryhackme.com/room/alerttriagewithelastic](https://tryhackme.com/room/alerttriagewithelastic)

---

## 📌 About This Lab

Our team manages servers and infrastructure for a few small businesses. One client, SomeCorp, starts throwing alerts: file uploads, odd requests to a web page, an admin logging in outside business hours, a new user account, and unusual command-line activity. As the on-call analyst, the job is to use Kibana (part of the Elastic Stack) to check each alert, connect the dots between them, and figure out whether this is one real attack or a handful of unrelated noise.

**Objectives:**
- Use Kibana to look through web and Windows logs
- Spot indicators of compromise (IOCs) in that data
- Connect events across different log sources into one timeline
- Confirm whether a set of SOC alerts adds up to a real breach

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **Alert Triage** | Working through a queue of alerts one at a time to decide which are real (True Positive) and which aren't |
| **IOC (Indicator of Compromise)** | A clue, like a suspicious IP, a strange command, or an odd file, that points toward an attack |
| **Log Correlation** | Lining up events from different sources (web logs, Windows Security logs, Sysmon, PowerShell logs) by time and account to see the full story, not just one piece of it |
| **Web Shell** | A malicious script uploaded to a server that lets an attacker run commands remotely through the website itself |
| **Living-off-the-Land** | Using tools already built into Windows (like `net.exe` or a scheduled task) instead of custom malware, which makes the activity blend in more |

---

## 🔍 Investigation

**Q: How many logs are available for analysis? What is the client IP tied to the suspicious activity?**

**Method:** Selected the right data view and time range in Kibana, then looked at the total hit count.

> **Answer:** `1467` logs total, suspicious IP: `203.0.113.55`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Elastic/Screenshots/a1.jpg" width="700"/>
</p>

**Q: Which user-agent made the POST requests to proxyLogon.ecp?**

**Method:** Filtered the web logs for that IP and the POST requests, then checked the user-agent field. It didn't look like a normal browser, it looked automated.

> **Answer:** `python-requests/2.25.1`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Elastic/Screenshots/a2.jpg" width="700"/>
</p>

**Q: Which command was run through the web shell on Jul 20, 2025?**

**Method:** Filtered for GET requests to the suspicious file and read the command tucked into the URL.

> **Answer:** `Hostname`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Elastic/Screenshots/a3.jpg" width="700"/>
</p>

**Q: What is the record ID of the Administrator's logon event?**

**Method:** Switched from web logs to Windows Security logs and looked for a logon event (Event ID 4624) tied to the Administrator account, from the same IP address seen earlier.

> **Answer:** `17166`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Elastic/Screenshots/a4.jpg" width="700"/>
</p>

**Q: What is the name of the new user account that was created?**

**Method:** Searched Windows Security logs around the same time window for an account creation event (Event ID 4720).

> **Answer:** `svc_backup`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Elastic/Screenshots/a5.jpg" width="700"/>
</p>

**Q: What command added the new account to the "Remote Desktop Users" group?**

**Method:** Filtered for command-line activity launched under `cmd.exe` by the Administrator account.

> **Answer:** `net localgroup "Remote Desktop Users" svc_backup /add`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Elastic/Screenshots/a6.jpg" width="700"/>
</p>

**Q: What is the record ID of the event where the new account got added to the Administrators group?**

**Method:** Correlated that same command-line activity against the matching Security Group Management event (Event ID 4732).

> **Answer:** `17254`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Elastic/Screenshots/a7.jpg" width="700"/>
</p>

**Q: What PowerShell command did the attacker run afterward?**

**Method:** Filtered PowerShell script block logging for activity from the same account. The commands leading up to it (`whoami`, `whoami /priv`) were the attacker checking their own access level.

> **Answer:** `net group "Domain Admins" /domain`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Elastic/Screenshots/a8.jpg" width="700"/>
</p>

**Q: What is the name of the archive the attacker created with Rar.exe?**

**Method:** This one never triggered a SOC alert on its own, `Rar.exe` is legitimate software the client uses, so nothing flagged it automatically. Searched Sysmon logs directly for that process to see what the newly created account was doing with it.

> **Answer:** `finance_it_archive.rar`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Elastic/Screenshots/a9.jpg" width="700"/>
</p>

---

## 💬 Honest Self-Assessment

**What I need to improve:**
Like with Splunk, the Kibana query syntax here isn't something I have memorized yet. I followed the room's suggested filters closely instead of building most of them from scratch. I want to keep practicing until picking the right field names and query structure feels automatic instead of something I have to look up each time.

---
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Elastic/Screenshots/afinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
