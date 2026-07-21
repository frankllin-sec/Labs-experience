# 🕵️ Man-in-the-Middle Detection — TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Network%20Forensics-blue?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-green?style=for-the-badge"/>
</p>

---

## 📌 About This Lab

A routine network monitoring alert at **Acme Corp** revealed unusual traffic patterns suggesting a possible **Man-in-the-Middle (MITM)** attack inside the corporate LAN. Over several days, an attacker quietly intercepted communications, redirected connections, and captured user credentials.

As a SOC Analyst, the task was to investigate a packet capture (`network_traffic.pcap`) and uncover evidence of **three chained MITM techniques:**

- **ARP Spoofing** — inetwork interception by poisoning ARP cache
- **DNS Spoofing** �@ traffic redirection via forged DNS responses
- **SSL Stripping** ℠�TLS downgrade leading to credential capture in plaintext

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Man-in-the-Middle-Detection/Screenshots/minicio.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Man-in-the-Middle-Detection/Screenshots/minicio.jpg" width="600"/>
  </a>
</p>

---

## 🔍 Investigation — Part 1: ARP Spoofing Detection

### Scenario

Starting by isolating ARP traffic in Wireshark and applying filters to identify abnormal volume, duplicate IP-to-MAC mappings, and spoofed gateway ARP replies.

---

### ❓ Q1 — How many ARP packets from the gateway MAC Address were observed?

**Method:** Applied the filter to isolate ARP traffic from the legitimate gateway IP and MAC address:

```
arp && arp.src.proto_ipv4 == 192.168.10.1 && eth.src == 02:aa:bb:cc:00:01
```

> **Answer:** `10`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Man-in-the-Middle-Detection/Screenshots/m1.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Man-in-the-Middle-Detection/Screenshots/m1.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q2 — What MAC address was used by the attacker to impersonate the gateway?

**Method:** Filtered ARP replies for the gateway IP and identified the spoofed MAC address in the source field — different from the legitimate gateway MAC `02:aa:bb:cc:00:01`.

> **Answer:** `02:fe:fe:fe:55:55`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Man-in-the-Middle-Detection/Screenshots/m2.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Man-in-the-Middle-Detection/Screenshots/m2.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q3 — How many Gratuitous ARP replies were observed for 192.168.10.1?

**Method:** Used the same ARP filter and counted the Gratuitous ARP reply packets — unsolicited ARP replies sent to poison the network's ARP cache.

```
arp && arp.src.proto_ipv4 == 192.168.10.1 && eth.src == 02:aa:bb:cc:00:01
```

> **Answer:** `2`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Man-in-the-Middle-Detection/Screenshots/m3.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Man-in-the-Middle-Detection/Screenshots/m3.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q4 — How many unique MAC addresses claimed the same IP (192.168.10.1)?

**Method:** Filtered all ARP replies for the gateway IP — both the legitimate MAC and the attacker's spoofed MAC appeared:

```
arp.opcode == 2 && arp.src.proto_ipv4 == 192.168.10.1
```

> **Answer:** `2`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Man-in-the-Middle-Detection/Screenshots/m4.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Man-in-the-Middle-Detection/Screenshots/m4.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q5 — How many ARP spoofing packets were observed in total from the attacker?

**Method:** Applied a more specific filter targeting only the attacker's MAC address to isolate spoofed packets:

```
arp.opcode == 2 && arp.src.proto_ipv4 == 192.168.10.1 && eth.src == 02:fe:fe:fe:55:55
```

> **Answer:** `14`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Man-in-the-Middle-Detection/Screenshots/m5.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Man-in-the-Middle-Detection/Screenshots/m5.jpg" width="600"/>
  </a>
</p>

---

## 🔍 Investigation — Part 2: DNS Spoofing Detection

### Scenario

After successfully poisoning the ARP table and positioning between the victim and the gateway, the attacker forged DNS responses to redirect the victim's DNS queries to a malicious IP.

---

### ❓ Q6 — How many DNS responses were observed for corp-login.acme-corp.local?

**Method:** Filtered all DNS responses for the target domain to see the total volume of responses — both legitimate and spoofed:

```
dns.flags.response == 1 && dns.qry.name == "corp-login.acme-corp.local"
```

