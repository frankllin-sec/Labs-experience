# 🖥️ Introduction to EDR — TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-EDR%20Investigation-blue?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-green?style=for-the-badge"/>
</p>

---

## 📌 About This Lab

This lab introduces **Endpoint Detection and Response (EDR)** — one of the most widely adopted security solutions in enterprise environments. Unlike traditional antivirus, EDR provides deep visibility into endpoint activity, process chains, network connections, and behavioral analysis.

As a SOC analyst at TECH THM, the task was to triage multiple medium and high-severity detections across three endpoints using the EDR console — analyzing process chains, identifying malicious behavior, and extracting key IOCs.

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Introduction-to-EDR/Screenshots/edrintro.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Introduction-to-EDR/Screenshots/edrintro.jpg" width="600"/>
  </a>
</p>

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **EDR** | Endpoint Detection and Response — monitors, detects, and responds to advanced threats at endpoint level |
| **Process Chain** | Visual representation of parent-child process relationships — reveals attack execution flow |
| **IOC** | Indicator of Compromise — file hash, IP, URL, or process name linked to malicious activity |
| **Credential Dumping** | Extracting credentials from memory (LSASS) to enable lateral movement |
| **Staging** | Downloading a payload to disk without executing it — preparing for later execution |
| **Threat Intel** | Contextual information about known threats, tools, and actor behavior |

---

## 🔍 Investigation — EDR Dashboard Triage

### Scenario

SOC analyst at TECH THM with access to the EDR console. Multiple medium and high-severity detections were present across three endpoints. The task was to analyze each detection's process chain, behavior, and IOCs to answer investigation questions.

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Introduction-to-EDR/Screenshots/edr1.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Introduction-to-EDR/Screenshots/edr1.jpg" width="600"/>
  </a>
</p>

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Introduction-to-EDR/Screenshots/edr7.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Introduction-to-EDR/Screenshots/edr7.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q1 — Which tool was launched by CMD.exe to download the payload on DESKTOP-HR01?

**Detection:** Initial Access via Malicious Office Document | Severity: HIGH | Host: `DESKTOP-HR01` | User: `alice.thomas`

**Investigation:** A macro-enabled Office document (`invoice.docm`) was opened via `WINWORD.EXE`. The document triggered `CMD.EXE`, which launched a download tool to retrieve a payload from an external domain. The file was saved to the Public folder but never executed — consistent with **malware staging tactics**.

**Process Chain:** `WINWORD.EXE → CMD.EXE → cURL.EXE → INSTALL.EXE`

> **Answer:** `curl.exe`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Introduction-to-EDR/Screenshots/edr2.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Introduction-to-EDR/Screenshots/edr2.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q2 — What is the absolute path to the downloaded malware on DESKTOP-HR01?

**Method:** Inspected the `INSTALL.EXE` node in the process chain. Reviewed the Path field which showed the full absolute path where the payload was dropped. Status: **Quarantined by EDR**. Execution: **Not launched**.

> **Answer:** `C:\Users\Public\install.exe`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Introduction-to-EDR/Screenshots/edr3.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Introduction-to-EDR/Screenshots/edr3.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q3 — What is the absolute path to the suspicious syncsvc.exe on WIN-ENG-LAPTOP03?

**Detection:** Credential Dumping via LSASS Memory Access | Severity: HIGH | Host: `WIN-ENG-LAPTOP03` | User: `haris.khan`

**Investigation:** `syncsvc.exe` (unsigned, not a legitimate Windows process) was launched from a temp directory. It accessed the memory of `lsass.exe` and wrote a dump file to disk — classic **credential dumping** behavior matching **T1003.001 (LSASS Memory)**. It also attempted to exfiltrate the dump file via WeTransfer.

**Process Chain:** `explorer.exe → syncsvc.exe → lsass.exe`

> **Answer:** `C:\Users\haris.khan\AppData\Local\Temp\syncsvc.exe`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Introduction-to-EDR/Screenshots/edr4.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Introduction-to-EDR/Screenshots/edr4.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q4 — On which URL was the exfiltration attempt made on WIN-ENG-LAPTOP03?

