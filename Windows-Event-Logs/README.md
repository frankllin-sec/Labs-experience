# 🛡️ Windows Event Logs: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Log%20Analysis-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Event-Logs/Screenshots/linicio.jpg" width="700"/>
</p>

---

## 📌 About This Lab

Event logs record what happens on a system to provide an audit trail, essential for troubleshooting, and, from a defender's seat, essential for spotting what an adversary did. This lab covers the three main ways to access Windows Event Logs (Event Viewer, wevtutil.exe, Get-WinEvent), how to build XPath queries against them, and closes with a full practical analysis of a real `merged.evtx` file across four incident scenarios.

**Objectives:**
- Navigate Event Viewer's panes and filter logs by Event ID
- Query event logs from the command line with wevtutil.exe
- Query event logs with the PowerShell Get-WinEvent cmdlet, including FilterHashtable
- Build XPath queries to filter events by log, provider, or Event ID
- Apply all three tools to investigate a real `merged.evtx` file across four SOC scenarios

---

## 🔑 Key Concepts

**Windows Event Log categories:**

| Log Type | What it Records |
|---|---|
| **System** | OS-level activity: hardware changes, device drivers, system changes |
| **Security** | Logon/logoff activity per the audit policy, the main source for investigating unauthorized access |
| **Application** | Errors, events, and warnings from installed applications |
| **Directory Service** | Active Directory changes, mainly on domain controllers |
| **File Replication Service** | Group Policy and logon script sharing across domain controllers |
| **DNS** | DNS server domain events |
| **Custom** | Application-defined logs with their own size and ACL settings |

**Three ways to access event logs:**

| Tool | Type | Best for |
|---|---|---|
| **Event Viewer** | GUI (MMC snap-in) | Quick interactive filtering and inspection |
| **wevtutil.exe** | Command-line | Scripting log queries, exports, and log management |
| **Get-WinEvent** | PowerShell cmdlet | Combining multiple sources, FilterHashtable and XPath queries at scale |

**XPath queries:** the Windows Event Log supports a subset of XPath 1.0 to filter events. A query starts with `*` or `Event`, then walks down the XML tree (`System`, `EventID`, `Provider/@Name`, etc.). Event Viewer's Details tab in XML View is the fastest way to find the exact tag names needed to build a query, and both `wevtutil` and `Get-WinEvent` accept XPath as a filter.

---

## 🔍 Investigation

### Part 1: Event Viewer

**Q: Filter the PowerShell Operational log for Event ID 4104. What is the Event ID for the earliest recorded event (excluding the warning event)?**

> **Answer:** `4103`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Event-Logs/Screenshots/l1.jpg" width="700"/>
</p>

**Q: Filter on Event ID 4104. What was the 2nd command executed in the PowerShell session?**

**Note:** This question gave me trouble. After filtering for 4104 I couldn't clearly see the 2nd command in the session. After digging through a lot of events, I searched online and found other people reporting the exact same issue, it looks like TryHackMe may have changed the underlying file without updating the expected answer.

> **Answer:** `whoami`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Event-Logs/Screenshots/l2.jpg" width="700"/>
</p>

**Q: What is the Task Category for Event ID 4104? Analyse the Windows PowerShell log, what is the Task Category for Event ID 800?**

**Method:** For the second part, I realized I was checking the wrong log. Event ID 800 lives in the **Windows PowerShell** log, not **PowerShell/Operational**.

> **Answer:** `Execute a Remote Command` (4104), `Pipeline Execution Details` (800)

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Event-Logs/Screenshots/l3.jpg" width="700"/>
</p>

---

### Part 2: wevtutil.exe

**Q: How many log names are in the machine?**

**Note:** I ran `(wevtutil el).Count` and separately `wevtutil el | Measure-Object`, both returned 1072, but the room's expected answer was 1071. A quick search showed other people hitting the same off-by-one mismatch, likely a stale answer key rather than a mistake on my end.

> **Answer:** `1071`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Event-Logs/Screenshots/l4.jpg" width="700"/>
</p>

**Q: What event files would be read when using the query-events command?**

> **Answer:** `event log, log file, structured query`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Event-Logs/Screenshots/l5.jpg" width="700"/>
</p>

**Q: What option would you use to provide a path to a log file? What is the VALUE for /q?**

> **Answer:** `/lf:true`, and `/q`'s value is an `Xpath query`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Event-Logs/Screenshots/l6.jpg" width="700"/>
</p>

**Q: Based on `wevtutil qe Application /c:3 /rd:true /f:text`, what is the log name?**

> **Answer:** `Application`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Event-Logs/Screenshots/l7.jpg" width="700"/>
</p>

**Q: What is the /rd option for? What is the /c option for?**

> **Answer:** `/rd` sets the event read direction, `/c` sets the maximum number of events to read

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Event-Logs/Screenshots/l8.jpg" width="700"/>
</p>

---

### Part 3: Get-WinEvent

**Q: Execute the command from Example 1. What are the names of the logs related to OpenSSH?**

**Method:** `Get-WinEvent -ListLog *`

> **Answer:** `OpenSSH/Admin, OpenSSH/Operational`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Event-Logs/Screenshots/l9.jpg" width="700"/>
</p>

