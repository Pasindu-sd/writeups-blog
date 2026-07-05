
# #PortSwigger 


![[Pasted image 20260603143145.png]]


## Lab Description (from PortSwigger)

> This lab is subtly vulnerable to username enumeration and password brute-force attacks. It has an account with a predictable username and password, which can be found in the following wordlists:
> 
> - [Candidate usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames)
> - [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)
> 
> **Objective:** Enumerate a valid username, brute-force this user's password, then access their account page.

---
---

## Step 1: Understanding the Vulnerability

**The application has a subtle difference in error messages:**

|Scenario|Error Message|
|---|---|
|Invalid username|`Invalid username or password.` (with period)|
|Valid username + wrong password|`Invalid username or password` (with trailing space, no period)|

This is a classic **username enumeration** vulnerability. The difference is barely noticeable - a trailing space instead of a period - but enough to distinguish valid from invalid usernames.

---

## Step 2: Reconnaissance

### Step 2.1: Test the Login Page

1. Navigate to the login page
2. Submit an invalid username and password (e.g., `aaa:bbb`)
3. Capture the `POST /login` request in Burp

**Request:**
![[Pasted image 20260603143852.png]]

**Response:**
![[Pasted image 20260603143936.png]]

### Step 2.2: Test a Valid Username

If you know any valid username (e.g., `wiener` from the lab description), test it with a wrong password:
```
username=wiener&password=wrong
```

**Response:**
```
Invalid username or password 
```

Note the **trailing space** instead of a period at the end!

This difference allows us to enumerate valid usernames.

---

## Step 3: Username Enumeration

### Step 3.1: Send Request to Intruder

1. Highlight the `username` parameter value
2. Right-click --> **Send to Intruder**

### Step 3.2: Configure Payload

|Setting|Value|
|---|---|
|Payload position|`username=§aaa§`|
|Payload type|Simple list|
|Payload values|Candidate usernames (provided list)|

![[Pasted image 20260603144259.png]]

### Step 3.3: Configure Grep-Extract

We need to extract the error message to see the subtle difference.
1. Go to **Settings** tab

![[Pasted image 20260603144826.png]]

2. Under **Grep - Extract**, click **Add**
3. In the response preview, find the error message
4. Highlight the exact error message text (including the period or space)
5. Burp will automatically configure the extraction settings

![[Pasted image 20260603145104.png]]

**Example extraction configuration:**
- Start after: `Invalid username or password`
- Extract up to: the end of the line

### Step 3.4: Start the Attack

Click **Start attack**

### Step 3.5: Analyze Results

Sort by the extracted column. Look for responses that are **different** from the majority.

**Expected result:**
![[Pasted image 20260603152242.png]]


| Username | Extracted Message                               |
| -------- | ----------------------------------------------- |
| aaa      | `Invalid username or password.`                 |
| bbb      | `Invalid username or password.`                 |
| app01    | `Invalid username or password` (trailing space) |
| ddd      | `Invalid username or password.`                 |

`app01` is a valid username!

---

## Step 4: Password Brute-Force

### Step 4.1: Modify the Request

Change the request to target the valid username:
```
username=app01&password=§invalid-password§
```

### Step 4.2: Configure Password Payload

1. Clear the username list
2. Add the candidate passwords list
3. Set payload position on `password`

![[Pasted image 20260603152456.png]]

### Step 4.3: Configure Detection Method

This time, we can detect success by:
- **302 redirect** (instead of 200)
### Step 4.4: Start the Attack

Click **Start attack**

### Step 4.5: Analyze Results

Look for:
- A response with **status code 302** (redirect)
- Or a response containing `Location: /my-account`

![[Pasted image 20260603152659.png]]

**Expected result:** One password will succeed where others fail.

---

## Step 5: Logging In

### Step 5.1: Use the Found Credentials

| Field    | Value     |
| -------- | --------- |
| Username | `app01`   |
| Password | `letmein` |

### Step 5.2: Access My Account

![[Pasted image 20260603153115.png]]

After successful login, navigate to **My account**.

---

## Step 6: Lab Solved

Success message displayed:

![[Pasted image 20260603153132.png]]

---
---

