# 🛡️ Shadow Trace: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Malware%20Analysis-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Shadow-Trace/Screenshots/sinicio.jpg" width="700"/>
</p>

---

## 📌 About This Lab

It's the middle of the night shift. I'm the only analyst in the SOC when a manager calls in urgently: a suspicious file was found on a user's machine and needs immediate review.

I open the file and start digging. Something doesn't look normal for a company updater, and at the same time the EDR throws a couple of alerts. The task: analyse the file, collect anything to identify it, gather any potential IOCs, correlate and analyse the alerts for potential malicious behaviour, and piece together what's happening before it spreads further.

**Objectives:**
- Extract IOCs from suspicious binaries
- Correlate alerts with malicious activity
- Perform basic SOC triage actions

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **Static Analysis** | Examining a binary without executing it, using tools like pestudio to pull metadata, headers, and embedded strings |
| **SHA-256 Hashing** | A unique fingerprint of a file, used to identify it and check it against threat intelligence sources |
| **Strings Analysis** | Scanning a binary for readable text (URLs, domains, error messages) that can reveal its intended behaviour |
| **Imported Libraries (DLLs)** | The Windows libraries a binary loads. `WS2_32.dll`, for example, indicates the binary performs network socket communication |
| **IOC (Indicator of Compromise)** | A URL, domain, hash, or file name that can be used to detect or block related malicious activity |
| **Base64 Encoding/Decoding** | A common technique attackers use to obfuscate URLs, commands, or payloads inside strings or scripts |
| **EDR Alert Correlation** | Connecting multiple EDR alerts (e.g. a suspicious PowerShell execution and a suspicious browser download) to reconstruct what a compromised host actually did |

---

## 🔍 Investigation

### Part 1: Static Binary Analysis (pestudio)

**Scenario:** Analyse the binary located at `C:\Users\DFIRUser\Desktop\windows-update.exe` on the attached machine.

**Q: What is the architecture of the binary file windows-update.exe?**

**Method:** Used pestudio to open and inspect the file. The file-header view shows the architecture directly.

> **Answer:** `64-bit`

**Q: What is the hash (SHA-256) of the file windows-update.exe?**

**Method:** Same pestudio file view, under `file > sha256`.

> **Answer:** `b2a88de3e3bcfae4a4b38fa36e884c586b5cb2c2c283e71fba59efdb9ea64bfc`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Shadow-Trace/Screenshots/s1.jpg" width="700"/>
</p>

**Q: Identify the URL within the file to use it as an IOC.**

**Method:** Opened the `indicators (imports > flag)` section in pestudio and found a suspicious URL pattern flagged automatically.

> **Answer:** `http://tryhatme.com/update/security-update.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Shadow-Trace/Screenshots/s2.jpg" width="700"/>
</p>

**Q: With the URL identified, can you spot a domain that can be used as an IOC?**

**Method:** Switched to the `strings` tab and looked for anything suspicious. The first candidate tried was `external-attacker.thm`, which turned out to be incorrect. Continuing to scan the strings led to the actual answer.

> **Answer:** `responses.tryhatme.com`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Shadow-Trace/Screenshots/s3.jpg" width="700"/>
</p>

**Q: Input the decoded flag from the suspicious domain.**

**Method:** While scanning the strings, spotted `tryhatme.com/VEhNe3lvdV9nMHRfc29tZV9JT0NzX2ZyaWVuZH0=`, an unusually long string for a URL path. That length was the signal it was Base64-encoded, so it was decoded through CyberChef.

> **Answer:** `THM{you_g0t_some_IOCs_friend}`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Shadow-Trace/Screenshots/s4.jpg" width="700"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Shadow-Trace/Screenshots/s5.jpg" width="700"/>
</p>

**Q: What library related to socket communication is loaded by the binary?**

**Method:** Opened the `libraries` tab in pestudio and matched the description text against socket-related functionality.

> **Answer:** `WS2_32.dll`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Shadow-Trace/Screenshots/s6.jpg" width="700"/>
</p>

---

### Part 2: Alerts Analysis (EDR Correlation)

**Scenario:** Two EDR alerts fired on the same host, `WIN-SRV-01.tryhackme.local`, tied to the `CORPsvc_backup` process context: a suspicious PowerShell execution and a suspicious browser download triggered by chrome.exe.

**Q: Can you identify the malicious URL from the trigger by the process powershell.exe?**

**Method:** The PowerShell command was a Base64-encoded `DownloadString` call. Decoded the Base64 blob through CyberChef to reveal the URL.

> **Answer:** `https://tryhatme.com/dev/main.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Shadow-Trace/Screenshots/s7.jpg" width="700"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Shadow-Trace/Screenshots/s8.jpg" width="700"/>
</p>

**Q: Can you identify the malicious URL from the alert triggered by chrome.exe?**

**Method:** This command was JavaScript, not PowerShell, using an array of character codes instead of a plain string. Getting an AI assistant to help translate the character-code array into readable text made it possible to identify the URL and confirm the file name it saved.

> **Answer:** `https://reallysecureupdate.tryhatme.com/update.exe`

**Q: What's the name of the file saved in the alert triggered by chrome.exe?**

> **Answer:** `test.txt`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Shadow-Trace/Screenshots/s9.jpg" width="700"/>
</p>

---

## 🧠 What I Learned

- How to use pestudio for static binary analysis: file metadata, hashes, indicators, strings, and imported libraries
- How to recognize an unusually long string as a signal it might be Base64-encoded, and how to confirm it with CyberChef
- How an imported DLL like `WS2_32.dll` points to network activity the binary is capable of
- How to correlate two separate EDR alerts on the same host to a single incident, instead of treating them as unrelated
- How to work through obfuscated JavaScript (character-code arrays) instead of just plain Base64, and when to bring in outside help to speed that up

---

## 💬 Honest Self-Assessment

**What went well:**
The pestudio workflow felt intuitive: file metadata, hashes, indicators, and strings all pointed to the IOCs pretty directly once I knew where to look. Spotting the Base64-encoded string by its unusual length, then confirming it in CyberChef, was a small win that clicked fast.

**What I need to improve:**
My first guess for the suspicious domain (`external-attacker.thm`) was wrong, so I need to slow down and look for other candidates before committing to an answer. The chrome.exe alert was also a real gap: it wasn't Base64, it was a JavaScript character-code array, and I didn't recognize that pattern on my own. I had to get AI help to decode it. I want to get comfortable enough with obfuscation techniques like this that I can spot and convert them myself next time.

---
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Shadow-Trace/Screenshots/sfinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
