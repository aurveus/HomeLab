# Wazuh Deployment and Agent Integration Lab

## 1. Wazuh Deployment

Wazuh was deployed as the centralized SIEM platform within the internal lab network.

### Access
```
https://10.200.200.20
```

### Notes
- HTTPS is required to access the dashboard
- A self-signed certificate warning must be accepted

### Core Components
- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard

---

## 2. Wazuh Dashboard

The Wazuh dashboard provides centralized visibility into monitored systems, alerts, and security events.

![Wazuh Dashboard](./wazuh-dashboard.png)

---

## 3. Network Configuration

All systems were configured using static IP addressing within the same subnet.

### Network Details
```
Network: 10.200.200.0/24
Gateway: 10.200.200.254
DNS:     10.200.200.254
```

### Assigned IPs
```
Wazuh   → 10.200.200.20
Windows → 10.200.200.30
Ubuntu  → 10.200.200.40
```

---

## 4. Ubuntu Static IP Configuration

Ubuntu Server uses **netplan** for network configuration.

### Configuration File
```
/etc/netplan/00-installer-config.yaml
```

### Static IP Configuration
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: no
    enp0s8:
      dhcp4: no
      addresses:
        - 10.200.200.40/24
      routes:
        - to: default
          via: 10.200.200.254
      nameservers:
        addresses: [10.200.200.254]
```

### Apply Configuration
```
sudo netplan apply
```

### Important Fixes
- Disabled DHCP on all interfaces
- Disabled cloud-init to prevent configuration override
- Removed conflicting netplan files
- Ensured correct YAML indentation

---

## 5. Windows Agent Deployment

### Steps
1. Open Wazuh Dashboard  
2. Navigate to:
```
Agents → Deploy new agent
```
3. Select OS: Windows  
4. Copy the generated PowerShell command  
5. Run the command in **PowerShell (Administrator)** on the Windows VM  

### Result
- Agent installs and registers automatically  
- Appears in Wazuh dashboard as **Active**  

---

## 6. Ubuntu Agent Deployment

### Package Selection
- OS: Linux  
- Package: DEB (amd64)  

### Installation Steps
1. Navigate to:
```
Agents → Deploy new agent
```
2. Copy the Linux (bash) installation command  
3. Execute on Ubuntu:
```
sudo <installation_command>
```

### Start and Enable Agent
```
sudo systemctl start wazuh-agent
sudo systemctl enable wazuh-agent
```

### Verification
```
systemctl status wazuh-agent
```

### Result
- Ubuntu agent successfully connects to Wazuh  
- Agent appears as **Active** in the dashboard  

---

## 7. Agent Verification

The following screenshot confirms that both Windows and Ubuntu agents are successfully deployed and active.

![Wazuh Agents](./agents.png)

---

## 8. Connectivity Verification

### Test Gateway
```
ping 10.200.200.254
```

### Test Wazuh Server
```
ping 10.200.200.20
```

---

## 9. Key Issues and Fixes

### Ubuntu IP Reset After Reboot
- Cause: cloud-init overriding netplan  
- Fix:
  - Disabled cloud-init networking  
  - Removed conflicting netplan files  
  - Re-applied netplan configuration  

### Wazuh Dashboard Not Accessible
- Cause: Using HTTP instead of HTTPS  
- Fix:
```
https://10.200.200.20
```

### Multiple IP Addresses on Wazuh
- Cause: DHCP client still active  
- Fix:
- Disabled DHCP configuration  
- Removed dhclient persistence  
- Restarted network services  

### Agent Not Connecting
- Cause: Service not started or network misconfiguration  
- Fix:
```
sudo systemctl start wazuh-agent
```

---

## 10. Outcome

- Wazuh successfully deployed and accessible  
- Static IP addressing implemented across all systems  
- Windows and Ubuntu machines integrated as monitored agents  
- Centralized logging and monitoring operational  
