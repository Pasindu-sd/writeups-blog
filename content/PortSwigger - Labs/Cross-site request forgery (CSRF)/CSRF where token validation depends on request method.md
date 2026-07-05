
# #PortSwigger 


![[Pasted image 20260508183131.png]]



### **Description**
	*Use the exploit server to host an HTML page that uses a CSRF attack to change the viewer's email address. The server only validates CSRF tokens for certain HTTP methods.
**Your credentials:** `wiener:peter`*
	*The server implements CSRF protection by validating a **CSRF token** for `POST` requests. However, when the request method is changed to `GET`, the server **does not validate** the token, making it vulnerable to CSRF.*

---
---

## Solution Steps

### Step 1: Log In and Capture the Request

1. Log in to your account using `wiener:peter`
![[Pasted image 20260508221912.png]]

2. Go to **"Update email"** form
![[Pasted image 20260508221956.png]]

3. Submit a test email change `haha@haha.com`
4. Capture the request in **Burp Proxy** (HTTP history)

The POST request looks like:
![[Pasted image 20260508222529.png]]






### Step 2: Test Token Validation

Send the request to **Burp Repeater** and:
1. Change the `csrf` parameter value
2. Observe that the request is **rejected** (CSRF token validation works for POST)

![[Pasted image 20260508222727.png]]






### Step 3: Convert to GET Request

1. Right-click on the request in Repeater
2. Select **"Change request method"**
3. The request becomes a GET request:

![[Pasted image 20260508224110.png]]






### Step 4: Test GET Request

Send the GET request and observe that:
- The CSRF token is **no longer verified**
- The email change is successful even with an invalid token

**You can even remove the `csrf` parameter entirely:**

![[Pasted image 20260508223951.png]]

![[Pasted image 20260508224345.png]]





### Step 5: Generate CSRF PoC

Use the following HTML template (note: no `csrf` parameter needed):
```
<form action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email">
    <input type="hidden" name="email" value="anything%40web-security-academy.net">
</form>
<script>
    document.forms[0].submit();
</script>
```

- **Important:** The form uses `method="GET"` by default (or you can specify it).





### Step 6: Upload to Exploit Server

1. Go to the **exploit server**    
2. Paste your exploit HTML into the **"Body"** section
3. Click **"Store"**

![[Pasted image 20260508224716.png]]
- Use - `anything@web-security-academy.net`




### Step 8: Test the Exploit

1. Click **"View exploit"** to test it on yourself
2. Check that your email address changes
3. **Important:** Change the email address in your exploit so it doesn't match your own

![[Pasted image 20260508224914.png]]





### Step 9: Deliver to Victim

1. Click **"Deliver to victim"**
2. The victim's email address is changed    
3. The lab is marked as **Solved**

***Note:** You cannot register an email address that is already taken by another user. If you change your own email address while testing your exploit, make sure you use a different email address for the final exploit you deliver to the victim.*


![[Pasted image 20260508225058.png]]
