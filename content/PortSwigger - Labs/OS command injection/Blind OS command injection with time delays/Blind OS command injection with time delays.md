
# #PortSwigger 



![[Pasted image 20251215003359.png]]


## Lab Description

>This lab contains a blind OS command injection vulnerability in the feedback function. The application executes a shell command containing the user-supplied details, but the output from the command is not returned in the response.
>
>**Objective:** Exploit the blind OS command injection vulnerability to cause a 10-second delay.


---
----


### Step 1: Capture the Feedback Request

1. In Burp's browser, access the lab
2. Use the **feedback** feature
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


![[Pasted image 20251215003612.png]]


---

### Step 2: Inject the Time-Delay Command

Modify the `email` parameter:

**Original:**
```
email=test%40test.com
```

**Modified:**
```
email=x||ping+-c+10+127.0.0.1||
```


![[Pasted image 20251215004354.png]]

**What this does:**

- `x` --> A dummy value (may fail, but that's fine)
- `||` --> OR operator (executes if the previous command fails)
- `ping -c 10 127.0.0.1` --> Pings localhost 10 times (~10 seconds delay)
- `||` --> Ends the command injection


---

### Step 3: Send the Request

1. Click **Send** in Repeater
2. Observe the response time

**Expected behavior:**
- The response takes **~10 seconds** to return
- This confirms the command injection vulnerability

---

### Step 4: Lab Solved

The lab is solved when the time delay is detected.

![[Pasted image 20251215004805.png]]


---
---
