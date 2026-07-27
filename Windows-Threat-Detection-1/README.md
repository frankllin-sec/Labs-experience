# 🛡️ Windows Threat Detection 1: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Windows%20Threat%20Detection-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-1/Screenshots/tinicio.jpg" width="700"/>
</p>

---

## 📌 About This Lab

This room builds on Windows event logging fundamentals and walks through the most common **Initial Access** methods threat actors use against Windows machines, then teaches how to detect each one using Windows Security logs and Sysmon.

Two broad attack surfaces were covered:
- **Exposed services** (RDP, mail servers, public web apps) that get scanned and brute-forced the moment they're internet-facing
- **User-driven attacks** (phishing attachments, LNK files, infected USB drives) that rely on a person clicking something they shouldn't

**Objectives:**
- Explore how threat actors access and breach Windows machines
- Learn common Initial Access techniques via real-world examples
- Practice detecting every technique using Windows event logs and Sysmon

---

## 🎯 MITRE ATT&CK Techniques Covered

| Technique ID | Name | Description |
|---|---|---|
| **T1190** | Exploit Public-Facing Application | Threat actors target misconfigured or vulnerable websites/apps (e.g. a vulnerable mail server) |
| **T1133** | External Remote Services | Threat actors look for exposed RDP/VNC/SSH with weak passwords for remote access |
| **T1566** | Phishing | Threat actors trick users into launching malware themselves through email attachments |
| **T1091** | Replication Through Removable Media | Threat actors infect a USB device and rely on it being reused across machines |

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **RDP Brute Force** | Repeated failed logon attempts (Event ID 4625) against exposed RDP, usually targeting default accounts like Administrator |
| **Logon Type 10** | RemoteInteractive logon, the signature of a successful RDP session |
| **Double Extension** | A malicious file disguised with two extensions (e.g. `best-cat.jpg.exe`) so the fake one is what the user notices |
| **LNK Attachment** | A shortcut file whose Target field runs a hidden command (commonly PowerShell) instead of opening a real document or site |
| **Sysmon Event IDs** | Event ID 1 (process create), 11 (file create), 22 (DNS query), each used to reconstruct an attack chain |

---

## 🔍 Investigation

### Part 1: RDP Breach Detection

**Scenario:** An IT admin exposed RDP on a production server with weak credentials (`Administrator:Summer2025`) for weekend remote access. The goal was to reconstruct the breach from `RDP-Security.evtx`.

**Q: Which MITRE technique ID describes Initial Access via a vulnerable mail server?**
> `T1190`

**Q: Which Initial Access method relies on a user opening a malicious email attachment?**
> `Phishing`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-1/Screenshots/t1.jpg" width="700"/>
</p>

**Q: Which user seems to be most actively brute-forced by botnets?**

**Method:** Scrolled through the filtered Event ID 4625 logs and observed that the large majority of failed logon attempts targeted one account. Without a formal count, this was a visual observation; a more rigorous approach would be exporting the filtered log to CSV and building a pivot table by account and source IP, but Excel wasn't available on the remote lab machine.

> **Answer:** `Administrator`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-1/Screenshots/t2.jpg" width="700"/>
</p>

**Q: Which IP managed to breach the host via RDP (Logon Type 10)?**

**Method:** Filtered for Event ID 4624 (successful logon), then narrowed to Logon Type 10 to find the source IP of the successful remote session.

> **Answer:** `203.205.34.107`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-1/Screenshots/t3.jpg" width="700"/>
</p>

**Q: Can you get the real Workstation Name (hostname) of the threat actor?**

**Method:** Looked for Logon Type 3 (network authentication), the stage of RDP where the real source Workstation Name gets recorded.

> **Answer:** `DESKTOP-QNBC4UU`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-1/Screenshots/t4.jpg" width="700"/>
</p>

---

### Part 2: Phishing Attachments

**Scenario:** Three phishing attachment examples stored under `Phishing Case 1-3`, each illustrating a different disguise technique.

**Q: Run the www.skype.com file from Phishing Case 1, which flag do you get?**

**Method:** Executed the `.com` file directly, playing the role of an untrained user who assumes it's a link rather than a binary.

> **Answer:** `THM{misleading_extension}`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-1/Screenshots/t5.jpg" width="700"/>
</p>

**Q: From which URL does the malicious LNK in Phishing Case 2 download the next stage malware?**

**Method:** Right-clicked the shortcut, opened Properties, and inspected the Target field, which hid a URL behind what looked like a legitimate path. Copied the Target string into Notepad to read it in full.

