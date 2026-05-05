# 🛡️ SOC Alert Triage Lab — TryHackMe

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Alert%20Triage-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Status-Completed-green?style=for-the-badge"/>
</p>

---

## 📌 About This Lab

This lab simulates a real **SOC Analyst Tier 1** workflow  triaging security alerts, investigating incidents, writing case reports, and classifying alerts as True Positive or False Positive.

The scenario involved a phishing campaign that targeted employees of a simulated company. My job was to analyze each alert, correlate the events, and document my findings professionally.

---

## 🔍 Alerts Investigated

---

### Alert #8814 — Inbound Email Containing Suspicious External Link
**Severity:** Medium | **Type:** Phishing | **Classification:** ✅ False Positive

**Analysis:**
The email was sent from `onboarding@hrconnex.thm` to employee `j.garcia@thetrydaily.thm`. After investigation, the sender domain `hrconnex.thm` is the company's internal HR onboarding system. The link directed to the same trusted internal domain. No malicious indicators were found  the alert was a false positive triggered by an automated HR workflow.

**Key Decision Factor:** Internal domain + legitimate onboarding context = False Positive.

---

### Alert #8815 — Inbound Email Containing Suspicious External Link
**Severity:** Medium | **Type:** Phishing | **Classification:** 🚨 True Positive

**Analysis:**
Email received by `h.harris@thetrydaily.thm` from `urgents@amazon.biz` impersonating Amazon. Multiple phishing indicators identified:

- Fake sender domain: `amazon.biz` instead of `amazon.com`
- Urgency language: "48 hours or package returned"
- URL shortener: `http://bit.ly/3sHkX3da12340` hiding real destination
- Unencrypted link: HTTP (port 80) instead of HTTPS (port 443)

**Key Decision Factor:** Fake domain + urgency + URL shortener = True Positive. Escalated.

---

### Alert #8816 — Access to Blacklisted External URL Blocked by Firewall
**Severity:** High | **Type:** Firewall | **Classification:** 🚨 True Positive

**Analysis:**
Internal IP `10.20.2.17`  attempted to access `http://bit.ly/3sHkX3da12340`  the exact same malicious URL from the phishing email. This confirmed the employee clicked the phishing link 1 minute after receiving the email. The firewall successfully blocked the connection before it reached the destination IP.

**Key Decision Factor:** Same URL as phishing email + user clicked = confirmed compromise. Escalated immediately.

**Correlation:** This alert directly linked to Alert #8815 — connecting phishing delivery to user interaction.

---

### Alert #8817 — Inbound Email Containing Suspicious External Link
**Severity:** Medium | **Type:** Phishing | **Classification:** 🚨 True Positive

**Analysis:**
Additional phishing email identified as part of the same campaign. Classified as True Positive based on consistent phishing indicators matching the previous alerts.


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

---

## 🧠 What I Learned

### Technical Skills
- How to correlate multiple alerts to reconstruct an attack timeline
- Identifying phishing indicators: fake domains, urgency language, URL shorteners, HTTP vs HTTPS
- Difference between True Positive and False Positive 
- How firewall logs confirm user interaction with malicious links


### Analyst Mindset
- Always check if alerts are connected Alert #8816 only made full sense after investigating #8815
- Port 80 (HTTP) on an external link is a red flag, legitimate services use HTTPS (port 443)
- Timestamp correlation is critical — the 1-minute gap between email receipt and firewall alert confirmed the click

---

## 💬 Honest Self-Assessment

**What went well:**
I identified the correct classification for all 4 alerts quickly. The technical analysis — recognizing fake domains, URL shorteners, HTTP vs HTTPS, and correlating alerts  felt natural and came fast, i`ve been studing a lot these days 

**What I need to improve:**
The part that took the most time was writing the incident reports. Grammar, structure, and making sure I covered all the required fields (Who, What, When, Where, Why) consistently slowed me down.

This is something I expect to improve naturally with daily practice, the more I do it, the more automatic it becomes. My goal is to build this muscle memory.

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

<a href="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/soc-alert-triage-lab/lab2.jpg">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/soc-alert-triage-lab/lab2.jpg" width="400"/>
</a>

<a href="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/soc-alert-triage-lab/lab1.jpg">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/soc-alert-triage-lab/lab1.jpg" width="400"/>
</a>

---

## 🛠️ Skills Demonstrated

| Skill | Application |
|---|---|
| Alert Triage | Classified 4 alerts with 100% accuracy |
| Phishing Analysis | Identified fake domains, URL shorteners, urgency tactics |
| Alert Correlation | Connected phishing email to firewall alert using timestamps and IOCs |
| Incident Reporting | Wrote professional case reports with IOCs and remediation steps |
| False Positive Identification | Correctly identified internal HR email as benign |
| HTTP vs HTTPS | Identified unencrypted connection as phishing indicator |

---


---

<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
