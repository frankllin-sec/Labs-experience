# 🎣 Phishing Analysis Tools — TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Phishing%20Analysis-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

---

## 📌 About This Lab

This lab simulates the role of a **Level 1 SOC Analyst** on the defensive frontline, investigating suspicious emails escalated by coworkers. The objective was to analyze phishing emails, extract key indicators of compromise, and identify malicious activity using real-world tools including **Thunderbird**, **ANYRUN sandbox**, and static analysis techniques.

The findings from this investigation contribute directly to building detection rules to prevent similar threats from reaching other users in the future.

> ⚠️ **Note:** The domains, URLs, and IP addresses referenced in this lab have hosted real malicious content. Do not access or interact with them outside of a controlled lab environment.

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Phishing-Analysis-Tools/Screenshots/pstarting.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Phishing-Analysis-Tools/Screenshots/pstarting.jpg" width="600"/>
  </a>
</p>

---

## 🔍 Investigation — Part 1: Phishing Email Analysis (Phish3Case1.eml)

### Scenario

A suspicious email impersonating Netflix was escalated by a coworker. The task was to open the `.eml` file in Thunderbird, review the email headers and body, and extract key indicators.

---

### ❓ Q1 — What reputable brand is this email tailored to impersonate?

**Method:** Reviewed the email subject line and body content in Thunderbird. The email used Netflix branding and claimed the user's account was on hold.

> **Answer:** `Netflix`

---

### ❓ Q2 — Based on the email headers, who is the intended recipient?

**Method:** Checked the **To:** field in the email header in Thunderbird.

> **Answer:** `redacted@yahoo.com`

---

### ❓ Q3 — What is the Received: from IP address?

**Method:** In Thunderbird, used **View → Message Source** to open the raw email headers. Located the `Received: from` field at the top of the header chain.

> **Answer:** `10.197.37.234`

---

### ❓ Q4 — What domain of interest appears in the Return-Path field?

**Method:** In the message source, located the `Return-Path` field. The domain in the return address was suspicious and not associated with Netflix.

> **Answer:** `etekno.xyz`

---

### ❓ Q5 — What is the shortened URL behind the UPDATE ACCOUNT NOW button?

**Method:** Hovered over the **UPDATE ACCOUNT NOW** button in Thunderbird to reveal the underlying URL. The link used a URL shortener to redirect to a phishing page.

> **Answer:** `https://t.co/yuxfZm8KPg?amp=1`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Phishing-Analysis-Tools/Screenshots/p1.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Phishing-Analysis-Tools/Screenshots/p1.jpg" width="600"/>
  </a>
</p>

---

## 🔍 Investigation — Part 2: PDF Attachment Analysis (ANYRUN Sandbox)

### Scenario

The phishing email contained a PDF attachment masquerading as a Netflix payment update. The file was submitted to ANYRUN sandbox for behavioral analysis. The task was to review process activity, network connections, and IOCs.

---

### ❓ Q6 — How does ANYRUN classify this suspected phishing email?

**Method:** Reviewed the ANYRUN sandbox verdict banner at the top of the analysis results.

> **Answer:** `Suspicious activity`

---

### ❓ Q7 — What is the name of the PDF attachment?

**Method:** Identified the filename from the ANYRUN analysis dashboard and process details.

> **Answer:** `Payment-updateid.pdf`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Phishing-Analysis-Tools/Screenshots/p2.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Phishing-Analysis-Tools/Screenshots/p2.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q8 — What is the SHA256 hash of the PDF file?

**Method:** Opened the **Static Discovering** panel in ANYRUN and located the Hashes section for `Payment-updateid.pdf`. Identified the SHA256 value.

> **Answer:** `CC6F1A64B10BCB168AEEC8D870B97BD7C20FC161E8310B5BCE1AF8ED420E2C24`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Phishing-Analysis-Tools/Screenshots/p3.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Phishing-Analysis-Tools/Screenshots/p3.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q9 — Which IP address associated with AcroRd32.exe is flagged as malicious?

**Method:** Reviewed the ANYRUN text report network connections section. Filtered by process `AcroRd32.exe` and identified the IP address flagged with a **malicious** reputation tag.

> **Answer:** `2.16.107.24`

---

### ❓ Q10 — Which Windows process is classed as Potentially Bad Traffic?

**Method:** Reviewed the **Threats** section in the ANYRUN network report. Identified the process flagged with the **Potentially Bad Traffic** classification.

> **Answer:** `svchost.exe`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Phishing-Analysis-Tools/Screenshots/p4.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Phishing-Analysis-Tools/Screenshots/p4.jpg" width="600"/>
  </a>
</p>

---

## 🔍 Investigation — Part 3: Excel Attachment Analysis (ANYRUN Sandbox)

### Scenario

A second phishing attachment — an Excel spreadsheet — was submitted to a different ANYRUN sandbox. The file exploited a known Microsoft Office vulnerability to execute malicious code. The task was to identify the file details, hashes, network IOCs, and the vulnerability exploited.

---

### ❓ Q11 — How does ANYRUN classify the .xlsx attachment?

**Method:** Reviewed the ANYRUN verdict for the Excel file analysis.

> **Answer:** `Malicious activity`

---

### ❓ Q12 — What is the file name of the Excel attachment?

**Method:** Identified the filename from the ANYRUN General Info section.

> **Answer:** `CBJ200620039539.xlsx`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Phishing-Analysis-Tools/Screenshots/p5.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Phishing-Analysis-Tools/Screenshots/p5.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q13 — What is the SHA256 hash of the Excel attachment?

**Method:** Located the SHA256 hash value in the ANYRUN General Info section under file hashes.

> **Answer:** `5F94A66E0CE78D17AFC2DD27FC17B44B3FFC13AC5F42D3AD6A5DCFB36715F3EB`

---

### ❓ Q14 — What IP address is associated with the malicious domain biz9holdings.com?

**Method:** Reviewed the DNS Requests section in the ANYRUN text report. Located the entry for `biz9holdings.com` and noted the resolved IP address flagged as malicious.

> **Answer:** `204.11.56.48`

---

### ❓ Q15 — Which other domain is classified as malicious?

**Method:** Continued reviewing the DNS Requests section. Identified the second domain with a **malicious** reputation tag.

> **Answer:** `findresults.site`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Phishing-Analysis-Tools/Screenshots/p6.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Phishing-Analysis-Tools/Screenshots/p6.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q16 — What vulnerability does this malicious attachment attempt to exploit?

**Method:** Reviewed the ANYRUN General Info tags and Behavior Activities section. The **MALICIOUS** activity section identified the Equation Editor exploit and the associated CVE tag was visible in the analysis tags.

> **Answer:** `CVE-2017-11882`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Phishing-Analysis-Tools/Screenshots/p7.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Phishing-Analysis-Tools/Screenshots/p7.jpg" width="600"/>
  </a>
</p>

---

## 🧠 What I Learned

### Technical Skills
- How to analyze phishing emails using **Thunderbird** — reading raw headers via View → Message Source
- How to identify spoofed sender domains using **Return-Path** and **SPF** failures in email headers
- How to use **ANYRUN** sandbox to analyze malicious PDF and Excel attachments without risk
- How to extract IOCs from sandbox reports — process chains, malicious IPs, suspicious domains, and file hashes
- What **CVE-2017-11882** is — a Microsoft Equation Editor RCE vulnerability that allows arbitrary code execution when a malicious Office file is opened
- How phishing attachments chain processes: `EXCEL.EXE → EQNEDT32.EXE → malicious payload download`

### Analyst Mindset
- The **Return-Path** field is one of the most reliable indicators of a spoofed sender legitimate organizations use matching domains
- **URL shorteners** in phishing emails are always a red flag they hide the real destination and bypass basic URL reputation checks
- A PDF or XLSX opening and immediately spawning unexpected child processes (like `EQNEDT32.EXE` or `AcroRd32.exe` making outbound connections) is a critical indicator of exploitation
- Always check the SHA256 hash against threat intel — it's the most reliable way to identify a known malicious file regardless of filename changes

---

## 💬 Honest Self-Assessment

**What went well:**
The email header analysis was intuitive — connecting the Return-Path domain to the spoofed sender and the Received: from IP to the originating mail server made the investigation feel like real SOC work. The ANYRUN sandbox interface was clear and the process chain visualization made it easy to understand how each attachment executed.

---

## 📁 Repository Structure

```
Phishing-Analysis-Tools/
├── README.md
└── Screenshots/
```

---

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Phishing-Analysis-Tools/Screenshots/pFinish.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Phishing-Analysis-Tools/Screenshots/pFinish.jpg" width="600"/>
  </a>
</p>

---

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
<p align="center"><a href="https://github.com/frankllin-sec">🔗 Visit my GitHub Profile</a></p>
