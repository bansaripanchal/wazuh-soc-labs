# 🔐 Lab 01 — Linux SSH Brute-Force Detection

## 🎯 Objective

This lab demonstrates a controlled SSH password-guessing attack against an authorized Ubuntu Agent VM and the detection and investigation of the activity using Wazuh.

The complete workflow was performed inside my isolated SOC lab:

**Kali Linux → Ubuntu SSH Target → Wazuh Agent → Wazuh Manager → Threat Hunting**

---

## 🖥️ Lab Environment

| Component | Role | IP / Identifier |
|---|---|---|
| Wazuh Manager | SIEM / Detection Platform | `192.168.51.102` |
| Ubuntu Agent | SSH Target | `192.168.51.104` |
| Kali Linux | Attack Simulation | Lab Attacker |
| Target Account | SSH Account | `vboxuser` |

---

## ⚔️ Attack Simulation

A controlled password list containing intentionally incorrect passwords was created on Kali Linux.

### Password Test File

wrong1
wrong2
wrong3
wrong4
wrong5
wrong6
wrong7
wrong8
wrong9
wrong10

### Hydra Command
hydra -v -P ~/ssh-test.txt ssh://192.168.51.104 -t 2 -V

Observed Result
Hydra performed:
- Target: 192.168.51.104
- Service: SSH
- Port: 22
- Username: vboxuser
- Password attempts: 10
- Tasks: 2
The attack was performed only against my own Ubuntu lab VM.

### 🔎 Wazuh Detection

The generated SSH activity was ingested by the Wazuh Agent and displayed in Wazuh Threat Hunting.
Observed Threat Hunting results:
Metric	Result
Total Events	477
Authentication Failures	28
Authentication Successes	29
Level 12+ Alerts	0

The Wazuh dashboard also displayed MITRE-related activity including:
- Password Guessing
- Brute Force
- SSH
- Valid Accounts
- Sudo and Sudo Caching
- 
### 📊 Wazuh Event Detection

The Threat Hunting event list showed multiple authentication-related detections.
Rule ID	Level	Description
5760	5	SSH authentication failed
5763	10	SSH brute-force attempt
2502	10	User missed the password more than one time
5503	5	PAM user login failed


### The important correlation was:
Rule 5760
SSH authentication failed
        ↓
Repeated authentication failures
        ↓
Rule 5763
SSH brute force trying to get access

### 🔍 Event Investigation

A Wazuh event was opened in Document Details to investigate the individual authentication failure.
Observed Event Details
Field	Value
Agent ID	001
Agent Name	wazuhagent
Agent IP	192.168.51.104
Target User	vboxuser
Source IP	192.168.51.1
Source Port	10712
Decoder	sshd
Program	sshd
Location	journald
Manager	wazuh-server

Raw Event
Failed password for vboxuser from 192.168.51.1 port 10712 ssh2

### Detection Rule Analysis
The investigated Wazuh event contained:
Field	Value
Rule ID	5760
Rule Level	5
Description	sshd: authentication failed.
Rule Fired Times	9
Groups	syslog, sshd, authentication_failed

### 🧩 MITRE ATT&CK Mapping

Wazuh mapped the observed activity to the following MITRE ATT&CK techniques:
MITRE ID	Technique
T1110.001	Password Guessing
T1021.004	SSH

### 🕐 Investigation Timeline

1. Kali Linux was prepared for the attack simulation.
2. SSH connectivity to the Ubuntu Agent was verified.
3. A controlled password list was created.
4. Hydra generated SSH authentication attempts.
5. Ubuntu recorded authentication failures.
6. Wazuh Agent collected the authentication telemetry.
7. Wazuh Manager received the events.
8. Threat Hunting displayed the authentication activity.
9. Wazuh identified repeated SSH authentication failures.
10. The event was investigated using source IP, target account, rule information, and MITRE ATT&CK mapping.

### 🛡️ SOC Takeaways

- Repeated SSH authentication failures can indicate password-guessing activity.
- Source IP is an important investigation pivot.
- The targeted username helps identify the account under attack.
- Wazuh rule metadata explains how the event was classified.
- Correlation rules can identify repeated authentication failures as brute-force behavior.
- MITRE ATT&CK mapping provides adversary-technique context.
- Host logs and SIEM alerts should be investigated together.
  
### 📸 Evidence

The repository contains screenshots from the actual lab execution:
1. Kali Hydra attack
2. Wazuh Threat Hunting results
3. Wazuh event list
4. Wazuh event details
5. Wazuh rule details
6. MITRE ATT&CK mapping

### ✅ Lab Status

Completed
Detection Platform: Wazuh
Attack Simulation: Kali Linux / Hydra
Target: Ubuntu Linux
Technique: SSH Password Guessing
MITRE ATT&CK: T1110.001, T1021.004
Environment: Authorized isolated SOC lab
