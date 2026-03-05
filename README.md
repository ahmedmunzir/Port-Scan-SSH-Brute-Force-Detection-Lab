# Port-Scan-SSH-Brute-Force-Detection-Lab
In this project I built an isolated attacker–target environment to capture port scans at the packet level, compare scan behaviours, and investigate authentication failures through system logs.

## Overview

This project explores what real attack activity looks like from a defender’s perspective.  
Using an isolated attacker and target environment, I generated port scans and SSH brute force attempts and then analysed how those actions appear at both the network and system level.

The goal was to connect attacker behaviour with the signals a defender would investigate.

Key areas covered in this lab:

• Packet level analysis of port scanning  
• Differences between stealth scans and full connection scans  
• How firewalls change scan behaviour  
• Investigating brute force activity through system authentication logs  

---

## Lab Environment

The lab was built using a small Proxmox virtual environment.

**Attacker**
- Kali Linux

**Target**
- Ubuntu Server

**Network Setup**
- Dedicated isolated internal network bridge
- Kali connected to two interfaces  
  - Management / internet access  
  - Internal lab network
- Ubuntu server connected only to the internal network
- Static IP address configured on the target

**Services Enabled**
- SSH installed and enabled on Ubuntu
- Connectivity verified using:
  - `ping`
  - `ssh`

This environment ensured that all attack traffic stayed inside the lab.

---

## Tools Used

- Kali Linux
- Ubuntu Server
- Proxmox
- Nmap
- tcpdump
- Wireshark
- UFW Firewall

---

## Part 1 — Stealth SYN Scan Analysis

A stealth SYN scan was launched from Kali while capturing traffic on the Ubuntu server using `tcpdump`.

The capture was transferred to Kali and analysed in Wireshark.

### Observations

- Around **1000 SYN packets** were sent during the scan
- Only the **SSH port returned a SYN/ACK response**
- All other ports responded with **RST packets**

### Packet behaviour for the open port

```
SYN → SYN/ACK → RST
```

This sequence confirms how a **half open SYN scan works**.

Key characteristics:

- The scanner confirms the port is open
- The connection is **not fully established**
- A **reset packet is sent immediately**

This allows attackers to discover open ports while leaving a smaller footprint.

---

## Part 2 — Full TCP Connect Scan Comparison

The scan was repeated using a **TCP connect scan**.

```
nmap -sT
```

Traffic was captured again using `tcpdump` and analysed in Wireshark.

### Observations

For the open SSH port, the connection completed the full handshake.

```
SYN → SYN/ACK → ACK
```

Additional packets were exchanged before the connection closed.

### Key Differences from SYN Scan

• Full TCP connection established  
• Higher number of ACK packets  
• More normal looking network traffic  
• Activity more likely to appear in host logs  

This demonstrates why TCP connect scans are **easier to detect** than stealth scans.

---

## Part 3 — Firewall Behaviour and Filtered Ports

A firewall was enabled on the Ubuntu server using **UFW**.

Firewall configuration:

- Deny all incoming connections
- Allow SSH

Another SYN scan was performed while capturing packets.

### Observations

The behaviour changed significantly.

- Closed ports **no longer returned RST packets**
- Traffic was **silently dropped**
- Nmap reported **999 filtered ports**
- SYN retransmissions increased
- The scan took **significantly longer**

### What this demonstrates

• Closed ports actively respond  
• Filtered ports **do not respond at all**  
• Firewalls alter scan signatures and detection patterns  

This helped illustrate how defensive controls affect reconnaissance behaviour.

---

## Part 4 — SSH Brute Force Log Investigation

The final part of the project focused on **host level log analysis**.

Repeated SSH login attempts were generated from the Kali machine.

Authentication logs were then analysed on the Ubuntu server.

Log file analysed:

```
/var/log/auth.log
```

### Observations

The logs showed:

- **53 failed login attempts**
- Attempts targeting a **non existent user**
- All attempts originating from the **same source IP**
- Repeated log entries such as:

```
Failed password for invalid user
```

Each log entry included:

- Source IP address
- Target username
- Source port
- Timestamp

### Key Indicators Identified

• High number of failed login attempts  
• Same source IP repeatedly attempting access  
• Attempts occurring within a short time window  

These are common indicators of **SSH brute force activity**.

---

## Key Takeaways

This project demonstrates how attacker behaviour translates into signals that defenders can investigate.

Skills developed during the lab:

- Packet level traffic analysis
- Understanding Nmap scan behaviour
- Identifying reconnaissance patterns
- Understanding firewall impact on scans
- Investigating authentication logs
- Recognising brute force attack indicators

---

## Evidence

The project includes screenshots showing:

- Nmap scan behaviour
- Wireshark packet analysis
- Firewall filtered traffic behaviour
- SSH authentication log entries
