# Wazuh SOC Lab – Docker-based Detection & Response Environment

## 📌 Project Overview

This project simulates a real-world SOC environment using Wazuh, Docker, and Linux containers.

The goal was to:
- Deploy a SIEM environment
- Onboard endpoints
- Simulate realistic attacks
- Create custom detection rules
- Perform incident investigation
- Generate professional SOC reports

---

## 🏗 Architecture

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

## 🛠 Stack Used

- Wazuh 4.x
- OpenSearch
- Docker & Docker Compose
- Ubuntu container (victim)
- Kali Linux (host & attacker)
- Netcat, Hydra, Nmap

---

## 🚀 Attack Scenarios Simulated

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

## 🔎 Custom Detection Rules

Custom rules created in:
