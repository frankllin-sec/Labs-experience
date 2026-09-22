# 🛡️ Boogeyman 3 - Full Attack Chain Investigation: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Incident%20Response-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/cinicio.jpg" width="700"/>
</p>

**Room link:** [tryhackme.com/room/boogeyman3](https://tryhackme.com/room/boogeyman3)

The Boogeyman returns for a third time. After the previous attacks, Quick Logistics LLC hired a managed security service provider to run its SOC, but the threat group was still lurking, waiting for the right moment to strike again. This time the investigation runs entirely through an Elastic Stack (Kibana), covering the full chain from initial phishing to a domain-wide ransomware deployment.

---

## 📌 About This Lab

Evan Hutchinson, the CEO of Quick Logistics LLC, received a phishing email and opened the attachment. It looked like nothing happened, so he reported it to the security team, who found the attachment sitting in his downloads folder, along with a suspicious file inside an ISO payload. From there, the investigation traces the entire attack: initial execution, persistence, C2 communication, UAC bypass, credential dumping, lateral movement across multiple machines, a DCSync attack against the domain controller, and finally a ransomware deployment.

**Tools used:** Elastic Stack / Kibana (KQL queries against Sysmon and Windows event logs), Mimikatz (identified as the attacker's credential dumping tool)

**Objectives:**
- Query Sysmon and Windows event logs in Kibana to trace process execution, persistence, and C2 activity
- Identify a UAC bypass and follow the attacker's credential dumping and lateral movement across the network
- Trace the attack through to a DCSync attack against the domain controller and the final ransomware deployment

---

## 🔍 Investigation

### Part 1: Initial Access & Payload Execution

**Q: What is the PID of the process that executed the initial stage 1 payload?**

**Method:** Narrowed the search to the incident window (August 29 to 30, 2023) and filtered on the attachment's name, `ProjectFinancialSummary_Q3.pdf`. Since the file was named like a PDF but nothing visibly happened when Evan opened it, it was likely disguised, so I also searched for `html`, since HTML smuggling attacks deliver malicious payloads as HTML files that only look like PDFs to the victim.

> **Answer:** `6392`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/C1.jpg" width="700"/>
</p>

**Q: The stage 1 payload attempted to implant a file to another location. What is the full command-line value of this execution?**

**Method:** Filtered on the same attachment name, `ProjectFinancialSummary_Q3.pdf`, to follow its next action in the logs.

> **Answer:** `"C:\Windows\System32\xcopy.exe" /s /i /e /h D:\review.dat C:\Users\EVAN~1.HUT\AppData\Local\Temp\review.dat`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/c2.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/c2-1.jpg" width="700"/>
</p>

**Q: The implanted file was eventually used and executed by the stage 1 payload. What is the full command-line value of this execution?**

> **Answer:** `"C:\Windows\System32\rundll32.exe" D:\review.dat,DllRegisterServer`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/c3.jpg" width="700"/>
</p>

**Q: The stage 1 payload established a persistence mechanism. What is the name of the scheduled task created by the malicious script?**

> **Answer:** `Review`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/c4.jpg" width="700"/>
</p>

---

### Part 2: C2 Connection & Privilege Escalation

**Q: The execution of the implanted file has initiated a potential C2 connection. What is the IP and port used by this connection? (format: IP:port)**

**Method:** Filtered with `event.provider: "Microsoft-Windows-Sysmon" AND event.code: 3`, since Event Code 3 is Sysmon's network connection log, the only event type that records the destination IP and port a process connected to. I included the provider because Event Code 3 alone isn't unique, other log sources can reuse the same code for unrelated events, so specifying Sysmon ensures I'm pulling from the right log type.

> **Answer:** `165.232.170.151:80`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/C5.jpg" width="700"/>
</p>

**Q: The attacker has discovered that the current access is a local administrator. What is the name of the process used by the attacker to execute a UAC bypass?**

**Method:** I wasn't sure how to build the right filter for this one, so I asked Claude for help identifying the best query for this specific situation. It suggested filtering on the known UAC bypass binaries: `event.provider: "Microsoft-Windows-Sysmon" AND event.code: 1 AND process.name: ("fodhelper.exe" OR "eventvwr.exe" OR "computerdefaults.exe" OR "sdclt.exe")`.

> **Answer:** `fodhelper.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/c6.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/c6-1.jpg" width="700"/>
</p>

---

### Part 3: Credential Access & Lateral Movement

**Q: Having high privilege machine access, the attacker attempted to dump the credentials inside the machine. What is the GitHub link used by the attacker to download a tool for credential dumping?**

**Method:** Searched Google for common credential dumping tools, which pointed to Mimikatz, then searched the logs for that tool's name directly.

> **Answer:** `https://github.com/gentilkiwi/mimikatz/releases/download/2.2.0-20220919/mimikatz_trunk.zip`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/C7.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/c7-1.jpg" width="700"/>
</p>

**Q: After successfully dumping the credentials inside the machine, the attacker used the credentials to gain access to another machine. What is the username and hash of the new credential pair? (format: username:hash)**

**Method:** Searched the logs for `mimikatz.exe` output and dug deeper around the CEO's account, `Evan.Hutchinson`, which surfaced the new username and hash.

> **Answer:** `itadmin:F84769D250EB95EB2D7D8B4A1C5613F2`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/c8.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/c8-1.jpg" width="700"/>
</p>

**Q: Using the new credentials, the attacker attempted to enumerate accessible file shares. What is the name of the file accessed by the attacker from a remote share?**

**Method:** Filtered by the CEO's infected machine hostname, `WKSTN-0051`, combined with `powershell.exe`, since PowerShell is a common tool for enumerating file shares. Added a `*file*` wildcard to narrow the command line results.

> **Answer:** `IT_Automation.ps1`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/c9.jpg" width="700"/>
</p>

**Q: After getting the contents of the remote file, the attacker used the new credentials to move laterally. What is the new set of credentials discovered by the attacker? (format: username:password)**

> **Answer:** `QUICKLOGISTICS\allan.smith:Tr!ckyP@ssw0rd987`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/c10.jpg" width="700"/>
</p>

**Q: What is the hostname of the attacker's lab machine for its lateral movement attempt?**

> **Answer:** `WKSTN-1327`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/c11.jpg" width="700"/>
</p>

**Q: Using the malicious command executed by the attacker from the first machine to move laterally, what is the parent process name of the malicious command executed on the second compromised machine?**

**Method:** Filtered by the second machine's hostname combined with Event Code 1, since that log shows every process that starts on a machine and what launched it. This surfaced `powershell.exe` running under `wsmprovhost.exe`, a process that handles remote connections, confirming the attacker connected to this machine from the first one and ran their command through it.

> **Answer:** `wsmprovhost.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/c12.jpg" width="700"/>
</p>

**Q: The attacker then dumped the hashes in this second machine. What is the username and hash of the newly dumped credentials? (format: username:hash)**

**Method:** Filtered using the second machine's hostname, `WKSTN-1327`, combined with `mimikatz`, scoping the search to activity related to the credential dumping tool on that machine.

> **Answer:** `administrator:00f80f2538dcb54e7adc715c0e7091ec`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/c13.jpg" width="700"/>
</p>

---

### Part 4: Domain Compromise & Ransomware Deployment

**Q: After gaining access to the domain controller, the attacker attempted to dump the hashes via a DCSync attack. Aside from the administrator account, what account did the attacker dump?**

**Method:** Already knew Mimikatz was the tool in use from the earlier findings, and the question specified a DCSync attack, so I filtered `mimikatz AND dcsync` to find the exact command and the account it targeted.

> **Answer:** `backupda`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/c14.jpg" width="700"/>
</p>

**Q: After dumping the hashes, the attacker attempted to download another remote file to execute ransomware. What is the link used by the attacker to download the ransomware binary?**

**Method:** Filtered `"WKSTN-1327" and *file*` to look for anything suspicious, which surfaced a file named `ransomboogey.exe`. Filtered again with `"WKSTN-1327" and *ransomboogey.exe*` to confirm the full download command.

> **Answer:** `ransomboogey.exe` (full download command confirmed in the query results below)

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/c15-1.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/c15-2.jpg" width="700"/>
</p>

---

## 🧠 What I Learned

- How to trace a full attack chain end to end inside Kibana, from initial phishing execution to a domain-wide ransomware deployment
- That Sysmon Event Code 3 (network connections) and Event Code 1 (process creation) cover most of what's needed to trace C2 activity and parent-child process relationships
- How attackers escalate access step by step: local admin, UAC bypass, credential dumping, lateral movement, then domain compromise, before finally deploying ransomware

---

## 💬 Honest Self-Assessment

**What I need to improve:**
I'm still entry level in cybersecurity, so for a few of the filters in this lab, especially the UAC bypass one, I wasn't sure what I needed to search for. I asked Claude to help me understand what should be done and why, instead of just giving me the answer, which helped me actually learn the logic behind the filter instead of just copying it.

Stepping back, this whole lab is really just one continuous story: the attacker got their first foothold on the victim's computer through a phishing email, and from there kept escalating step by step (gaining local admin, bypassing UAC, stealing credentials, moving to other computers on the network, and finally taking over the domain controller) until they had enough access to deploy ransomware. Seeing that full chain laid out helped me understand why each stage of an attack matters, not just the individual technical steps.

---

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-3/Screenshots/cfinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
<p align="center"><a href="https://github.com/frankllin-sec">🔗 Visit my GitHub Profile</a></p>
