# 🔥 Firewall Fundamentals — TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Firewall%20Analysis-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

---

## 📌 About This Lab

This lab covers the fundamentals of **firewalls** — one of the most essential security controls in any network. Just like a security guard controlling who enters and exits a building, a firewall monitors and controls incoming and outgoing network traffic based on pre-configured rules.

The lab includes hands-on work with **Windows Defender Firewall**, where the task was to analyze existing rules created by a security team to block and allow specific network traffic on a critical Windows system.

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **Firewall** | A network security control that monitors and filters traffic based on rules |
| **Inbound Rule** | Controls traffic coming INTO the system |
| **Outbound Rule** | Controls traffic going OUT of the system |
| **Allow Rule** | Permits specific traffic matching defined criteria |
| **Block Rule** | Denies specific traffic matching defined criteria |
| **SSH (Port 22)** | Secure Shell — remote access protocol, common attack target |

---

## 🔍 Investigation — Windows Defender Firewall Analysis

### Scenario

The security team noticed suspicious incoming and outgoing traffic on a critical Windows system. They created rules on **Windows Defender Firewall** to block and allow specific network traffic. The task was to analyze those rules and extract key information.

**Method:** Opened Windows Defender Firewall → Advanced Settings → Inbound Rules. Filtered and analyzed existing rules to identify blocks and exceptions on port 22 (SSH).

---

### ❓ Q1 — What is the name of the rule that was created to block all incoming traffic on the SSH port?

**SSH Port:** `22`

**Method:** Navigated to Inbound Rules in Windows Defender Firewall Advanced Settings. Filtered rules by port `22` and looked for entries with **Action = Block** to identify the rule blocking all incoming SSH traffic.

> **Answer:** `Core Op`

---

### ❓ Q2 — A rule was created to allow SSH from one single IP address. What is the rule name?

**Method:** Using the same port `22` filter, switched focus to rules with **Action = Allow** instead of Block. Identified the rule that permits SSH access from a specific IP address.

> **Answer:** `Infra team`

---

### ❓ Q3 — Which IP address is allowed under this rule?

**Method:** Opened the `Infra team` rule details and inspected the **Remote IP Address** field to identify the single IP address permitted to connect via SSH.

> **Answer:** `192.168.13.7`

---

## 🗺️ MITRE ATT&CK Mapping

| Field | Detail |
|---|---|
| **Tactic** | Initial Access (TA0001) |
| **Technique** | T1190 — Exploit Public-Facing Application |
| **Relevance** | SSH (port 22) is a common vector for unauthorized remote access |
| **Defense** | Restrict SSH access via firewall rules — block all, allow only trusted IPs |

---

## 🧠 What I Learned

### Technical Skills
- How to navigate **Windows Defender Firewall Advanced Settings**
- How to filter inbound rules by port to quickly find relevant entries
- The difference between **Block** and **Allow** actions in firewall rules
- How to restrict SSH access to a single trusted IP using firewall rules — a real-world hardening technique

### Analyst Mindset
- Blocking all traffic on a port and then creating an allow exception for a specific IP is the correct **least privilege** approach to remote access
- SSH on port 22 should never be open to all IPs on a production system — always restrict by source IP
- Firewall rules are a first line of defense, but they must be reviewed regularly — misconfigured or overly permissive rules are a common finding in security audits

---

## 💬 Honest Self-Assessment

**What went well:**
The logic of block-all + allow-exception clicked immediately. Filtering by port in the firewall rules made it straightforward to find the relevant entries and extract the answers.

**What I need to improve:**
Getting comfortable with more complex rule sets — real environments have hundreds of firewall rules. Building the instinct to quickly identify anomalies in large rule lists will take more practice.

---

## 📁 Repository Structure

```
Firewall-Fundamentals/
├── README.md          # Lab documentation
└── Screenshots/       # Evidence screenshots from the lab
```

---

## 🖼️ Screenshots

> Screenshots of the Windows Defender Firewall rules and investigation steps will be available in the [`Screenshots`](./Screenshots/) folder.

---

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>