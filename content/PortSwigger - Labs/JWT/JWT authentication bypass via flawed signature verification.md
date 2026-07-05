
# #PortSwigger 



![[Pasted image 20251218230256.png]]


## Lab Description

> This lab uses a JWT-based mechanism for handling sessions. The server is insecurely configured to > accept unsigned JWTs.
> 
> **Objective:** Modify your session token to gain access to the admin panel at `/admin`, then > delete the user `carlos`.
> 
> **Credentials:** `wiener:peter`

---
---

## Step-by-Step Solution

---

### Step 1: Log In and Capture the JWT

1. Log in as `wiener:peter`
2. In Burp Proxy, find the `GET /my-account` request
3. Observe the `session` cookie - it's a JWT


![[Pasted image 20251218230914.png]]


---

### Step 2: Send to Repeater

1. Right-click the `GET /my-account` request
2. Select **Send to Repeater**

---

### Step 3: Test Admin Access

1. In Repeater, change the path to `/admin`
2. Send the request
3. Access denied (only administrators can access)

---

### Step 4: Modify the JWT

1. In the **JSON Web Token** tab (added by JWT Editor extension), select the **Payload**
2. Change the `sub` claim from `wiener` to `administrator`
3. Click **Apply changes**
4. Select the **Header**
5. Change the `alg` parameter from `RS256` to `none`
6. Click **Apply changes**


![[Pasted image 20251218231044.png]]

---

### Step 5: Remove the Signature

1. In the message editor, locate the JWT in the `session` cookie
2. Remove the **signature** part (the third segment after the second dot)
3. **Leave the trailing dot** after the payload

**Example JWT format:**
```
header.payload.signature
```

Change to:
```
header.payload.
```


---

### Step 6: Send the Forged Request

1. Send the request in Repeater
2. Access granted to `/admin    


![[Pasted image 20251218231121.png]]


---

### Step 7: Delete Carlos

1. In the response, find the delete link:
```
/admin/delete?username=carlos
```

2. Change the path to: `/admin/delete?username=carlos`
3. Send the request

![[Pasted image 20251218231212.png]]


---

### Step 8: Lab Solved

![[Pasted image 20251218231236.png]]

---
---
