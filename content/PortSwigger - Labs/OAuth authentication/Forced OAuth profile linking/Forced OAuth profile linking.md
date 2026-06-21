
# #PortSwigger 



![[Pasted image 20251224143312.png]]


## Lab Description

>This lab gives you the option to attach a social media profile to your account so that you can log in via OAuth instead of using the normal username and password. Due to the insecure implementation of the OAuth flow by the client application, an attacker can manipulate this functionality to obtain access to other users' accounts.
>
>**Objective:** Use a CSRF attack to attach your own social media profile to the admin user's account on the blog website, then access the admin panel and delete carlos.
>
>**Credentials:**
- Blog website account: `wiener:peter`
- Social media profile: `peter.wiener:hotdog`

>The admin user will open anything you send from the exploit server and always has an active session on the blog website.


---
---


### Step 1: Log In and Attach Social Profile

1. Log in to the blog website with `wiener:peter`
2. Go to **My account**
3. Click **"Attach a social profile"**
4. Complete the OAuth flow using your social media credentials (`peter.wiener:hotdog`)
5. Verify the profile is attached

![[Pasted image 20251224143513.png]]


---

### Step 2: Turn on Intercept and Re-attach

1. Turn on **Proxy Intercept** in Burp
2. Click **"Attach a social profile"** again
3. Forward requests until you see the `GET /oauth-linking?code=...` request

**The request:**
```
GET /oauth-linking?code=STOLEN-CODE HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```

---

### Step 3: Copy the URL and Drop the Request

1. Right-click the request --> **Copy URL**
2. **Drop** the request (do NOT forward it)    

**Why drop?** The authorization code is single-use. Dropping keeps it valid.


![[Pasted image 20251224143611.png]]


![[Pasted image 20251224143817.png]]


---

### Step 4: Log Out

Log out of the blog website.

![[Pasted image 20251224144129.png]]


---

### Step 5: Create the Exploit

1. Go to the **Exploit server**
2. In the Body section, paste:
```
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/oauth-linking?code=STOLEN-CODE"></iframe>
```

**Replace:**
- `YOUR-LAB-ID` with your lab ID
- `STOLEN-CODE` with the code you copied

3. Click **Store**

![[Pasted image 20251224144744.png]]


---

### Step 6: Deliver to the Victim

1. Click **Deliver exploit to victim**
2. The victim's browser loads the iframe
3. The OAuth flow completes using your social media profile
4. The victim's account is now linked to your social profile    

---

### Step 7: Log In as Admin

1. Go back to the blog website
2. Click **"Log in with social media"**    
3. You are instantly logged in as **admin**!


![[Pasted image 20251224145431.png]]

---

### Step 8: Delete Carlos

1. Access the admin panel
2. Click **Delete** next to `carlos`

---

### Step 9: Lab Solved

![[Pasted image 20251224145457.png]]


---
---

