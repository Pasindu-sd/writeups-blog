
# #PortSwigger 


![[Pasted image 20260508230559.png]]



**Description**
	*Use the exploit server to host an HTML page that uses a CSRF attack to change the viewer's email address. The server validates the CSRF token **only if it is present** — if the token is omitted entirely, the request is accepted.

**Your credentials:** `wiener:peter`*




## Solution Steps


### Step 1: Log In and Capture the Request

1. Log in to your account using `wiener:peter`
2. Go to **"Update email"** form
3. Submit a test email change
4. Capture the request in **Burp Proxy** (HTTP history)

The POST request looks like:

![[Pasted image 20260508230817.png]]

![[Pasted image 20260508230853.png]]






### Step 2: Test Token Validation

Send the request to **Burp Repeater** and:
1. Change the `csrf` parameter value
2. Observe that the request is **rejected** (invalid token)

![[Pasted image 20260508231056.png]]






### Step 3: Delete the CSRF Token

**Delete the `csrf` parameter entirely** from the request:

![[Pasted image 20260508231132.png]]

 - **Observe:** The request is now **accepted** even with no token!





### Step 4: Generate CSRF PoC

Since the CSRF token is **not needed**, the exploit is simple.

Use the following HTML template (no csrf parameter):
```
<form method="POST" action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email">
    <input type="hidden" name="email" value="anything@web-security-academy.net">
</form>
<script>
    document.forms[0].submit();
</script>
```

**Replace:**
- `YOUR-LAB-ID` with your actual lab ID
- `hihi@hihi.com` with any unused email address

**Note:** There is **no** `csrf` input field because the token is omitted entirely!





### Step 6: Upload to Exploit Server

1. Go to the **exploit server**
2. Paste your exploit HTML into the **"Body"** section
3. Click **"Store"**

![[Pasted image 20260508231527.png]]





### Step 7: Test the Exploit

1. Click **"View exploit"** to test it on yourself
2. Check that your email address changes
3. **Important:** Change the email address in your exploit so it doesn't match your own

![[Pasted image 20260508231619.png]]




### Step 8: Deliver to Victim

1. Click **"Deliver to victim"**
2. The victim's email address is changed
3. The lab is marked as **Solved**

![[Pasted image 20260508231719.png]]

---

## Key Points

|Requirement|Value|
|---|---|
|HTTP Method|`POST`|
|Endpoint|`/my-account/change-email`|
|Parameters|`email=` **ONLY** (no csrf)|
|Auto-submit|`document.forms[0].submit()`|

---

## Important Notes
1. **Do NOT include a `csrf` parameter** in your exploit
2. **Test on yourself first** to verify it works
3. **Use a different email** for the final exploit

---
