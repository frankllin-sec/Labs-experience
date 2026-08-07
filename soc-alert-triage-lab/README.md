# 🛡️ SOC Alert Triage Lab: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Alert%20Triage-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

---

## 📌 About This Lab

This lab simulates a real SOC Analyst Tier 1 workflow: triaging security alerts, investigating incidents, writing case reports, and classifying alerts as True Positive or False Positive.

The scenario involved a phishing campaign targeting employees of a simulated company. The job was to analyze each alert, correlate the events, and document the findings in a professional case report.

**Objectives:**
- Practice triaging real-world style SOC alerts
- Classify alerts as True Positive or False Positive
- Correlate related alerts into a single attack timeline
- Write professional incident case reports

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **True Positive vs False Positive** | A True Positive is a correctly identified malicious alert. A False Positive looks suspicious but turns out to be legitimate activity |
| **Phishing Indicators** | Fake or lookalike sender domains, urgency language, URL shorteners hiding the real destination, and HTTP instead of HTTPS links |
| **Alert Correlation** | Connecting multiple alerts, like a phishing email and a firewall block, using timestamps and shared IOCs to reconstruct an attack timeline |
| **MTTR / Dwell Time** | Mean Time to Resolve measures how fast an alert was closed. Dwell Time measures how long the threat was present before detection |
| **Incident Report Structure** | A complete case report answers Who, What, When, Where, and Why for each alert |

---

## 🔍 Investigation

### Alert #8814: Inbound Email Containing Suspicious External Link
**Severity:** Medium | **Type:** Phishing | **Classification:** ✅ False Positive

**Analysis:** The email was sent from `onboarding@hrconnex.thm` to employee `j.garcia@thetrydaily.thm`. After investigation, the sender domain `hrconnex.thm` turned out to be the company's internal HR onboarding system, and the link directed to that same trusted internal domain. No malicious indicators were found.

> **Key Decision Factor:** Internal domain plus legitimate onboarding context equals False Positive.

---

### Alert #8815: Inbound Email Containing Suspicious External Link
**Severity:** Medium | **Type:** Phishing | **Classification:** 🚨 True Positive

**Analysis:** Email received by `h.harris@thetrydaily.thm` from `urgents@amazon.biz` impersonating Amazon. Multiple phishing indicators identified:

- Fake sender domain: `amazon.biz` instead of `amazon.com`
- Urgency language: "48 hours or package returned"
- URL shortener: `http://bit.ly/3sHkX3da12340` hiding the real destination
- Unencrypted link: HTTP (port 80) instead of HTTPS (port 443)

> **Key Decision Factor:** Fake domain plus urgency plus URL shortener equals True Positive. Escalated.

---

### Alert #8816: Access to Blacklisted External URL Blocked by Firewall
**Severity:** High | **Type:** Firewall | **Classification:** 🚨 True Positive

**Analysis:** Internal IP `10.20.2.17` attempted to access `http://bit.ly/3sHkX3da12340`, the exact same malicious URL from the phishing email. This confirmed the employee clicked the phishing link one minute after receiving it. The firewall successfully blocked the connection before it reached the destination IP.

> **Key Decision Factor:** Same URL as the phishing email plus a user click equals confirmed compromise. Escalated immediately.

**Correlation:** This alert directly linked to Alert #8815, connecting phishing delivery to user interaction.

---

### Alert #8817: Inbound Email Containing Suspicious External Link
**Severity:** Medium | **Type:** Phishing | **Classification:** 🚨 True Positive

**Analysis:** Additional phishing email identified as part of the same campaign. Classified as True Positive based on consistent phishing indicators matching the previous alerts.

---

## 📊 Final Results

| Metric | Result |
|---|---|
| 🔔 Closed Alerts | 4 alerts |
| ✅ True Positive Rate | 100% |
| ✅ False Positive Rate | 100% |
| ⏱️ Mean Time to Resolve | 5 minutes |
| ⏱️ Mean Dwell Time | 12 minutes |
| 🏆 Ranking | 1st place |
| 🎯 Score | 160 pts |

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/soc-alert-triage-lab/lab2.jpg" width="700"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/soc-alert-triage-lab/lab3.jpg" width="700"/>
</p>

---

## 🧠 What I Learned

- How to correlate multiple alerts to reconstruct an attack timeline
- How to identify phishing indicators: fake domains, urgency language, URL shorteners, and HTTP vs HTTPS
- The difference between True Positive and False Positive classification
- How firewall logs confirm user interaction with a malicious link
- Port 80 (HTTP) on an external link is a red flag since legitimate services use HTTPS (port 443)
- Timestamp correlation is critical: the one-minute gap between email receipt and the firewall alert confirmed the click

---

## 💬 Honest Self-Assessment

**What went well:**
I identified the correct classification for all 4 alerts quickly. The technical analysis, recognizing fake domains, URL shorteners, HTTP vs HTTPS, and correlating alerts, felt natural and came fast, since I had been studying a lot in the days before this lab.

**What I need to improve:**
The part that took the most time was writing the incident reports. Grammar, structure, and making sure I covered all the required fields (Who, What, When, Where, Why) consistently slowed me down. This is something I expect to improve naturally with daily practice, the more I do it, the more automatic it becomes. My goal is to build that muscle memory.

---
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/soc-alert-triage-lab/lab1.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