**Q: Execute the command from Example 8, searching `*PowerShell*` instead of `*Policy*`. What is the name of the 3rd log provider?**

**Method:** `Get-WinEvent -ListProvider *Powershell*`

**Q: Execute the command from Example 9 with Microsoft-Windows-PowerShell as the provider. How many event IDs are displayed? How do you specify the number of events to display?**

**Method:** `(Get-WinEvent -ListProvider Microsoft-Windows-PowerShell).Events | Format-Table Id, Description | Measure-Object`

> **Answer:** `192` event IDs, and events displayed are limited with the `-MaxEvents` parameter

**Q: When using the FilterHashtable parameter and filtering by level, what is the value for Informational?**

**Method:** Googled "FilterHashtable parameter and filtering by level" and confirmed it against Microsoft's own documentation.

> **Answer:** `4`

---

### Part 4: Putting Theory Into Practice, Investigating merged.evtx

**Scenario 1: PowerShell Visibility.** Management approved PowerShell usage company-wide. The team needs visibility to confirm no coverage gaps.

**Q: What event ID is used to detect a PowerShell downgrade attack?**

**Method:** Searched Microsoft/MITRE-adjacent guidance for detecting downgrade attacks.

> **Answer:** `400`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Event-Logs/Screenshots/l11.jpg" width="700"/>
</p>

**Q: What is the Date and Time this attack took place?**

**Method:** Filtered `merged.evtx` for Event ID 400.

> **Answer:** `12/18/2020 7:50:33 AM`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Event-Logs/Screenshots/l13.jpg" width="700"/>
</p>

**Scenario 2: Log Clearing.** The Security Team wants to know if event logs are ever cleared.

**Q: A Log Clear event was recorded. What is the Event Record ID?**

**Method:** Searched for the correct Event ID first ("log clear event id windows" surfaced 1102, 104, and 1100 as candidates). Event ID 104 (System/Application log cleared) was the right match for this log.

> **Answer:** `27736`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Event-Logs/Screenshots/l14.jpg" width="700"/>
</p>

**Q: What is the name of the computer?**

> **Answer:** `PC01.example.corp`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Event-Logs/Screenshots/l15.jpg" width="700"/>
</p>

**Scenario 3: Emotet.** The threat intel team's research pointed to Event ID 4104 and the text "ScriptBlockText" inside the EventData element to find the encoded PowerShell payload.

**Q: What is the name of the first variable within the PowerShell command? What is the Date and Time this attack took place?**

**Method:** Filtered for Event ID 4104 and reviewed the General tab.

> **Answer:** `$Va5w3n8`, on `8/25/2020 10:09:28 PM`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Event-Logs/Screenshots/l16.jpg" width="700"/>
</p>

**Q: What is the Execution Process ID?**

> **Answer:** `6620`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Event-Logs/Screenshots/l17.jpg" width="700"/>
</p>

**Scenario 4: Suspicious Enumeration.** An intern was suspected of enumerating members of the Administrators group. A senior analyst suggested searching for `C:\Windows\System32\net1.exe` to confirm.

**Q: What is the Group Security ID of the group she enumerated?**

**Method:** I needed to find the Event ID first to filter properly, so I answered that question before this one.

> **Answer:** `S-1-5-32-544`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Event-Logs/Screenshots/l20.jpg" width="700"/>
</p>

**Q: What is the event ID?**

**Method:** Googled "What is the Group Security ID windows?" and cross-referenced against Windows Security Log documentation to confirm Event ID 4799 (security-enabled local group membership enumerated).

> **Answer:** `4799`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Event-Logs/Screenshots/l18.jpg" width="700"/>
</p>

---

## 🧠 What I Learned

- The difference between the three tools for accessing event logs, and when each one actually makes sense: Event Viewer for quick interactive digging, wevtutil.exe for scripted CLI queries, Get-WinEvent for combining and filtering at scale
- That the same Event ID (like 4104) can live in different logs (PowerShell/Operational vs Windows PowerShell) with different meanings, checking the wrong log burned time on this lab
- How to build an XPath query by walking the XML tree in Event Viewer's Details/XML View tab, System, EventID, Provider/@Name
- Practical detection knowledge: Event ID 400 for PowerShell downgrade attacks, Event ID 104 for log clears, Event ID 4799 for group membership enumeration
- How to cross-reference a real forensic file (`merged.evtx`) against four distinct SOC scenarios instead of a single flat Q&A list

---

## 💬 Honest Self-Assessment

**What went well:**
Once I understood the difference between PowerShell/Operational and Windows PowerShell logs, correlating events across the four scenarios in `merged.evtx` went smoothly. Building XPath queries by reading the XML view tag-by-tag made sense quickly, and I got comfortable pivoting between Event Viewer, wevtutil, and Get-WinEvent depending on what a question actually needed.

**What I need to improve:**
A couple of answers (the 2nd PowerShell command, and the log name count) didn't match what my tools were actually showing me, and searching online confirmed other people hit the same mismatches, likely a stale answer key rather than something I did wrong. Still, it's a good reminder to sanity-check twice before assuming I made the mistake. I also want to get faster at picking the right log on the first try instead of checking Operational by default and correcting course afterward.

---
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Event-Logs/Screenshots/lfinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
