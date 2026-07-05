
# #PortSwigger 



![[Pasted image 20251218224914.png]]


## Lab Description

> This lab uses a JWT-based mechanism for handling sessions. Due to implementation flaws, the > server doesn't verify the signature of any JWTs that it receives.
> 
> **Objective:** Modify your session token to gain access to the admin panel at `/admin`, then > delete the user `carlos`.
> 
> **Credentials:** `wiener:peter`

---
---

### Step 1: Log In and Capture the JWT

1. Log in as `wiener:peter`
2. In Burp Proxy, find the `GET /my-account` request
3. Observe the `session` cookie - it's a JWT

**Decoded JWT from the request:**
```
Header: {"kid": "...", "alg": "RS256"}
Payload: {"iss": "portswigger", "exp": 1766082227, "sub": "wiener"}
Signature: <some signature>
```


![[Pasted image 20251218225529.png]]


---

### Step 2: Send to Repeater

1. Right-click the `GET /my-account` request
2. Select **Send to Repeater

---

### Step 3: Test Admin Access

1. In Repeater, change the path to `/admin`
2. Send the request
3. Access denied (only administrators can access)


---

### Step 4: Modify the JWT Payload



![[Pasted image 20251218225551.png]]
![[Pasted image 20251218225614.png]]



1. In the **JSON Web Token** tab, select the **Payload**
2. Change the `sub` claim:
    - From: `wiener`
    - To: `administrator`
3. Click **Apply changes**

**Modified Payload:**
```
{"iss": "portswigger", "exp": 1766082227, "sub": "administrator"}
```


---

### Step 5: Send the Forged Request

1. **Do NOT modify the signature** - keep it as is
2. Send the request in Repeater
3. Access granted to `/admin`


![[Pasted image 20251218225855.png]]



---

### Step 6: Delete Carlos

1. In the response, find the delete link:
```
/admin/delete?username=carlos
```

2. Change the path to: `/admin/delete?username=carlos`
3. Send the request

![[Pasted image 20251218225926.png]]



---

## Step 7: Lab Solved

![[Pasted image 20251218230025.png]]

---
---

