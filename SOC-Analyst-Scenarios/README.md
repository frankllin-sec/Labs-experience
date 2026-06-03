# 🛡️ SOC Analyst Scenarios — TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-SOC%20Scenarios-blue?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-green?style=for-the-badge"/>
</p>

---
<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/SOC-Analyst-Scenarios/Screenshots/systematacintro.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Analyst-Scenarios/Screenshots/systematacintro.jpg" width="600"/>
  </a>
</p>

## 📌 About This Lab

This lab covers two progressive SOC analyst scenarios at TryHackMe. Unlike basic monitoring labs, these simulations show a SOC team deeply embedded in company operations working with IT, HR, and employees directly. Scenarios cover malware delivery, phishing, deepfake vishing, credential phishing, CVE patching, brute-force response, supply chain attacks, and proactive threat intelligence.

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **Alert Triage** | Reviewing and prioritizing security alerts from SIEM and employees |
| **Social Engineering** | Manipulating people into performing actions or revealing information |
| **Phishing** | Fraudulent emails designed to steal credentials or deliver malware |
| **Vishing / Deepfake** | Voice-based social engineering — attacker impersonated CEO via deepfake call |
| **Data Stealer** | Malware that silently exfiltrates credentials, files, and browser data |
| **CVE Patching** | Applying security patches to fix known vulnerabilities before exploitation |
| **Supply Chain Attack** | Malicious code injected into a trusted software update |
| **Brute Force** | Automated login attempts to guess valid credentials |

---

## 🔍 Lab 1 — Employees at Risk

---

### ❓ Scenario 1 — Malware Delivery via Fake Software Site

**Alert:** Employee Lucas Martinez reported 7-Zip not launching after download.

**Investigation:** Identified `Setup.exe` downloaded from `best-freeapps-2025.top` — a suspicious freeware hosting site. Antivirus blocked execution 6 times. Quarantine analysis confirmed the file was a **data stealer**.

**Response:** Confirmed quarantine was correct. Informed Lucas the file was malicious — not a security tool conflict. Domain blocked.

> **Verdict:** ✅ Lucas protected — data stealer prevented before execution.

---

### ❓ Scenario 2 — Phishing Email with Malicious Attachment

**Alert (SIEM):** Suspicious email attachment flagged.

```
From:    noreply@stripe-payments.xyz
To:      Mark Phillips, Finance Director
Subject: Stripe Invoice #38291 — $23,650.00
Attach:  Invoice.rar (password: 1111)
```

**Investigation:** `stripe-payments.xyz` is a fake domain impersonating Stripe. Password-protected archive used to bypass email filters. Contents: malicious DOCX document.

**Response:** Blocked email immediately. Confirmed fake domain. Analyzed archive contents — malicious DOCX confirmed.

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/SOC-Analyst-Scenarios/Screenshots/human%20vet%201.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Analyst-Scenarios/Screenshots/human%20vet%201.jpg" width="600"/>
  </a>
</p>
---

### ❓ Scenario 3 — Deepfake Vishing Attack on CEO

**Alert:** IT Support (Isabella) reset CEO Ben's Gmail password after a 9 PM call from a hidden number.

