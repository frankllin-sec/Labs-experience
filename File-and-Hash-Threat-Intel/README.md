# 🛡️ File and Hash Threat Intel: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Threat%20Intelligence-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/File-and-Hash-Threat-Intel/Screenshots/finicio.jpg" width="700"/>
</p>

---

## 📌 About This Lab

It's a Monday in April. Try Daily is preparing a significant release when the EDR tool flags multiple binaries across various workstations during a routine alert sweep. As the L1 analyst shadowing an L2 mentor, I receive a curated triage package containing those samples. Within 60 minutes, I need evidence to show whether each file is bait, benign, or malicious.

**Objectives:**
- Interpret suspicious filepaths and filenames using heuristics
- Generate and validate file hashes
- Leverage VirusTotal and MalwareBazaar to enrich newly observed binaries
- Extract behaviour from sandbox telemetry and map it to MITRE ATT&CK

---

## 🔑 Key Concepts

**Filepath and filename heuristics:**

| Heuristic | Example | Why it works for an attacker |
|---|---|---|
| **Double extensions** | `invoice.pdf.exe` | Abuses Windows' default setting that hides file extensions |
| **System binary impersonation** | `scvhost.exe` | Abuses familiarity with core system process names |
| **High-entropy strings** | `jh8F21.exe` | Suggests automated packing or polymorphic generation |
| **Masquerading** | `backup-2300.exe` | Blends with routine files or uses single-character substitution |

Suspicious storage locations follow the same logic: `C:\Users\Public\`, `C:\Windows\Temp\`, and `C:\ProgramData\` all offer high-traffic or writable paths that evade strict monitoring.

**Hashing:** file names are trivial for an attacker to change, but a SHA256 or MD5 hash is an immutable fingerprint. Any byte change produces a different hash, so hashes are the reliable way to tie renamed samples back to the same binary. Store hashes in lowercase, hash both an archive and its extracted contents, and always keep the context of where and when a hash was encountered.

**Reading a VirusTotal report:**

| Section | Key Question | Red Flags |
|---|---|---|
| **Detection Score** | How many vendors flag it malicious? | 5+ solid vendors, conflicting classifications |
| **Upload Time** | When was it first submitted? | Sudden detection spike after days or weeks |
| **Signatures** | Is the file properly signed? | Invalid or missing certificate |
| **Properties** | Any anomalies in file metadata? | Compile timestamp at odd hours, high entropy |
| **Relations** | What infrastructure does it connect to? | Known-bad IPs, DGA-like domains |
| **Behavioural** | What post-execution actions occur? | Registry key changes, process injection |

**MalwareBazaar** complements VirusTotal with malware family tagging, YARA rule integration for future hunting, and campaign attribution tags (e.g. a threat actor group), which help link an isolated alert to a known adversary.

**Sandbox limitations:** sandbox results should never be trusted unquestioningly. Malware can evade sandboxes through environment-awareness checks and anti-debugging tricks, most sandboxes only run for 2-5 minutes so multi-stage or time-delayed payloads may not fully execute, encrypted C2 traffic can hide payloads entirely, and fileless Living-off-the-Land techniques (PowerShell, WMI) never touch disk at all.

---

## 🔍 Investigation

### Part 1: Filepath and Filename Analysis

**Q: One of the files in the CTI Files folder shows one of the indicators mentioned. Can you identify the file and the indicator?**

**Method:** Reviewed the files on the Desktop's `CTI Files` folder against the heuristics covered.

> **Answer:** `payroll.pdf, Double extensions`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/File-and-Hash-Threat-Intel/Screenshots/f1.jpg" width="700"/>
</p>

---

### Part 2: Hash Generation and VirusTotal / MalwareBazaar Analysis

**Q: What is the SHA256 hash of the file bl0gger?**

**Method:** Ran `certutil -hashfile bl0gger.exe SHA256` in the command prompt.

> **Answer:** `2672b6688d7b32a90f9153d2ff607d6801e6cbde61f509ed36d0450745998d58`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/File-and-Hash-Threat-Intel/Screenshots/f2.jpg" width="700"/>
</p>

**Q: What is the threat classification label used to identify the malicious file? When was it first submitted for analysis?**

**Method:** Pasted the generated SHA256 hash into TryDetectThis to pull the vendor intelligence report.

> **Answer:** `trojan.graftor/flystudio`, submitted `2025-05-15 12:03:49`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/File-and-Hash-Threat-Intel/Screenshots/f3.jpg" width="700"/>
</p>

**Q: Which vendor classified the Morse-Code-Analyzer file as non-malicious?**

**Method:** Generated the hash for `Morse-Code-Analyzer.exe` the same way, then checked the vendor detection table for the one outlier verdict among the rest.

> **Answer:** `CyberFortress`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/File-and-Hash-Threat-Intel/Screenshots/f4.jpg" width="700"/>
</p>

**Q: What MITRE technique has been flagged for persistence and privilege escalation for the Morse-Code-Analyzer file?**

> **Answer:** `DLL Side-Loading (T1574.002)`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/File-and-Hash-Threat-Intel/Screenshots/f5.jpg" width="700"/>
</p>

---

### Part 3: Sandbox Analysis (Hybrid Analysis)

**Q: What tags are used to identify the bl0gger.exe malicious file on Hybrid Analysis?**

> **Answer:** `BlackMoon, Discovery, windows-server-utility`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/File-and-Hash-Threat-Intel/Screenshots/f6.jpg" width="700"/>
</p>

**Q: What was the stealth command line executed from the file? Which other process was spawned according to the process tree?**

**Method:** Checked the Process Tree tab under Sandbox Analysis for the parent process and its children.

> **Answer:** `regsvr32 %WINDIR%\Media\ActiveX.ocx /s`, which spawned `werfault.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/File-and-Hash-Threat-Intel/Screenshots/f7.jpg" width="700"/>
</p>

