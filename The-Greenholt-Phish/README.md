# 🎣 The Greenholt Phish — TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Phishing%20Analysis-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

---

## 📌 About This Lab

A sales executive at **Greenholt PLC** reported a suspicious email received from a known customer. The message raised several red flags: a generic greeting, an unexpected request for a money transfer, and an unsolicited attachment. The behavior did not align with the customer's usual communication style. The email was escalated to the SOC for further investigation.

The objective was to analyze the email sample, extract key artifacts, investigate its origin and authenticity, and assess the potential maliciousness of the attachment using analysis tools.

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/The-Greenholt-Phish/Screenshots/lpinicial.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/The-Greenholt-Phish/Screenshots/lpinicial.jpg" width="600"/>
  </a>
</p>

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **Email Header Analysis** | Examining Received, Return-Path, Reply-To fields to identify spoofed senders |
| **SPF Record** | DNS record that specifies which mail servers are authorized to send email for a domain |
| **DMARC Record** | Policy that tells receiving servers how to handle emails that fail SPF/DKIM checks |
| **SHA256 Hash** | Cryptographic fingerprint used to uniquely identify a file for threat intel lookup |
| **File Type Spoofing** | Disguising a malicious file with a fake extension (e.g., .CAB that is actually a RAR) |
| **VirusTotal** | Online service that scans files and URLs against 60+ antivirus engines |
| **IPInfo** | Tool used to geolocate and identify the owner of an IP address |
| **MXToolbox** | Online tool for SPF, DMARC, MX, and DNS record lookups |

---

## 🔍 Investigation — Email Header Analysis

### Scenario

A `.eml` file (`challenge.eml`) was opened in Thunderbird and the Message Source was reviewed to extract key header artifacts and identify signs of phishing.

---

### ❓ Q1 — What is the Transfer Reference Number listed in the Subject line?

**Method:** Reviewed the email subject line in Thunderbird: `webmaster@redacted.org your: Transfer Reference Number:(09674321)`

> **Answer:** `09674321`

---

### ❓ Q2 — What is the display name of the sender?

**Method:** Checked the **From:** field in Thunderbird — the display name shown to the recipient.

> **Answer:** `Mr. James Jackson`

---

### ❓ Q3 — What is the sender's email address?

**Method:** Inspected the **From:** field in the message source to get the actual sending address behind the display name.

> **Answer:** `info@mutawamarine.com`

---

### ❓ Q4 — What email address will receive a reply to this email?

**Method:** Located the **Reply-To:** field in the message source. This is a classic phishing technique — the Reply-To address differs from the From address to redirect responses to an attacker-controlled inbox.

> **Answer:** `info.mutawamarine@mail.com`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/The-Greenholt-Phish/Screenshots/lp1.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/The-Greenholt-Phish/Screenshots/lp1.jpg" width="600"/>
  </a>
</p>

---

## 🔍 Investigation — IP & DNS Analysis

### ❓ Q5 — What is the originating IP address of this email?

**Method:** In Thunderbird, used **View → Message Source** and located the `Received: from` header at the top of the chain — the first hop showing the true originating IP before it entered the legitimate mail infrastructure.

> **Answer:** `192.119.71.157`

---

### ❓ Q6 — Who is the owner of the originating IP?

**Method:** Submitted the IP `192.119.71.157` to **https://ipinfo.io** to perform a geolocation and ownership lookup. The ASN field identified the hosting provider.

> **Answer:** `HostPapa`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/The-Greenholt-Phish/Screenshots/lp2.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/The-Greenholt-Phish/Screenshots/lp2.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q7 — What is the full SPF record for the Return-Path domain?

**Method:** Identified the **Return-Path** domain (`mutawamarine.com`) from the message source — SPF validates the Return-Path domain, not the From domain. Ran an **SPF Record Lookup** on MXToolbox for `mutawamarine.com`.

> **Answer:** `v=spf1 include:spf.protection.outlook.com -all`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/The-Greenholt-Phish/Screenshots/lp3.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/The-Greenholt-Phish/Screenshots/lp3.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q8 — What is the complete DMARC record for the Return-Path domain?

**Method:** Using the same domain (`mutawamarine.com`), switched the MXToolbox lookup type to **DMARC Lookup**. This queries the `_dmarc.mutawamarine.com` subdomain and returns the DMARC policy.

> **Answer:** `v=DMARC1; p=quarantine; fo=1`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/The-Greenholt-Phish/Screenshots/lp4.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/The-Greenholt-Phish/Screenshots/lp4.jpg" width="600"/>
  </a>
