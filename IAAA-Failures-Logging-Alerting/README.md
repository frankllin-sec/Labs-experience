# 🔐 IAAA Failures - Logging & Alerting Failure 

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-OWASP%20Top%2010-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

---
<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/IAAA-Failures-Logging-Alerting/Screenshots/logging%20%26%20alerting%20failuresintro.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/IAAA-Failures-Logging-Alerting/Screenshots/logging%20%26%20alerting%20failuresintro.jpg" width="600"/>
  </a>
</p>


## 📌 About This Lab

This lab covers **OWASP Top 10 2025 A01, A07, and A09** in the context of the **IAAA model** (Identification, Authentication, Authorization, and Accountability), with a focus on **logging and alerting failures**.

When applications fail to record or alert on security-relevant events, defenders lose the ability to detect and investigate attacks. Good logging underpins accountability proving who did what, when, and from where. In practice, failures look like:

- Missing authentication events
- No alerting on brute-force attempts or privilege changes
- Short log retention periods

The task was to analyze a static site simulating an application under attack, investigate the logs, and identify attacker activity.

---

## 🔍 Investigation — Logging & Alerting Failure Analysis

### Scenario

An application under attack was analyzed through its logs. The goal was to identify attacker activity specifically a brute-force attack and understand how logging failures would make investigation significantly harder or impossible.

---

### ❓ Q1 It looks like an attacker tried to perform a brute-force attack. What is the IP of the attacker?

**Method:** Analyzed the application logs in the static site. Looked for repeated failed authentication attempts from a single IP address within a short timeframe — the classic pattern of a brute-force attack.

> **Answer:** `203.0.113.45`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/IAAA-Failures-Logging-Alerting/Screenshots/logging%20%26%20alerting%20failures.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/IAAA-Failures-Logging-Alerting/Screenshots/logging%20%26%20alerting%20failures.jpg" width="600"/>
  </a>
</p>

### ❓ Q2  Looks Like they were able to gain access to an account! What is the username associated with that account??
> **Answer:** `admin`


<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/IAAA-Failures-Logging-Alerting/Screenshots/logging%20%26%20alerting%20failures2.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/IAAA-Failures-Logging-Alerting/Screenshots/logging%20%26%20alerting%20failures2.jpg" width="600"/>
  </a>
</p>

**Method:** Analyzed the application logs in the static site. Looked for repeated failed authentication attempts from a single IP address within a short timeframe the classic pattern of a brute-force attack.

---

## 🧠 What I Learned

### Technical Skills
- How logging failures (OWASP A09) directly impact the ability to detect and investigate attacks
- How to identify brute-force patterns in application logs repeated failed attempts from a single IP

## 💬 Honest Self-Assessment

**What went well:**
Connecting OWASP A09 to a real investigation scenario made the concept concrete. Identifying the brute-force pattern in the logs multiple failed attempts from the same IP  felt intuitive once I knew what to look for.


---

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/IAAA-Failures-Logging-Alerting/Screenshots/logging%20%26%20alerting%20failures4.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/IAAA-Failures-Logging-Alerting/Screenshots/logging%20%26%20alerting%20failures4.jpg" width="600"/>
  </a>
</p>

---

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
