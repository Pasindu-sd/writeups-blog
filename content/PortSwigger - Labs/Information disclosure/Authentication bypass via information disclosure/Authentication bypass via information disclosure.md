
# #PortSwigger 


![[Pasted image 20260528235106.png]]


## Lab Description

> This lab's administration interface has an authentication bypass vulnerability, but it is impractical to exploit without knowledge of a custom HTTP header used by the front-end.
> 
> **Objective:** Obtain the header name then use it to bypass the lab's authentication. Access the admin interface and delete the user `carlos`.
> 
> **Credentials:** `wiener:peter`

---
---

## Step 1: Understanding the Vulnerability

**This lab combines two concepts:**
1. **Information disclosure** — The `TRACE` HTTP method reveals a custom header
2. **Authentication bypass** — The custom header can be manipulated to gain admin access

**The authentication mechanism:**
- The admin panel (`/admin`) checks if the request comes from `127.0.0.1` (localhost)
- A front-end proxy adds a custom header with the client's IP address
- The back-end server trusts this header to determine access

**The vulnerability:**
- `TRACE` method echoes back the request, revealing the custom header name
- Attacker can add this header with `127.0.0.1` to bypass authentication

---

## Step 2: Reconnaissance

### Step 2.1: Log in to Your Account

1. Log in with credentials: `wiener:peter`
2. Note that you cannot access `/admin` directly
![[Pasted image 20260528235504.png]]

### Step 2.2: Test Access to Admin Panel

In **Burp Repeater**, send a request to `/admin`:

![[Pasted image 20260528235753.png]]

This reveals:
- Admin access is restricted
- Two ways to access: administrator role OR local IP (127.0.0.1)


---

## Step 3: Discovering the Custom Header Using TRACE

### Step 3.1: What is the TRACE Method?

`TRACE` is an HTTP method designed for debugging. It echoes back the exact request received by the server.

**Request:**
![[Pasted image 20260528235920.png]]

**Response:**
![[Pasted image 20260528235944.png]]

The server's response includes **any headers that were added by intermediate servers** (proxies, load balancers).

**Key finding:** The `X-Custom-IP-Authorization` header was automatically appended by the front-end proxy. It contains your IP address.

This header is likely used to determine the client's IP address for access control decisions.


---

## Step 4: Understanding the Authentication Bypass

**How authentication works:**
1. Front-end proxy receives client request
2. Proxy adds `X-Custom-IP-Authorization: client-ip` header
3. Back-end server checks this header
4. If header contains `127.0.0.1` (localhost), access is granted

**Attack:**
- If we can add `X-Custom-IP-Authorization: 127.0.0.1` to our requests
- The back-end server will think we are localhost
- We bypass authentication and access `/admin`

---

## Step 5: Configuring Burp to Add the Custom Header

### Step 5.1: Open Match and Replace Settings

1. In Burp Suite, go to **Proxy --> Options**
2. Scroll to **Match and replace** section
3. Click **Add** to create a new rule

### Step 5.2: Configure the Rule

|Setting|Value|
|---|---|
|**Match**|(leave empty)|
|**Type**|Request header|
|**Replace**|`X-Custom-IP-Authorization: 127.0.0.1`|

![[Pasted image 20260529000343.png]]

### Step 5.3: Test the Rule

1. Click **Test**
2. In the **Auto-modified request** field, verify Burp adds the header
3. Click **OK**

**How it works:**
- This rule adds the custom header to **every request** sent through Burp Proxy
- No match condition means it always applies
- The header tells the server the request comes from localhost


---

## Step 6: Accessing the Admin Panel

### Step 6.1: Browse to the Homepage

With the match/replace rule active:
1. Turn on **Intercept** or just browse normally
2. Navigate to the lab homepage
3. Burp automatically adds the header to every request

### Step 6.2: Access /admin

Now navigate to:
```
https://YOUR-LAB-ID.web-security-academy.net/admin
```

You should see the admin panel with a list of users:

![[Pasted image 20260529000648.png]]


---

## Step 7: Deleting User Carlos

### Step 7.1: Find the Delete Endpoint

Look for the delete action, typically:
```
/admin/delete?username=carlos
```


### Step 7.2: Send the Delete Request

Click the **Delete** button next to `carlos` or send the appropriate request:

![[Pasted image 20260529001027.png]]


### Step 7.3: Verify Deletion

After deletion, the lab should display the success message.
![[Pasted image 20260529001037.png]]


---

## Step 8: Lab Solved

Success message displayed:

![[Pasted image 20260529001054.png]]

---
---

