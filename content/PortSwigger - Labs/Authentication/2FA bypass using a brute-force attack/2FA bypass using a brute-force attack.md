
# #PortSwigger 


![[Pasted image 20260604142330.png]]


## Lab Description (from PortSwigger)

> This lab's two-factor authentication is vulnerable to brute-forcing. You have already obtained a valid username and password, but do not have access to the user's 2FA verification code. To solve the lab, brute-force the 2FA code and access Carlos's account page.
> 
> - **Victim's credentials:** `carlos:montoya`
>     
> - **Note:** The verification code resets while you're running your attack. You may need to repeat this attack several times before you succeed.

---
---

## Step 1: Understanding the Vulnerability

**The 2FA mechanism:**
- User logs in with username/password
- Server sends 4-digit verification code
- User must submit correct code within a time window

**The vulnerabilities:**
1. **No rate limiting** on 2FA code attempts
2. **Short code length** (4 digits = 10,000 possibilities)
3. **Code resets after timeout**, but can be brute-forced

**The challenge:**
- After 2 wrong codes, the user is logged out
- Need to automate re-login before each code attempt

---

## Step 2: Reconnaissance

### Step 2.1: Test the 2FA Flow

1. Log in as `carlos:montoya`
2. Capture the `POST /login` request
![[Pasted image 20260604143108.png]]

3. Capture the `GET /login2` (2FA page)
![[Pasted image 20260604143125.png]]

4. Capture the `POST /login2` (code submission)
![[Pasted image 20260604143052.png]]

### Step 2.2: Observe the Behavior

- Submit wrong 2FA code --> error message
- Submit wrong code twice --> logged out, need to re-authenticate    
- This is why we need a **macro** to auto-login before each attempt

![[Pasted image 20260604143331.png]]


---

## Step 3: Creating a Burp Macro for Auto-Login

### Step 3.1: Open Session Handling Rules

1. Go to **Burp Suite** --> **Settings** (gear icon)
2. Select **Sessions** from the left menu
![[Pasted image 20260604143456.png]]

3. In **Session Handling Rules**, click **Add**
![[Pasted image 20260604143520.png]]

### Step 3.2: Configure the Rule

**Rule name:** `2FA Auto-Login`

**Scope tab:**
- **URL Scope:** Include all URLs (or specific lab domain)
- **Tool Scope:** Select **Intruder**

![[Pasted image 20260604143828.png]]


**Details tab --> Rule Actions:**
1. Click **Add** --> **Run a macro**
2. Click **Add** to create a new macro

![[Pasted image 20260604144000.png]]

### Step 3.3: Record the Macro

Select the following 3 requests in order:

| Step | Request       | Purpose                                    |
| ---- | ------------- | ------------------------------------------ |
| 1    | `GET /login`  | Load login page (get CSRF token if needed) |
| 2    | `POST /login` | Submit username/password                   |
| 3    | `GET /login2` | Get 2FA page (establish session)           |
|      |               |                                            |

**How to select:**
1. In Macro Recorder, you'll see a list of recent requests
2. Select the 3 requests in the correct chronological order
3. Click **OK**

![[Pasted image 20260604144216.png]]

### Step 3.4: Test the Macro

1. In Macro Editor, click **Test macro**
2. Verify the final response contains the 2FA code input page
3. If successful, click **OK** through all dialogs

![[Pasted image 20260604144244.png]]


---

## Step 4: Configuring the Intruder Attack

### Step 4.1: Capture the 2FA Request

Find the `POST /login2` request containing the `mfa-code` parameter:
```
POST /login2 HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: session=...
Content-Type: application/x-www-form-urlencoded

mfa-code=1234
```

![[Pasted image 20260604144430.png]]
### Step 4.2: Send to Intruder

1. Highlight the `1234` value
2. Right-click --> **Send to Intruder**

### Step 4.3: Configure Payload

|Setting|Value|
|---|---|
|Payload type|Numbers|
|Number range|0 - 9999|
|Step|1|
|Min integer digits|4|
|Max integer digits|4|
|Max fraction digits|0|
![[Pasted image 20260604144700.png]]

This generates all 4-digit codes: `0000` to `9999`

### Step 4.4: Configure Resource Pool

**Critical:** Set **Maximum concurrent requests** to **1**
1. Go to **Resource pool** tab
2. Create new pool or edit existing
3. Set **Maximum concurrent requests** = **1**

**Why?** If requests run in parallel, session cookies will conflict. Sequential requests ensure each login session is used correctly.

---

## Step 5: Starting the Attack

### Step 5.1: Launch Intruder

Click **Start attack**

### Step 5.2: Monitor for Success

Look for a request with:
- **Status code:** `302 Found`
- **Or:** `Location: /my-account`

### Step 5.3: Handle Code Resets

**Note from the lab:** The verification code resets while you're running the attack. You may need to restart the attack if the code expires.

**Why this happens:**
- The 2FA code has a short expiration time (e.g., 30-60 seconds)
- Brute-forcing 10,000 codes takes time
- The code may change during the attack

**Solution:**
- If you reach the end without success, restart the attack
- The correct code will be a new number each time
- Eventually, Intruder will hit the correct code before it expires

---

## Step 6: Accessing Carlos's Account

### Step 6.1: Get the Successful Response

When a `302` response is found:
1. Right-click on that request
2. Select **Show response in browser**
3. Copy the URL

### Step 6.2: Load in Browser

Paste the URL into your browser.

### Step 6.3: Navigate to My Account

Click **My account** to verify access.

---

## Step 7: Lab Solved

Success message displayed: