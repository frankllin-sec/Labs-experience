# 🛡️ Invite Only: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Threat%20Intelligence-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Invite-Only/Screenshots/iinicio.jpg" width="700"/>
</p>

---

## 📌 About This Lab

I'm a SOC analyst at Managed Server Provider TrySecureMe, supporting an L3 analyst investigating flagged IPs, hashes, URLs, and domains as part of incident response activities. An L1 analyst flagged two suspicious findings early in the morning and escalated them:

- **Flagged IP:** `101[.]99[.]76[.]120`
- **Flagged SHA256 hash:** `5d0509f68a9b7c415a726be75a078180e3f02e59866f193b0a99eee8e39c874f`

The task is to analyse both findings using the company's new threat intelligence search application, TryDetectThis2.0, and distil the results into usable threat intelligence, tracing the malware's lineage and connecting it back to a real, publicly documented campaign.

**Objectives:**
- Trace a flagged file hash through its execution parents and dropped files
- Pivot from a flagged IP to public community threat intelligence
- Correlate internal findings to an external, published threat research report
- Extract adversary TTPs (phishing technique, tooling, delivery platform) from that report

---

## 🔑 Key Concepts

**Execution parents and dropped files:** malware rarely arrives alone. An "execution parent" is the file or process that spawned the flagged sample, tracing these chronologically reconstructs how the infection actually got onto the machine. "Dropped files" are what the malware itself deploys once running, following that chain from parent to sample to dropped files rebuilds the full infection lineage instead of looking at one isolated file.

**Community comments as CTI:** platforms like VirusTotal let researchers attach comments to an indicator, often containing IOC context, MISP galaxy tags, and links to the original threat research. These comments can shortcut hours of analysis by pointing straight to a named malware family and a public report.

**Correlating to public threat research:** once an internal finding (a hash, an IP) is enriched enough to identify the malware family, the next step is finding whether a security vendor has already published research on the exact campaign. This turns a stack of isolated IOCs into a full narrative: who's behind it, how they deliver the payload, and what they're after.

---

## 🔍 Investigation

### Tracing the Flagged Hash

**Q: What is the name of the file identified with the flagged SHA256 hash? What is the file type?**

**Method:** Searched the flagged hash in TryDetectThis2.0.

> **Answer:** `syshelpers.exe`, `Win32 EXE`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Invite-Only/Screenshots/i1.jpg" width="700"/>
</p>

**Q: What are the execution parents of the flagged hash? What file does it drop?**

**Method:** Checked the Relations tab for the flagged hash's execution parents and dropped files sections.

> **Answer:** Execution parents (chronological): `361GJX7J, installer.exe`. Dropped file: `Aclient.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Invite-Only/Screenshots/i2.jpg" width="700"/>
</p>

**Q: Research the second hash from the execution parents and list the four malicious dropped files, in the order they appear.**

**Method:** Looked up the `installer.exe` hash and reviewed its own dropped files list.

> **Answer:** `searchhost.exe, syshelpers.exe, nat.vbs, runsys.vbs`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Invite-Only/Screenshots/i3.jpg" width="700"/>
</p>

---

### Pivoting from the Flagged IP

**Q: Analyse the files related to the flagged IP. What is the malware family that links these files?**

**Method:** Looked up `101[.]99[.]76[.]120` in TryDetectThis2.0 and read through the community comments, which named the malware family directly and linked to the original vendor report.

> **Answer:** `AsyncRAT` (Remote Access Trojan)

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Invite-Only/Screenshots/i4.jpg" width="700"/>
</p>

**Q: What is the title of the original report where these flagged indicators are mentioned?**

**Method:** Googled the IP address to find the publicly published research behind it.

> **Answer:** `From Trust to Threat: Hijacked Discord Invites Used for Multi-Stage Malware Delivery`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Invite-Only/Screenshots/i5.jpg" width="700"/>
</p>

**Q: Which tool did the attackers use to steal cookies from the Google Chrome browser? Which phishing technique did they use?**

**Method:** Read through the Check Point Research report's Key Takeaways. My first guess for the cookie-theft tool was "malware delivery", which was wrong, I'd misread the takeaway and needed to slow down and reread the report more carefully to find the actual named tool.

> **Answer:** `ChromeKatz` (cookie theft), `ClickFix` (phishing technique)

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Invite-Only/Screenshots/i6.jpg" width="700"/>
</p>

**Q: What is the name of the platform used to redirect a user to malicious servers?**

**Method:** Per the report, the attackers hijacked expired and released invite links through vanity link registration.

> **Answer:** `Discord`

---

## 🧠 What I Learned

- How to trace a malware sample's lineage through execution parents and dropped files instead of analysing one file in isolation
- Why community comments on a threat intel platform are worth reading closely, they often name the exact malware family and link straight to the original research
- How to pivot from an internal indicator (a flagged IP or hash) to a real, publicly published threat report using nothing more than a targeted Google search
- The mechanics of the AsyncRAT/ClickFix Discord invite hijacking campaign: attackers reclaim expired invite links, then use the ClickFix phishing technique and tools like ChromeKatz to steal browser cookies and deliver multi-stage payloads

---

## 💬 Honest Self-Assessment

**What went well:**
Tracing the execution parent chain and dropped files across two related hashes felt methodical and clear once I understood the relationship, follow the parent, note the hash, check that hash's own drops. Pivoting from the flagged IP's community comments straight to the original Check Point report was a satisfying shortcut.

**What I need to improve:**
On the cookie-theft tool question, I answered "malware delivery" first, which was wrong. I'd skimmed the report's Key Takeaways instead of reading them carefully, and picked up the wrong phrase. I need to slow down on report-based questions specifically and make sure I'm answering what was actually asked, not just pattern-matching on nearby keywords.

---
<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
