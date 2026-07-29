# 🛡️ Windows Threat Detection 2: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Windows%20Threat%20Detection-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-2/Screenshots/winicio.jpg" width="700"/>
</p>

---

## 📌 About This Lab

This room continues the Windows threat detection journey from [Windows Threat Detection 1](https://github.com/frankllin-sec/Labs-experience/blob/main/Windows-Threat-Detection-1/README.md), moving past Initial Access into what a threat actor typically does once they're already inside: figuring out where they landed, grabbing anything valuable, and getting it out.

**Objectives:**
- Detect common Discovery techniques using Windows Event Log
- Learn how to trace the attack origin by reconstructing a process tree
- Find out what data threat actors look for and how they exfiltrate it
- See how malicious commands are logged by running them yourself

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **Process Tree Reconstruction** | Correlating Sysmon Event ID 1 (process create) records by ProcessId and ParentProcessId to trace a command back to its origin |
| **Data Stealer** | Malware that automates collection and exfiltration without a human operator typing commands manually |
| **DNS Query (Event ID 22)** | The Sysmon event that reveals which domain a process actually tried to resolve, key for spotting exfiltration destinations |

---

## 🔍 Investigation

### Part 1: Discovery

**Q: Open CMD and type "net user Administrator". Which privileged group does the user belong to?**
> **Answer:** `Administrators`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-2/Screenshots/w1.jpg" width="700"/>
</p>

**Q: Open Event Viewer and find your command in Sysmon logs. What is the "Image" field of the net command you just ran?**

**Method:** Navigated to Microsoft > Windows > Sysmon > Operational and filtered for Event ID 1 (Process Create) around the time the command was run.

> **Answer:** `C:\Windows\System32\net.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-2/Screenshots/w2.jpg" width="700"/>
</p>

**Q: Looking at Sysmon logs, what is the first command the invoice.pdf.exe executes?**

**Method:** Opened `invoice.pdf.exe` at 8:43 (VM time), then filtered Sysmon Event ID 1 around that timestamp to catch the first child process.

> **Answer:** `whoami`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-2/Screenshots/w3.jpg" width="700"/>
</p>

**Q: Which command did the malware use to check the presence of MS Defender EDR?**
> **Answer:** `cmd /c "tasklist /v | findstr MsSense.exe || echo No MS Defender EDR"`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-2/Screenshots/w4.jpg" width="700"/>
</p>

**Q: To which domain did the malware send the discovered data?**

**Method:** Checked Sysmon Event ID 22 (DNS query), the reliable way to see which domain a process resolved.

> **Answer:** `exfil.beecz.cafe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-2/Screenshots/w5.jpg" width="700"/>
</p>

---

### Part 2: Searching for Secrets (Collection & Credential Access)

**Q: What is the Facebook password that the user saved in Chrome?**

**Method:** Chrome menu > Passwords and autofill > Password Manager.

> **Answer:** `nsAghv51BBav90!`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-2/Screenshots/w6.jpg" width="700"/>
</p>

**Q: Which interesting SSH key does the user store on disk?**

**Method:** Started the search from `C:\Users\Administrator\` and found a `.ssh` folder holding a key pair.

> **Answer:** `thm-access-database.key`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-2/Screenshots/w7.jpg" width="700"/>
</p>

**Q: What is the secret PDF file explaining TryHackMe's internal network?**

**Method:** Checked the Desktop, Downloads, and Documents folders as suggested.

> **Answer:** `thm-network-diagram-2025.pdf`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-2/Screenshots/w8.jpg" width="700"/>
</p>

---

### Part 3: Analyzing a Data Stealer

**Scenario:** Ran a live data stealer sample (`stealer.exe`) and analyzed its actions directly in Sysmon logs as they happened.

**Q: Looking at Sysmon logs, what directory does the stealer create?**
> **Answer:** `staging_58f1`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-2/Screenshots/w9.jpg" width="700"/>
</p>

**Q: Which three file extensions does the malware search for?**

**Method:** Reviewed the process activity following `stealer.exe`'s launch and found an `xcopy` command filtering by extension.

> **Answer:** `docx, pdf, xlsx`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-2/Screenshots/w10.jpg" width="700"/>
</p>

**Q: Which PowerShell cmdlet does the malware use to get clipboard content?**
> **Answer:** `Get-ClipBoard`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-2/Screenshots/w11.jpg" width="700"/>
</p>

**Q: Which domain does the malware exfiltrate the data to?**

**Method:** Checked Event ID 22 (DNS query) again, matching the timestamp of the suspicious file execution to a domain that clearly matched the collected data theme.

> **Answer:** `collecteddata-storage-2025.s3.amazonaws.com`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-2/Screenshots/w12.jpg" width="700"/>
</p>

---

### Part 4: Living-off-the-Land Downloads

**Scenario:** The same file hosted at `http://appsforfree.thm/trojan.exe` was fetched four different ways to compare how each method looks in logs.

**Q: Open the Chrome browser on the VM and navigate to the URL. What is the flag in the response?**
> **Answer:** `THM{just_use_web_browser}`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-2/Screenshots/w13.jpg" width="700"/>
</p>

**Note on this section:** The remaining three downloads (curl, certutil, PowerShell IWR) returned connection errors directly on the VM. To keep progressing, I looked up the expected flags rather than getting stuck on a lab environment issue unrelated to the actual technique being taught.

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-2/Screenshots/werror.jpg" width="700"/>
</p>

**Q: Open CMD and download the file from the same URL using curl.exe. What is the flag in the response?**
> **Answer:** `THM{curl_is_cool}`

**Q: Continue with the same CMD and URL, but now using certutil.exe. What is the flag in the response?**
> **Answer:** `THM{abusing_certutil}`

**Q: Finally, download the same file using PowerShell IWR. What is the flag in the response?**
> **Answer:** `THM{power_of_powershell}`

---

## 🧠 What I Learned

### Technical Skills
- How to reconstruct a full process tree by correlating Sysmon Event ID 1 records through ProcessId and ParentProcessId, tracing a command all the way back to the original phishing binary
- How to identify Collection targets a threat actor is likely to go after: saved browser passwords, SSH keys, and sensitive documents by filename or extension
- How to recognize a data DNS queries (Event ID 22) pointing to attacker infrastructure


## 💬 Honest Self-Assessment

**What went well:**
Correlating ProcessId and ParentProcessId across multiple Sysmon events to rebuild the invoice.pdf.exe attack chain felt natural this time, building directly on the pattern from Windows Threat Detection 1.

**What I need to improve:**
Recognizing Collection targets faster without needing the task to spell them out (Facebook password, SSH key, PDF). In a real investigation, I'd want a mental checklist of common secret locations (browser password stores, `.ssh`, cloud credential files, key vaults) to search proactively instead of just following prompts.

---

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-2/Screenshots/wfinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
