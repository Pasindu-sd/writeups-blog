
# #PortSwigger 


![[Pasted image 20260529111451.png]]


## Lab Description

> This lab makes an assumption about the privilege level of the user based on the HTTP Host header.
> 
> **Objective:** Access the admin panel and delete the user carlos.

---

## Step 1: Understanding the Vulnerability

**The application incorrectly assumes:**
- Requests from `localhost` (127.0.0.1) are from trusted administrators
- The `Host` header can be spoofed to impersonate localhost

**Why this works:**
- The server uses the `Host` header for access control decisions
- No validation ensures the request actually originated from localhost

---

## Step 2: Reconnaissance

### Step 2.1: Test Host Header Flexibility

Send a normal `GET /` request to Burp Repeater. Change the `Host` header to any arbitrary value:

![[Pasted image 20260529112506.png]]


**Change HOST:**
![[Pasted image 20260529112552.png]]


### Step 2.2: Discover Admin Panel

Check `robots.txt`:
![[Pasted image 20260529112859.png]]

### Step 2.3: Test Access to /admin

![[Pasted image 20260529113131.png]]

**Response:**
![[Pasted image 20260529113203.png]]

- The error message tells us exactly what's needed: **localhost access**.


---

## Step 3: Host Header Spoofing Attack

### Step 3.1: Change Host Header to localhost

In Repeater, modify the `Host` header:
![[Pasted image 20260529113443.png]]

**Response:**
![[Pasted image 20260529113505.png]]
- Access granted!

### Step 3.2: Delete User Carlos

Change the request to delete Carlos:

![[Pasted image 20260529113615.png]]

Send the request.

![[Pasted image 20260529113650.png]]

---

## Step 4: Lab Solved

Success message displayed:

![[Pasted image 20260529113731.png]]


---
---

