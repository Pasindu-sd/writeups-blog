
# #HTB 


![[Pasted image 20251228193812.png|281]]



# HTB: Crocodile

**Machine IP:** `10.129.49.130`  
**Difficulty:** Easy  
**OS:** Linux

---

## Tools Used
- `nmap` - Port and service enumeration
- `rustscan` - Fast port discovery
- `ftp` - File transfer protocol client
- Web browser - Login interface access


---

## Step 1: Reconnaissance - Initial Port Scan

### FTP Service Discovery

Initial scan focused on port 21 (FTP)

**Result:**
![[Pasted image 20251228193835.png]]


**Key findings:**
- Anonymous FTP login **allowed** (vulnerability)
- Two interesting files discovered:
    - `allowed.userlist` (33 bytes)
    - `allowed.userlist.passwd` (62 bytes)

![[Pasted image 20251228194159.png]]





## Step 2: FTP Exploitation - Anonymous Access

### Connect to FTP Server

![[Pasted image 20251228194257.png]]


### Download Sensitive Files
![[Pasted image 20251228194501.png]]


### File Contents

```
cat allowed.userlist
```

```
cat allowed.userlist.passwd
```

![[Pasted image 20251228221646.png]]






## Step 3: Web Application Discovery

### Full Port Scan

```
rustscan -a 10.129.49.130
```

**Open ports discovered:**
- Port 21 (FTP)
- Port 80 (HTTP)

![[Pasted image 20260507002105.png]]

### HTTP Service Enumeration

```
sudo nmap 10.129.49.130 -sV -p 80
```

**Result:**
![[Pasted image 20260507002255.png]]






## Step 4: Web Login & Flag Capture

### Login Page Discovery

Navigate to `http://10.129.49.130/login.php`

Using credentials discovered from FTP:
- **Username:** `admin` (from allowed.userlist)
- **Password:** `rKXM59ESxesUFHAd` (from allowed.userlist.passwd)

![[Pasted image 20251228221853.png]]



### Dashboard Access

After successful login, redirected to `/dashboard/index.php`
**Flag obtained:** `c7110277ac44d78b6a9fff2232434d16`

![[Pasted image 20251228221815.png]]






## Step 5: Machine Owned

![[Pasted image 20251228221924.png]]


## Flags

|Flag|Value|
|---|---|
|User/Root|`c7110277ac44d78b6a9fff2232434d16`|

_Note: This machine appears to have a single flag accessible after web login._



---- 

## Attack Chain Summary
```
Port Scan (21,80)
       ↓
Anonymous FTP Access
       ↓
Download allowed.userlist & allowed.userlist.passwd
       ↓
Extract Credentials (admin / rKXM59ESxesUFHAd)
       ↓
Web Login (login.php)
       ↓
Dashboard Access → Flag Captured
```


