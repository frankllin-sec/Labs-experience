# 🔍 Windows Event Log Analysis Lab 

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Log%20Analysis-green?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

---
<p align="center">
  <a href="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/windows-event-log-analysis/screenshots/logs%20fundaments.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/windows-event-log-analysis/screenshots/logs%20fundaments.jpg" width="600"/>
  </a>
</p>

## 📌 About This Lab

This lab simulates a real **SOC Analyst Tier 1** investigation using **Windows Event Logs** and **Web Server Access Logs**. The goal was to trace the attacker's activities on a compromised system before they pivoted to a file server and exfiltrated critical data.

The investigation involved filtering Windows Security logs by Event ID to reconstruct the attacker's account manipulation actions, and manually analyzing a web server `access.log` using Linux commands.

---

## 🔑 Critical Event IDs Used

| Event ID | Description |
|---|---|
| `4720` | A user account was **created** |
| `4722` | A user account was **enabled** |
| `4724` | An attempt was made to **reset an account's password** |

---

## 🔍 Investigation - Part 1: Windows Event Log Analysis

### Scenario

On a Friday, a critical organization reported being a victim of a cyber attack. Critical data was exfiltrated from a file server in the organization's network. The security team identified the **username and IP address** of the compromised system that had access to the file server at the time of the attack.

**Mission:** Investigate the attacker's activities on the compromised system — *before* they pivoted to the file server.
<a href="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/windows-event-log-analysis/screenshots/events-logs.jpg">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/windows-event-log-analysis/screenshots/events-logs.jpg" width="600"/>
</a>

---

### ❓ Q1 - What is the name of the last user account created on this system?

**Event ID used:** `4720` - *A user account was created*

**Method:** Opened Event Viewer → Windows Logs → Security. Used **Filter Current Log** and filtered by Event ID `4720`. Identified the most recent entry to find the last account created.

<a href="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/windows-event-log-analysis/screenshots/events-logs-anser%201.jpg">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/windows-event-log-analysis/screenshots/events-logs-anser%201.jpg" width="600"/>
</a>
---

### ❓ Q2 - Which user account created the above account?

**Event ID used:** `4720` — *Subject field reveals who performed the action*

**Method:** Opened the Event ID `4720` log entry and inspected the **Subject: Account Name** field — this shows the account that performed the creation.

<a href="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/windows-event-log-analysis/screenshots/events-logs-anser%202.jpg">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/windows-event-log-analysis/screenshots/events-logs-anser%202.jpg" width="600"/>
</a>

---

### ❓ Q3 - On what date was this user account enabled? *(Format: M/D/YYYY)*

**Event ID used:** `4722` - *A user account was enabled*

**Method:** Filtered Security log by Event ID `4722`. Located the entry matching the account found in Q1 and checked the event timestamp.

<a href="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/windows-event-log-analysis/screenshots/events-logs-anser%203.jpg">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/windows-event-log-analysis/screenshots/events-logs-anser%203.jpg" width="600"/>
</a>

---

### ❓ Q4 - Did this account undergo a password reset as well? *(Format: Yes/No)*

**Event ID used:** `4724` — *An attempt was made to reset an account's password*

**Method:** Filtered by Event ID `4724` and confirmed a password reset event was logged for the account identified in Q1.

<a href="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/windows-event-log-analysis/screenshots/events-logs-anser%204.jpg">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/windows-event-log-analysis/screenshots/events-logs-anser%204.jpg" width="600"/>
</a>

---

## 🔍 Investigation - Part 2: Web Server Access Log Analysis

### Setup

The `access.log` file was located at `/root/Rooms/logs` on the AttackBox.

```bash
# Navigate to the log directory
cd Rooms/logs

# List available files
ls

# Read the log file
cat access.log
```

<a href="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/windows-event-log-analysis/screenshots/events-logs-anser%205.jpg">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/windows-event-log-analysis/screenshots/events-logs-anser%205.jpg" width="600"/>
</a>
---
<a href="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/windows-event-log-analysis/screenshots/events-logs-anser%207.jpg">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/windows-event-log-analysis/screenshots/events-logs-anser%207.jpg" width="600"/>
</a>

---
<a href="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/windows-event-log-analysis/screenshots/events-logs-8.jpg">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/windows-event-log-analysis/screenshots/events-logs-8.jpg" width="600"/>
</a>
---

## 🧠 What I Learned

### Technical Skills
- How to use **Windows Event Viewer** to filter logs by Event ID during an active investigation
- The difference between account **creation** (4720), **enablement** (4722), and **password reset** (4724)
- Correlating Windows Security events to reconstruct an attacker's timeline

### Analyst Mindset
- A new account being created, enabled, and having its password reset in quick succession is not normal.
- Always check the **Subject** field in Security events - it reveals *who* performed the action, not just *what* happened

---

## 💬 Honest Self-Assessment

**What went well:**
Filtering by Event ID made the investigation straightforward. The correlation between account creation and the password reset was intuitive once I understood what each Event ID represents.

**What I need to improve:**
Getting faster at navigating Event Viewer and knowing which log category (Security, System, Application) to check first without hesitation. Building that instinct takes repetition.

---

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>

<p align="center">Thank you for visiting! 🙏</p>

<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
