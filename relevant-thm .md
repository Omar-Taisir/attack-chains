# Red Team Engagement Report – Relevant

External Compromise → SYSTEM Privilege Escalation

**Author:** Omar Taisir Abu Zahra                                                                
**Engagement Type:** Red Team / Adversary Simulation                                              
**Target Host:** Relevant (Windows Server 2016)                                                        
**Platform:** TryHackMe (Lab Environment)                                                       
**Assessed Difficulty:** Medium                                                                                   
**Date:** 09 Jan 2026

## Executive Summary

This engagement assessed the security posture of the Windows Server host “Relevant” from an external attacker perspective.

The assessment resulted in full system compromise, culminating in NT AUTHORITY\SYSTEM access. The compromise was achieved by chaining multiple configuration weaknesses and operational security failures, rather than exploiting advanced vulnerabilities.

Key contributing factors included:

Insecure SMB share exposure

Plaintext credential disclosure

Writable IIS web directories

Abuse of Windows token impersonation privileges

Both authenticated user-level access and SYSTEM-level access were successfully obtained, demonstrating a complete host takeover and highlighting real-world risks associated with misconfigurations.

## Scope & Objectives
```bash
In Scope

External network access to the target host

IIS web services

SMB file-sharing services

Local privilege escalation paths

Objectives

Obtain an initial foothold

Escalate privileges

Demonstrate full system compromise
```
## Phase 1 – External Reconnaissance
```bash
Objective

Identify exposed services and determine the external attack surface.

Actions

Network and service enumeration using TCP scans

OS fingerprinting and service detection

Key Findings

HTTP (IIS 10.0) on port 80

SMB services on ports 139 and 445

RDP on port 3389

Secondary IIS service exposed on high TCP port 49663

The target was identified as a Windows Server 2016 system named Relevant, presenting multiple externally accessible services suitable for further investigation.
```
## Phase 2 – SMB Enumeration & Credential Access
```bash
Objective

Identify accessible file shares and assess information disclosure risks.

Actions

Enumeration of SMB shares

Manual inspection of accessible directories

Findings

A non-standard SMB share named nt4wrksv was discovered. The share:

Was accessible with weak or misconfigured permissions

Contained a file named passwords.txt

Credential Disclosure

The file contained Base64-encoded credentials. After decoding, valid credentials for a local user account (e.g., bob) were recovered.

Credential reuse was confirmed through successful authentication against SMB services, providing authenticated access without exploitation.
```
## Phase 3 – Web Application Abuse
```bash
Objective

Leverage misconfigured web services to achieve remote code execution.

Actions

Identification of IIS services mapped to local directories

Correlation of SMB shares with IIS web roots

Critical Misconfiguration

The nt4wrksv SMB share was mapped directly to an IIS web root directory, allowing files written via SMB to be executed by the web server.

This configuration flaw enabled a direct path from file upload to code execution.

Initial Access

A malicious ASPX reverse shell payload was generated and uploaded to the writable SMB share. When accessed via the web service, the payload executed successfully, resulting in a reverse shell under the IIS application pool context.
```
## Phase 4 – Post-Exploitation
```bash
Objective

Validate access and identify privilege escalation opportunities.

Actions

Verification of execution context

Retrieval of user-level proof files

Enumeration of local privileges and token permissions

Findings

Privilege enumeration revealed the presence of:

SeImpersonatePrivilege

This privilege is frequently abused on modern Windows systems and represents a high-confidence privilege escalation vector.
```
## Phase 5 – Privilege Escalation
```bash
Objective

Escalate privileges to full SYSTEM-level control.

Technique

The PrintSpoofer token impersonation technique was used to abuse SeImpersonatePrivilege.

Outcome

Successful token impersonation resulted in escalation to:

NT AUTHORITY\SYSTEM


This granted unrestricted access to the operating system.
```
## Impact Validation
```bash
SYSTEM-Level Access Confirmation

SYSTEM access was validated by:

Accessing protected directories

Retrieving administrator-level proof files

This confirmed complete compromise of the target host.
```
## Risk Assessment
```bash
Overall Risk Rating: Critical

Identified Issues

Weak or anonymous SMB share access

Plaintext credential storage

Shared SMB and IIS web root directories

Excessive Windows token privileges

Lack of service and permission isolation

These weaknesses significantly reduce the effort required for a full system takeover.
```
## Defensive Recommendations
```bash
Restrict SMB share access and enforce strict authentication

Remove sensitive files from shared directories

Separate web application roots from file shares

Disable unnecessary privileges such as SeImpersonatePrivilege

Apply least-privilege principles to service accounts

Monitor for token impersonation and suspicious privilege usage
```
## Conclusion
```bash
This engagement demonstrates how multiple low-to-medium severity misconfigurations can be chained together to achieve full SYSTEM compromise.

An external attacker with no prior access was able to fully control the host without exploiting memory corruption vulnerabilities or zero-day exploits, emphasizing the importance of secure configuration, credential hygiene, and privilege management.
```
## MITRE ATT&CK Mapping

> Target: Relevant (Windows Server 2016)
> Difficulty: Medium

Initial Access
```bash

T1190 – Exploit Public-Facing Application

T1133 – External Remote Services (SMB)
```
Execution
```bash
T1059.003 – Windows Command Shell

T1059.001 – PowerShell
```
Persistence
```bash
T1505.003 – Web Shell
```
Privilege Escalation
```bash
T1134.001 – Token Impersonation

T1068 – Exploitation for Privilege Escalation
```
Credential Access
```bash
T1552.001 – Credentials in Files
```
Discovery
```bash
T1083 – File and Directory Discovery

T1018 – Remote System Discovery
```
Lateral Movement
```bash
T1021.002 – SMB/Windows Admin Shares
```
Command and Control
```bash
T1071.001 – Web-based Command and Control
```

