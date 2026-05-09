
# #PortSwigger 


![[Pasted image 20260509154438.png]]



**Description**
	*Use the exploit server to host an HTML page that uses a CSRF attack to change the viewer's email address. The CSRF token is **not tied to the user's session** — a token from one user can be used by another user.

**Credentials:**
- `wiener:peter`
- `carlos:montoya`*



## Solution Steps

### Step 1: Log in as wiener and Get a CSRF Token

1. Open Burp's browser and log in as `wiener:peter`
2. Go to the **"Update email"** form
3. Submit a test email change
4. **Intercept the request** in Burp Proxy
![[Pasted image 20260509155142.png]]

The request will look like:
![[Pasted image 20260509155930.png]]

5. **Make a note of the CSRF token value**
	- `csrf = UTs94TY17ejIzu2KtgvoqUWqqzjBLNOV  `
6. **Drop the request** (do not send it)






### Step 2: Verify Token Works for Another User
1. Open a **private/incognito browser window**
2. Log in as `carlos:montoya`
3. Send the email change request to **Burp Repeater**
![[Pasted image 20260509160205.png]]

4. Replace the CSRF token with the token you captured from wiener
![[Pasted image 20260509160247.png]]

5. Send the request

**Observe:** The request is **accepted**! The token from wiener works for carlos.
![[Pasted image 20260509161423.png]]






### Step 3: Understand the Limitations

- Tokens are **single-use** — once used, they cannot be used again
- You need a **fresh token** for the attack
- The token must be obtained before the victim uses their account






### Step 4: Create the CSRF Exploit

Since tokens are single-use, you need to:
1. Get a fresh token from your account (wiener)


2. Create an exploit that uses that specific token
3. Deliver the exploit before the token expires or is used

**Exploit HTML (without token first):**
```
<form method="POST" action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email">
    <input type="hidden" name="csrf" value="PUT_FRESH_TOKEN_HERE">
    <input type="hidden" name="email" value="hacked@attacker.com">
</form>
<script>
    document.forms[0].submit();
</script>
```
- PUT_FRESH_TOKEN_HERE *Replace With *






### Step 5: Get a Fresh Token

1. Log in as `wiener:peter` again (if needed)
2. Intercept a new email change request
![[Pasted image 20260509161650.png]]

3. **Copy the fresh CSRF token**
	- New CSRF = `4BBRLZKmRN8KKI3ElIpDlcah4gCboTNn`
4. **Drop the request** (don't use it yet)






### Step 6: Build the Complete Exploit

Replace `PUT_FRESH_TOKEN_HERE` with your fresh token:
- New CSRF = `4BBRLZKmRN8KKI3ElIpDlcah4gCboTNn`

```
<form method="POST" action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email">
    <input type="hidden" name="csrf" value="PUT_FRESH_TOKEN_HERE">
    <input type="hidden" name="email" value="hacked@attacker.com">
</form>
<script>
    document.forms[0].submit();
</script>
```






### Step 7: Upload to Exploit Server

1. Go to the **exploit server**
2. Paste your exploit HTML into the **"Body"** section    
3. Click **"Store"**

![[Pasted image 20260509161853.png]]





### Step 8: Test and Deliver

1. **Test on yourself first** — click "View exploit"
2. Verify your (wiener's) email changes
3. **Change the email address** to something different (not your own)
4. **Get another fresh token** (the previous one was used in testing)
5. **Update the exploit** with the new fresh token
6. Click **"Deliver to victim"**


![[Pasted image 20260509161915.png]]
