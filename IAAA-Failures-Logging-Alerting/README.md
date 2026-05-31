# 🔐 IAAA Failures — Logging & Alerting Failure — TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-OWASP%20Top%2010-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

---

## 📌 About This Lab

This lab covers **OWASP Top 10 2025 — A01, A07, and A09** in the context of the **IAAA model** (Identification, Authentication, Authorization, and Accountability), with a focus on **logging and alerting failures**.

When applications fail to record or alert on security-relevant events, defenders lose the ability to detect and investigate attacks. Good logging underpins accountability — proving who did what, when, and from where. In practice, failures look like:

- Missing authentication events
- Vague or incomplete error logs
- No alerting on brute-force attempts or privilege changes
- Short log retention periods
- Logs stored where attackers can tamper with them

The task was to analyze a static site simulating an application under attack, investigate the logs, and identify attacker activity.

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **IAAA Model** | Identification, Authentication, Authorization, Accountability |
| **A01 - Broken Access Control** | Attackers bypass access restrictions to unauthorized resources |
| **A07 - Auth Failures** | Weak authentication allows credential attacks like brute-force |
| **A09 - Logging Failures** | Missing or inadequate logs blind defenders to ongoing attacks |
| **Brute-Force Attack** | Automated repeated login attempts to guess valid credentials |
| **Log Integrity** | Logs must be protected from tampering — stored securely, off-system |

---

## 🔍 Investigation — Logging & Alerting Failure Analysis

### Scenario

An application under attack was analyzed through its logs. The goal was to identify attacker activity — specifically a brute-force attack — and understand how logging failures would make investigation significantly harder or impossible.

---

### ❓ Q1 — It looks like an attacker tried to perform a brute-force attack. What is the IP of the attacker?

**Method:** Analyzed the application logs in the static site. Looked for repeated failed authentication attempts from a single IP address within a short timeframe — the classic pattern of a brute-force attack.

> **Answer:** `_______`

---

## 🗺️ MITRE ATT&CK Mapping

| Field | Detail |
|---|---|
| **Tactic** | Credential Access (TA0006) |
| **Technique** | T1110 — Brute Force |
| **Sub-technique** | T1110.001 — Password Guessing |
| **Defense** | Log all authentication events, alert on repeated failures, implement account lockout |

---

## 🧠 What I Learned

### Technical Skills
- How logging failures (OWASP A09) directly impact the ability to detect and investigate attacks
- How to identify brute-force patterns in application logs — repeated failed attempts from a single IP
- The IAAA model and how each component (Identification, Authentication, Authorization, Accountability) can fail
- Why log integrity matters — logs stored where attackers can reach them are worthless as evidence

### Analyst Mindset
- No logs = no visibility = no detection. Logging is not optional — it's the foundation of incident response
- A brute-force attack is trivial to detect **if** authentication events are properly logged
- Always ask: if a key piece of this log was missing, could I still reconstruct what happened?
- Log retention, alerting thresholds, and tamper protection are as important as the logs themselves

---

## 💬 Honest Self-Assessment

**What went well:**
Connecting OWASP A09 to a real investigation scenario made the concept concrete. Identifying the brute-force pattern in the logs — multiple failed attempts from the same IP — felt intuitive once I knew what to look for.

**What I need to improve:**
Building faster pattern recognition in raw logs. In a real SOC, logs come in high volume and without a clean UI — developing the instinct to spot anomalies quickly in unformatted data is the next step.

---

## 📁 Repository Structure

```
IAAA-Failures-Logging-Alerting/
├── README.md          # Lab documentation
└── Screenshots/       # Evidence screenshots from the lab
```

---

## 🖼️ Screenshots

> Screenshots of the log investigation and brute-force analysis will be available in the [`Screenshots`](./Screenshots/) folder.

---

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