**Method:** Reviewed the Network Activity section of the `syncsvc.exe` detection. Identified an attempted upload of `dump_2025.dmp` to a WeTransfer URL. Status: **Blocked by EDR**.

> **Answer:** `https://files-wetransfer.com/upload/session/ab12cd34ef56/dump_2025.dmp`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Introduction-to-EDR/Screenshots/edr5.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Introduction-to-EDR/Screenshots/edr5.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q5 — What was UpdateAgent.exe labelled by Threat Intel on DESKTOP-DEV01?

**Detection:** Execution from AppData Directory | Severity: MEDIUM | Host: `DESKTOP-DEV01` | User: `daniel.richards`

**Investigation:** `UpdateAgent.exe` was launched from `AppData\Roaming` — a common path used by malware to avoid detection. However, Threat Intel classified it as a known internal IT utility tool. The process initiated an outbound connection to `10.10.20.5` on port 8080 via HTTP — an internal IP, consistent with legitimate IT activity.

**Process Chain:** `explorer.exe → UpdateAgent.exe`

> **Answer:** `Known internal IT utility tool`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Introduction-to-EDR/Screenshots/edr6.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Introduction-to-EDR/Screenshots/edr6.jpg" width="600"/>
  </a>
</p>

---

## 🗺️ MITRE ATT&CK Mapping

| Detection | Tactic | Technique |
|---|---|---|
| Initial Access via Malicious Office Document | Initial Access (TA0001) | T1566.001 — Spearphishing Attachment |
| CMD launching cURL for payload download | Execution (TA0002) | T1059.003 — Windows Command Shell |
| Credential Dumping via LSASS | Credential Access (TA0006) | T1003.001 — LSASS Memory |
| Exfiltration attempt via WeTransfer | Exfiltration (TA0010) | T1048 — Exfiltration Over Alternative Protocol |
| Execution from AppData | Defense Evasion (TA0005) | T1036 — Masquerading |

---

## 🧠 What I Learned

### Technical Skills
- How to navigate an EDR dashboard and analyze process chains to trace attack execution flow
- How macro-enabled Office documents trigger CMD → cURL download chains — a common initial access technique
- How credential dumping via LSASS works and how EDR detects it via behavioral signatures
- How to read Threat Intel labels in EDR to distinguish malicious tools from legitimate IT utilities
- How EDR blocks exfiltration attempts at the network level before data leaves the endpoint

### Analyst Mindset
- Process chains are the most valuable artifact in EDR — always trace from parent to child to understand the full attack flow
- AppData execution is suspicious by default, but Threat Intel context can confirm legitimacy — never assume without checking
- A file saved to disk but not executed is still a threat — it is malware staging and requires immediate containment
- EDR telemetry gives you answers that SIEM alone cannot — process hashes, command lines, and network connections in one view

---

## 💬 Honest Self-Assessment

**What went well:**
Reading the process chains and connecting each node to the right MITRE technique felt natural. The credential dumping detection was particularly clear — `syncsvc.exe` accessing `lsass.exe` from a temp directory with no signature is an immediate red flag.

**What I need to improve:**
Getting faster at distinguishing legitimate tools from malicious ones without relying on Threat Intel labels. In a real SOC, Threat Intel may not always have a verdict — building the instinct to evaluate process behavior independently is the next level.

---

## 📁 Repository Structure

```
Introduction-to-EDR/
├── README.md
└── Screenshots/
```

---

## 🖼️ Screenshots

> Screenshots of the EDR dashboard, process chains, and detection details are available in the [`Screenshots`](./Screenshots/) folder.

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Introduction-to-EDR/Screenshots/edrend.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Introduction-to-EDR/Screenshots/edrend.jpg" width="600"/>
  </a>
</p>

---

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
<p align="center"><a href="https://github.com/frankllin-sec">🔗 Visit my GitHub Profile</a></p>