</p>

---

## 🔍 Investigation — Attachment Analysis

### ❓ Q9 — What is the file name of the attachment?

**Method:** Reviewed the bottom of the email in Thunderbird. The attachment was visible with its full filename.

> **Answer:** `SWT_#09674321____PDF__.CAB`

---

### ❓ Q10 — What is the SHA256 hash of the attachment?

**Method:** Downloaded the attachment to the Desktop in the virtual environment. Opened the Ubuntu terminal and ran `sha256sum` to generate the cryptographic hash.

```bash
cd ~/Desktop
sha256sum "SWT_#09674321____PDF__.CAB"
```

> **Answer:** `2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/The-Greenholt-Phish/Screenshots/lp5.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/The-Greenholt-Phish/Screenshots/lp5.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q11 — What is the attachment's file size in KB?

**Method:** Submitted the SHA256 hash to **VirusTotal**. The file size was displayed in the Details section of the analysis report.

> **Answer:** `400.26 KB`

---

### ❓ Q12 — What is the actual file type of the attachment?

**Method:** Continued reviewing the VirusTotal analysis. Despite the `.CAB` extension, the actual file type reported by VirusTotal and confirmed by 50/63 security vendors was different — a classic file type spoofing technique used to bypass email security filters.

> **Answer:** `RAR`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/The-Greenholt-Phish/Screenshots/lp6.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/The-Greenholt-Phish/Screenshots/lp6.jpg" width="600"/>
  </a>
</p>

---

## 🗺️ MITRE ATT&CK Mapping

| Technique | Tactic | Detail |
|---|---|---|
| T1566.001 — Spearphishing Attachment | Initial Access (TA0001) | Malicious RAR attachment disguised as .CAB delivered via phishing email |
| T1036.008 — Masquerading: File Extension | Defense Evasion (TA0005) | `.CAB` extension used to disguise a RAR archive containing malware |
| T1598 — Phishing for Information | Reconnaissance (TA0043) | Generic greeting and fake payment notification used as social engineering lure |
| T1071 — Application Layer Protocol | Command and Control (TA0011) | Trojan.MSIL/Loki family identified — known for C2 via HTTP |

---

## 🧠 What I Learned

### Technical Skills
- How to extract key artifacts from email headers using **Thunderbird Message Source** — From, Reply-To, Return-Path, and Received fields
- The difference between **From** (display), **Return-Path** (SPF validation), and **Reply-To** (where replies go) — three fields that can each be different in a spoofed email
- How to perform **SPF** and **DMARC** lookups using MXToolbox to validate domain authentication records
- How to use **IPInfo** to geolocate an originating IP and identify the hosting provider
- How to use `sha256sum` in Linux terminal to generate a file hash for threat intel lookup
- How **file type spoofing** works — the `.CAB` extension was a fake; the actual file was a **RAR** archive containing ransomware/Trojan (Loki family, 50/63 detections on VirusTotal)

### Analyst Mindset
- A **Reply-To** address that differs from the **From** address is one of the clearest indicators of phishing — it redirects responses to attacker-controlled infrastructure
- The **Return-Path** domain is what SPF actually validates — not the visible From address — which is why spoofing the display name is so effective against users
- File extensions cannot be trusted — always verify the actual file type using hash analysis and VirusTotal
- 50/63 VirusTotal detections with **Trojan.MSIL/Loki** classification means this is a confirmed, well-known threat — not a false positive

---

## 💬 Honest Self-Assessment

**What went well:**
The email header investigation flowed naturally — identifying the Reply-To mismatch, tracing the originating IP, and correlating the Return-Path domain to SPF/DMARC records felt like a complete investigation workflow. Running `sha256sum` on the attachment and confirming the file type via VirusTotal brought the analysis to a satisfying conclusion.

**What I need to improve:**
Speed at recognizing file type spoofing without needing VirusTotal confirmation. A trained analyst should be able to spot a `.CAB` file that's actually a RAR by checking magic bytes with a tool like `file` in the terminal — building that reflex will come with more malware analysis practice.

---

## 📁 Repository Structure

```
The-Greenholt-Phish/
├── README.md
└── Screenshots/
```

---

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/The-Greenholt-Phish/Screenshots/lpfinal.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/The-Greenholt-Phish/Screenshots/lpfinal.jpg" width="600"/>
  </a>
</p>

---

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
<p align="center"><a href="https://github.com/frankllin-sec">🔗 Visit my GitHub Profile</a></p>
