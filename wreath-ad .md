# Red Team Engagement Report – Wreath

**Multi-Host Active Directory Compromise**

**Author:** Omar Taisir Abu Zahra                                                                             
**Lab Name:** Wreath                                                                    
**Platform:** TryHackMe (Lab Environment)                                                                    
**Engagement Type:** Red Team / Adversary Emulation                                                          
**Assessed Difficulty:** Easy                                                   
**Final Outcome:** Full domain compromise                                                            
**Date:** Jan 2026                                                               

---

## Executive Summary

This engagement simulated a realistic multi-stage attack against a small enterprise Active Directory environment. Starting from an externally exposed Linux host, the attacker achieved an initial foothold, pivoted into the internal network, compromised a Windows host, and ultimately obtained **Domain Administrator–level control**.

The compromise was achieved by chaining common weaknesses including exposed services, weak credentials, insecure trust relationships, and excessive privileges. No advanced exploits or zero-day vulnerabilities were required.

---

##  Scope & Objectives

### Scope
- External-facing Linux server
- Internal Windows hosts
- Active Directory domain services

### Objectives
- Gain initial external access  
- Perform internal pivoting  
- Compromise Windows systems  
- Escalate privileges to domain administrator  

---

## Attack Path Overview

External Reconnaissance
→ Initial Foothold (Linux)
→ Credential Access
→ Internal Network Pivot
→ Windows Host Compromise
→ Active Directory Enumeration
→ Privilege Escalation
→ Domain Admin Access


This path reflects common real-world intrusions against hybrid Linux/Windows environments.

---

## Phase 1 – External Reconnaissance

### Objective
Identify exposed services and attack surface.

```bash
nmap -sC -sV -Pn <target-ip>
Findings
SSH service exposed

Web service accessible

Target identified as Linux-based external host
```
## Phase 2 – Initial Foothold
Objective
Gain execution on the external host.

Weak authentication and exposed services were abused to obtain user-level access on the Linux system.

```bash
ssh user@<target-ip>
Initial shell access confirmed.
```
## Phase 3 – Credential Access
Objective
Identify credentials for lateral movement.

Post-exploitation enumeration revealed locally stored credentials and configuration files containing authentication material reused across systems.

```bash
cat /etc/passwd
cat config.php
Recovered credentials were later reused against internal Windows hosts.
```
## Phase 4 – Internal Pivoting
Objective
Access internal-only network resources.

SSH tunneling and port forwarding were used to reach internal services not exposed externally.

```bash
ssh -L 3389:<internal-ip>:3389 user@<external-host>
This enabled access to Windows systems within the internal network.
```
## Phase 5 – Windows Host Compromise
Objective
Gain access to an internal Windows machine.

Recovered credentials were successfully reused to authenticate to a Windows host via SMB / RDP.

```bash
xfreerdp /u:user /p:password /v:<internal-ip>
User-level access obtained on a domain-joined Windows system.
```
## Phase 6 – Active Directory Enumeration
Objective
Identify privilege escalation paths within the domain.

Tools and native commands were used to enumerate domain users, groups, and permissions.

```bash
whoami /groups
net user /domain
Misconfigured privileges and weak group memberships were identified.
```
## Phase 7 – Privilege Escalation
Objective
Escalate privileges to Domain Administrator.

Abuse of delegated permissions and credential reuse allowed escalation to high-privilege domain accounts.

```bash
net group "Domain Admins" /domain
Domain administrator credentials were obtained.
```
## Impact Validation
Domain-Level Control Confirmed
Administrative access to domain controllers

Ability to manage users and systems

Full compromise of the Active Directory environment

## Risk Assessment
Overall Risk Rating: Critical

Key Issues Identified
Credential reuse across systems

Insecure storage of credentials

Excessive user privileges

Lack of network segmentation

Weak internal monitoring

## Defensive Recommendations
Enforce strong, unique credentials and MFA

Eliminate plaintext credential storage

Apply least-privilege principles

Segment internal networks

Monitor lateral movement and credential abuse

Regularly audit Active Directory permissions

## MITRE ATT&CK Mapping
Reconnaissance	Active Scanning	T1595                                                       
Initial Access	Valid Accounts	T1078                                                      
Credential Access	Credential Dumping	T1003                                         
Lateral Movement	Remote Services	T1021                                            
Discovery	Account Discovery	T1087                                     
Privilege Escalation	Abuse of Privileged Accounts	T1068                                         

## Conclusion
The Wreath engagement demonstrates how common misconfigurations and poor credential hygiene can allow an external attacker to fully compromise an Active Directory environment.

The attack required no advanced exploitation techniques, emphasizing that basic security controls, if improperly implemented, can lead to catastrophic outcomes. Strengthening identity management, access controls, and internal monitoring is critical to preventing similar attacks.