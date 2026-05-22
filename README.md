
# 🔍 Network Reconnaissance with Nmap — Kali Linux & Ubuntu

> Using Nmap to perform network reconnaissance on an Ubuntu VM from Kali Linux, and applying defensive countermeasures based on findings.

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Environment & Tools](#environment--tools)
- [What is Network Reconnaissance?](#what-is-network-reconnaissance)
- [Setup](#setup)
- [Scans Performed](#scans-performed)
- [Results & Analysis](#results--analysis)
- [Defensive Countermeasures Applied](#defensive-countermeasures-applied)
- [Key Findings](#key-findings)
- [Lessons Learned](#lessons-learned)

---

## Project Overview

The goal of this project was to perform network reconnaissance against an Ubuntu VM using Nmap from a Kali Linux VM. Both machines were connected via a NAT Network in VirtualBox. The project covered offensive scanning techniques as well as applying real defensive countermeasures based on the findings.

---

## Environment & Tools

| Component        | Details                          |
|------------------|----------------------------------|
| Hypervisor       | Oracle VirtualBox                |
| Attacker VM      | Kali Linux                       |
| Target VM        | Ubuntu                           |
| Target IP        | 10.0.2.4                         |
| Network Type     | NAT Network                      |
| Primary Tool     | Nmap 7.95                        |

---

## What is Network Reconnaissance?

Network reconnaissance is the process of gathering information about a target system or network before an attack. It is one of the first steps in the **penetration testing methodology** and helps identify:

- Open ports and services
- Software versions running on those services
- Operating system information
- Potential vulnerabilities to investigate further

Understanding reconnaissance from an attacker's perspective helps defenders know what information is exposed and how to reduce it.

---

## Setup

### Services Installed on Ubuntu (Target)

To simulate a real server environment, the following services were installed on the Ubuntu VM:

**SSH Server:**
```bash
sudo apt install openssh-server -y
sudo systemctl start ssh
sudo systemctl enable ssh
```
<img width="1512" height="982" alt="Screenshot 2026-05-20 at 9 07 45 PM" src="https://github.com/user-attachments/assets/5d67f2f6-29bd-4a9e-88f8-6ce4d32c4b6c" />


**Apache Web Server:**
```bash
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2
```
<img width="1512" height="982" alt="Screenshot 2026-05-20 at 9 14 59 PM" src="https://github.com/user-attachments/assets/2af39138-ef39-4251-be75-afe17676e170" />


### Verifying Connectivity

Before scanning, connectivity between the VMs was confirmed:
```bash
ping 10.0.2.4
```

---

## Scans Performed

### 1. Basic Port Scan
```bash
nmap 10.0.2.4
```
Discovers open ports and their associated services.

### 2. Service Version Scan
```bash
nmap -sV 10.0.2.4
```

Identifies the exact software and version running on each open port.

### 3. Aggressive Scan
```bash
sudo nmap -A 10.0.2.4
```
Combines OS detection, version detection, script scanning, and traceroute.

### 4. Saving Results to File
```bash
sudo nmap -A 10.0.2.4 -oN nmap_results.txt
```

---

## Results & Analysis

### Basic Scan Results

```
PORT    STATE  SERVICE
22/tcp  open   ssh
80/tcp  open   http
```
<img width="626" height="461" alt="Screenshot 2026-05-20 at 9 15 54 PM" src="https://github.com/user-attachments/assets/e2bd8633-0ce6-4f72-ab53-f6a3c6850bbd" />


### Version Scan Results (Before Hardening)

```
PORT    STATE  SERVICE  VERSION
22/tcp  open   ssh      OpenSSH 10.2p1 Ubuntu 2ubuntu3.2 (Ubuntu Linux; protocol 2.0)
80/tcp  open   http     Apache httpd 2.4.66 ((Ubuntu))
```
<img width="1512" height="982" alt="Screenshot 2026-05-20 at 9 21 36 PM" src="https://github.com/user-attachments/assets/894792bc-99db-488b-953e-edc7bb6753c8" />


### Aggressive Scan Results (After Hardening)

```
PORT    STATE  SERVICE  VERSION
22/tcp  open   ssh      OpenSSH 10.2p1 Ubuntu 2ubuntu3.2 (Ubuntu Linux; protocol 2.0)
80/tcp  open   http     Apache httpd

MAC Address: 08:00:27:76:AC:25 (Oracle VirtualBox virtual NIC)
```
<img width="1512" height="982" alt="Screenshot 2026-05-20 at 10 23 58 PM" src="https://github.com/user-attachments/assets/67d58af4-fbd3-45da-8d59-305f3c64ae3b" />


---

## Defensive Countermeasures Applied

### Hiding the Apache Version Banner

The version scan revealed Apache 2.4.66 which could allow an attacker to search for version-specific exploits. This was mitigated by editing the Apache security config:

```bash
sudo nano /etc/apache2/conf-available/security.conf
```

Changed the following values:
```
ServerTokens Prod
ServerSignature Off
```
<img width="551" height="489" alt="Screenshot 2026-05-20 at 9 31 56 PM copy" src="https://github.com/user-attachments/assets/6ea1a153-5d7c-444a-a617-b32242507df5" />


Then restarted Apache:
```bash
sudo systemctl restart apache2
```

**Result:** Running `nmap -sV` after this change showed `Apache httpd` with no version number — successfully hiding the version from attackers.  
<img width="1512" height="982" alt="Screenshot 2026-05-20 at 9 45 23 PM" src="https://github.com/user-attachments/assets/4a358484-e41e-4e34-bd21-e9f6f4b1f78d" />


---

## Key Findings

| Finding | Risk | Action Taken |
|---|---|---|
| Port 22 open (SSH) | Medium — brute force risk | Next project: SSH Hardening |
| Port 80 open (HTTP) | Medium — unencrypted traffic | Consider HTTPS in future |
| Apache version exposed | Medium — version-specific exploits | Fixed with ServerTokens Prod |
| Default Apache page visible | Low — reveals fresh install | Should be replaced with real content |
| VirtualBox NIC detected | Low — reveals virtualized environment | Expected in lab environment |
| OS fingerprint incomplete | Low | VirtualBox interferes with raw packet detection |

---

## Lessons Learned

- **Basic scans reveal a lot** — just two open ports told an attacker the OS, software versions, and that the machine was freshly set up.
- **Version banners should be hidden** — exposing exact software versions makes it easy for attackers to find known exploits.
- **Virtualization affects scanning** — VirtualBox's network layer interferes with raw packet OS detection, which is a known limitation of scanning inside virtual environments.
- **Reconnaissance is the foundation** — everything an attacker does next (exploitation, brute forcing) is based on what they find during reconnaissance.
- **Defenders should scan themselves** — running Nmap against your own systems shows exactly what an attacker would see.

---

## 📎 References

- [Nmap Official Documentation](https://nmap.org/docs.html)
- [Apache ServerTokens Documentation](https://httpd.apache.org/docs/2.4/mod/core.html#servertokens)
- [OpenSSH Security](https://www.openssh.com/security.html)
- [CVE Database for vulnerability research](https://cve.mitre.org/)
