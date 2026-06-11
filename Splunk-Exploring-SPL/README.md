# 📊 Splunk: Exploring SPL — TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Splunk%20SPL-green?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

---

## 📌 About This Lab

This lab covers **Splunk's Search Processing Language (SPL)** — the core skill for any SOC analyst using Splunk as their SIEM. SPL turns raw log data into actionable intelligence. The lab progressed through search fundamentals, filtering, structuring results, transforming commands, and anomaly detection against real Windows and VPN log datasets.

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Splunk-Exploring-SPL/Screenshots/sp-intro.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Splunk-Exploring-SPL/Screenshots/sp-intro.jpg" width="600"/>
  </a>
</p>

---
### 🔎 Search & Reporting

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Splunk-Exploring-SPL/Screenshots/sp-1.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Splunk-Exploring-SPL/Screenshots/sp-1.jpg" width="600"/>
  </a>
</p>

**❓ Q1 — How many total events?**
> **Answer:** `12,256`

**❓ Q2 — Which SourceIP has the most events?**

**Method:** SourceIp was not in the default Interesting Fields — clicked All Fields, searched for SourceIp, identified the top value.

> **Answer:** `172.90.12.11`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Splunk-Exploring-SPL/Screenshots/sp-2.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Splunk-Exploring-SPL/Screenshots/sp-2.jpg" width="600"/>
  </a>
</p>

---

### 🔎 Search Operators

**❓ Q3 — Events on 04/15/2022 from 08:05 AM to 08:06 AM?**

**Method:** Clicked All Time, selected Date & Time Range, set the specific window, clicked Apply.

> **Answer:** `134`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Splunk-Exploring-SPL/Screenshots/sp-3.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Splunk-Exploring-SPL/Screenshots/sp-3.jpg" width="600"/>
  </a>
</p>

**❓ Q4 — Events with EventID = 4624?**

```
index="windowslogs" EventID="4624"
```
> **Answer:** `26`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Splunk-Exploring-SPL/Screenshots/sp-4.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Splunk-Exploring-SPL/Screenshots/sp-4.jpg" width="600"/>
  </a>
</p>

**❓ Q5 — Events with DestinationIp = 172.18.39.6 AND DestinationPort = 135?**

```
index="windowslogs" DestinationIp="172.18.39.6" AND DestinationPort="135"
```
> **Answer:** `4`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Splunk-Exploring-SPL/Screenshots/sp-5.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Splunk-Exploring-SPL/Screenshots/sp-5.jpg" width="600"/>
  </a>
</p>

**❓ Q6 — Highest SourceIp for Hostname=Salena.Adam DestinationIp=172.18.38.5?**

**Method:** Ran the query and clicked SourceIp in the Interesting Fields panel.

> **Answer:** `172.90.12.11`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Splunk-Exploring-SPL/Screenshots/sp-6.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Splunk-Exploring-SPL/Screenshots/sp-6.jpg" width="600"/>
  </a>
</p>

**❓ Q7 — Events returned with Cyber*?**

```
index=windowslogs Cyber*
```
> **Answer:** `12,256`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Splunk-Exploring-SPL/Screenshots/sp-7.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Splunk-Exploring-SPL/Screenshots/sp-7.jpg" width="600"/>
  </a>
</p>

**❓ Q8 — Which operator has the lowest priority?**
> **Answer:** `AND`

---

### 🔎 Filtering Results

**❓ Q9 — Which SourceProcessId has the highest value?**

**Method:** Used fields command to highlight Domain, SourceProcessId, TargetProcessId. Clicked All Fields and selected highlighted fields.

> **Answer:** `9496`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Splunk-Exploring-SPL/Screenshots/sp-8.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Splunk-Exploring-SPL/Screenshots/sp-8.jpg" width="600"/>
  </a>
</p>

**❓ Q10 — Which TargetObject has highest results using regex Manager$?**

```
index=windowslogs | regex TargetObject="Manager$"
```
> **Answer:** `HKLM\SOFTWARE\Microsoft\SecurityManager`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Splunk-Exploring-SPL/Screenshots/sp-9.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Splunk-Exploring-SPL/Screenshots/sp-9.jpg" width="600"/>
  </a>
</p>

---

### 🔎 Structuring Results

**❓ Q11 — Which AccountName appears first?**

```
index=windowslogs | table _time EventID AccountName AccountType
```
> **Answer:** `System`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Splunk-Exploring-SPL/Screenshots/sp-10.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Splunk-Exploring-SPL/Screenshots/sp-10.jpg" width="600"/>
  </a>
</p>

**❓ Q12 — After adding reverse, which EventID appears first?**
> **Answer:** `800`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Splunk-Exploring-SPL/Screenshots/sp-11.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Splunk-Exploring-SPL/Screenshots/sp-11.jpg" width="600"/>
  </a>