**Investigation:** Login logs showed access from USA (matches Ben's location). Ben unreachable. Red flags: after-hours call, hidden number, urgency, no confirmation.

**Response:** Flagged as social engineering. Escalated. Ben confirmed he never made the call — **deepfake voice attack**.

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/SOC-Analyst-Scenarios/Screenshots/human%20vet%203.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Analyst-Scenarios/Screenshots/human%20vet%203.jpg" width="600"/>
  </a>
</p>

---

### ❓ Scenario 4 — Credential Phishing via Fake Login Page

**Alert (SIEM):** Anomalous login location for HR Assistant Rose Lewis.

```
User:             Rose Lewis, HR Assistant
Login to:         Microsoft 365
Login location:   London, UK  (typical: Oxford, UK)
URLs before login:
  - http://login[.]micrsoft365-online[.]ru
  - https://hroyhiqtspqgkp[.]info
```

**Investigation:** `micrsoft365-online[.]ru` — typosquatted fake Microsoft 365 login page. Rose entered credentials on the fake page before the real login.

**Response:** Analyzed pre-login URL history. Confirmed phishing domain. Initiated password reset and account review.

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/SOC-Analyst-Scenarios/Screenshots/human%20vet%203.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Analyst-Scenarios/Screenshots/human%20vet%203.jpg" width="600"/>
  </a>
</p>

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/SOC-Analyst-Scenarios/Screenshots/human%20vet%204.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Analyst-Scenarios/Screenshots/human%20vet%204.jpg" width="600"/>
  </a>
</p>
---

## 🔍 Lab 2 — Systems at Risk & Remediation

---

### ❓ Scenario 5 — CVE-2024-49040 — Exchange Server

**Alert:** Pentest team confirmed Exchange mail server `HQ-MAIL-02` is vulnerable to **CVE-2024-49040** and Internet-exposed.

**Investigation:** Root cause identified — unpatched vulnerability actively exploitable. Server exposure increases risk of mass exploitation.

**Response:** Patched the vulnerability immediately. Performed threat hunt to identify any compromise that occurred before the patch was applied.

 
---

### ❓ Scenario 6 — WordPress Admin Panel Brute-Force

**Alert:** Threat actors brute-forced the WordPress admin panel and replaced the main page with malware links and gambling ads.

**Investigation:** Root cause — compromised/weak admin credentials allowed brute-force success.

**Response:** Mitigated root cause by addressing breached credentials. Restored defaced pages. Initiated backdoor hunt across the WordPress installation.


---

### ❓ Scenario 7 — Proactive Threat Intel — Cisco Firewall CVE

**Alert (Threat Intel):** Neighboring company hit by ransomware via old Cisco firewall exploitation. Advisory issued to audit all Cisco devices.

**Investigation:** Audited all corporate firewalls. Identified **outdated Cisco firewall in London office** with known CVE.

**Response:** Applied latest patches to the London office firewall before exploitation occurred.


---

### ❓ Scenario 8 — Supply Chain Attack via Trusted App Update

**Alert:** Unusual spike of security events from designer's laptop `LPT-01518`. Trusted 3D design application started running malicious CMD commands after a recent update.

**Investigation:** Trusted application behavior changed post-update — classic **supply chain attack** indicator. Malicious code injected into a legitimate software update.

**Response:** Isolated the affected laptop. Identified the compromised update as the attack vector. Initiated rollback and forensic investigation.
<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/SOC-Analyst-Scenarios/Screenshots/systematac3.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Analyst-Scenarios/Screenshots/systematac3.jpg" width="600"/>
  </a>
</p>
 

---
## 🧠 What I Learned

### Technical Skills
- How to identify fake/typosquatted domains impersonating legitimate services
- How to analyze pre-login URL history to identify credential phishing
- How to recognize supply chain attacks from behavioral changes post-update
- The importance of proactive threat intelligence acting before exploitation occurs

### Analyst Mindset
- Every employee interaction is a potential security event SOC is not just about SIEM alerts
- Urgency + unusual timing + hidden number = immediate red flags for social engineering
- A login from the correct country does not confirm legitimacy context always matters
- Patching is reactive threat hunting after patching is proactive and equally important
- When a trusted app starts behaving maliciously after an update, always consider supply chain first

---

## 💬 Honest Self-Assessment

**What went well:**
Both labs had clear red flags once I knew what to look for, the fake domain, the after-hours call, the pre-login phishing URL, the post-update behavioral change. Connecting each indicator to a real technique made the analysis feel like genuine SOC work.

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/SOC-Analyst-Scenarios/Screenshots/systematacend.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/SOC-Analyst-Scenarios/Screenshots/systematacend.jpg" width="600"/>
  </a>
</p>

---

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
