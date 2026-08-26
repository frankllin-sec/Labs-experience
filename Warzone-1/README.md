# 🛡️ Warzone 1: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Network%20Traffic%20Analysis-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Warzone-1/Screenshots/zinicio.jpg" width="700"/>
</p>

---

## 📌 About This Lab

I'm a Tier 1 Security Analyst at a Managed Security Service Provider (MSSP), monitoring network alerts. A few minutes into the shift, the first case comes in: **Potentially Bad Traffic** and **Malware Command and Control Activity Detected**. The clock starts, the task is to inspect the PCAP and retrieve the artefacts needed to confirm whether this alert is a true positive.

**Tools used:** Brim, NetworkMiner, Wireshark

**Objectives:**
- Identify the alert signature and the source/destination IPs involved
- Pivot from an IP and domain to VirusTotal for threat attribution
- Inspect web traffic for user-agent and additional attacker infrastructure
- Retrace the full attack, from initial HTTP request to dropped files on disk

---

## 🔍 Investigation

**Q: What was the alert signature for Malware Command and Control Activity Detected? What is the source IP address? What IP address was the destination IP in the alert?**

**Method:** Opened `zone1.pcap` in Brim. On the left panel, ran the "Suricata Alerts by Category" query, right-clicked the "Malware Command and Control Activity Detected" alert, and pulled the signature plus the source and destination IPs directly from the result grid.

> **Answer:** `ET Malware MirrorBlast CnC Activity M3`, source `172[.]16[.]1[.]102`, destination `169[.]239[.]128[.]11`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Warzone-1/Screenshots/z1.jpg" width="700"/>
</p>

**Q: Still in VirusTotal, under Community, what threat group is attributed to this IP address? What is the malware family?**

**Method:** Searched the destination IP in VirusTotal and checked the Community tab for analyst-tagged graphs and campaign names.

> **Answer:** `TA505` (threat group), `MirrorBlast` (malware family)

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Warzone-1/Screenshots/z2.jpg" width="700"/>
</p>

**Q: Search VirusTotal for the domain. What was the majority file type listed under Communicating Files?**

**Note:** This question originally asked for a domain from an earlier question, but that question's answer wasn't actually a domain, it looks like TryHackMe updated this room without updating the question. I searched Google, worked out the domain name myself from the PCAP, and searched it directly in VirusTotal.

> **Answer:** `Windows Installer`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Warzone-1/Screenshots/z3.jpg" width="700"/>
</p>

**Q: Inspect the web traffic for the flagged IP address. What is the user-agent in the traffic?**

**Method:** Filtered Brim for the flagged IP and reviewed the HTTP requests.

> **Answer:** `REBOL View 2.7.8.3.1`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Warzone-1/Screenshots/z5.jpg" width="700"/>
</p>

**Q: Retrace the attack, there were multiple IP addresses associated with it. What were two other IP addresses?**

**Method:** Filtered for HTTP requests to see every host the infected machine reached out to beyond the primary C2 IP.

> **Answer:** `185[.]10[.]68[.]235, 192[.]36[.]27[.]92`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Warzone-1/Screenshots/z6.jpg" width="700"/>
</p>

**Q: What were the file names of the downloaded files, in order of the IP addresses from the previous question?**

**Method:** Filtered on each of the two IP addresses in turn to see what file each one served.

> **Answer:** `filter.msi, 10opd3r_load.msi`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Warzone-1/Screenshots/z7.jpg" width="700"/>
</p>

**Q: Inspect the traffic for the first downloaded file. Two files are saved to the same directory. What is the full file path and the names of the two files?**

**Method:** Filtered Brim for the `filter.msi` extension, opened the matching packet directly in Wireshark, then followed the TCP stream and searched the stream for the file names to recover the full install path.

> **Answer:** `C:\ProgramData\001\arab.bin, C:\ProgramData\001\arab.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Warzone-1/Screenshots/z8.jpg" width="700"/>
</p>

**Q: Now do the same for the second downloaded file. What is the full file path and the names of the two files?**

**Method:** Filtered Wireshark for frames containing "10opd3r", followed that TCP stream, and searched for `c:\` inside it to find the drop path.

**Note:** This was the hardest question in the room for me. My first two attempts (both visible in the screenshots below) came back incorrect. The stream clearly contains install paths under `C:\ProgramData\Local\Google\`, but I wasn't fully confident I had the exact two file names TryHackMe expected by the time I finished the room, this is the one answer in this lab I'd want to revisit and double-check against a fresh read of the stream.

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Warzone-1/Screenshots/z9.jpg" width="700"/>
</p>

---


## 💬 Honest Self-Assessment

**What I need to improve:**
The second dropped-file question took two incorrect attempts and I'm still not fully confident in the final path I landed on. I need more practice reading raw TCP stream output under time pressure, picking the right file path . 

---
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Warzone-1/Screenshots/zfinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
