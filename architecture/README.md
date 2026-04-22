# Network Architecture

![Lab Diagram](diagram.png)

## Description

This lab simulates a segmented network environment using an OPNsense firewall.

- The **WAN interface** connects to the internet via VirtualBox NAT  
- The **LAN interface** connects to an isolated internal network (`intnet`)  
- Internal machines (e.g., Kali Linux, Windows clients) reside on the LAN  
- The **OPNsense firewall** acts as the gateway, routing and controlling traffic between WAN and LAN  

All internal traffic must pass through the firewall, enforcing network segmentation and preventing direct exposure to the external network.
