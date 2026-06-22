
# #PortSwigger 


![[Pasted image 20251213104258.png]]


## Lab Description

This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics and performs a SQL query containing the value of the submitted cookie. The results are not returned, and the application does not respond differently based on whether the query returns any rows or causes an error. However, since the query is executed synchronously, it is possible to trigger conditional time delays to infer information.

**Objective:** Exploit the SQL injection vulnerability to cause a 10 second delay.

---
---


### Step 1: Capture the Request

1. Visit the front page of the shop
2. In Burp Proxy, find the request containing the `TrackingId` cookie
3. Send it to Repeater

**Example request:**
```
GET / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: TrackingId=xyz; session=...
```



![[Pasted image 20251213143036.png]] 



---

### Step 2: Inject Time-Delay Payload

**Modify the `TrackingId` cookie:**
```
'|| pg_sleep(10)-- -
```


![[Pasted image 20251213143626.png]]

**What this does:**
- `x'` --> Closes the current string
- `||` --> String concatenation operator
- `pg_sleep(10)` --> PostgreSQL function that pauses for 10 seconds
- `--` --> Comments out the rest of the query


---

### Step 3: Send the Request

1. Click **Send** in Repeater
2. Observe the response time

**Expected behavior:**
- The response takes **~10 seconds** to return    
- This confirms the SQL injection vulnerability


---

### Step 4: Lab Solved

![[Pasted image 20251213143856.png]]


---
---

