
# #PortSwigger 



![[Pasted image 20251216184605.png]]




**Description**
	*Exploit flawed access controls that are based partly on the HTTP method of requests. Using the credentials `wiener:peter`, promote yourself to become an administrator.*


---
---


## Solution Steps


### Step 1: Log in as Administrator (Initial Recon)

First, log in using the provided admin credentials:
```
Username: administrator
Password: admin
```
Browse to the admin panel and observe how admin functionality works. Notice that promoting a user (e.g., `carlos`) is done via a specific HTTP request.





### Step 2: Capture the Admin Request

When you promote a user, capture the HTTP request in Burp Suite and send it to **Burp Repeater**.

The request likely looks something like:

![[Pasted image 20251216190356.png]]


![[Pasted image 20251216190615.png]]






### Step 3: Log in as Non-Admin User

Open a **private/incognito browser window** and log in with:
```
Username: wiener
Password: peter
```





### Step 4: Test the Admin Request with Non-Admin Cookie
1. Copy the non-admin user's session cookie from the incognito browser
2. In Burp Repeater, replace the admin session cookie with the non-admin cookie
3. Send the request

**Result:** Response says **"Unauthorized"** (as expected - non-admin can't perform admin actions)





### Step 5: Identify the Vulnerability

The access control is based on the **HTTP method**. The server checks:
- Does it authorize `POST` requests to `/admin-roles`?
- Does it authorize `GET` requests?

**Test:** Change the method from `POST` to `POSTX` (an invalid method)
**Result:** Response changes to **"missing parameter"** (not "Unauthorized"!)

This indicates that the method is part of the access control check. Certain methods bypass the check.





### Step 6: Convert to GET Method

In Burp Repeater:
1. Right-click on the request
2. Select **"Change request method"**
3. This converts `POST` to `GET`

**GET request becomes:
```
GET /admin-roles?username=carlos&role=administrator HTTP/1.1
```

![[Pasted image 20251216190745.png]]


![[Pasted image 20251216190843.png]]





### Step 7: Test GET Request with Non-Admin Cookie

Send the GET request with the non-admin cookie.
**Result:** The request is accepted! No "Unauthorized" response!





### Step 8: Promote Yourself

Change the `username` parameter from `carlos` to `wiener`:
```
GET /admin-roles?username=wiener&role=administrator HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: session=NON_ADMIN_SESSION_COOKIE
```


![[Pasted image 20251216191120.png]]
- Send the request.





### Step 9: Verify and Solve

The request succeeds! You have promoted `wiener` to administrator. The lab is marked as **Solved**.

![[Pasted image 20251216191139.png]]


