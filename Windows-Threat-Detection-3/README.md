# 🛡️ Windows Threat Detection 3: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Windows%20Threat%20Detection-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-3/Screenshots/dinicio.jpg" width="700"/>
</p>

---

## 📌 About This Lab

This room completes the Windows threat detection series after [Windows Threat Detection 1](https://github.com/frankllin-sec/Labs-experience/blob/main/Windows-Threat-Detection-1/README.md) and [2](https://github.com/frankllin-sec/Labs-experience/blob/main/Windows-Threat-Detection-2/README.md). It covers what happens when a threat actor decides to stay: setting up Command and Control, establishing Persistence through multiple methods, and understanding why that matters for Impact (ransomware).

**Objectives:**
- Remind the concept of Command and Control (C2)
- Learn why and how threat actors maintain control of their victims
- Use Windows event logs to uncover various persistence methods
- See how the learned techniques work in a hands-on environment

---

## 🎯 MITRE ATT&CK Coverage

| ID | Name | Description |
|---|---|---|
| **TA0011** | Command and Control | Maintaining a channel back to the attacker to receive further commands |
| **TA0003** | Persistence | Surviving reboots, password changes, and time to keep long-term access |
| **T1136** | Create Account | Creating a new backdoor user account for repeated access |
| **T1098** | Account Manipulation | Adding a backdoored account to a privileged group like Administrators |
| **T1543.003** | Create or Modify System Process: Windows Service | Creating a malicious service to run malware on startup |
| **T1547.001** | Boot or Logon Autostart Execution: Registry Run Keys / Startup Folder | Using the Run registry key or Startup folder to launch malware at logon |
| **TA0040** | Impact | The final stage attackers work toward, most commonly ransomware |

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **C2 Channel** | A process that phones home to attacker infrastructure and waits for commands, often disguised as a normal-looking executable |
| **Backdoored User** | A new or repurposed account created specifically to preserve remote access, usually added to a privileged group |
| **Service Persistence** | A malicious Windows service configured to auto-start, detectable via Security Event ID 4697 or Sysmon Event ID 1 |
| **Scheduled Task Persistence** | A task created via `schtasks.exe` that runs malware on a trigger (startup, schedule), detectable via Security Event ID 4698 |
| **Run Key / Startup Folder Persistence** | Per-user autostart methods: a registry value under `...CurrentVersion\Run` (Sysmon Event ID 13) or a file dropped in the Startup folder (Sysmon Event ID 11) |

---

## 🔍 Investigation

### Part 1: Command and Control

**Scenario:** Detecting a C2 setup from `Sysmon.evtx` after a phishing-driven infection.

**Q: Which suspicious archive did the user download?**

**Method:** Correlated Sysmon Event ID 1 (process started it), Event ID 11 (file landed), and Event ID 15 (Windows marked it as downloaded from the internet).

> **Answer:** `URGENT!.zip`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-3/Screenshots/d1.jpg" width="700"/>
</p>

**Q: Where did the attackers hide the C2 malware file? What is the domain of the Command and Control server?**

**Method:** Both answers came from the same Sysmon Event ID 22 (DNS query) entry.

> **Answers:** `C:\Users\Administrator\AppData\Roaming\update.exe` connecting to `route.m365officesync.workers.dev`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-3/Screenshots/d2.jpg" width="700"/>
</p>

---

### Part 2: Persistence via a Backdoored User Account

**Scenario:** Detecting persistence from `Security.evtx` after an RDP-based breach.

**Q: How many times did the threat actor fail to log in to the Administrator account?**

**Method:** Filtered Security Event ID 4625 (failed logon) targeting the Administrator account.

> **Answer:** `6`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-3/Screenshots/d3.jpg" width="700"/>
</p>

**Q: After the successful login, which backdoor user did the attacker create?**

**Method:** Looked for Security Event ID 4720 (user account created).

> **Answer:** `support`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-3/Screenshots/d4.jpg" width="700"/>
</p>

**Q: Which privileged group was the backdoor user added to?**

**Method:** Followed up with Security Event ID 4732 (member added to a security-enabled local group).

> **Answer:** `Administrators`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-3/Screenshots/d5.jpg" width="700"/>
</p>

---

### Part 3: Malware Persistence via Services and Scheduled Tasks

**Scenario:** Two backdoors were left behind and the system was restarted. Both had to be uncovered using Security and Sysmon logs.

**Q: Which Windows service was created to persist the Nessie malware?**

**Method:** Filtered Security Event ID 4697 (service installed) and spotted an unusual service file name pointing to `nessie.exe`, alongside two legitimate-looking Google/Microsoft Edge services in the same log.

> **Answer:** `Data Protection Service`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-3/Screenshots/d6.jpg" width="700"/>
</p>

**Q: Which scheduled task was created to persist the Troy malware?**

**Method:** Opened Task Scheduler and found three tasks configured to run at startup. One named `AmazonSync` stood out, and its Actions tab pointed to a file that was actually a trojan.

> **Answer:** `AmazonSync`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-3/Screenshots/d7.jpg" width="700"/>
</p>

**Q: What flag do you get after finding and running the Troy malware?**

**Method:** Located the Sysmon Event ID 1 entry for the process that `AmazonSync` launched and ran it to retrieve the flag.

> **Answer:** `THM{c2_is_on_schedule!}`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-3/Screenshots/d8.jpg" width="700"/>
</p>

---

### Part 4: Run Keys and Startup Persistence

**Scenario:** More backdoors to uncover, this time using per-user autostart methods (Run keys and the Startup folder).

**Q: What is the parent process image of the "Odin" malware?**
> **Answer:** `C:\Windows\explorer.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-3/Screenshots/d9.jpg" width="700"/>
</p>

**Q: What is the last line that the "Odin" malware outputs?**
> **Answer:** `Done doing bad stuff!`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-3/Screenshots/d10.jpg" width="700"/>
</p>

**Q: What flag do you get after finding and running the "Kitten" malware?**
> **Answer:** `THM{persisting_in_basket!}`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-3/Screenshots/d11.jpg" width="700"/>
</p>

---

### Part 5: Threat Detection Recap

**Q: What is the biggest threat to most corporate Windows networks?**
> **Answer:** `Ransomware`

**Q: At which stage is it best to detect and stop the attack?**
> **Answer:** `Initial Access`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-3/Screenshots/dfinal.jpg" width="700"/>
</p>

---

## 🧠 What I Learned

### Technical Skills
- How to trace a C2 channel from an initial phishing download through file-drop, internet-mark-of-the-web, and DNS query events (Sysmon Event IDs 1, 11, 15, 22)
- How to investigate a backdoored user end to end: failed logons (4625), account creation (4720), and privileged group membership (4732)
- How to spot malicious Windows services (Security Event ID 4697) hiding among legitimate-looking ones
- How to identify persistence through scheduled tasks by checking the Actions tab in Task Scheduler and correlating with Sysmon process creation
- How Run key (Sysmon Event ID 13) and Startup folder (Sysmon Event ID 11) persistence differ from services and tasks, and why both share explorer.exe as a parent process

### Analyst Mindset
- Persistence investigations benefit from asking who, what, and which: who created the account, what was the source IP and timing, and which other suspicious activity happened in that same session
- Don't rely on suspicious-sounding names alone (a service called "Data Protection Service" looks completely legitimate). Cross-reference the actual file path and creation context instead
- Scheduled tasks are the most attractive persistence method for attackers precisely because they're easy to hide among dozens of legitimate ones, which means a SOC analyst needs a habit of reviewing Task Scheduler entries against a known-good baseline
- The whole point of detecting Persistence, Discovery, and Collection early is to stop the attack chain well before it reaches Impact. Ransomware is the disaster; everything covered across these three rooms is the opportunity to catch it earlier

---

## 💬 Honest Self-Assessment

**What went well:**
Connecting the C2 domain and dropped file back to the same Sysmon Event ID 22 entry felt efficient, and the pattern of correlating multiple Security event IDs (4625 to 4720 to 4732) to build the full backdoor-user story came together smoothly by this point in the series.

**What I need to improve:**
Building a faster mental checklist for "is this service/task/registry entry legitimate?" without needing the room to flag which one is suspicious. In a live environment, I'd want a baseline of expected services and scheduled tasks for a given host so anomalies stand out immediately.

---

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-3/Screenshots/d12.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
