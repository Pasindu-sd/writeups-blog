
# #PortSwigger 



![[Pasted image 20251219110951.png]]


## Lab Description

> This lab uses a JWT-based mechanism for handling sessions. The server uses the `kid` parameter > in the JWT header to fetch the relevant key from its filesystem.
> 
> **Objective:** Forge a JWT that gives you access to the admin panel at `/admin`, then delete the > user `carlos`.
> 
> **Credentials:** `wiener:peter`


---
---

### Step 1: Install JWT Editor Extension

1. Go to **Burp Suite** -> **Extender** -> **BApp Store**
2. Search for **"JWT Editor"**
3. Click **Install**

---

### Step 2: Log In and Capture the JWT

1. Log in as `wiener:peter`
2. In Burp Proxy, find the `GET /my-account` request
3. Send it to Repeater

---

### Step 3: Test Admin Access

1. In Repeater, change the path to `/admin`
2. Send the request
3. Access denied

![[Pasted image 20251219111135.png]]


---

### Step 4: Generate a Symmetric Key with Empty Secret

1. Go to **JWT Editor** tab -> **Keys** tab
2. Click **New Symmetric Key**
3. Click **Generate** (auto-generates a key)
4. **Replace the `k` property with an empty string:**
```
{
  "kty": "oct",
  "kid": "generated-id",
  "k": ""
}
```

5. Click **OK**

---

### Step 5: Modify the JWT Header

1. In Repeater, switch to the **JSON Web Token** tab
2. In the **Header**, change the `kid` parameter:

**Original:**
```
"kid": "8f7c4d80-fbdc-4fc6-a040-9f61c4981eb3"
```

**Modified (path traversal):**
```
"kid": "../../../../../../../dev/null"
```


---

### Step 6: Modify the Payload

1. In the **Payload**, change the `sub` claim:
    - From: `wiener`        
    - To: `administrator`


---

### Step 7: Sign the Token

1. At the bottom, click **Sign**
2. Select the **symmetric key** you created (with empty `k`)
3. Ensure **"Don't modify header"** is selected
4. Click **OK**

---

### Step 8: Send the Forged Request

1. Click **Send** in Repeater
2. Access granted to `/admin`

![[Pasted image 20251219111419.png]]



---

### Step 9: Delete Carlos

1. In the response, find the delete link:
```
/admin/delete?username=carlos
```

2. Change the path to: `/admin/delete?username=carlos`
3. Send the request


---

## Step 10: Lab Solved

![[Pasted image 20251219111450.png]]


---
---
