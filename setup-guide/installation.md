# 🛠️ Wazuh SOC Lab Installation Guide

## 📋 Prerequisites

* VirtualBox installed
* Minimum 8–16 GB RAM recommended (Lab tested with 32 GB RAM)
* Kali Linux, Windows 10, Ubuntu VM images
* Stable internet connection

---

## 📌 Lab Architecture

* Kali Linux (Attacker) – 192.168.10.5
* Windows 10 (Victim) – 192.168.10.10
* Ubuntu Wazuh Server – 192.168.10.20

---

## ⚙️ Step 1: Setup Virtual Machines

* Install VirtualBox
* Import Kali Linux, Windows 10, and Ubuntu

### 🌐 Network Configuration

Each VM is configured with two network adapters:

1. **Internal Network (Adapter 1)**

   * Used for communication between lab machines
   * Example IP range: 192.168.10.0/24

2. **NAT (Adapter 2)**

   * Used for internet access (updates, package installation)

---

👉 In this lab, **Internal Network + NAT** is used.

👉 Alternatively, you can also use **NAT Network** to allow communication between VMs along with internet access, depending on your lab setup.

---

### 📡 IP Addressing

* Kali Linux – 192.168.10.5
* Windows 10 – 192.168.10.10
* Ubuntu (Wazuh Server) – 192.168.10.20

👉 You can configure IP addresses based on your lab environment, but ensure all systems are in the same subnet.

---

### 🧾 Manual IP Configuration (Temporary)

#### 🔹 Ubuntu (Wazuh Server)

sudo nano /etc/netplan/00-installer-config.yaml

network:
version: 2
ethernets:
enp0s3:
dhcp4: no
addresses:
- 192.168.10.20/24

sudo netplan apply

---

#### 🔹 Kali Linux

sudo ip addr add 192.168.10.5/24 dev eth0
sudo ip link set eth0 up

---

#### 🔹 Windows 10

* Open Network Settings
* Select Internal Network Adapter
* Configure IPv4 manually

IP Address: 192.168.10.10
Subnet Mask: 255.255.255.0

---

### 🟢 Optional: Permanent IP Configuration

#### 🔹 Ubuntu (Netplan)

sudo nano /etc/netplan/00-installer-config.yaml

network:
version: 2
ethernets:
enp0s3:
dhcp4: no
addresses:
- 192.168.10.20/24
gateway4: 192.168.10.1
nameservers:
addresses: [8.8.8.8]

sudo netplan apply

---

#### 🔹 Kali Linux

sudo nano /etc/network/interfaces

auto eth0
iface eth0 inet static
address 192.168.10.5
netmask 255.255.255.0
gateway 192.168.10.1

sudo systemctl restart networking

---

## 🧱 Step 2: Install Wazuh Server (Ubuntu)

curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash wazuh-install.sh -a

Access Dashboard:
https://192.168.10.20

---

### ⚠️ Troubleshooting: Wazuh Dashboard Not Ready

If the dashboard shows **"Dashboard not ready yet"**, run:

sudo systemctl restart wazuh-manager
sudo systemctl restart wazuh-indexer
sudo systemctl restart wazuh-dashboard

Refresh browser after a few seconds.

---

## 🖥️ Step 3: Install Wazuh Agent (Windows)

* Download Wazuh agent
* Install the agent

Configure:
Manager IP: 192.168.10.20

* Start Wazuh agent service

---

### ⚠️ Important Compatibility Note

* Always install a **Wazuh agent version equal to or lower than server version**
* Higher agent version may cause connection issues

---

### 🔑 Step 3.1: Get Wazuh Agent Registration Key

sudo /var/ossec/bin/manage_agents

1. Press **A** → Add agent
2. Enter agent name and IP (192.168.10.10)
3. Press **Y**
4. Press **E** → Extract key
5. Copy key

---

### 🔐 Step 3.2: Add Key in Windows Agent

**Method 1 (CLI):**

"C:\Program Files (x86)\ossec-agent\manage_agents.exe"

* Select **Import key**
* Paste key

**Method 2 (GUI):**

* Open Wazuh Agent Manager
* Paste key and save

---

### 🔄 Step 3.3: Restart Agent

- net stop wazuh
- net start wazuh

---

### ✅ Step 3.4: Verify Agent Connection

* Open Dashboard → **Overview**
* Check agent status → **Active**

---

### 🔎 Step 3.5: Verify Agent from Server (Optional)

sudo /var/ossec/bin/agent_control -l

---

## 🔍 Step 4: Enable Log Collection

* Enable Windows Security logs
* Monitor Event ID 4625

---

### 🌐 Step 4.1: Check Network Connectivity

ping 192.168.10.20
ping 192.168.10.10

---

## 🔐 Step 5: Simulate Brute Force Attack

From Kali Linux terminal:

ping 192.168.10.10

**Credentials:**

* Username: Tester
* Wordlist: rockyou.txt

**SMBv1:**
hydra -l Tester -P rockyou.txt 192.168.10.10 smb

**SMB (General):**
hydra -l Tester -P rockyou.txt smb://192.168.10.10

**Optional RDP:**
hydra -l Tester -P rockyou.txt rdp://192.168.10.10

---

## 🟡 Step 6: File Integrity Monitoring (FIM)

### ⚙️ Step 6.1

Edit:
C:\Program Files (x86)\ossec-agent\ossec.conf

---

### 🧾 Step 6.2
- Add this line inside the <syscheck> 
```xml
<syscheck>
  <!-- existing configuration -->

  <frequency>60</frequency>

  <directories check_all="yes" realtime="yes">C:\Users\Tester\Desktop</directories>

</syscheck>
```

---

### 🔄 Step 6.3

net stop wazuh
net start wazuh

---

## 📊 Step 7: View Alerts

### 🔴 Brute Force Alerts

* Go to **Threat Hunting**

### 🟡 FIM Alerts

* Go to **File Integrity Monitoring**

---

## 🔎 Step 8: Create Alert Filters in Discover

### 📍 Navigate

* Open Dashboard → **Discover**

---

### 🔴 Failed Login

* Add filter → rule.id → is → 60122

---

### 🟡 FIM Events

* Add filter → syscheck.event → is → added
* Add filter → syscheck.event → is → modified
* Add filter → syscheck.event → is → deleted

---

### 🔵 Successful Login

* Add filter → win.system.eventID → is → 4624

---

### 🟣 Attacker IP

* Add filter → data.win.eventdata.ipAddress → is → 192.168.10.5

---

### 🟢 Logon Type

* Add filter → data.win.eventdata.logonType → is → 3

---

### 💾 Save Filters

Save filters for quick threat hunting and analysis.

---

## 🎯 Outcome

* Brute-force attack detected
* File integrity monitoring working
* Alerts visible in dashboard
* Threat hunting enabled

---

## 🎯 MITRE ATT&CK Mapping

* T1110 – Brute Force
* T1565 – Data Manipulation (FIM)