</p>

**❓ Q13 — What password was given to user A1berto?**

```
index=windowslogs Alberto EventID=1
| table _time ParentProcessId ProcessId ParentCommandLine CommandLine
| reverse
```

**Method:** Added username Alberto to filter results. Identified the password in the CommandLine field.

> **Answer:** `paw0rd1`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Splunk-Exploring-SPL/Screenshots/sp-12.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Splunk-Exploring-SPL/Screenshots/sp-12.jpg" width="600"/>
  </a>
</p>

---

### 🔎 Transforming Commands

**❓ Q14 — Which Image has the most occurrences?**

```
index=windowslogs | top image
```
> **Answer:** `C:\windows\system32\svchost.exe`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Splunk-Exploring-SPL/Screenshots/sp-13.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Splunk-Exploring-SPL/Screenshots/sp-13.jpg" width="600"/>
  </a>
</p>

**❓ Q15 — Which Region do the IPs originate from?**

```
index=windowslogs | iplocation SourceIp
```
> **Answer:** `California`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Splunk-Exploring-SPL/Screenshots/sp-14.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Splunk-Exploring-SPL/Screenshots/sp-14.jpg" width="600"/>
  </a>
</p>

**❓ Q16 — Which Image has the highest RiskScore?**

```
index=windowslogs
| lookup image_riskscore Image OUTPUT RiskScore
| stats count by Image RiskScore
| sort - RiskScore
```
> **Answer:** `C:\Windows\System32\WindowsPowerShell1.0\powershell.exe`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Splunk-Exploring-SPL/Screenshots/sp-15.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Splunk-Exploring-SPL/Screenshots/sp-15.jpg" width="600"/>
  </a>
</p>

---

### 🔎 Anomaly Detection

**❓ Q17 — Which user is an outlier? Which country is anomalous?**

```
index=vpnlogs
| eventstats count as logins_by_user by user
| eventstats count as logins_by_user_country by user src_country
| eval country_freq=logins_by_user_country/logins_by_user
| where country_freq < 0.1
| table _time user src_ip src_country country_freq
```
> **Outlier user:** `jsmith` | **Anomalous country:** `JP`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Splunk-Exploring-SPL/Screenshots/sp-16.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Splunk-Exploring-SPL/Screenshots/sp-16.jpg" width="600"/>
  </a>
</p>

**❓ Q18 — Which user suspiciously logged in at 3 AM?**

```
index=vpnlogs
| eval hour=tonumber(strftime(_time, "%H")) + tonumber(strftime(_time, "%M"))/60
| eventstats avg(hour) as typical_hour stdev(hour) as stdev_hour by user
| eval zscore=abs(hour - typical_hour) / stdev_hour
| where zscore > 3
| table _time user src_ip src_country hour typical_hour stdev_hour zscore
```
> **Answer:** `njackson`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Splunk-Exploring-SPL/Screenshots/sp-17.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Splunk-Exploring-SPL/Screenshots/sp-17.jpg" width="600"/>
  </a>
</p>

---

## 🧠 What I Learned

### Technical Skills
- How to build and chain SPL queries using index, fields, table, top, stats, regex, reverse, iplocation, and lookup
- How to use time picker to filter events to a specific date and time window
- How eventstats and eval calculate statistical baselines for anomaly detection
- How zscore identifies logins that deviate from a user's typical behavior
-

### Analyst Mindset
- SPL is not about memorizing commands — it is about knowing what question to ask
- Anomaly detection is powerful because attackers can steal credentials but cannot always mimic normal user behavior
- A login from an unusual country at 3 AM with a high zscore is not proof of compromise — but it absolutely warrants investigation
- The lookup command connecting process names to RiskScores simulates real threat intel enrichment in enterprise SIEMs

---

## 💬 Honest Self-Assessment

**What went well:**
SPL pipe logic clicked quickly — each command transforms the previous output intuitively. The anomaly detection section was the most interesting, connecting statistical concepts to real SOC use cases.

**What I need to improve:**
The most challenging part of this lab was the **Anomaly Detection** section — specifically understanding how `zscore` works mathematically and how to apply it to detect behavioral outliers in login data. This is where I spent the most time and still need more practice. Building multi-stage SPL pipelines like the anomaly detection queries independently from memory will require repetition with real datasets before it becomes natural.

---


<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Splunk-Exploring-SPL/Screenshots/sp-end.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Splunk-Exploring-SPL/Screenshots/sp-end.jpg" width="600"/>
  </a>
</p>

---

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
<p align="center"><a href="https://github.com/frankllin-sec">🔗 Visit my GitHub Profile</a></p>
