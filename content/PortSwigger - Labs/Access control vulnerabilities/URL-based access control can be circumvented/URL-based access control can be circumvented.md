
# #PortSwigger 



![[Pasted image 20251216180808.png]]



## Description

This website has an unauthenticated admin panel at `/admin`, but a **front-end system** blocks external access to that path. However, the back-end application is built on a framework that supports the `X-Original-URL` header.

**Objective:** Access the admin panel and delete the user `carlos`.

## Vulnerability Explanation

A **front-end proxy** or load balancer blocks direct access to `/admin`. However, the back-end framework (e.g., Spring Boot, [ASP.NET](https://asp.net/)) supports the `X-Original-URL` header, which overrides the request path.

By setting `X-Original-URL: /admin`, you can bypass the front-end restriction and directly access the back-end admin functionality.


---
---


## Solution Steps

### Step 1: Test Direct Access to /admin

First, try to load `/admin` normally:
```
https://YOUR-LAB-ID.web-security-academy.net/admin
```

![[Pasted image 20251216181128.png]]

**Observe:** The request is **blocked**. The response is very plain (no HTML formatting), suggesting it comes from a front-end system (not the actual application).




### Step 2: Send Request to Burp Repeater

1. Capture the request to `/admin`
2. Send it to **Burp Repeater**

![[Pasted image 20251216181207.png]]






### Step 3: Test the X-Original-URL Header

Change the URL in the request line to `/` (or anything else) and add the `X-Original-URL` header:

```
GET / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
X-Original-URL: /invalid
```

**Observe:** The application returns a **"not found"** response. This confirms that the back-end system is processing the URL from the `X-Original-URL` header.





### Step 4: Access /admin Using X-Original-URL

Change the `X-Original-URL` header value to `/admin`:

![[Pasted image 20251216181535.png]]

**Observe:** The admin page loads successfully! The front-end restriction has been bypassed.






### Step 5: Delete carlos

To delete the user `carlos`:
1. Add the query parameter `?username=carlos`
2. Change the `X-Original-URL` path to `/admin/delete`

![[Pasted image 20251216181935.png]]

- So we need convert POST method
```
GET /?username=carlos HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
X-Original-URL: /admin/delete
```

![[Pasted image 20251216182034.png]]






### Step 6: Solve the Lab

Send the request. The user `carlos` is deleted, and the lab is marked as **Solved**.

```
# Missing The proof of Solved Lab
```

---
