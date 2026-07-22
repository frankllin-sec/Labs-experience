# Windows Logging for SOC - TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Windows%20Log%20Analysis-blue?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-green?style=for-the-badge"/>
</p>

---

## About This Lab

SOC analysts spend most of their time triaging alerts and hunting threats using logs in SIEM. This room begins the journey into **Windows logging** - a key skill for any SOC analyst or DFIR professional. The lab covers three critical log sources: **Security Event Logs**, **Sysmon**, and **PowerShell History** - each revealing a different layer of attacker activity on a compromised Windows endpoint.

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Windows-Logging-for-SOC/Screenshots/loginicio.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Logging-for-SOC/Screenshots/loginicio.jpg" width="600"/>
  </a>
</p>

---

## Investigation - Part 1: Security Event Log Analysis (RDP Brute Force)

### Scenario

Opened `Practice-Security.evtx` in Event Viewer and investigated failed and successful logon events to trace an RDP brute force attack and its aftermath.

---

### Q1 - Which event ID describes a successful login?

**Method:** Researched which Event ID corresponds to successful logons in Windows Security logs. Event ID 4624 is the standard Windows Security event for a successful account logon.

> **Answer:** `Security / 4624`

---

### Q2 - Which IP performed a brute force of the THM-PC?

**Method:** Filtered Security logs for **Event ID 4625** (Failed Logon) with Logon Type 3 and 10 (Network and RDP). Identified the Source Network Address with a high volume of repeated failed login attempts against the Administrator account.

> **Answer:** `10.10.53.248`

---

### Q3 - Which user has been breached as a result of the attack?

**Method:** After identifying the brute force source IP, filtered for **Event ID 4624** (Successful Logon) with Logon Type 10 from the same source IP to find the successful RDP login.

> **Answer:** `Administrator`

---

### Q4 - What was the Logon ID of the malicious RDP login?

**Method:** Opened the Event ID 4624 with Logon Type 10 and noted the **Logon ID** field - a unique session identifier used to correlate all subsequent events in the same attacker session.

> **Answer:** `0x183C36D`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Windows-Logging-for-SOC/Screenshots/log1.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Logging-for-SOC/Screenshots/log1.jpg" width="600"/>
  </a>
</p>

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Windows-Logging-for-SOC/Screenshots/log2.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Logging-for-SOC/Screenshots/log2.jpg" width="600"/>
  </a>
</p>

---

## Investigation - Part 2: User Management Events (Backdoor Account)

### Scenario

Continued with `Practice-Security.evtx`, hunting for **Event ID 4720** (User Created) and **4732** (User Added to Group) to identify backdoor accounts created by the attacker after the RDP login.

---

### Q5 - Which user was created by the attacker soon after the RDP login?

**Method:** Filtered for **Event ID 4720** and matched the Logon ID `0x183C36D` to confirm the action was performed within the same attacker session.

> **Answer:** `svc_sysrestore`

---

### Q6 - Which two privileged groups was the backdoor user added to?

**Method:** Filtered for **Event ID 4732** (Member Added to Security Group). Found the backdoor account added to two built-in privileged groups linked to the same Logon ID.

> **Answer:** `Backup Operators, Remote Desktop Users`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Windows-Logging-for-SOC/Screenshots/log3.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Logging-for-SOC/Screenshots/log3.jpg" width="600"/>
  </a>
</p>

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Windows-Logging-for-SOC/Screenshots/log4.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Logging-for-SOC/Screenshots/log4.jpg" width="600"/>
  </a>
</p>

---

## Investigation - Part 3: Sysmon - Process Analysis

### Scenario

Opened `Practice-Sysmon.evtx` and analyzed **Sysmon Event ID 1** (Process Create) to trace malware execution on `sarah.miller`'s account.

---

### Q7 - Which web browser does Sarah use?

**Method:** Filtered Sysmon for Event ID 1 and reviewed the Image field. Found Chrome launched by `sarah.miller` from the standard Google Chrome installation path.

> **Answer:** `Google Chrome`

---

### Q8 - Which file did Sarah download from the browser?

**Method:** Identified a suspicious Sysmon Event ID 1 where a process was launched from the Downloads folder - a red flag for malware staging.

> **Answer:** `C:\Users\sarah.miller\Downloads\ckjg.exe`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Windows-Logging-for-SOC/Screenshots/log5.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Logging-for-SOC/Screenshots/log5.jpg" width="600"/>
  </a>
</p>

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Windows-Logging-for-SOC/Screenshots/log6.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Logging-for-SOC/Screenshots/log6.jpg" width="600"/>
  </a>
</p>

---

### Q9 - Which URL was the file downloaded from?

**Method:** Used Sysmon **Event ID 15** (FileCreateStreamHash) which captures the Zone.Identifier - a Windows feature storing the original download URL. The HostUrl field in Contents revealed the source.

> **Answer:** `http://gettsvenff.com/bgj3/ckjg.exe`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Windows-Logging-for-SOC/Screenshots/log7.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Logging-for-SOC/Screenshots/log7.jpg" width="600"/>
  </a>