**Q: The payroll.pdf application seems to be masquerading as which known Windows file?**

**Method:** Hashed `payroll.pdf.exe` and checked its Impersonation Behaviour tab.

> **Answer:** `svchost.dll`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/File-and-Hash-Threat-Intel/Screenshots/f8.jpg" width="700"/>
</p>

**Q: What associated URL is linked to the file?**

> **Answer:** `hxxp://121.182.174.27:3000/server.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/File-and-Hash-Threat-Intel/Screenshots/f9.jpg" width="700"/>
</p>

**Q: How many extracted strings were identified from the sandbox analysis of the file?**

> **Answer:** `454`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/File-and-Hash-Threat-Intel/Screenshots/f10.jpg" width="700"/>
</p>

---

### Part 4: Threat Intelligence Challenge, Analysing Challenge.bin.sample

**Scenario:** A suspected file, `Challenge.bin.sample`, needs the same full workup applied independently.

**Q: What is the SHA256 hash of the file?**

> **Answer:** `43b0ac119ff957bb209d86ec206ea1ec3c51dd87bebf7b4a649c7e6c7f3756e7`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/File-and-Hash-Threat-Intel/Screenshots/f11.jpg" width="700"/>
</p>

**Q: What family labels are assigned to the file on VirusTotal?**

**Method:** 61 of 71 vendors flagged the file malicious, with a popular threat label pointing to a known ransomware family.

> **Answer:** `akira, filecryptor`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/File-and-Hash-Threat-Intel/Screenshots/f12.jpg" width="700"/>
</p>

**Q: When was the first time the file was recorded in the wild?**

> **Answer:** `2024-10-30 17:17:24 UTC`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/File-and-Hash-Threat-Intel/Screenshots/f13.jpg" width="700"/>
</p>

**Q: Name the text file dropped during the execution of the malicious file.**

**Method:** Checked the Dropped Files section of the report for anything with active detections.

> **Answer:** `akira_readme.txt`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/File-and-Hash-Threat-Intel/Screenshots/f14.jpg" width="700"/>
</p>

**Q: What PowerShell command is observed to be executed?**

> **Answer:** `Get-WmiObject Win32_Shadowcopy | Remove-WmiObject`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/File-and-Hash-Threat-Intel/Screenshots/f15.jpg" width="700"/>
</p>

**Q: What MITRE ATT&CK ID is associated with the actions of the command?**

**Method:** The command deletes volume shadow copies, a classic ransomware move to block recovery, but the report didn't name the technique directly. Googled `get wmiobject win32_shadowcopy remove wmiobject mitre` to confirm it.

> **Answer:** `T1490` (Inhibit System Recovery)

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/File-and-Hash-Threat-Intel/Screenshots/f16.jpg" width="700"/>
</p>

---

## 🧠 What I Learned

- How to read filepaths and filenames as early heuristics before any technical analysis, double extensions, system binary impersonation, high-entropy strings, and masquerading
- Why hashes, not filenames, are the reliable way to track a binary across renames
- How to read a VirusTotal report section by section: detection score, upload time, signatures, properties, relations, and behavioural indicators
- What MalwareBazaar adds on top of VirusTotal: family tagging, YARA rules, and campaign attribution
- How to pull process trees, command lines, and network indicators out of a Hybrid Analysis sandbox report
- The specific limitations of sandboxing (evasion, execution time limits, encrypted traffic, fileless malware) and why sandbox output alone is never the full picture
- Recognising `Get-WmiObject Win32_Shadowcopy | Remove-WmiObject` as a shadow-copy deletion technique mapped to MITRE T1490, common across ransomware families like Akira

---

## 💬 Honest Self-Assessment

**What went well:**
Once I had the hash, pivoting into TryDetectThis to pull classification, vendor verdicts, and sandbox behaviour felt fast and repeatable, the same workflow worked for `bl0gger.exe`, `Morse-Code-Analyzer.exe`, `payroll.pdf.exe`, and the final challenge file. Reading the process tree and pulling out the stealth `regsvr32 /s` command made sense once I knew where to look.

**What I need to improve:**
The last question in the challenge, mapping the PowerShell shadow-copy deletion command to its MITRE ATT&CK ID, wasn't something I could answer directly from the tool. I had to Google the command to confirm it was T1490. I want to get to the point where I recognise common ransomware pre-encryption behaviours like shadow-copy deletion on sight, instead of needing to look them up each time.

---
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/File-and-Hash-Threat-Intel/Screenshots/ffinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
