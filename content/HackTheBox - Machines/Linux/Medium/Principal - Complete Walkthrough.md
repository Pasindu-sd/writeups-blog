
# #HTB 


![[Pasted image 20260805234633.png|281]]


# HTB: Principal

**Machine IP:** `10.129.244.220`  
**Difficulty:** Medium  
**OS:** Linux

---

## Tools Used
- `nmap` - Port discovery
- `feroxbuster` - Web directory enumeration
- `Burp Suite` - Request inspection
- `jwcrypto` / `requests` (Python) - JWT forging
- `hydra` - SSH password spraying
- `ssh-keygen` - Certificate signing
- `ssh` - Remote access

---
---

## Step 1: Reconnaissance - Port Scanning

### Nmap Results

```text
nmap -sCV -p 22,8080 10.129.244.220
```

**Open ports discovered:**
- Port 22 (SSH) - OpenSSH 9.6p1 Ubuntu 3ubuntu13.14
- Port 8080 (HTTP) - Jetty web server


![[Pasted image 20260805223420.png]]


### Technology Stack
- **Web Server:** Jetty (Java)
- **Authentication:** pac4j-jwt v6.0.3
- **Framework:** Spring Boot-based


---

## Step 2: Web Enumeration - Principal Internal Platform

### Website Overview

Visiting `http://10.129.244.220:8080` redirects to `/login`, revealing "Principal Internal Platform" - a unified operations dashboard.

![[Pasted image 20260805223441.png]]


### Application Analysis

Taking a look at the page source, it loads `static/js/app.js`, which handles the auth. At the top, it defines the JWT structure in comments and some endpoints:

![[Pasted image 20260805223825.png]]

**Key observations from the JavaScript:**
- JWT tokens are **JWE-encrypted** using RSA-OAEP-256 + A128GCM
- Public key available at `/api/auth/jwks`
- Inner JWT is signed with RS256
- Claims include `sub` (username) and `role` (ROLE_ADMIN, ROLE_MANAGER, ROLE_USER)

![[Pasted image 20260805223540.png]]


---

## Step 3: CVE-2026-29000 - pac4j-jwt Authentication Bypass

### Vulnerability Discovery

Research revealed **CVE-2026-29000** - a critical vulnerability in pac4j-jwt versions prior to 6.3.3 (the version used is 6.0.3).

**Vulnerability summary:**
- The library decrypts JWE tokens, then attempts to extract a signed JWT
- If the inner JWT has no signature (`alg: none`), `toSignedJWT()` returns `null`
- The signature verification step is **skipped entirely** when `signedJWT` is `null`
- Attackers with the server's public RSA key can forge arbitrary authentication tokens

### Exploitation Script

Create a Python script to forge the token using the public key from `/api/auth/jwks`:


![[Pasted image 20260805225532.png]]


![[Pasted image 20260805225516.png]]


![[Pasted image 20260805225557.png]]


![[Pasted image 20260805230825.png]]


![[Pasted image 20260805230847.png]]


#### Password Spray

I’ll save the usernames from the dashboard into `users.txt` and spray the encryptionKey as a password against SSH. `netexec` can do this spray, but it runs serially, waiting for each attempt to timeout before doing the next, so `hydra` is a better tool here:

![[Pasted image 20260805230810.png]]


It finds a match using the `encryptionKey` for the svc-deploy account.

#### Shell

I’ll use the creds to get a shell using SSH:

![[Pasted image 20260805230944.png]]


And grab the user flag:

![[Pasted image 20260805231005.png]]


![[Pasted image 20260805233338.png]]


![[Pasted image 20260805233314.png]]



![[Pasted image 20260805233530.png]]


![[Pasted image 20260805233723.png]]


![[Pasted image 20260805233743.png|700]]