</p>

---

## Investigation - Part 4: Sysmon - Persistence and C2

### Scenario

Continued Sysmon analysis using the malware ProcessId to identify persistence mechanisms and C2 communications.

---

### Q10 - Which file was created by the malware to persist on the host?

**Method:** Used Sysmon **Event ID 11** (FileCreate) filtered by the malware ProcessId. Found a file dropped to the Windows Startup folder - a classic persistence technique that executes the payload on every system reboot.

> **Answer:** `C:\Users\sarah.miller\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\DeleteApp.url`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Windows-Logging-for-SOC/Screenshots/log8.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Logging-for-SOC/Screenshots/log8.jpg" width="600"/>
  </a>
</p>

---

### Q11 - What is the C2 server the malware connected to?

**Method:** Used Sysmon **Event ID 3** (NetworkConnect) filtered by the malware ProcessId. Found an outbound TCP connection from `ckjg.exe` to an external IP on a non-standard port.

> **Answer:** `193.46.217.4:7777`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Windows-Logging-for-SOC/Screenshots/log9.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Logging-for-SOC/Screenshots/log9.jpg" width="600"/>
  </a>
</p>

---

### Q12 - Which domain does the malicious IP correspond to?

**Method:** Used Sysmon **Event ID 22** (DNS Query) filtered by the malware ProcessId. The QueryName field revealed the C2 domain.

> **Answer:** `hkfasfsafg.click`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Windows-Logging-for-SOC/Screenshots/log10.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Logging-for-SOC/Screenshots/log10.jpg" width="600"/>
  </a>
</p>

---

## Investigation - Part 5: PowerShell History Analysis

### Scenario

Reviewed PowerShell ConsoleHost_history.txt files for multiple users to identify attacker activity and recover a hidden flag.

---

### Q13 - Which PowerShell command was executed first?

**Method:** Located the `ConsoleHost_history.txt` file for the Administrator account and reviewed commands in chronological order.

> **Answer:** `Get-ComputerInfo`

---

### Q14 - When did the Administrator run the first PS command?

**Method:** Right-clicked the history file, opened Properties, and checked the **Created** date.

> **Answer:** `May 18, 2025`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Windows-Logging-for-SOC/Screenshots/log11.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Logging-for-SOC/Screenshots/log11.jpg" width="600"/>
  </a>
</p>

---

### Q15 - What is the flag stored in the PowerShell history?

**Method:** Checked PowerShell history for all local users - not just Administrator. Found the flag in `thm.bob`'s history file, echoed into a `flag.txt` file.

> **Answer:** `THM{it_was_me!}`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Windows-Logging-for-SOC/Screenshots/log12.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Logging-for-SOC/Screenshots/log12.jpg" width="600"/>
  </a>
</p>

---

## What I Learned

### Technical Skills
- How to navigate Event Viewer and use filters to isolate specific Event IDs and attack patterns
- Event ID 4625 (Failed Logon) + Logon Type 10 = RDP brute force detection
- Event ID 4624 (Successful Logon) + Logon ID = thread to correlate all attacker actions in one session
- Event ID 4720 / 4732 = backdoor account creation and privileged group membership detection
- Sysmon Event ID 1 (Process Create) - reveals execution path, parent process, hash, and user
- Sysmon Event ID 15 (FileCreateStreamHash) - captures Zone.Identifier with original download URL
- Sysmon Event ID 11 (FileCreate) - detects persistence via Startup folder drops
- Sysmon Event ID 3 (NetworkConnect) - reveals C2 IP and port
- Sysmon Event ID 22 (DNS Query) - reveals C2 domain from malware DNS lookups
- PowerShell ConsoleHost_history.txt persists across reboots and records all commands for all users

### Analyst Mindset
- The Logon ID is the thread that connects all events in an attacker's session - always note it when you find a malicious login
- Processes launched from Downloads, Temp, or Public directories are immediate red flags
- The Zone.Identifier (Sysmon ID 15) is one of the most underrated forensic artifacts - it tells you exactly where a file came from
- Always check PowerShell history for all users - attackers often leave traces in accounts other than the one they breached

---

## Honest Self-Assessment

**What went well:**
The correlation between Event IDs clicked naturally - connecting the brute force (4625) to the successful login (4624) to the backdoor creation (4720/4732) using the same Logon ID felt like real incident response. Sysmon's process chain was particularly satisfying to trace from download to execution to C2 contact.

**What I need to improve:**
Knowing which Sysmon Event ID to use for each type of artifact without referencing documentation. Right now I still need to check which event covers network connections vs file creation vs DNS queries. With more practice this will become second nature.

---

## Repository Structure

```
Windows-Logging-for-SOC/
├── README.md
└── Screenshots/
```

---

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Windows-Logging-for-SOC/Screenshots/logfinal.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Logging-for-SOC/Screenshots/logfinal.jpg" width="600"/>
  </a>
</p>

---

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
<p align="center"><a href="https://github.com/frankllin-sec">Visit my GitHub Profile</a></p>
