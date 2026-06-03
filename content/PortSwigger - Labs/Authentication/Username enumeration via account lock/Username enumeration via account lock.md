
# #PortSwigger 


![[Pasted image 20260604002123.png]]


## Lab Description

> This lab is vulnerable to username enumeration. It uses account locking, but this contains a logic flaw. To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.
> 
> - **Candidate usernames** (provided list)
> - **Candidate passwords** (provided list)

---
---

## Step 1: Understanding the Vulnerability

**The account lock mechanism:**
- After multiple failed login attempts, an account becomes locked
- The error message changes when an account is locked

**The logic flaw:**
- The application reveals whether an account exists through the lock behavior
- Invalid username: No account lock message (account doesn't exist to lock)
- Valid username: After X failures, you get account lock message

**Enumeration method:**
1. Send multiple login attempts for each candidate username
2. If an account lock message appears → username exists
3. Once username is found, brute-force password

---

## Step 2: Reconnaissance

### Step 2.1: Test Account Lock Behavior

1. Submit an invalid username (e.g., `aaa`) with wrong password multiple times
2. Observe no account lock message
3. Submit a valid username (e.g., `wiener` from lab description) with wrong password multiple times
4. After several attempts, you get: `You have made too many incorrect login attempts`    

Valid usernames trigger account lock messages after enough failures.

![[Pasted image 20260604002804.png]]

---

## Step 3: Username Enumeration

### Step 3.1: Configure Cluster Bomb Attack

We need to test each username multiple times (enough to trigger account lock if it exists).

**Attack type:** Cluster Bomb (multiple payload sets)

**Request:**
![[Pasted image 20260604003106.png]]

### Step 3.2: Configure Payloads

|Position|Payload Type|Value|
|---|---|---|
|Position 1 (username)|Simple list|Candidate usernames|
|Position 2 (blank)|Null payloads|Generate 5 payloads|
**Position 1:**
![[Pasted image 20260604003300.png]]

**Position 2:**
![[Pasted image 20260604003402.png]]

**Why this works:**
- Each username will be tested 5 times (or enough to trigger account lock)
- The second payload position adds a dummy value (or nothing) to repeat each username
- 5 failed attempts should trigger account lock for valid usernames    

### Step 3.3: Start the Attack

Click **Start attack**

### Step 3.4: Analyze Results

Look for responses that are **longer** than others or contain a different error message.

**Expected result:**

|Username|Response Length|Error Message|
|---|---|---|
|aaa|2500|`Invalid username or password`|
|bbb|2500|`Invalid username or password`|
|**carlos**|**2700**|**`You have made too many incorrect login attempts`**|
|ddd|2500|`Invalid username or password`|

The username with the account lock message is valid: `carlos`

---

## Step 4: Password Brute-Force

### Step 4.1: Wait for Account Lock to Reset

**Important:** After triggering account lock, you need to wait (~1 minute) for the lock to reset before brute-forcing the password.

### Step 4.2: Configure Sniper Attack

**Attack type:** Sniper (single payload position)

**Request:**
```
POST /login HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

username=carlos&password=§invalid-password§
```

### Step 4.3: Configure Payload

|Setting|Value|
|---|---|
|Payload type|Simple list|
|Payload values|Candidate passwords|

### Step 4.4: Configure Grep-Extract or Grep-Match

Add a **Grep-Match** rule to detect successful login:
- `302` (redirect)
- Or `Location: /my-account`

Alternatively, use **Grep-Extract** to capture error messages and look for absence of error.

### Step 4.5: Start the Attack

Click **Start attack**

### Step 4.6: Analyze Results

Look for:
- **302 status code** (instead of 200)
- Or **no error message** in response

**Expected result:** One password returns a successful login response.

---

## Step 5: Logging In

### Step 5.1: Use the Found Credentials

|Field|Value|
|---|---|
|Username|`carlos`|
|Password|[found password]|

### Step 5.2: Access My Account

After successful login, navigate to **My account**.

---

## Step 6: Lab Solved

Success message displayed: