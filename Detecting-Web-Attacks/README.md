# 🌐 Detecting Web Attacks: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Web%20Security%20Monitoring-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Detecting-Web-Attacks/Screenshots/dtinicio.jpg" width="700"/>
</p>

---

## 📌 About This Lab

This room builds on [Web Security Essentials](https://github.com/frankllin-sec/Labs-experience/blob/main/Web-Security-Essentials/README.md) by moving from "what controls exist" to "how do you actually catch an attack happening." It covers the client-side vs server-side attack split, then walks through detecting a real breach scenario (TryBankMe) using access logs and a packet capture, and closes with how Web Application Firewalls fit into the picture.

**Objectives:**
- Learn common client-side and server-side attack types
- Understand the benefits and limitations of log-based detection
- Explore network traffic-based detection methods
- Understand how and why Web Application Firewalls are used
- Practice identifying common web attacks using the methods covered

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **Client-Side Attack** | Abuses the user's browser or behavior; largely invisible to SOC tools like server logs or network captures |
| **XSS (Cross-Site Scripting)** | The most common client-side attack: malicious scripts run inside a trusted site and execute in the victim's browser |
| **CSRF (Cross-Site Request Forgery)** | Tricks the victim's browser into sending unauthorized requests while they're authenticated |
| **Clickjacking** | Invisible elements overlaid on legitimate content trick users into interacting with something they didn't intend to |
| **Server-Side Attack** | Exploits the web server, application code, or backend directly, and leaves evidence in logs and network traffic |
| **Brute-Force** | Repeated login attempts against a form until valid credentials are found |
| **SQL Injection (SQLi)** | Malicious input alters a dynamically built SQL query, letting an attacker read or manipulate the database |
| **WAF (Web Application Firewall)** | Inspects and filters requests before they reach the server, using rules, IP reputation, and challenge-response mechanisms like CAPTCHA |

---

## 🔍 Investigation

### Part 1: Client-Side Attacks

**Q: What class of attacks relies on exploiting the user's behavior or device?**
> **Answer:** `Client-Side`

**Q: What is the most common client-side attack?**
> **Answer:** `XSS`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Detecting-Web-Attacks/Screenshots/dt1.jpg" width="700"/>
</p>

---

### Part 2: Server-Side Attacks

**Q: What class of attacks relies on exploiting vulnerabilities within web servers?**
> **Answer:** `Server-Side`

**Q: Which server-side attack lets attackers abuse forms to dump database contents?**
> **Answer:** `SQLi`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Detecting-Web-Attacks/Screenshots/dt2.jpg" width="700"/>
</p>

---

### Part 3: Detecting Attacks in Logs

**Scenario:** TryBankMe, a small online banking platform, suffered a breach and leaked customer data. Management believes the intrusion started through the public website. The mission: analyze `access.log` and retrace the attacker's steps.

**Q: What is the attacker's User-Agent while performing the directory fuzz?**
> **Answer:** `FFUF v2.1.0`

**Q: What is the name of the page on which the attacker performs a brute-force attack?**

**Method:** Spotted repeated POST requests to the same page in quick succession, ending in a 302 redirect that signaled a successful login.

> **Answer:** `/login.php`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Detecting-Web-Attacks/Screenshots/dt3.jpg" width="700"/>
</p>

**Q: What is the complete, decoded SQLi payload the attacker uses on the /changeusername.php form?**

**Method:** Copied the URL-encoded query string from the log and ran it through CyberChef's URL Decode operation.

> **Answer:** `%' OR '1'='1`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Detecting-Web-Attacks/Screenshots/dt4.jpg" width="700"/>
</p>

---

### Part 4: Detecting Attacks in Network Traffic

**Scenario:** The log analysis showed evidence of the attack, but not which user was breached or what data was actually stolen. Continued the investigation using `traffic.pcap`.

**Q: What password does the attacker successfully identify in the brute-force attack?**

**Method:** Filtered Wireshark for POST requests to the login form and followed the HTTP stream on the successful attempt to read the submitted credentials in clear text.

> **Answer:** `astrongpassword123`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Detecting-Web-Attacks/Screenshots/dt5.jpg" width="700"/>
</p>

**Q: What is the flag the attacker found in the database using SQLi?**

**Method:** Built a filter to catch structural SQL characters: `http contains "%27" or http contains "'" or http contains "--" or http contains "%23"`. This caught a GET request with an encoded suspicious payload in the URL. Followed the HTTP stream to reconstruct the full request and response, confirming the SQL injection and the dumped table contents.

> **Answer:** `THM{dumped_the_db}`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Detecting-Web-Attacks/Screenshots/dt6.jpg" width="700"/>
</p>

---

### Part 5: Web Application Firewalls

**Q: What do WAFs inspect and filter?**
> **Answer:** `Web Requests`

**Q: Create a custom firewall rule to block any User-Agent that matches "BotTHM".**
> **Answer:** `IF User-Agent CONTAINS "BotTHM" THEN block`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Detecting-Web-Attacks/Screenshots/dt7.jpg" width="700"/>
</p>

---

## 🧠 What I Learned

### Technical Skills
- How to distinguish client-side attacks (invisible to server-side tooling) from server-side attacks (which leave a trail in logs and network traffic)
- How to retrace a full attack chain in access logs: directory fuzzing, then brute-force, then SQL injection, by reading User-Agent strings, status codes, and request timing
- How to decode a URL-encoded SQLi payload with CyberChef instead of trying to read it by eye
- How to build a targeted Wireshark filter for SQL injection indicators (`%27`, `'`, `--`, `%23`) when I don't already know the exact payload, and follow the HTTP stream to confirm it
- Why access logs alone weren't enough here: the actual submitted credentials and the stolen data only became visible once I moved to the packet capture
- How WAF rules are structured (block, deny by reputation, custom, rate-limit) and how a challenge-response approach (CAPTCHA) fits in when blocking outright risks false positives

### Analyst Mindset
- Client-side attacks are a genuine blind spot for a SOC without endpoint or browser-side monitoring. That gap is worth calling out explicitly rather than assuming logs will catch everything
- Logs and packet captures are complementary, not redundant. Logs are quicker to search but often miss POST bodies and full payloads. Packet captures are more complete but heavier to review, so knowing when to escalate from one to the other saves time
- Following the actual sequence of an attack (recon, then brute-force, then exploitation) rather than jumping straight to the flag helped build a mental model of how a real breach unfolds end to end

---

## 💬 Honest Self-Assessment

**What went well:**
Reconstructing the full attack chain across both the log file and the packet capture felt natural, and building a custom Wireshark filter for SQLi indicators instead of relying on a pre-made one was a good stretch.

**What I need to improve:**
Getting faster at recognizing common attack tool signatures (like `sqlmap` or `FFUF`) in User-Agent strings on sight, without needing to search them. Building a personal reference list of common attack tool fingerprints would speed up future log reviews.

---

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Detecting-Web-Attacks/Screenshots/dtfinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
