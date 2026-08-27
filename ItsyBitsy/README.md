# 🛡️ ItsyBitsy: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-SIEM%20Triage-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/ItsyBitsy/Screenshots/tinicio.jpg" width="700"/>
</p>

---

## 📌 About This Lab

During normal SOC monitoring, Analyst John observed an alert on an IDS solution indicating a potential C2 communication from a user, Browne, in the HR department. A suspicious file was accessed containing a malicious pattern (`THM{________}`). A week-long HTTP connection log was pulled and ingested into a `connection_logs` index in Kibana. The task: examine the network connection logs of this user, find the link and the content of the flagged file, and answer the investigation questions using Kibana/ELK.

**Objectives:**
- Filter and count events in Kibana Discover by date range
- Identify a suspicious source IP by its rarity against normal traffic
- Recognize a living-off-the-land binary (LOLBin) via User-Agent strings
- Trace a C2 URL back to a filesharing site abused for command and control
- Retrieve and decode the payload left on that site

---

## 🔍 Investigation

**Q: How many events were returned for the month of March 2022?**

**Method:** Filtered the Kibana Discover time range to March 2022 against the `connection_logs` index.

> **Answer:** `1482`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/ItsyBitsy/Screenshots/t1.jpg" width="700"/>
</p>

**Q: What is the IP associated with the suspected user in the logs?**

**Method:** Selected the `source_ip` field and reviewed the Top 5 values. One IP made up 99.6% of traffic, ordinary user activity, while a second IP sat at just 0.4%. That rarity was the tell, so I investigated it further and found it communicating with the host `pastebin.com`, a strong signal for something abnormal.

> **Answer:** `192.166.65.54`

**Q: The infected machine connected with a famous filesharing site in this period, which also acts as a C2 server. What is the name of the filesharing site?**

**Method:** Expanded the document details for that suspicious IP's traffic and found the host field directly.

> **Answer:** `pastebin.com`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/ItsyBitsy/Screenshots/t2.jpg" width="700"/>
</p>

**Q: The user's machine used a legit Windows binary to download a file from the C2 server. What is the name of the binary? What is the full URL of the C2?**

**Method:** Selected the `user_agent` field and compared values. Most traffic showed a normal `Mozilla/5.0` browser string, but one entry stood out: `bitsadmin`, a legitimate Windows binary abused to download files. Filtering on that user-agent revealed the full request URI.

> **Answer:** `bitsadmin` (binary), `pastebin.com/yTg0Ah6a` (full C2 URL)

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/ItsyBitsy/Screenshots/t3.jpg" width="700"/>
</p>

**Q: A file was accessed on the filesharing site. What is the name of the file accessed? The file contains a secret code with the format `THM{_____}`.**

**Method:** Visited `pastebin.com/yTg0Ah6a` to check the paste directly.

**Note:** In a real investigation I wouldn't browse a suspected C2 URL directly from my own machine, I'd use an isolated VM or a tool like urlscan.io to preview the page safely before ever opening it myself.

> **Answer:** File name `secret.txt`, secret code `THM{SECRET_CODE}`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/ItsyBitsy/Screenshots/t4.jpg" width="700"/>
</p>

---

## 🧠 What I Learned

- How to use Kibana Discover's date range and field statistics (Top 5 values) to spot a rare, anomalous source IP hiding inside otherwise normal traffic
- How to recognize a LOLBin (Living-off-the-Land Binary) like `bitsadmin` by its User-Agent string standing out against normal browser traffic
- How trusted filesharing platforms like Pastebin get abused as free, hard-to-block C2 infrastructure
- The correct operational security practice for investigating a suspected malicious URL: never browse it directly from an analyst workstation, use an isolated VM or a preview tool like urlscan.io instead

---

## 💬 Honest Self-Assessment

**What went well:**
The investigation flowed cleanly from one pivot to the next: date filter to rare IP to LOLBin user-agent to C2 URL to payload. Recognizing `bitsadmin` as a LOLBin rather than just noise in the user-agent field was a good instinct, and I liked that I caught myself before treating the pastebin link like a normal one and noted the safer way to have handled it.

**What I need to improve:**
This is a good habit I want to make automatic rather than something I only remember after the fact: before opening any URL tied to an investigation, default to an isolated VM or urlscan.io first, every time, not just when it occurs to me partway through.

---
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/ItsyBitsy/Screenshots/tfinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