> **Answer:** `211`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Man-in-the-Middle-Detection/Screenshots/m6.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Man-in-the-Middle-Detection/Screenshots/m6.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q7 — How many DNS requests were observed from IPs other than 8.8.8.8?

**Method:** Modified the filter to exclude the legitimate DOS server (`8.8.8.8`) and identify responses coming from unexpected sources — revealing the rogue DNS server:

```
dns.flags.response == 1 && ip.src != 8.8.8.8 && dns.qry.name == "corp-login.acme-corp.local"
```

> **Answer:** `2`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Man-in-the-Middle-Detection/Screenshots/m7.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Man-in-the-Middle-Detection/Screenshots/m7.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q8 — What IP did the attacker's forged DNS response return for the domain?

**Method:** Inspected the forged DNS response packets from the rogue server. The Answer section of the spoofed DNS reply revealed the malicious IP the victim was redirected to.

> **Answer:** `192.168.10.55`

---

## 🔍 Investigation — Part 3: SSL Stripping Detection

### Scenario

With the victim redirected to the attacker's IP via forged DNS, the attacker downgraded the victim's HTTPS connection to plain HTTP — allowing credentials to be captured in cleartext.

---

### ❓ Q9 — How many POST requests were observed for corp-login.acme-corp.local?

**Method:** Filtered HTTP traffic between the victim and the attacker's server. Identified the POST request containing the login form submission over plain HTTP.

```
http && ip.src == 192.168.10.10 && ip.dst == 192.168.10.55
```

> **Answer:** `1`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Man-in-the-Middle-Detection/Screenshots/m8.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Man-in-the-Middle-Detection/Screenshots/m8.jpg" width="600"/>
  </a>
</p>

---

### ❓ Q10 — What is the victim's password found in the plaintext after SSL stripping?

**Method:** Right-clicked on the POST packet → **Follow → HTTP Stream**. The HTTP stream revealed the form data in cleartext, including the username and password submitted by the victim.

> **Answer:** `Secret123!`

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Man-in-the-Middle-Detection/Screenshots/m9.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Man-in-the-Middle-Detection/Screenshots/m9.jpg" width="600"/>
  </a>
</p>

---

## 🧠 What I Learned

### Technical Skills
- How to detect **ARP Spoofing** using Wireshark filters — identifying duplicate IP-to-MAC mappings and gratuitous ARP replies from unexpected MAC addresses
- How **DNS Spoofing** works as a second stage of MITM — forging DNS responses to redirect victims to attacker-controlled servers
- How **SSL Stripping** downgrades HTTPS to HTTP — enabling cleartext credential capture via HTTP stream analysis
- How to use **Follow → HTTP Stream** in Wireshark to reconstruct and read form submissions in plaintext
- How the three attack stages chain together: ARP poisoning → DNS redirection → SSL downgrade → credential theft

### Analyst Mindset
- Gratuitous ARP replies that don't match the expected gateway MAC are an immediate red flag — this is how ARP poisoning begins
- DNS responses coming from IPs other than the configured DNS server are always suspicious — a legitimate response should only come from `8.8.8.8`
- Credentials transmitted over HTTP instead of HTTPS on a corporate login page indicate SSL stripping — even if the user thinks they(re on a secure connection
- The attack chain is only visible when you correlate all three layers — ARP, DNS, and HTTP ☠ across the same packet capture

---

## 💬 Honest Self-Assessment

**What went well:**
Following the filters step by step and seeing each attack layer unfold in Wireshark was very clear. Connecting the three chained techniques — ARP → DNS → SSL  unto a single attack narrative made the investigation feel like real incident response work.

**What I need to improve:**
Memorizing Wireshark filters from scratch is still a challenge. Right now I rely heavily on references and the lab guidance to build the queries. In a real SOC environment I would likely have a playbook or cheat sheet to reference — and I believe that with experience and repetition, these filters will become second nature over time.

---

## 📁 Repository Structure

```
Man-in-the-Middle-Detection/
├── README.md
└── Screenshots/
```

---

<p align="center">
  <a href="https://github.com/frankllin-sec/Labs-experience/blob/main/Man-in-the-Middle-Detection/Screenshots/mfinal.jpg">
    <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Man-in-the-Middle-Detection/Screenshots/mfinal.jpg" width="600"/>
  </a>
</p>

---

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
<p align="center"><a href="https://github.com/frankllin-sec">🔗 Visit my GitHub Profile</a></p>
