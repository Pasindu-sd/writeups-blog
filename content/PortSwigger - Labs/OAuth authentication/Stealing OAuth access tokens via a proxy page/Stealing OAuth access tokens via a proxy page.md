
# #PortSwigger 


![[Pasted image 20260602223732.png]]


## Lab Description (from PortSwigger)

> This lab uses an OAuth service to allow users to log in with their social media account. Flawed validation by the OAuth service makes it possible for an attacker to leak access tokens to arbitrary pages on the client application.
> 
> **Objective:** Identify a secondary vulnerability in the client application and use this as a proxy to steal an access token for the admin user's account. Use the access token to obtain the admin's API key and submit the solution.
> 
> - **Your credentials:** `wiener:peter`
>     
> - The admin user will open anything you send from the exploit server and always has an active session with the OAuth service.
>     
> - Victim uses Chrome — test your exploit in Chrome.
>     

---

## Step 1: Understanding the Attack Chain

**This lab uses two vulnerabilities together:**
1. OAuth `redirect_uri` path traversal (like the previous lab)
2. **postMessage** vulnerability on the comment form (acts as a proxy)

**The attack chain:**
1. OAuth redirect_uri points to a vulnerable page (`/post/comment/comment-form`)
2. That page sends `window.location.href` via `postMessage` to any origin (`*`)
3. Attacker's page listens for this message and extracts the access token
4. Token is sent to exploit server logs

**Why this is "Expert":**
- No open redirect available this time
- Need to find a different proxy page
- Use `postMessage` to exfiltrate the token

---

## Step 2: Reconnaissance

### Step 2.1: Study the OAuth Flow

1. Log in with `wiener:peter` via OAuth
2. Capture the `GET /auth` request in Burp:
```
GET /auth?client_id=YOUR-CLIENT-ID&redirect_uri=https://YOUR-LAB-ID.web-security-academy.net/oauth-callback&response_type=token&nonce=123456&scope=openid%20profile%20email
```

