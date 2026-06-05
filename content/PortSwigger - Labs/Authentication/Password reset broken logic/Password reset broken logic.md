
# #PortSwigger 


![[Pasted image 20251211215338.png]]


## Lab Description

> This lab's password reset functionality is vulnerable. To solve the lab, reset Carlos's password then log in and access his "My account" page.
> 
> - **Your credentials:** `wiener:peter
> - **Victim's username:** `carlos`

---
---

## Step 1: Understanding the Vulnerability

**The password reset functionality has a critical logic flaw:**
- The reset token is sent in the email (as a URL parameter)
- However, when submitting the new password, the token is **NOT validated**
- Any user can reset any other user's password by simply changing the `username` parameter

**The attack:**
1. Request a password reset for any account
2. Capture the password reset submission request
3. Remove the token parameter (it's not checked)
4. Change the `username` to the victim
5. Set a new password of your choice

---

## Step 2: Reconnaissance

### Step 2.1: Request Password Reset

1. Click **"Forgot your password?"**
2. Enter your username: `wiener`
3. Click **Submit**

![[Pasted image 20251211215547.png]]

### Step 2.2: Check the Email

1. Go to the **Email client** (on exploit server)
2. Find the password reset email

![[Pasted image 20251211215643.png]]
- The token is: `36ovcusirdtmgh6617x8t6r0wf1b9my3`

### Step 2.3: Reset Your Password

Click the link and reset your password to anything (e.g., `1234`).

![[Pasted image 20251211215712.png]]


---

## Step 3: Analyzing the Password Reset Request

### Step 3.1: Capture the Password Reset Submission

In Burp Proxy, find the `POST /forgot-password` request that submits the new password.

![[Pasted image 20251211215805.png]]

### Step 3.2: Send to Repeater

Right-click --> **Send to Repeater**

---

## Step 4: Testing the Vulnerability

### Step 4.1: Try Removing the Token

In Repeater, delete the token value:

**Modified request:**
```
POST /forgot-password?temp-forgot-password-token= HTTP/2
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

temp-forgot-password-token=&username=wiener&new-password-1=1234&new-password-2=1234
```
**Result:** The password reset still works!  The token is **not being validated**.

### Step 4.2: Change the Username to Carlos

Now modify the request to target Carlos:
```
POST /forgot-password?temp-forgot-password-token= HTTP/2
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

temp-forgot-password-token=&username=carlos&new-password-1=password123&new-password-2=password123
```


### Step 4.3: Send the Request

Click **Send**. Carlos's password is now changed to `password123`.

---

## Step 5: Logging in as Carlos

### Step 5.1: Use the New Password

1. Go to the login page
2. Enter:
    - **Username:** `carlos`
    - **Password:** `password123` (or whatever you set)

### Step 5.2: Access My Account

After successful login, navigate to **My account**.

---

## Step 6: Lab Solved

Success message displayed:

![[Pasted image 20251211215904.png]]

---
---
