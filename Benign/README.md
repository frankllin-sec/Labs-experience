# 🛡️ Benign: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Host%20Investigation-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Benign/Screenshots/binicio.jpg" width="700"/>
</p>

**Room link:** [tryhackme.com/room/benign](https://tryhackme.com/room/benign)

---

## 📌 About This Lab

An IDS flagged suspicious process execution on a host in the HR department. Some of the tools that ran are normally associated with network information gathering and scheduled tasks, which is exactly the kind of thing that shows up right before something worse happens. Only the process execution logs (Event ID 4688) could be pulled, and they were ingested into Splunk under the index `win_eventlogs`. The job: work through those logs and figure out what actually happened on that host.

**Network layout for context:**
- **IT:** James, Moin, Katrina
- **HR:** Haroon, Chris, Diana
- **Marketing:** Bell, Amelia, Deepak

---

## 🔍 Investigation

**Q: How many logs are ingested from the month of March 2022?**

**Command:** `index=win_eventlogs`

**Logic:** Just filtered the time range in Splunk to all of March 2022 and read the total event count.

> **Answer:** `13959`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Benign/Screenshots/b1.jpg" width="700"/>
</p>

**Q: Imposter Alert, there seems to be an imposter account in the logs. What is the name?**

**Command:** `index=win_eventlogs UserName=* | table UserName`

**Logic:** Listed every username showing up in the logs and sorted the column alphabetically. One name looked almost identical to a real employee (Amelia), but with the lowercase `l` swapped in for the `i`, a classic lookalike account trick.

> **Answer:** `Amel1a`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Benign/Screenshots/b2.jpg" width="700"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Benign/Screenshots/b2-1.jpg" width="700"/>
</p>

**Q: Which user from the HR department was observed running scheduled tasks?**

**Command:** `index=win_eventlogs schtasks`

**Logic:** That search returned four different usernames. Cross-checked each one against the HR department list from the scenario, only one of them actually belonged to HR.

> **Answer:** `Chris.fort`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Benign/Screenshots/b3.jpg" width="700"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Benign/Screenshots/b3-1.jpg" width="700"/>
</p>

**Q: Which system process (LOLBIN) was used to download a payload from the internet, bypassing security controls?**

**Command:** `index=win_eventlogs UserName="chris.fort" | table CommandLine | dedup CommandLine`

**Logic:** Pulled every command line that user ran and removed duplicates. Normal, everyday activity tends to repeat itself, malicious activity usually doesn't, so stripping out the repeated lines made the one unusual command much easier to spot.

> **Answer:** `certutil.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Benign/Screenshots/b4.jpg" width="700"/>
</p>

**Q: What date was that binary executed? Which third-party site delivered the payload? What file got saved on the host?**

**Logic:** The full command line that stood out in the previous step already contained all three answers, the date, the destination site, and the file name it saved locally.

> **Answer:** Executed on `2022-03-04`, payload delivered from `controlc.com`, file saved as `benign.exe`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Benign/Screenshots/b5.jpg" width="700"/>
</p>

**Q: The downloaded file contained a pattern `THM{...}`. What is it?**

**Method:** Checked the flagged URL through urlscan.io to safely preview the page content without visiting it directly.

> **Answer:** `THM{KJ&*H^B0}`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Benign/Screenshots/b6.jpg" width="700"/>
</p>

**Q: What is the full URL that the infected host connected to?**

> **Answer:** `https[:]//controlc[.]com/e4d11035`

---

## 🧠 What I Learned

- How to filter Splunk logs by index and time range to get a basic feel for the data before diving into specifics
- Spotting a lookalike account by sorting usernames and reading closely, `Amel1a` vs `Amelia` is an easy thing to miss if you're skimming
- Cross-referencing a username against a department list to figure out who's actually who in an investigation
- Using `dedup` to strip out repeated, normal activity so the one unusual command line stands out on its own
- Recognizing `certutil.exe` as a LOLBIN, a legitimate Windows tool getting abused to download something it was never meant to download
- Checking a suspicious URL safely through urlscan.io instead of opening it directly

---

## 💬 Honest Self-Assessment

**What I need to improve:**
The Splunk commands I used here (`table`, `dedup`, filtering by field) were still things I had to think through rather than type automatically. I want to get to the point where building a search like this is second nature, since a real investigation won't wait for me to look up the right syntax.

---
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Benign/Screenshots/bfinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
