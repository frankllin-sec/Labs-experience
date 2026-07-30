# 🌐 Web Security Essentials: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Web%20Security-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Web-Security-Essentials/Screenshots/weinicio.jpg" width="700"/>
</p>

---

## 📌 About This Lab

This room steps back from log analysis to cover the security fundamentals behind the sites and services a SOC analyst is defending. It walks through why web applications are such attractive targets, how web infrastructure is put together, and then puts that knowledge into practice on a fictional site called **Secure-A-Site**, hardening it across three layers: the web application, the web server, and the host machine.

**Objectives:**
- Understand the shift from desktop applications to web applications
- Learn why web applications are common targets for attackers
- Explore web infrastructure and the tools used to protect the web
- Practice applying security measures to harden a new web application

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **Access Control** | Restricting users so they only see or change data relevant to their role |
| **Input Validation & Sanitization** | Cleaning and checking user-submitted data before it's used, to block code injection |
| **Secure Coding** | Reviewing application code for vulnerabilities early in development |
| **Web Application Firewall (WAF)** | A protective barrier that filters traffic and blocks requests matching malicious patterns |
| **Access Logging** | Maintaining a log of requests so anomalies can be investigated later |
| **CDN (Content Delivery Network)** | Serving cached content from edge servers to cut latency and shield the origin server from direct traffic |
| **Antivirus** | Endpoint-level protection that detects and blocks known malware |
| **Least Privilege** | Running services under dedicated low-privilege accounts instead of admin rights |
| **System Hardening** | Ensuring only what's actually needed is running and open on a host or server |

---

## 🔍 Securing Secure-A-Site

### Part 1: Web Application Security

**Q: An employee can see the admin dashboard. What security measure helps stop this?**
> **Answer:** `Access Control`: restrict users so they only see appropriate data.

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Web-Security-Essentials/Screenshots/we1.jpg" width="700"/>
</p>

**Q: Attackers can inject code into your login form. How do you block it?**
> **Answer:** `Input Validation & Sanitization`: clean and check any user-submitted data before using it.

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Web-Security-Essentials/Screenshots/we2.jpg" width="700"/>
</p>

**Q: Your app leaks detailed errors when it crashes. What should developers do early in development to secure it?**
> **Answer:** `Secure Coding`: review the app for vulnerabilities in code.

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Web-Security-Essentials/Screenshots/we3.jpg" width="700"/>
</p>

**Q: What flag did you receive for securing the Web Application?**
> **Answer:** `THM{web_app_secured!}`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Web-Security-Essentials/Screenshots/we4.jpg" width="700"/>
</p>

---

### Part 2: Web Server Security

**Q: When configuring your web server, what should you enable so unusual traffic patterns can be investigated later?**
> **Answer:** `Access Logging`: maintain an access log to spot anomalies and support incident response.

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Web-Security-Essentials/Screenshots/we5.jpg" width="700"/>
</p>

**Q: Which security measure helps ensure malicious requests never reach your server?**
> **Answer:** `Web Application Firewall`: set up a protective barrier between users and the web server.

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Web-Security-Essentials/Screenshots/we6.jpg" width="700"/>
</p>

**Q: How can you reduce your server's exposure while also speeding up content delivery?**
> **Answer:** `Content Delivery Network (CDN)`: serve cached content from edge servers to cut latency and improve security.

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Web-Security-Essentials/Screenshots/we7.jpg" width="700"/>
</p>

**Q: What flag did you receive for securing the Web Server?**
> **Answer:** `THM{server_security_expert!}`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Web-Security-Essentials/Screenshots/we8.jpg" width="700"/>
</p>

---

### Part 3: Host Machine Security

**Q: How can you protect your endpoint from harmful or unauthorized software?**
> **Answer:** `Antivirus`: detects and blocks known malware.

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Web-Security-Essentials/Screenshots/we9.jpg" width="700"/>
</p>

**Q: Your web server runs with admin rights. What safer setup should you use?**
> **Answer:** `Least Privilege`: use dedicated low-privilege accounts to run your site.

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Web-Security-Essentials/Screenshots/we10.jpg" width="700"/>
</p>

**Q: Unused ports are open and outdated services are still running. How do you reduce the risk?**
> **Answer:** `System Hardening`: ensure only what you need is running and open.

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Web-Security-Essentials/Screenshots/we11.jpg" width="700"/>
</p>

**Q: What flag did you receive for securing the Host Machine?**
> **Answer:** `THM{the_final_security_layer!}`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Web-Security-Essentials/Screenshots/we12.jpg" width="700"/>
</p>

---

## 🧠 What I Learned

### Technical Skills
- How to map a specific security weakness (visible admin dashboard, injectable login form, verbose error messages, unpatched exposure) to the exact control that fixes it, rather than a generic "add security" answer
- How defense in depth applies across three distinct layers of a deployment: the application code itself, the server serving it, and the host machine underneath both
- Why a CDN is a security control and not just a performance one: shielding the origin server's real IP from direct traffic reduces its attack surface

### Analyst Mindset
- Each layer (app, server, host) has its own failure modes and its own fix. A WAF doesn't replace secure coding, and antivirus doesn't replace least privilege. Real hardening means applying the right control at the right layer instead of over-relying on one
- Access logging showed up as the answer for both the web app and web server layers, which is a good reminder that visibility (logs) is foundational to nearly every other detection or investigation, not just a nice-to-have
- Least privilege for service accounts matters just as much as least privilege for users. A web server compromised while running as admin hands the attacker a much bigger blast radius than the same compromise under a dedicated low-privilege account

---

## 💬 Honest Self-Assessment

**What went well:**
Recognizing which control belonged to which scenario felt intuitive by the Host Machine section, since the pattern (identify the gap, pick the matching defensive control) repeats consistently across all three layers.

**What I need to improve:**
Going a level deeper on implementation. This room is great for mapping symptom to control name, but I want to follow up by actually configuring a WAF rule set or a CDN cache policy hands-on, rather than just knowing the term.

---

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Web-Security-Essentials/Screenshots/wefinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
