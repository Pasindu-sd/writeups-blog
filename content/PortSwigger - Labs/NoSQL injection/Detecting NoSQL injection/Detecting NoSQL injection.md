
# #PortSwigger 



![[Pasted image 20251223094519.png]]


<<<<<<< Updated upstream
=======
## Lab Description

>The product category filter for this lab is powered by a MongoDB NoSQL database. It is vulnerable to NoSQL injection.
>
 **Objective:** Perform a NoSQL injection attack that causes the application to display unreleased products.

---
---

### Step 1: Capture a Category Filter Request

1. In Burp's browser, access the lab
2. Click on a product category filter (e.g., "Gifts")
3. In Burp Proxy, find the request

**Example request:**
```
GET /filter?category=Gifts HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```


>>>>>>> Stashed changes
![[Pasted image 20251223094542.png]]


![[Pasted image 20251223094654.png]]


![[Pasted image 20251223094956.png]]


![[Pasted image 20251223095049.png]]


![[Pasted image 20251223095115.png]]
