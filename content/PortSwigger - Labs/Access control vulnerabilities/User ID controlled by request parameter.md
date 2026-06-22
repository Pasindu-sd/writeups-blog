
# #PortSwigger 



![[Pasted image 20251216194826.png]]




## Description

This lab has a **horizontal privilege escalation** vulnerability on the user account page. The user ID is controlled by a request parameter.

**Credentials:** `wiener:peter`

**Objective:** Obtain the API key for the user `carlos` and submit it as the solution.


---
---

## Solution Steps

### Step 1: Log in to Your Account

1. Log in using `wiener:peter`
2. Go to your **account page**






### Step 2: Examine the URL

Observe that the URL contains your username in an `id` parameter:

```
https://YOUR-LAB-ID.web-security-academy.net/my-account?id=wiener
```





### Step 3: Send Request to Burp Repeater

1. Capture the request to `/my-account?id=wiener`    
2. Send it to **Burp Repeater**

![[Pasted image 20251216195351.png]]





### Step 4: Change the id Parameter

Change the `id` parameter from `wiener` to `carlos`:

```
GET /my-account?id=carlos HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: session=YOUR_SESSION_COOKIE
```

![[Pasted image 20251216195423.png]]





### Step 5: Send the Request

Send the modified request.

**Observe:** The response contains **carlos's account page**, including his API key!





### Step 6: Extract the API Key

From the response, find carlos's API key. It will look something like:

```
API Key: BESwkplUqqep99DA0HPoEsiJ2wXUxbw6
```

![[Pasted image 20251216195506.png]]





### Step 7: Submit the API Key

1. Go back to the lab page
2. Enter carlos's API key in the submission field
3. Click **"Submit solution"**
4. The lab is marked as **Solved**

![[Pasted image 20251216195524.png]]


---
