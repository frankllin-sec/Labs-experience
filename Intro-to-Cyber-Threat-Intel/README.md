# 🛡️ Intro to Cyber Threat Intel: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Threat%20Intelligence-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Intro-to-Cyber-Threat-Intel/Screenshots/cinicio.jpg" width="700"/>
</p>

---

## 📌 About This Lab

A junior analyst logs in for their shift and finds two hundred fresh alerts waiting, everything from benign network scans to a PowerShell beacon that feels out of place. The ticket queue does not politely pace itself, it demands quick, confident decisions. This is where Cyber Threat Intelligence (CTI) earns its keep.

Threat intelligence provides the context that helps an analyst decide which of those alerts represents genuine danger, so the SOC spends energy on the right issues instead of chasing noise.

**Objectives:**
- Understand what CTI is and why it matters at Tier 1
- Learn the data, information, and intelligence pyramid
- Learn the difference between IOC, IOA, and TTP
- Understand indicator types and where to enrich each one
- Learn the difference between a feed and a platform
- Learn the main sources of CTI and the four intelligence classifications
- Apply the concepts by building a threat actor profile from simulated alerts

---

## 🔑 Key Concepts

CTI tries to answer three essential questions for every alert: who or what is on the other end of it, what their past behaviour has been, and how the organisation should respond right now.

**From raw data to usable intelligence:**

| Layer | Definition | Alert-queue example | SOC L1 action |
|---|---|---|---|
| **Data** | An unprocessed observable | `45.155.205.3:443` | Capture the artefact |
| **Information** | Data plus factual annotation | IP registered to Hetzner, first seen 2023-07-14 | Record attributes |
| **Intelligence** | Analysed information that answers "so what" | IP belongs to a known C2, block immediately | Escalate or suppress |

**Three labels every analyst needs:**

| Term | Meaning |
|---|---|
| **IOC (Indicator of Compromise)** | Evidence a breach already happened, like a C2 address in the logs |
| **IOA (Indicator of Attack)** | A malicious action currently underway, like PowerShell launching an unknown service |
| **TTP (Tactics, Techniques, Procedures)** | An adversary's methodology, expressed in MITRE ATT&CK IDs and descriptions |

**Common indicator types and first resources to check:**

| Indicator | Example | First Resources |
|---|---|---|
| **IPv4 / IPv6** | `45.155.205.3` | WHOIS, VirusTotal Relations, Shodan |
| **Domain / FQDN** | `malicious-updates[.]net` | WHOIS age, passive DNS, urlscan.io |
| **URL** | `hxxp://malicious-updates[.]net/login` | URLhaus, urlscan.io, Any.Run |
| **File hash** | `e99a18c428cb38d5...` | VirusTotal, Hybrid-Analysis, MalShare |
| **Email address** | `billing@evil-corp.com` | MXToolbox, Have I Been Pwned |
| **Local artefact** | `HKCU\Software\Run\updater.exe` | Sigma rules, EDR prevalence query |

**Feed vs Platform:**
- **Feed:** a scheduled stream of indicators (CSV, JSON, STIX, TAXII). Over-ingesting without curation drowns analysts in false positives.
- **Platform:** a structured repository that stores indicators, tracks enrichment, and maps relationships. MISP and OpenCTI are leading open-source examples.

**Sources of CTI:**
- **Internal telemetry:** SIEM logs, EDR detections, phishing-mailbox submissions
- **Commercial services:** vendor premium feeds, paid sandboxes, closed-source analytics
- **OSINT:** AbuseIPDB, URLhaus, public blogs, academic research
- **Communities & ISACs:** sector-specific lists with rich context (e.g. FS-ISAC)

**Threat intelligence classifications:**

| Classification | Focus | Example |
|---|---|---|
| **Strategic** | High-level trends shaping business decisions | Annual ransomware trends report |
| **Tactical** | Adversary TTPs | Advisory on new VBA abuse in malspam |
| **Operational** | Campaign-specific motive and intent | Which assets a campaign is likely targeting |
| **Technical** | Atomic indicators and artefacts | IPs, hashes tied to an attack |

An L1 analyst mostly works at the Technical layer (escalating IOCs), documents Tactical IOAs when spotted, and feeds patterns upward into Operational reporting.

---

## 🔍 Investigation

### Task 5: Practical Analysis, Building a Threat Profile

**Scenario:** A simulated security monitoring tool shows a run of alerts on `badbank.com`: a phishing email from `vipivillain@badbank.com`, a file download (`flbpfuh.exe`), outbound traffic to `91.185.23.222`, and a successful Administrator login. The task is to correlate these alerts into a threat actor profile (attacker, malware, victim, and the relationships between them).

**Q: What was the threat actor's extraction IP address?**

> **Answer:** `91.185.23.222`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Intro-to-Cyber-Threat-Intel/Screenshots/c1.jpg" width="700"/>
</p>

**Q: What was the threat actor's email address?**

> **Answer:** `vipivillain@badbank.com`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Intro-to-Cyber-Threat-Intel/Screenshots/c2.jpg" width="700"/>
</p>

**Q: What software tool was used in the extraction?**

> **Answer:** `flbpfuh.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Intro-to-Cyber-Threat-Intel/Screenshots/c3.jpg" width="700"/>
</p>

**Q: What user account was logged in by the threat actor?**

> **Answer:** `Administrator`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Intro-to-Cyber-Threat-Intel/Screenshots/c4.jpg" width="700"/>
</p>

**Q: Who was the targeted victim?**

> **Answer:** `John Doe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Intro-to-Cyber-Threat-Intel/Screenshots/c5.jpg" width="700"/>
</p>

**Q: After building the threat profile, what message do you receive?**

> **Answer:** `THM{NOW_I_CAN_CTI}`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Intro-to-Cyber-Threat-Intel/Screenshots/c6.jpg" width="700"/>
</p>

---

## 🧠 What I Learned

- The data, information, intelligence pyramid, and why raw artefacts need enrichment before they're actionable
- The difference between IOC (evidence a breach happened), IOA (an attack in progress), and TTP (the adversary's methodology)
- Which first resources to check for each indicator type (IP, domain, URL, hash, email, local artefact)
- Why a feed and a platform serve different purposes, and why over-ingesting feeds without curation hurts more than it helps
- The four classifications of threat intel (Strategic, Tactical, Operational, Technical) and where an L1 analyst's work actually sits
- How to correlate scattered alerts (email, file, IP, login, victim) into a single threat actor profile

---

## 💬 Honest Self-Assessment

**What went well:**
The practical part (Task 5) was straightforward: matching each alert to the right field in the threat profile tool wasn't hard once I understood the scenario. Correlating the email, the file, the IP, and the account login into one story felt natural.

**What I need to improve:**
This lab was much heavier on theory than the hands-on labs I've done so far, and the terminology is where I need more repetition. IOC vs IOA vs TTP, and feed vs platform, are the kind of distinctions that are easy to nod along to while reading but harder to recall cold during an actual triage. I want to drill these definitions until I can explain each one without looking it up.

---
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Intro-to-Cyber-Threat-Intel/Screenshots/cfinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
