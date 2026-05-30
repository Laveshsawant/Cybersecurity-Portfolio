# 🛡️ Lavesh Sawant — Cybersecurity Portfolio
## 👋 About Me

I am a fresher cybersecurity enthusiast focused on **SOC Analysis** and **Ethical Hacking**. I build hands-on home labs to demonstrate real attack and detection scenarios rather than just studying theory.

> "Most people study cybersecurity. I build it."

---

## 🗂️ Projects

### 🔴 Project 1 — SOC Home Lab: Brute Force Attack & Detection
📁 Folder: `01-SOC-Brute-Force-Lab/`

| Detail | Info |
|--------|------|
| Type | Red Team + Blue Team |
| Attack | SMB Brute Force via Hydra |
| Detection | Wazuh SIEM + Splunk |
| Alert | Gmail email notification |
| MITRE | T1110.001 — Password Guessing |

**What I did:**
- Performed SMB brute force attack from Kali Linux against Windows 10 VM
- Cracked credentials using Hydra with rockyou.txt wordlist
- Detected attack in Wazuh SIEM via Event ID 4625
- Configured automated email alerts when attack detected
- Built Splunk dashboard showing attack timeline
- Mapped attack to MITRE ATT&CK framework

**Skills:** Kali Linux, Hydra, Nmap, Wazuh, Splunk, Windows Event Logs, MITRE ATT&CK

---

### 🌐 Project 2 — CCNA Networking Practical Lab
📁 Folder: `02-CCNA-Networking-Lab/`

| Detail | Info |
|--------|------|
| Type | Networking Fundamentals |
| Tool | Cisco Packet Tracer |
| Topics | Routing, VLANs, ACLs, DNS, DHCP |
| Goal | Understand network traffic for SOC |

**What I did:**
- Configured routers and switches in Packet Tracer
- Set up VLANs and inter-VLAN routing
- Configured Access Control Lists (ACLs)
- Practiced subnetting and IP addressing
- Documented all commands and configurations

**Skills:** TCP/IP, Routing, Switching, VLANs, ACLs, Subnetting, Cisco IOS

---

### 🎣 Project 3 — Phishing Attack Simulation *(Coming Soon)*
📁 Folder: `03-Phishing-Attack-Lab/`

| Detail | Info |
|--------|------|
| Type | Social Engineering + Initial Access |
| Tools | SET, Metasploit, Kali Linux |
| MITRE | T1566 — Phishing |
| Goal | CIA Triad demonstration |

**Planned:**
- Generate phishing payload using Social Engineering Toolkit
- Deliver payload to Windows 10 VM
- Establish reverse shell connection
- Demonstrate CIA Triad breach (Confidentiality, Integrity, Availability)
- Detect with Wazuh

---

### 🦠 Project 4 — YARA Malware Detection *(Coming Soon)*
📁 Folder: `04-YARA-Malware-Lab/`

| Detail | Info |
|--------|------|
| Type | Malware Analysis + Detection |
| Tools | YARA, Python, MalwareBazaar |
| Goal | Build mini malware detector |

---

## 🧰 Skills Summary

### Security Tools
![Kali](https://img.shields.io/badge/-Kali%20Linux-557C94?style=flat)
![Wazuh](https://img.shields.io/badge/-Wazuh-005571?style=flat)
![Splunk](https://img.shields.io/badge/-Splunk-000000?style=flat)
![Hydra](https://img.shields.io/badge/-Hydra-red?style=flat)
![Nmap](https://img.shields.io/badge/-Nmap-blue?style=flat)
![Metasploit](https://img.shields.io/badge/-Metasploit-2596be?style=flat)

### Concepts
- SIEM — Wazuh, Splunk
- Brute Force Detection
- Windows Event Log Analysis
- MITRE ATT&CK Framework
- Network Scanning & Enumeration
- Incident Detection & Alerting
- Virtualisation — Oracle VirtualBox

### Networking
- TCP/IP, UDP, DNS, DHCP
- VLANs, Routing, Switching
- Firewalls, NAT, VPN
- Subnetting, ACLs
- Cisco Packet Tracer

### Operating Systems
- Kali Linux
- Windows 10
- Ubuntu (Wazuh Server)

---

## 📊 MITRE ATT&CK Coverage

```
INITIAL ACCESS    EXECUTION    PERSISTENCE    DISCOVERY    LATERAL    CREDENTIAL
                                                          MOVEMENT    ACCESS
                                             ✅ T1046                 ✅ T1110
                                             Network Scan             Brute Force
                                                                      ✅ T1110.001
                                                                      Password Guess

COMING SOON:
✅ T1566 Phishing   ✅ T1059                               ✅ T1021
   Initial Access      Command Line                          Remote Services
```

---

## 🏆 What Makes This Portfolio Different

| Most Freshers | This Portfolio |
|--------------|----------------|
| Only theory knowledge | Hands-on lab evidence |
| Watch YouTube tutorials | Actually build and break things |
| List tools on CV | Show tools in action with screenshots |
| Know about attacks | Performed AND detected real attacks |
| Study MITRE ATT&CK | Map own attacks to framework |

---

## 📜 Certifications & Learning

- 🔄 CEH *(In Progress)*
- 🔄 TryHackMe — SOC Level 1 Path *(In Progress)*
- 📚 CCNA Networking Fundamentals
- 📚 Ethical Hacking Fundamentals

---

## 📫 Contact

- 📧 Email: sawantlavesh6@gmail.com
- 💼 LinkedIn :-(https://www.linkedin.com/in/lavesh-sawant-144576260?utm_source=share_via&utm_content=profile&utm_medium=member_android)
- 🐙 GitHub: *(https://github.com/Laveshsawant/cybersecurity-Portfolio)*

---

## ⚠️ Disclaimer

All projects in this portfolio were performed on **personally owned virtual machines** in an **isolated lab environment** for **educational purposes only**. No unauthorized systems were accessed at any time.

---

*Built with dedication, late nights, and lots of debugging 🔥*

