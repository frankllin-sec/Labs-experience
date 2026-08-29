# 🛡️ Log Analysis with SIEM: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-SIEM%20Triage-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Log-Analysis-with-SIEM/Screenshots/ainicio.jpg" width="700"/>
</p>

---

## 📌 About This Lab

Any SOC analyst needs to be comfortable using a SIEM (Security Information and Event Management) to search and correlate logs, and to quickly tell malicious activity apart from normal noise. This room walks through why SIEM matters, the main log sources an analyst runs into (Windows, Linux, and Web), and closes with practical Splunk scenarios for each one.

**Objectives:**
- Understand why SIEM tools are useful for a SOC (centralisation, correlation, historical data)
- Learn what Windows, Linux, and Web logs each show an analyst
- Practice writing simple Splunk searches to investigate suspicious activity

---

## 🔍 Investigation

**Q: What is the process of linking data from multiple sources to identify relationships between events called? What is the process of collecting log data from multiple systems into one place called?**

> **Answer:** `Correlation` and `Centralisation`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Log-Analysis-with-SIEM/Screenshots/a1.jpg" width="700"/>
</p>

**Q: Which log source type can be used to detect the execution of a malicious script?**

> **Answer:** `Host-Based`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Log-Analysis-with-SIEM/Screenshots/a2.jpg" width="700"/>
</p>

---

### Practice Scenario: Windows Logs

**Scenario:** A suspicious network connection was reported on port 5678, on host `WIN-105`. The logs live in the Splunk index `task4`.

**Q: Which IP address was the connection established with?**

**Method:** Filtered by the destination port (5678) to find the matching connection.

> **Answer:** `10.10.114.80`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Log-Analysis-with-SIEM/Screenshots/a3.jpg" width="700"/>
</p>

**Q: Which process initiated this suspicious connection?**

**Method:** Filtered by the process identifier (ProcessGuid) tied to that connection.

> **Answer:** `SharePoInt.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Log-Analysis-with-SIEM/Screenshots/a4.jpg" width="700"/>
</p>

**Q: What is the MD5 hash of that malicious process?**

**Method:** Added the hash field to the filter to pull it directly from the process event.

> **Answer:** `770D14FFA142F09730B415506249E7D1`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Log-Analysis-with-SIEM/Screenshots/a5.jpg" width="700"/>
</p>

**Q: What is the name of the scheduled task that was created on the system?**

**Method:** Filtered for SharePoint-related events and checked the command line field for a `schtasks` command.

> **Answer:** `Office365 Install`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Log-Analysis-with-SIEM/Screenshots/a6.jpg" width="700"/>
</p>

---

### Practice Scenario: Linux Logs

**Scenario:** An alert flagged possible persistence through a new `remote-ssh` user on an Ubuntu server. Logs for this task live in the Splunk index `task5`.

**Q: What was the timestamp of the remote-ssh account creation?**

**Method:** Filtered by the user field and checked when the `remote-ssh` account showed up.

> **Answer:** `2025-08-12 09:52:57`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Log-Analysis-with-SIEM/Screenshots/a7.jpg" width="700"/>
</p>

**Q: Which user successfully escalated their privileges to root prior to that?**

**Method:** Used the room's suggested search for `su` activity, sorted by time, and found the user right before the `remote-ssh` account got created.

> **Answer:** `jack-brown`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Log-Analysis-with-SIEM/Screenshots/a8.jpg" width="700"/>
</p>

**Q: From which IP address did that user successfully log in?**

**Method:** Filtered specifically for `jack-brown` to see the login source.

> **Answer:** `10.14.94.82`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Log-Analysis-with-SIEM/Screenshots/a9.jpg" width="700"/>
</p>

**Q: How many failed login attempts occurred before that successful login?**

**Method:** Filtered for `jack-brown` plus "failed password" entries and counted them.

> **Answer:** `4`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Log-Analysis-with-SIEM/Screenshots/a10.jpg" width="700"/>
</p>

**Q: Which port is the persistence mechanism configured to connect to?**

**Method:** Used the room's suggested cron search on the syslog source to find the scheduled connection.

> **Answer:** `7654`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Log-Analysis-with-SIEM/Screenshots/a11.jpg" width="700"/>
</p>

---

### Practice Scenario: Web Logs

**Scenario:** A spike in activity was reported on the organisation's web server. Logs live in the Splunk index `task6`.

**Q: Which URI path had the highest number of requests? Which IP was the source? How can this activity be classified? Which tool did the attacker use?**

**Method:** Used the room's suggested search filtering `POST` requests to `/wp-login.php`, grouped by client IP over 5-minute windows, and looked at the user-agent field for a tool signature.

> **Answer:** `/wp-login.php`, source IP `10.10.243.134`, classified as `Brute Force`, tool used: `WPScan`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Log-Analysis-with-SIEM/Screenshots/a12.jpg" width="700"/>
</p>

---

## 🧠 What I Learned

- Why centralisation and correlation are the two biggest reasons a SIEM makes an analyst's life easier, instead of chasing logs across ten separate systems
- The difference between what Windows logs, Linux logs, and Web logs each tend to show, and which one to reach for depending on the alert
- How to build a simple Splunk search step by step: pick the right index, filter by the field that matters (port, process, user), and narrow down from there
- That a user-agent string can be a giveaway on its own, tools like Hydra or WPScan often show up directly in the traffic

---

## 💬 Honest Self-Assessment

**What went well:**
Following the room's suggested searches and adapting them slightly for each scenario felt natural. Filtering step by step (first the port, then the process, then the hash) made each answer easier to find than trying to search for everything at once.

**What I need to improve:**
I relied a lot on the exact search syntax the room gave me. My next goal is to get comfortable writing these Splunk searches from scratch, without leaning on a template, so I can adapt faster when a real alert doesn't match a scenario I've already practiced.

---
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Log-Analysis-with-SIEM/Screenshots/afinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
