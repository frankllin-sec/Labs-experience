# 🛡️ IP and Domain Threat Intel: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Threat%20Intelligence-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/IP-and-Domain-Threat-Intel/Screenshots/dinicio.jpg" width="700"/>
</p>

---

## 📌 About This Lab

SOC runbooks revolve around one core loop: verify, enrich, decide. IP and domain enrichment is a core skill every L1 analyst needs. This lab pivots on geolocation, ASNs, open-service footprints, and DNS records to decide whether an indicator is malicious, and to pull additional context about the attack behind it.

**Objectives:**
- Enrich domains with WHOIS age, DNS records, and TLS details
- Learn the concepts of ASNs and geolocation for SOC triage
- Spot red-flag services using VirusTotal, Shodan, and Censys
- Detect VPN, proxy, and Tor exit nodes with IP2Proxy and Spur
- Correlate signals across sources instead of trusting one verdict

---

## 🔑 Key Concepts

**DNS records for domain enrichment:**

| Record | What it reveals |
|---|---|
| **A / AAAA** | Maps a domain to its IPv4/IPv6 address, revealing hosting provider or CDN |
| **TXT** | Mail security settings (SPF/DKIM) and tooling; empty or suspicious TXT records are a red flag |

**WHOIS / RDAP:** reveals registrar, registration date, and expiry. Domain age is a strong signal, attack infrastructure rarely stays online more than a few months, so a domain registered days ago is far more suspicious than one registered a decade ago.

**DNS-based attack techniques:**

| Technique | How it works |
|---|---|
| **CDN Abuse** | Routing malicious traffic through Cloudflare, Akamai, or Fastly to hide the real origin server |
| **Typosquatting** | Domains like `tryhakme[.]com` rely on visual similarity to a trusted brand |
| **IDN Attacks** | Substituting Cyrillic or Greek letters that look identical to Latin ones (e.g. `tryhаckme[.]com`). Converting to Punycode (`xn--`) reveals the trick |

**IP enrichment:** AbuseIPDB shows if an IP has been involved in port scans or brute-force attacks; VirusTotal gives overall reputation and community comments. If the IP isn't in a CDN range, even a single detection is worth flagging.

**Autonomous Systems (ASN):**

| ASN Type | Risk Signal | Example |
|---|---|---|
| **Residential** | May indicate VPN usage or a compromised consumer device | AS124888 (Vodafone) |
| **Server Hosting** | Highest risk, commonly used to distribute malware | AS215439 (PLAY2GO) |
| **Cloud / CDN** | Used by both legitimate services and adversaries, needs deeper analysis | AS16509 (Amazon AWS) |

