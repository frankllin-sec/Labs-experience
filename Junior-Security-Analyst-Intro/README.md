# 🛡️ Junior Security Analyst Intro 

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-SOC%20Alert%20Triage-blue?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-green?style=for-the-badge"/>
</p>

---
<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Junior-Security-Analyst-Intro/Screenshots/soc-1-intro.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Junior-Security-Analyst-Intro/Screenshots/soc-1-intro.jpg" width="600"/>
  </a>
</p>

## 📌 About This Lab

This lab simulates the real day-to-day experience of a **Junior SOC Analyst** on the defensive frontline. During a typical 8-hour shift, a SOC analyst is responsible for triaging a constant stream of alerts and tickets — identifying threats, investigating suspicious activity, escalating confirmed incidents, and documenting findings.

The lab places you directly in the role of a Security Analyst working an alert dashboard. The goal was to identify a malicious IP address in the alerts, investigate it using threat intelligence tools, escalate the incident to the appropriate team, and take containment action.

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **Alert Triage** | Process of reviewing, prioritizing, and responding to security alerts |
| **IP Reputation** | Checking an IP against threat intel sources to determine if it is malicious |
| **Escalation** | Passing a confirmed or high-severity alert to a senior analyst or team lead |
| **Firewall Block** | Adding a malicious IP to the firewall deny list to stop further communication |
| **SOC Tier 1** | First line of defense  responsible for monitoring, triage, and initial response |

---

## 🔍 Investigation — Alert Dashboard Analysis

### Scenario

During a shift on the SOC alert dashboard, multiple alerts were generated. The task was to identify the malicious IP address within the alerts, investigate it using IP Hunter (threat intelligence), escalate the incident to the appropriate person, and add the IP to the firewall block list as a containment measure.

---

### ❓ Q1 — What was the malicious IP address in the alerts?

**Method:** Navigated to the alert dashboard and reviewed active alerts. Identified the suspicious IP address flagged across multiple events. Cross-referenced using **IP Hunter** to confirm malicious reputation.

> **Answer:** `221.181.185.159`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Junior-Security-Analyst-Intro/Screenshots/soc-1-1.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Junior-Security-Analyst-Intro/Screenshots/soc-1-1.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q2 — To whom did you escalate the alert with the malicious IP?

**Method:** After confirming the IP was malicious via IP Hunter, clicked **Next** to proceed with the escalation workflow. Selected the appropriate escalation target based on severity. Added the IP `221.181.185.159` to the firewall block list as an immediate containment action.

> **Answer:** `SOC Team Lead`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Junior-Security-Analyst-Intro/Screenshots/soc-1-2.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Junior-Security-Analyst-Intro/Screenshots/soc-1-2.jpg" width="600"/>
  </a>
</p>

---

## 🛑 Containment Action Taken

| Action | Detail |
|---|---|
| **Escalation target** | SOC Team Lead |
| **Containment** | IP `221.181.185.159` added to firewall block list |
| **Tool used** | IP Hunter (threat intelligence lookup) |

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Junior-Security-Analyst-Intro/Screenshots/soc-1-3.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Junior-Security-Analyst-Intro/Screenshots/soc-1-3.jpg" width="600"/>
  </a>
</p>

---

## 🧠 What I Learned

### Technical Skills
- How to navigate a SOC alert dashboard and identify suspicious events
- How to use **IP Hunter** to investigate and confirm malicious IP reputation
- The proper escalation workflow when to escalate and to whom
- How to apply immediate containment adding a malicious IP to the firewall block list
- The importance of speed and accuracy in alert triage — false negatives have real consequences

### Analyst Mindset
- Not every alert is a real threat but every alert deserves a proper triage process
- Confirming an IP with threat intelligence before escalating is essential escalate with evidence, not assumptions
- Containment and escalation happen simultaneously in a real SOC don't wait to act
- Being a Tier 1 analyst means being the first line of defense. The decisions you make in triage directly impact the organization's security posture

---

## 💬 Honest Self-Assessment

**What went well:**
The alert triage workflow felt intuitive identify the alert, investigate the IP, confirm with threat intel, escalate, contain. Connecting each step to a real SOC process made the exercise click.

**What I need to improve:**
Building speed. In a real SOC, I would be handling multiple alerts simultaneously under time pressure. Developing the muscle memory to move through triage steps faster without skipping quality is the next level.

---

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Junior-Security-Analyst-Intro/Screenshots/soc-1-end.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Junior-Security-Analyst-Intro/Screenshots/soc-1-end.jpg" width="600"/>
  </a>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
