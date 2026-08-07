# 🛡️ Detecting Web DDoS: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Web%20Security%20Monitoring-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Detecting-Web-DDoS/Screenshots/dinicio.jpg" width="700"/>
</p>

---

## 📌 About This Lab

This room covers Denial-of-Service (DoS) and Distributed Denial-of-Service (DDoS) attacks against the application layer (Layer 7). It walks through how attackers overwhelm a web service so legitimate users cannot use it, why they do it, and how a defender can spot the attack in raw web logs and in Splunk.

**Objectives:**
- Learn how denial-of-service attacks function
- Understand attacker motives behind the disruptive attacks
- See how web logs can help reveal signs of web DoS and DDoS
- Practice analyzing denial-of-service attacks through log analysis
- Discover detection and mitigation techniques defenders can use

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **DoS vs DDoS** | A DoS attack comes from a single machine and is capped by its own resources. A DDoS attack uses a botnet, an army of compromised devices, to flood the target from many sources at once |
| **Slowloris** | Sending many partial HTTP requests to tie up server resources |
| **HTTP Flood** | Sending a large number of HTTP requests to overwhelm the server |
| **Oversized Query / Logic Abuse** | Forcing the server to process large, resource-intensive requests (e.g. `GET /products?limit=999999`) |
| **Targeted Endpoints** | Resource-heavy pages like `/login`, `/search`, `/api`, `/register`, or `/checkout` are prime targets since each request triggers database queries, validation, or session handling |

---

## 🔍 Investigation

### Part 1: Access Log Analysis (DoS)

**Scenario:** A bicycle parts e-commerce site went down. The `access.log` file on the Desktop mixes normal user traffic with attacker traffic.

**Q: What is the attacker's IP address?**

**Method:** Scanned the log for a single IP repeating the same request far more often than any normal user would.

> **Answer:** `203.12.23.195`

**Q: Which page is repeatedly targeted by the attacker's requests?**

> **Answer:** `/login`

**Q: After the attack, what error code do legitimate users receive?**

**Method:** Once the attacker's flood exhausted server resources, subsequent legitimate requests started returning server errors.

> **Answer:** `503`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Detecting-Web-DDoS/Screenshots/d1.jpg" width="700"/>
</p>

---

### Part 2: Splunk Investigation (DDoS)

**Scenario:** The same website suffered a suspected DDoS. This time the investigation moved to Splunk, searching `index="main"` over the attack window.

**Q: What was the most frequently requested uri?**

**Method:** Filtered through the `uri_path` field and checked the top values report.

> **Answer:** `/search`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Detecting-Web-DDoS/Screenshots/d2.jpg" width="700"/>
</p>

**Q: Which clientip made the most requests to the target uri?**

**Method:** Selected the `clientip` field and sorted by top values against the `/search` requests.

> **Answer:** `203.0.113.7`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Detecting-Web-DDoS/Screenshots/d3.jpg" width="700"/>
</p>

**Q: How many IP addresses were part of the botnet that attacked the website?**

**Method:** Checked how many distinct values the `clientip` field returned across the attack traffic.

> **Answer:** `60`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Detecting-Web-DDoS/Screenshots/d4.jpg" width="700"/>
</p>

**Q: Which useragent was most commonly used by the attacking traffic?**

**Method:** Filtered through the `useragent` field and checked the top values report.

> **Answer:** `Java/1.8.0_181`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Detecting-Web-DDoS/Screenshots/d5.jpg" width="700"/>
</p>

**Q: Using the `timechart` command, what is the peak number of requests made per second during the attack?**

**Method:** Ran `index="main" | timechart span=1s count by url limit=1`. The room's suggested `span=5m` was narrowed down to `1s` to catch the actual burst.

> **Answer:** `207`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Detecting-Web-DDoS/Screenshots/d6.jpg" width="700"/>
</p>

**Q: Which legitimate (non-attacking) clientip received the first 503 response status post-attack?**

**Method:** Filtered by `status=503`, identified the attack start time (1:12:31 PM), then walked forward through the results to find the first 503 response tied to a legitimate client after that point.

> **Answer:** `10.10.0.27`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Detecting-Web-DDoS/Screenshots/d7.jpg" width="700"/>
</p>

---

### Part 3: Detection and Mitigation Recap

**Q: What type of security challenge blocks bots by asking users to solve a simple puzzle?**

> **Answer:** `Captcha`

**Q: Which CDN feature spreads traffic across multiple servers to prevent overload?**

> **Answer:** `Load-balancing`

---

## 🧠 What I Learned

- How to use Splunk's `uri_path`, `clientip`, and `useragent` fields to isolate attack traffic from normal traffic
- How narrowing a `timechart` span (from 5m down to 1s) changes what the data reveals, coarse spans hide short bursts that matter most in a DDoS
- How to pinpoint the moment defenses recovered by correlating the attack start time with the first legitimate 503 response afterward
- Captcha and load-balancing as practical mitigation controls a SOC can recommend alongside detection

---

## 💬 Honest Self-Assessment

**What went well:**
Moving from raw log analysis to Splunk made the pattern recognition much faster. Pivoting on `clientip`, `uri_path`, and `useragent` to isolate the botnet traffic felt intuitive once the right fields were selected.

**What I need to improve:**
I still don't know most of the SPL filter commands by heart, I haven't memorized them yet, so sometimes it's faster for me to click through the fields on the side panel and drill into top values than to write the search command directly. I want to get more comfortable writing SPL from memory instead of relying on the field sidebar.

---
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Detecting-Web-DDoS/Screenshots/dfinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
