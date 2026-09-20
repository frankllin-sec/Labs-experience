# 🛡️ Boogeyman 2 - Full Attack Chain Investigation: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Incident%20Response-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-2/Screenshots/ainicio.jpg" width="700"/>
</p>

**Room link:** [tryhackme.com/room/boogeyman2](https://tryhackme.com/room/boogeyman2)

The Boogeyman is back. After a severe attack on Quick Logistics LLC, the security team improved its defences, but the same threat group returned with new and improved tactics, techniques, and procedures (TTPs). This room picks up where Boogeyman 1 left off, moving from spear phishing into full memory forensics with Volatility.

---

## 📌 About This Lab

Maxine, a Human Resource Specialist at Quick Logistics LLC, received what looked like a normal job application. The attached resume was malicious and compromised her workstation. The security team flagged suspicious commands running on her machine, kicking off this investigation. Two artefacts are provided: a copy of the phishing email and a memory dump of the victim's workstation.

**Tools used:** MD5sum, VirusTotal, olevba, Volatility 3 (windows.pstree, windows.cmdline, windows.netscan, windows.filescan), strings

**Objectives:**
- Analyse the phishing email and malicious attachment to trace initial access
- Extract and analyse the malicious macro to identify the stage 2 payload
- Use Volatility against a memory dump to reconstruct the full process chain, from initial execution to C2 communication and persistence

---

## 🔍 Investigation

### Part 1: Phishing Email & Initial Access

**Q: What email was used to send the phishing email?**

**Method:** Reviewed the sender field of the phishing email.

> **Answer:** `westaylor23@outlook.com`

**Q: What is the email of the victim employee?**

**Method:** Reviewed the recipient field of the phishing email.

> **Answer:** `maxine.beck@quicklogisticsorg.onmicrosoft.com`

**Q: What is the name of the attached malicious document?**

**Method:** Identified the file attached to the phishing email.

> **Answer:** `Resume_WesleyTaylor.doc`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-2/Screenshots/a1.jpg" width="700"/>
</p>

**Q: What is the MD5 hash of the malicious attachment?**

**Method:** Used the command `md5sum` against the file to generate its hash.

> **Answer:** `52c4384a0b9e248b95804352ebec6c5b`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-2/Screenshots/a2.jpg" width="700"/>
</p>

**Q: What URL is used to download the stage 2 payload based on the document's macro?**

**Method:** Took the MD5 hash and submitted it to VirusTotal to get more information about the file and its behavior.

> **Answer:** `https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.png`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-2/Screenshots/a3.jpg" width="700"/>
</p>

**Q: What is the name of the process that executed the newly downloaded stage 2 payload?**

**Method:** Used `olevba` (recommended by the room) to analyse and extract the VBA macro embedded in the malicious document.

> **Answer:** `wscript.exe`

---

### Part 2: Memory Forensics with Volatility

**Q: What is the full file path of the malicious stage 2 payload? What URL is used to download the malicious binary executed by the stage 2 payload?**

**Method:** Continued analysing the macro's behavior and follow-up network calls extracted from the memory dump.

> **Answer:** File path: `C:\ProgramData\update.js` — Download URL: `https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-2/Screenshots/a4.jpg" width="700"/>
</p>

**Q: What is the PID of the process that executed the stage 2 payload? What is the parent PID of that process?**

**Method:** Ran `windows.pstree` to reconstruct the process tree from the memory dump, since it shows parent-child relationships and lets me trace which process launched which. This confirmed that `wscript.exe` (stage 2) was spawned directly by `WINWORD.EXE` (stage 1), proving the malicious document triggered the payload execution.
```
vol -f WKSTN-2961.raw windows.pstree
```

> **Answer:** PID: `4260` — Parent PID: `1124`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-2/Screenshots/a5.jpg" width="700"/>
</p>

**Q: What is the PID of the malicious process used to establish the C2 connection?**

**Method:** Continued reviewing the reconstructed process tree to identify the next malicious process spawned after the stage 2 payload.

> **Answer:** `6216`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-2/Screenshots/a6.jpg" width="700"/>
</p>

**Q: What is the full file path of the malicious process used to establish the C2 connection?**

**Method:** Ran `windows.cmdline` against that PID to see exactly how the process was launched. The full command line shows the file path and arguments it executed.
```
vol -f WKSTN-2961.raw windows.cmdline --pid 6216
```

> **Answer:** `C:\Windows\Tasks\updater.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-2/Screenshots/a7.jpg" width="700"/>
</p>

**Q: What is the IP address and port of the C2 connection initiated by the malicious binary?**

**Method:** Ran `windows.netscan` to list all active/historical network connections found in memory, then located the row matching PID 6216 (`updater.exe`). The ForeignAddr and ForeignPort columns for that row gave the C2 server's IP and port.
```
vol -f WKSTN-2961.raw windows.netscan
```

> **Answer:** `128.199.95.189:8080`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-2/Screenshots/a8.jpg" width="700"/>
</p>

**Q: What is the full file path of the malicious email attachment based on the memory dump?**

**Method:** Ran `windows.filescan` to search memory for file object references, then filtered with `grep -i "resume"` to find any file paths matching the malicious attachment's name.
```
vol -f WKSTN-2961.raw windows.filescan | grep -i "resume"
```

> **Answer:** `C:\Users\maxine.beck\AppData\Local\Microsoft\Windows\INetCache\Content.Outlook\WQHGZCFI\Resume_WesleyTaylor (002).doc`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-2/Screenshots/a9.jpg" width="700"/>
</p>

**Q: The attacker implanted a scheduled task right after establishing the C2 callback. What is the full command used by the attacker to maintain persistent access?**

**Method:** The process behind the scheduled task had likely already exited, so it wasn't visible to Volatility's structured plugins. Instead, ran `strings` against the raw memory dump and filtered for `schtasks`, which surfaced the full command used to create the persistence mechanism.
```
strings WKSTN-2961.raw | grep -i "schtasks"
```

> **Answer:**
```
/Create /F /SC DAILY /ST 09:00 /TN Updater /TR 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -NonI -W hidden -c \"IEX ([Text.Encoding]::UNICODE.GetString([Convert]::FromBase64String((gp HKCU:\Software\Microsoft\Windows\CurrentVersion debug).debug)))\"';
```

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-2/Screenshots/a10.jpg" width="700"/>
</p>

---

## 🧠 What I Learned

- How to trace a full infection chain purely from a memory dump, from the initial malicious document through to persistence, using Volatility 3
- That `windows.pstree` is the fastest way to confirm parent-child process relationships and prove which process actually launched which payload
- That a process that has already exited can still leave its command line recoverable in raw memory via `strings`, even when it no longer shows up in structured Volatility plugins
- Reinforced how attackers chase persistence immediately after establishing C2, in this case with a daily scheduled task running an obfuscated, base64-encoded PowerShell command

---

## 💬 Honest Self-Assessment

**What I need to improve:**
This lab pushed my Volatility skills further than Boogeyman 1 did, since almost the entire investigation happened inside a single memory dump instead of across multiple artefact types. Chaining plugins together (pstree to find a PID, then cmdline to confirm how it was launched, then netscan to find its network activity) took a few attempts to get right. I want to get faster at knowing which Volatility plugin answers which kind of question without having to check the documentation each time.

---

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
<p align="center"><a href="https://github.com/frankllin-sec">🔗 Visit my GitHub Profile</a></p>