> **Answer:** `http://wp16.hqywlqpa.thm:8000/cgi-bin/f`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-1/Screenshots/t6.jpg" width="700"/>
</p>

**Q: What is the name of the double-extension file in Phishing Case 3?**

**Method:** Reviewed the folder contents and spotted the file relying on Windows' default behavior of hiding known extensions.

> **Answer:** `best-cat.jpg.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-1/Screenshots/t7.jpg" width="700"/>
</p>

---

### Part 3: Correlating the Attack Chain with Sysmon (Phishing Case 3)

**Q: Which file did the user download via the web browser?**
> **Answer:** `C:\Users\Administrator\Downloads\top-cats.zip`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-1/Screenshots/t8.jpg" width="700"/>
</p>

**Q: In which folder did the user unarchive the suspicious file?**

**Method:** The extraction process appeared just a few seconds after the download event in the timeline.

> **Answer:** `C:\Users\Administrator\Pictures`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-1/Screenshots/t9.jpg" width="700"/>
</p>

**Q: What is the process ID of the launched phishing malware?**

**Method:** Correlated the file extraction event's timestamp with a following Event ID 1 (process create), which confirmed the process ID.

> **Answer:** `5484`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-1/Screenshots/t10.jpg" width="700"/>
</p>

**Q: Which malicious domain did the malware try to connect to?**

**Method:** Used Event ID 22 (DNS query), the only Sysmon event that records the actual domain a process tried to resolve, correlated by Process ID.

> **Answer:** `rjj.store`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-1/Screenshots/t11.jpg" width="700"/>
</p>

---

### Part 4: Initial Access via USB

**Scenario:** A typical USB-borne infection chain, reconstructed from `USB-Sysmon.evtx`.

**Q: Which USB file was launched by the user?**
> **Answer:** `E:\Open Sandisk 4GB USB.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-1/Screenshots/t12.jpg" width="700"/>
</p>

**Q: Which suspicious file did the malware drop to the disk?**

**Method:** Checked the file creation event tied to the same process image and timestamp as the previous event.

> **Answer:** `C:\Users\Public\Documents\winupdate.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-1/Screenshots/t13.jpg" width="700"/>
</p>

**Q: To which other USB did the malware propagate?**
> **Answer:** `F:`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-1/Screenshots/t14.jpg" width="700"/>
</p>

---

## 🧠 What I Learned

### Technical Skills
- How to detect an RDP brute force and successful breach purely from default Windows Security logs (Event IDs 4624/4625) by pivoting on logon type and source IP
- How to identify the real Workstation Name of an attacker by reading Logon Type 3 events
- How Windows hides known file extensions, and how attackers abuse that with double-extension files and misleading icons
- How to read an LNK file's hidden PowerShell payload directly from its Target field
- How to correlate a full attack chain (browser download to unarchive to execution to DNS query) using Sysmon Event IDs 1, 11, and 22 and a shared Process ID

### Analyst Mindset
- RDP exposure is often called the "Ransomware Deployment Protocol" for a reason. Detection here is less about clever pivoting and more about knowing which default logs to check first
- Extension-hiding attacks (double extensions, misleading COM/SCR files) succeed because they exploit user assumptions, not technical gaps, so user awareness and file-type alerting both matter
- LNK-based phishing leaves very little direct execution trace since explorer.exe appears to be the one launching PowerShell. The real tell is the preceding file creation event showing the LNK arriving in Downloads
- USB-based Initial Access looks nearly identical to phishing in the logs since both rely on a user double-clicking a binary via explorer.exe. The differentiator is the drive letter (D:, E:, F:) in the process image path

---

## 📝 Key Takeaways

- The two most common Windows Initial Access methods are exposed services and user-driven attacks
- Initial Access via RDP can be easily detected using default authentication logs (4624/4625)
- User-driven attacks are best detected by process execution events, preferably Sysmon ones
- Each Initial Access method (like LNK) has unique features that come through with practice

---

## 💬 Honest Self-Assessment

**What went well:**
Pivoting between event IDs (4625 to 4624, then Sysmon 1 to 11 to 22) to rebuild a full attack timeline felt intuitive by the end. Correlating by Process ID and timestamp became a repeatable pattern across all three attack types (RDP, phishing, USB).

**What I need to improve:**
Doing an actual count of brute force attempts per account without relying on visual scanning. I'd want to get comfortable exporting Event Viewer logs to CSV and using a pivot table (or a quick Python/pandas script) instead of eyeballing it, since that won't scale on a real high-volume log.

---

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Windows-Threat-Detection-1/Screenshots/tfinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
