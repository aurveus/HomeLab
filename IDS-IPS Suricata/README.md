# OPNsense IDS/IPS Lab – Custom Nmap Detection

## 1. Overview

This lab demonstrates the implementation of an Intrusion Detection and Prevention System (IDS/IPS) using OPNsense. A custom Suricata rule was created to detect Nmap SYN stealth scans targeting the firewall. The lab validates detection by generating scan traffic from a Kali Linux machine and observing alerts within OPNsense.

---

## 2. Objectives

- Configure IDS/IPS functionality in OPNsense  
- Enable SSH access for remote file management  
- Create and deploy a custom Suricata rule  
- Host rule files via a local HTTP server  
- Import and activate the rule within OPNsense  
- Simulate reconnaissance activity using Nmap  
- Validate detection through IDS alerts  

---

## 3. Lab Architecture Summary

- **Firewall:** OPNsense (LAN: 10.200.200.254)  
- **Attacker Machine:** Kali Linux (10.200.200.x)  
- **Detection Engine:** Suricata (built into OPNsense)  
- **Rule Hosting:** Python HTTP server on Kali  
- **File Transfer:** SFTP using FileZilla  

---

## 4. IDS/IPS Configuration

IDS/IPS was enabled through the OPNsense interface with the following key settings:

- Intrusion Detection: Enabled  
- IPS Mode: Enabled (Netmap)  
- Promiscuous Mode: Enabled  
- Pattern Matcher: Hyperscan  
- Detection Profile: Medium  
- Interface: LAN  
- Home Network: 10.200.200.0/24  

![IDS Settings](ids settings.png)

---

## 5. Enabling SSH Access

SSH access was enabled to allow remote file transfer and system interaction.

- SSH enabled under system administration settings  
- Root login permitted (lab environment only)  
- Password authentication enabled  
- Port: 22  

![SSH Configuration](ssh.png)

---

## 6. Custom Rule Creation

A custom Suricata rule was written to detect SYN stealth scans based on:

- TCP SYN packets  
- High frequency (threshold-based detection)  
- Target: Firewall (10.200.200.254)  

The rule triggers an alert when scanning behavior is detected.

![Custom Rule](customnmap.png)

---

## 7. Rule Deployment via FileZilla

FileZilla was used to establish an SFTP connection to the OPNsense firewall.

- Connected using root credentials  
- Navigated to Suricata rules directory  
- Uploaded XML rule definition file  

![File Transfer](filezilla.png)

---

## 8. Hosting Rule Files

A Python HTTP server was used to serve the rule file so OPNsense could download it dynamically.

```bash
python3 -m http.server 80
