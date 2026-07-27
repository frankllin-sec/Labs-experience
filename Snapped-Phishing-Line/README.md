# 🎣 Snapped Phish-ing Line: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Phishing%20Analysis-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Snapped-Phishing-Line/Screenshots/phinicio.jpg" width="700"/>
</p>

---

## 📌 About This Lab

As a member of the IT department at **SwiftSpend Financial**, I was responsible for assisting employees with technical concerns. What started as a routine day escalated quickly when multiple employees across different departments reported receiving a suspicious email. Several users noticed unusual characteristics in the message, and some had already submitted their credentials and could no longer access their accounts.

With the potential for a wider compromise, the incident was escalated for investigation. The task was to analyze the available evidence, determine the scope of the attack, and uncover how the adversary operated: from the initial phishing email all the way to the phishing kit hosting the fake login page.

**Objectives:**
- Analyze the provided email samples to identify key artifacts
- Investigate phishing URLs to understand redirection
- Retrieve and examine the phishing kit used in the attack
- Use CTI tools to gather intelligence on the adversary
- Analyze the phishing kit to uncover additional indicators

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **Phishing Email** | A fraudulent email crafted to impersonate a legitimate sender and trick the victim into acting |
| **Redirection URL** | A link inside a malicious attachment that redirects the victim to an attacker-controlled site |
| **Phishing Kit** | A packaged set of files (HTML/PHP/assets) used to clone a legitimate login page and harvest credentials |
| **IOC (Indicator of Compromise)** | Evidence such as a domain, hash, or email address that points to malicious activity |
| **VirusTotal** | CTI tool used to check file hashes and URLs against multiple security vendors |
| **CyberChef** | Tool used to decode/encode data: used here to reverse a Base64-encoded flag |

---

## 🔍 Investigation: Phishing Email to Credential Harvesting

### Scenario

Multiple SwiftSpend Financial employees received a phishing email disguised as a **"Quote for Services Rendered"**. The investigation traced the email, its attachment, the redirection URL, and the phishing kit hosted behind it, ultimately exposing captured credentials and the adversary's collection email.

---

### ❓ Q1: Which individual received the email regarding a Quote for Services Rendered?

**Method:** Reviewed the emails in the `phish-emails` folder and located the message with the subject **"Quote for Services Rendered."**

> **Answer:** `William McClean`

### ❓ Q2: What email address was used by the adversary to send the phishing emails?

**Method:** Checked the **From** field of the phishing email, which spoofed a fake marketing company.

> **Answer:** `Accounts.Payable@groupmarketingonline.icu`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Snapped-Phishing-Line/Screenshots/ph1.jpg" width="700"/>
</p>

---

### ❓ Q3: Investigate the attachment in the email addressed to Zoe Duncan. What is the root domain of the redirection URL?

**Method:** Opened the attachment and traced the embedded link to its redirection target.

> **Answer:** `kennaroads.buzz`

### ❓ Q4: Open the attachment in the VM web browser. Which company is the login page impersonating?

**Method:** Opened the redirection URL in the VM's browser and inspected the rendered login page.

> **Answer:** `Microsoft`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Snapped-Phishing-Line/Screenshots/ph2.jpg" width="700"/>
</p>

---

### ❓ Q5: Navigate to the /data directory. What is the name of the archive file?

**Method:** The attacker had left directory listing enabled on `kennaroads.buzz`. Browsing to `/data` exposed the phishing kit archive.

> **Answer:** `Update365.zip`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Snapped-Phishing-Line/Screenshots/ph3.jpg" width="700"/>
</p>

---

### ❓ Q6: Download the phishing kit archive. Using sha256sum, what is the SHA256 hash?

**Method:** Since the VM had no internet access, the hash had to be generated locally. Downloaded `Update365.zip` and ran `sha256sum` at the Linux terminal.

> **Answer:** `ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Snapped-Phishing-Line/Screenshots/ph4.jpg" width="700"/>
</p>

---

### ❓ Q7: Investigate the file hash on VirusTotal. Aside from phishing, what other threat category is assigned to the ZIP archive?

**Method:** Submitted the SHA256 hash to VirusTotal and reviewed the assigned threat categories.

> **Answer:** `Trojan`

### ❓ Q8: Review the VirusTotal Details page. How many files are contained within the archive?

**Method:** Checked the **Contents Metadata** section on the VirusTotal Details tab.

> **Answer:** `49`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Snapped-Phishing-Line/Screenshots/ph5.jpg" width="700"/>
</p>

---

### ❓ Q9: Navigate to /data/Update365/ and investigate the log file. What email address submitted credentials more than once?

**Method:** Opened `log.txt` in the exposed `Update365` directory and reviewed every captured login attempt.

> **Answer:** `michael.ascot@swiftspend.finance` (submitted credentials 3 times)

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Snapped-Phishing-Line/Screenshots/ph6.jpg" width="700"/>
</p>

---

### ❓ Q10: Extract the phishing kit archive and locate submit.php. What email address is used by the adversary to collect compromised credentials?

**Method:** Extracted the kit and used `grep mail` inside the `submit.php` source to quickly surface every line referencing an email variable, which revealed the collection address.

> **Answer:** `m3npat@yandex.com`

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Snapped-Phishing-Line/Screenshots/ph7.jpg" width="700"/>
</p>

---

### ❓ Q11: Return to the phishing URL and locate flag.txt. Using CyberChef to decode it, what is the secret value?

**Method:** The `flag.txt` file wasn't linked anywhere: it had to be requested directly by appending its filename to the end of the `office365` folder's address to reach the hidden file. That returned an encoded string.

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Snapped-Phishing-Line/Screenshots/ph8.jpg" width="700"/>
</p>

In CyberChef, running the **Magic** function identified the encoding as Base64, but the decoded output came out reversed. Swapping Magic for the **From Base64** operation followed by **Reverse** produced the readable flag.

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Snapped-Phishing-Line/Screenshots/ph9.jpg" width="700"/>
</p>

> **Answer:** `THM{pL4y_w1Th_tH3_URL}`

---

## 🧠 What I Learned

### Technical Skills
- How to trace a phishing email from a spoofed sender through an attachment to its redirection domain
- How to safely investigate an exposed attacker web server (directory listing) to pull down a phishing kit
- How to hash a file locally with `sha256sum` and pivot that hash into VirusTotal for CTI enrichment
- How to read a phishing kit's `submit.php` source with `grep` to find the credential-exfiltration email
- How to decode a Base64 + reversed string in CyberChef by building the recipe manually instead of relying on Magic

### Analyst Mindset
- A phishing kit left exposed on the attacker's own server is a goldmine of IOCs: captured credentials, exfil email, and kit metadata all in one place
- Always correlate the **sender domain + redirection domain + hosting server** to fully scope a phishing campaign
- A victim who submits credentials more than once is a strong signal they later re-entered them believing the page was legitimate: worth flagging for priority containment and password reset
- CyberChef's Magic function is a great starting point, but building the recipe manually is sometimes the only way to fully unpack multi-layer encoding

---

## 💬 Honest Self-Assessment

**What went well:**
Pivoting from the email to the attachment to the exposed `/data` directory felt like a natural, logical chain: each step revealed the next artifact needed to keep moving forward.

**What I need to improve:**
Getting faster at reading phishing kit PHP source without a walkthrough: spotting the right file (`submit.php`) and the right grep pattern took a bit of trial and error. More repetition with real phishing kits will build that instinct.

---

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Snapped-Phishing-Line/Screenshots/phfinal.jpg" width="700"/>
</p>

<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
