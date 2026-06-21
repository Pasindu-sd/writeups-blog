
# #PortSwigger 



![[Pasted image 20251219011319.png]]


## Lab Description

> This lab uses a JWT-based mechanism for handling sessions. The server supports the `jwk` parameter in the JWT header, which embeds the verification key directly in the token. However, it > fails to check whether the provided key came from a trusted source.
> 
> **Objective:** Modify and sign a JWT that gives you access to the admin panel at `/admin`, then > delete the user `carlos`.
> 
> **Credentials:** `wiener:peter`

---
---

### Step 1: Install JWT Editor Extension

1. Go to **Burp Suite** → **Extender** → **BApp Store**
2. Search for **"JWT Editor"**
3. Click **Install**

---

### Step 2: Log In and Capture the JWT

1. Log in as `wiener:peter`
2. In Burp Proxy, find the `GET /my-account` request
3. Send it to Repeater

![[Pasted image 20251219011446.png]]

---

### Step 3: Test Admin Access

1. In Repeater, change the path to `/admin`
2. Send the request
3. Access denied


![[Pasted image 20251219011545.png]]


---

### Step 4: Generate a New RSA Key

1. Go to **JWT Editor** tab → **Keys** tab
2. Click **New RSA Key**
3. Click **Generate** (auto-generates key pair)
4. Click **OK** to save

![[Pasted image 20251219011628.png]]



---

### Step 5: Modify and Sign the JWT

1. In Repeater, switch to the **JSON Web Token** tab
2. In the **Payload**, change `sub` from `wiener` to `administrator`
3. At the bottom, click **Attack** → **Embedded JWK**
4. Select your newly generated RSA key
5. Click **OK**


![[Pasted image 20251219011820.png]]


**What this does:**
- Adds a `jwk` parameter to the header containing your public key
- Signs the token with your private key

**Resulting header:**
```
{
  "alg": "RS256",
  "kid": "your-key-id",
  "jwk": {
    "kty": "RSA",
    "e": "AQAB",
    "n": "your-public-key",
    "kid": "your-key-id"
  }
}
```


![[Pasted image 20251219012159.png]]


---

### Step 6: Send the Forged Request

1. Click **Send** in Repeater
2. Access granted to `/admin`


![[Pasted image 20251219012233.png]]



---

### Step 7: Delete Carlos

1. In the response, find the delete link:
```
/admin/delete?username=carlos
```

2. Change the path to: `/admin/delete?username=carlos`
3. Send the request


![[Pasted image 20251219012323.png]]


---

## Step 8: Lab Solved


![[Pasted image 20251219012342.png]]


---
---
