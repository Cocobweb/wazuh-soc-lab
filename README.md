# Wazuh SOC Lab – Docker-based Detection & Response Environment

## Project Overview

This project simulates a real-world SOC environment using Wazuh, Docker, and Linux containers.

The goal was to:
- Deploy a SIEM environment
- Onboard endpoints
- Simulate realistic attacks
- Create custom detection rules
- Perform incident investigation
- Generate professional SOC reports

---

## Architecture

Docker-based architecture running on Kali Linux:

Host (Kali Linux)  
│  
├── Wazuh Manager  
├── Wazuh Indexer (OpenSearch)  
├── Wazuh Dashboard  
└── Victim Ubuntu (with Wazuh Agent)  

All components run inside Docker.

Architecture diagram:
`docs/architecture.png`

---

## Stack Used

- Wazuh 4.14.2
- OpenSearch
- Docker & Docker Compose
- Ubuntu container (victim)
- Kali Linux (host & attacker)
- Netcat, Hydra, Nmap

---

## Attack Scenarios Simulated

### 1️⃣ SSH Brute Force
- Multiple failed login attempts
- Detection rule triggered
- Timeline reconstructed

### 2️⃣ Successful SSH Login After Brute Force
- Root login detected
- Privilege escalation attempt

### 3️⃣ File Integrity Monitoring
- Modification of sensitive files

### 4️⃣ Data Exfiltration via Netcat
- /etc/passwd exfiltration
- Network event correlation

---

## Custom Detection Rules

Custom rules created in:
rules/local_rules.xml


Examples:
- SSH brute force (5 fails in 60s)
- Multiple sudo failures
- Root SSH login detection
- Sensitive file modification
- Suspicious netcat usage

---

## Incident Investigation Example

Full investigation report available here:
docs/incidents/incident-ssh-bruteforce-2026-02-16.md


The report includes:
- Timeline reconstruction
- Source IP identification
- User involved
- Actions performed
- Indicators of compromise (IOCs)
- Lessons learned

---

## Active Response (Optional Module)

Automatic IP blocking via firewall-drop configured (optional).

---

## Dashboards Created

- Top source IPs
- Failed login attempts
- Alerts by severity
- Attack timeline visualization

Screenshots available in `/screenshots`

---

## Key Learnings

- Log correlation is crucial in detecting multi-stage attacks
- Brute force alone is low severity; successful login changes impact
- Custom rules significantly improve detection precision
- Timeline reconstruction is essential for SOC L2 workflow

---

## Why This Project Matters

This lab demonstrates:
- Hands-on SIEM deployment
- Attack simulation
- Detection engineering
- Incident investigation workflow
- Real SOC analytical thinking

---

## Disclaimer

This lab is for educational purposes only.
All attacks were performed in a controlled environment.
