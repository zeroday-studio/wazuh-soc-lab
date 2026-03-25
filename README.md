# wazuh-soc-lab
SOC Lab using Wazuh (Brute Force + FIM Detection)
🔐 Wazuh SOC Lab – Brute Force Detection & File Integrity Monitoring
📌 Overview

This project demonstrates a Security Operations Center (SOC) lab using Wazuh SIEM to detect brute-force attacks and monitor file integrity changes.

🧱 Lab Setup
Kali Linux (Attacker) – 192.168.10.5
Windows 10 (Victim) – 192.168.10.10
Ubuntu Wazuh Server – 192.168.10.20
Metasploitable – 192.168.10.50
⚙️ Tools Used
Wazuh SIEM
Hydra
VirtualBox
🚨 Use Cases
🔴 Brute Force Detection
Hydra attack on SMB (port 445)
Failed login detection (Event ID 4625)
Alerts in Wazuh dashboard
🟡 File Integrity Monitoring (FIM)
Monitoring critical files
Detecting file modification & deletion
Real-time alerts
📊 Detection
Authentication attacks detected
File changes monitored
📸 Screenshots

See /screenshots

🛠️ Installation Guide

See /setup-guide/installation.md

📄 Report

See /report

🎯 MITRE ATT&CK
T1110 – Brute Force
T1565 – Data Manipulation
👨‍💻 Author

Rakesh A R (zeroday-studio)
