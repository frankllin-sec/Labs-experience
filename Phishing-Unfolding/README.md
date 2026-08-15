# 🛡️ Phishing Unfolding: TryHackMe Lab

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Type-Alert%20Triage-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Role-SOC%20Analyst%20Tier%201-blue?style=for-the-badge"/>
</p>

---

## 📌 About This Lab

A live phishing attack unfolds in real time inside a simulated corporate network. Unlike a static walkthrough, alerts arrive as the attack actually progresses, and the job is to monitor, analyse, and document each phase of the breach as it happens, then submit a full case report.

**Scenario objectives:**
- Monitor and analyse real-time alerts as the attack unfolds
- Identify and document critical events such as PowerShell executions, reverse shell connections, and suspicious DNS requests
- Create detailed case reports based on observations to help the team understand the full scope of the breach

---

## 🔍 Investigation

This scenario runs live: alerts kept arriving as the attack chain progressed, so triage happened under real time pressure instead of against a fixed, already-complete log. For each alert, the job was to classify it as True Positive or False Positive, then build a case report tying the PowerShell executions, reverse shell activity, and suspicious DNS requests into a single coherent narrative of the breach.

---

## 📊 Final Results

| Metric | Result |
|---|---|
| ✅ True Positive Identification Rate | 100% |
| ✅ False Positive Identification Rate | 91% |
| 🔔 Closed Alerts | 36 alerts |
| ⏱️ Mean Time to Resolve | 7 minutes |
| ⏱️ Mean Dwell Time | 26 minutes |
| 🏆 Ranking | 1st place |
| 🎯 Score | 949 pts (+789 pts) |

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Phishing-Unfolding/Screenshots/pfinal.jpg" width="700"/>
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/frankllin-sec/Labs-experience/main/Phishing-Unfolding/Screenshots/pfinal2.jpg" width="700"/>
</p>

---

## 🧠 What I Learned

- How to triage under real time pressure, alerts kept coming while I was still writing up the previous one, which is a very different experience than working a static, already-complete alert queue
- How to hold onto details across a long, evolving attack chain instead of just reacting to whatever alert is newest
- That classification (True Positive vs False Positive) and report writing are two separate skills, being right about the verdict isn't the same as writing it up clearly
- Process-related alerts consistently took the longest to close, worth specifically practicing that category

---

## 💬 Honest Self-Assessment

This one gave me a headache. After an hour my brain was fried, and I'm sure I lost some details along the way, it's a lot to keep track of. I'll be honest: recognizing whether something is a true or false positive comes fairly easily to me. My hardest part is finding the right words while writing the report itself. I'd like to work from some examples of well-written reports, or just get more repetitions in, I think that's the kind of thing that gets faster with practice, not something I can shortcut.

---
<p align="center">
  <i>"Stay sharp, stay curious, stay secure."</i> 🔐
</p>
<p align="center">Thank you for visiting! 🙏</p>
<p align="center">Made with 🛡️ by <a href="https://github.com/frankllin-sec">Frankllin</a></p>
