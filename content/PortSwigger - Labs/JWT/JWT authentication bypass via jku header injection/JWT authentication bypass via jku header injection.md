
# #PortSwigger 


![[Pasted image 20260531224519.png]]


## Lab Description
> This lab uses a JWT-based mechanism for handling sessions. The server supports the `jku` parameter in the JWT header. However, it fails to check whether the provided URL belongs to a trusted domain before fetching the key.
> 
> **Objective:** Forge a JWT that gives you access to the admin panel at `/admin`, then delete the user `carlos`.
> 
> - **Your credentials:** `wiener:peter`

---
---


## Step 1: Understanding the jku Header

**JKU (JSON Key URL)** is a JWT header parameter that points to a URL containing the JSON Web Key Set (JWKS) used to verify the token's signature.

**Normal flow:**
1. Server receives JWT with `jku` header
2. Server fetches JWKS from the provided URL
3. Server uses the key from JWKS to verify signature

**The vulnerability:** The server does **not validate** that the `jku` URL belongs to a trusted domain. An attacker can host their own JWKS and sign tokens with their own private key.

**In this lab:**
- Server supports `jku` header
- No whitelist check on the URL
- We can host a malicious JWK Set on the exploit server

---

## Step 2: Reconnaissance

### Step 2.1: Log in to Your Account

1. Log in with `wiener:peter`
2. Capture the `GET /my-account` request after login

![[Pasted image 20260531225014.png]]

### Step 2.2: Test Admin Access

In Burp Repeater, change the path to `/admin`:
```
GET /admin HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: session=YOUR_JWT
```

![[Pasted image 20260531225049.png]]

![[Pasted image 20260531225119.png]]
**Response:** `403 Forbidden` (only administrator can access)

---

## Step 3: Generating an RSA Key Pair

### Step 3.1: Install JWT Editor Extension

1. Go to **Burp Suite** --> **Extender** --> **BApp Store**
2. Search for **"JWT Editor"**
3. Click **Install**

![[Pasted image 20260531225229.png]]

### Step 3.2: Create an RSA Key

1. Go to **JWT Editor** tab --> **Keys** tab
2. Click **New RSA Key**
3. Click **Generate** (auto-generates a key pair)
![[Pasted image 20260531225351.png]]

4. Click **OK**

The key appears in the list with a unique `kid` (Key ID).

---

## Step 4: Hosting the JWK Set on Exploit Server

### Step 4.1: Copy Public Key as JWK

1. In **JWT Editor Keys** tab, right-click on your RSA key
2. Select **Copy Public Key as JWK**

The JWK looks like:
```
{
    "kty": "RSA",
    "e": "AQAB",
    "kid": "893d8f0b-061f-42c2-a4aa-5056e12b8ae7",
    "n": "yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9mk6GPM9gNN4Y_qTVX67WhsN3JvaFYw"
}
```

### Step 4.2: Create JWK Set on Exploit Server

1. Go to the **Exploit server**
2. In the **Body** section, paste a JWK Set template:
```
{
    "keys": [
        
    ]
}
```

![[Pasted image 20260531225823.png]]

3. Paste your copied JWK into the `keys` array:
![[Pasted image 20260531225804.png]]

4. Click **Store**

### Step 4.3: Note the JWKS URL

Your JWK Set is now available at:
```
https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/
```


---

## Step 5: Modifying and Signing the JWT

### Step 5.1: Locate the Request in Repeater

In Burp Repeater, find the `GET /admin` request.

### Step 5.2: Switch to JSON Web Token Editor

Look for the **"JSON Web Token"** tab within Repeater (added by JWT Editor extension).

### Step 5.3: Modify the JWT Header

Original header:
```
{
    "alg": "RS256",
    "typ": "JWT",
    "kid": "original-kid"
}
```

Modified header:
```
{
    "alg": "RS256",
    "typ": "JWT",
    "kid": "893d8f0b-061f-42c2-a4aa-5056e12b8ae7",
    "jku": "https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/"
}
```

![[Pasted image 20260531230225.png]]

**Changes:**
- `kid` → Set to your key's `kid`
- `jku` → Add URL pointing to your JWK Set

### Step 5.4: Modify the Payload

Change the `sub` claim to `administrator`:
```
{
    "sub": "administrator",
    "iat": 1516239022
}
```

![[Pasted image 20260531231023.png]]

### Step 5.5: Sign the Token

1. Click **Sign** at the bottom of the tab
2. Select the **RSA key** you generated earlier
3. Ensure **"Don't modify header"** is selected    
4. Click **OK**
![[Pasted image 20260531231201.png]]


The JWT is now signed with your private key. The server will fetch your public key from the `jku` URL to verify it.

### Step 5.6: Send the Request

![[Pasted image 20260531232049.png]]

Click **Send** in Repeater.

![[Pasted image 20260531232031.png]]
**Response:** `200 OK` - Admin panel accessed!

---

## Step 6: Deleting User Carlos

### Step 6.1: Find the Delete Endpoint

In the admin panel response, look for the delete link:
```
<a href="/admin/delete?username=carlos">Delete</a>
```

![[Pasted image 20260531232135.png]]

### Step 6.2: Send the Delete Request

Modify the request to:
```
GET /admin/delete?username=carlos HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: session=[FORGED_JWT]
```

![[Pasted image 20260531232355.png]]

**Response:**
![[Pasted image 20260531232257.png]]


---

## Step 7: Lab Solved

Success message displayed:

![[Pasted image 20260531232412.png]]

---
---