**Geolocation (GeoIP):** useful for spotting anomalous logons (a US-based employee logging in from the Netherlands) and anomalous outbound traffic (a European company's traffic suddenly going to Vietnam).

**Exposed services (Shodan / Censys):** open ports and service banners are the first fingerprint of exposure. Old service versions usually mean vulnerable versions. Censys can also surface non-standard ports Shodan misses.

**TLS certificates:** a self-signed certificate is a strong sign a site is worth investigating. Newly-created or unusually long-lived certificates, and the certificate's Subject field, can point directly to the malware family behind it.

**VPN / Proxy / Tor detection:** tools like IP2Proxy and Spur label whether an IP is a known VPN, proxy, or Tor exit node. Without this, a login that looks geographically normal can hide an attacker behind a VPN wearing the victim's own city as a mask.

**SOC Analyst Workflow:**
- **Investigate the domain:** Does it look legit? What's its reputation? When was it registered? Resolve it to an IP.
- **Investigate the IP:** Is it a CDN range? What's its reputation? Is it a VPN node? What ASN type does it belong to, and does that match the alert context?

---

## 🔍 Investigation

### Part 1: Domain Enrichment, purematrixa[.]com

**Scenario:** On June 1, 2026, the SIEM raises a critical alert pointing to `purematrixa[.]com`.

**Q: Which CDN does the purematrixa[.]com domain use?**

**Method:** Looked up the domain on nslookup.io.

> **Answer:** `Cloudflare`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/IP-and-Domain-Threat-Intel/Screenshots/d1.jpg" width="700"/>
</p>

**Q: According to the report, how old was the domain when SIEM raised an alert?**

> **Answer:** `1 days old`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/IP-and-Domain-Threat-Intel/Screenshots/d2.jpg" width="700"/>
</p>

---

### Part 2: IP Enrichment, 2.58.56.50 (Potential C2)

**Q: What country does the malicious IP resolve to?**

**Method:** Looked up the IP on iplocation.net.

> **Answer:** `Netherlands`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/IP-and-Domain-Threat-Intel/Screenshots/d3.jpg" width="700"/>
</p>

**Q: Looking at VirusTotal comments, what C2 server is hosted behind the IP? What Autonomous System does the IP belong to?**

> **Answer:** `Remcos`, hosted under AS `1337 Services GmbH`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/IP-and-Domain-Threat-Intel/Screenshots/d4.jpg" width="700"/>
</p>

**Q: What two tags does BGP.Tools attribute to the ASN?**

> **Answer:** `Server Hosting, Tor Services`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/IP-and-Domain-Threat-Intel/Screenshots/d5.jpg" width="700"/>
</p>

---

### Part 3: Services Exposure, 64.89.160.44

**Scenario:** The IP is already confirmed malicious. The CTI team wants more context on its role.

**Q: What remote access service is exposed? How many ports have been identified as open on the server?**

**Method:** Reviewed the Censys host report for the IP.

> **Answer:** `RDP`, `9` open ports

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/IP-and-Domain-Threat-Intel/Screenshots/d6.jpg" width="700"/>
</p>

**Q: One of the exposed services leaks an active C2 server. What is the name of that C2?**

**Method:** Found a self-signed TLS certificate on an unusual port with the Subject DN naming the malware family directly.

> **Answer:** `AsyncRAT`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/IP-and-Domain-Threat-Intel/Screenshots/d7.jpg" width="700"/>
</p>

**Q: For how many days is the C2 server's certificate valid?**

**Method:** A second self-signed certificate on the same host, this one naming DcRat, gave the exact validity window in Censys.

> **Answer:** `3935` days

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/IP-and-Domain-Threat-Intel/Screenshots/d8.jpg" width="700"/>
</p>

---

### Part 4: VPN Detection, Triage Decisions

**Scenario:** A London-based user logs in from a London IP address, and the IP is confirmed to belong to a Mullvad VPN provider.

**Q: Would you raise the alarm and prioritize the alert?**

> **Answer:** `Yea`, since VPN use can mask an attacker's real location behind a stolen or legitimate-looking login.

**Q: Same scenario, but the IP doesn't match any VPN providers. Would you close the alert as a False Positive?**

> **Answer:** `Nay`, not matching a known VPN provider doesn't rule out a compromised account, many breaches start from a stolen VPN login.

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/IP-and-Domain-Threat-Intel/Screenshots/d9.jpg" width="700"/>
</p>

---

### Part 5: Challenge, raytracingengine[.]com (APT Infrastructure)

**Scenario:** An APT group struck the company. As an L1 analyst, the task is to gather everything possible on the malicious domain `raytracingengine[.]com` and identify the infrastructure the malware relies on.

**Q: What IP does the domain resolve to?**

**Method:** Checked VirusTotal's Relations tab for the domain's passive DNS resolution history.

> **Answer:** `35.188.105.97`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/IP-and-Domain-Threat-Intel/Screenshots/d10.jpg" width="700"/>
</p>

**Q: What cloud provider did the attacker use? What country is the malicious server located in?**

**Method:** Looked up the resolved IP on Shodan.

> **Answer:** `Google Cloud`, located in the `United States`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/IP-and-Domain-Threat-Intel/Screenshots/d11.jpg" width="700"/>
</p>

**Q: When was the malicious domain name created?**

**Method:** Checked whois.domaintools.com for the domain's creation date.

> **Answer:** `21.02.2026`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/IP-and-Domain-Threat-Intel/Screenshots/d12.jpg" width="700"/>
</p>

**Q: According to the exposed service, what is the attack server's OS?**

**Method:** Shodan listed the specific distribution (Ubuntu) under Web Technologies. My first answer was `ubuntu`, which was marked incorrect, the room wanted the general OS family instead.

> **Answer:** `Linux`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/IP-and-Domain-Threat-Intel/Screenshots/d13.jpg" width="700"/>
</p>

---

## 🧠 What I Learned

- How to read A/AAAA and TXT records for hosting and mail-security context, and why empty or odd TXT records are worth a second look
- Why domain age from WHOIS is one of the fastest ways to gauge risk, a domain that's one day old is a very different signal than one that's ten years old
- How to classify an ASN as residential, server hosting, or cloud/CDN, and what risk level each implies
- How to pivot from a flagged IP through VirusTotal community comments, BGP.Tools tags, Shodan, and Censys to build a full picture of the infrastructure behind it
- Why a self-signed TLS certificate's Subject DN can name the exact malware family running behind a port
- Why VPN match or non-match cuts both ways in triage: a confirmed VPN raises the alert, but a non-match doesn't automatically clear it, since credential theft doesn't require a VPN at all
- The difference between a specific OS distribution (Ubuntu) and the general OS family (Linux) the room expected as the answer

---

## 💬 Honest Self-Assessment

**What went well:**
Pivoting across tools felt natural by the end, domain to WHOIS to resolved IP to Shodan/Censys to TLS certificate is a workflow I could repeat without re-reading the task instructions. Spotting the AsyncRAT and DcRat certificates by their Subject DN was a genuinely satisfying find.

**What I need to improve:**
I answered "ubuntu" for the attack server's OS question when the room wanted the broader category "Linux." I need to pay closer attention to how a question is phrased, specific distribution versus general OS family aren't interchangeable, and assuming the more specific answer is always better cost me an attempt here.

---
<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/IP-and-Domain-Threat-Intel/Screenshots/dfinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
