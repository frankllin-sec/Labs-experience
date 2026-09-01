# 🛡️ Alert Triage With Splunk: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Alert%20Triage-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Splunk/Screenshots/atinicio.jpg" width="700"/>
</p>

---

## 📌 About This Lab

Three shifts, three alerts, three different environments. This room walks through a full L1 triage workflow across a Linux brute-force attack, a Windows persistence mechanism, and a web shell on a vulnerable web server, each investigated end-to-end in Splunk, from the initial alert context to the escalation decision.

**Objectives:**
- Investigate brute-force attacks on Linux systems
- Discover a persistence mechanism on a Windows host
- Analyse a web shell on a vulnerable web server
- Practice the full triage workflow: alert context first, SIEM second, escalate when confirmed

---

## 🔑 Key Concepts

**Alert context before SIEM.** Before touching the SIEM, look at what the alert itself tells you: is the source IP internal or external, does the hostname suggest a workstation or a server, does the activity time fall inside normal working hours. That context shapes what you look for once you do start searching.

**Escalation is the L1 job.** An L1 analyst's responsibility ends at identifying malicious activity and escalating it to L2, not resolving the full incident. Each scenario in this room closes the same way: classify as True Positive, escalate, and list the open questions L2 needs to chase.

**Common indicators across all three scenarios:**

| Indicator | What it Signals |
|---|---|
| Many failed logins targeting invalid usernames | Account enumeration before a brute-force attempt |
| A scheduled task using `certutil` to download a file | A classic Living-off-the-Land persistence technique |
| A referer pointing to an unexpected `.php` file | Possible web shell interaction |
| A known offensive tool in the User-Agent (e.g. Hydra) | A strong, direct signal of automated attack tooling |

---

## 🔍 Investigation

### Scenario 1: Brute Force on a Linux Host

**Alert:** Brute Force Activity Detection, target host `tryhackme-2404`, source IP `10.10.242.248` (internal), 9:00 AM (normal working hours).

**Q: How many failed login attempts were made on the user john.smith?**

**Method:** Searched `auth.log`-style events for that source IP, extracted the username field with a regex, and grouped counts by user. One user stood out with a disproportionately high count.

> **Answer:** `500`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Splunk/Screenshots/at1.jpg" width="700"/>
</p>

**Q: What was the duration of the brute-force attack, in minutes?**

**Method:** Compared the timestamp of the first and last failed attempt against `john.smith`.

> **Answer:** `5` minutes

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Splunk/Screenshots/at2.jpg" width="700"/>
</p>

**Q: What username was the attacker able to privilege escalate to? What is the name of the user account created for persistence?**

**Method:** Searched for `sudo`/`su` activity from `john.smith` and found a successfully opened session, confirming privilege escalation. Then searched for account creation events to find the new persistence account.

> **Answer:** Escalated to `root`, persistence account created: `system-utm`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Splunk/Screenshots/at3.jpg" width="700"/>
</p>

---

### Scenario 2: Persistence on a Windows Host

**Alert:** Potential Task Scheduler Persistence Identified, host `WIN-H015` (a workstation, based on naming convention), user `oliver.thompson` (a System Engineer), task name `AssessmentTaskOne`.

**Q: What is the ProcessId of the process that created this malicious task?**

**Method:** Queried Event ID 4698 (scheduled task creation) filtered to the task name, and read the ProcessId directly from the event.

> **Answer:** `5816`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Splunk/Screenshots/at4.jpg" width="700"/>
</p>

**Q: What is the name of the parent process for the process that created this malicious task?**

**Method:** Filtered by that ProcessId to trace it back to its parent.

> **Answer:** `cmd.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Splunk/Screenshots/at5.jpg" width="700"/>
</p>

**Q: Which local group did the attacker enumerate during discovery?**

**Method:** Searched for `oliver.thompson` activity involving group-related events.

> **Answer:** `Administrators`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Splunk/Screenshots/at6.jpg" width="700"/>
</p>

**Q: What is the name of the workstation from which the threat actor logged into this host?**

**Method:** Searched for `oliver.thompson` combined with Event ID 4624 (a successful logon event), and read the source workstation field.

> **Answer:** `DEV-QA-SERVER`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Splunk/Screenshots/at7.jpg" width="700"/>
</p>

---

### Scenario 3: Web Shell on a Vulnerable Server

**Alert:** Potential Web Shell Upload Detected, resource `http://web.trywinme.thm`, suspicious IP `171.251.232.40` (flagged malicious 3000+ times on AbuseIPDB).

**Q: What time did the brute-force activity using Hydra begin?**

**Method:** Filtered web logs for the flagged IP and sorted by time. The `Mozilla/5.0 (Hydra)` User-Agent immediately stood out as a brute-force tool targeting `wp-login.php`.

> **Answer:** `2025-09-14 21:20:27`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Splunk/Screenshots/at8.jpg" width="700"/>
</p>

**Q: Which User-Agent did the attacker use when interacting with the web shell? What was the number of requests made via the web shell?**

**Method:** Excluded the Hydra User-Agent from the search to see what else the IP was doing. A POST to `admin-ajax.php` with a referer pointing to `theme-editor.php?file=b374k.php` stood out, `b374k.php` is a known public web shell. Filtered specifically on that file to count the interactions.

> **Answer:** `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36`, `4` requests

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Splunk/Screenshots/at9.jpg" width="700"/>
</p>

---

## 🧠 What I Learned

- How to triage the same way across Linux, Windows, and Web alerts: read the alert context first, then confirm it in Splunk
- How to spot brute force, persistence, and web shell activity in raw logs

---

## 💬 Honest Self-Assessment

**What I need to improve:**
The Splunk filters and commands used in this room aren't things I know by heart yet. I had to research which command and syntax to use instead of writing it from memory. I want to practice until these searches come naturally instead of needing to look up the syntax every time.

---
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Alert-Triage-With-Splunk/Screenshots/atfinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
