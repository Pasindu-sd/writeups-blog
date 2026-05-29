
# #PortSwigger 



![[Pasted image 20251212013704.png]]


## Lab Description

> This lab is vulnerable due to a logic flaw in its brute-force protection. To solve the lab, brute-force Carlos's password, then access his account page.
> 
> - **Victim's username:** `carlos`
> - **Candidate passwords:** (provided list)

---
---

## Step 1: Understanding the Vulnerability

**The logic flaw:** The login endpoint accepts a **JSON array** of passwords in a single request, but the application only checks the first one that matches.

**In this lab:**
- Login request uses JSON format: `{"username": "carlos", "password": "string"}`
- The server accepts a **password array** instead of a single string
- It iterates through the array and logs in with the first valid password
- No rate limiting on password attempts within a single request

**Why brute-force protection fails:**
- Traditional protection limits requests per second or per minute
- By sending many passwords in **one request**, the attacker bypasses these limits entirely

---

## Step 2: Reconnaissance

### Step 2.1: Capture the Login Request

1. Go to the login page
2. Attempt to log in with any credentials
3. In Burp Proxy, capture the `POST /login` request

**The request format:**

![[Pasted image 20251212013430.png]]

### Step 2.2: Send to Repeater

Right-click --> **Send to Repeater**



![[Pasted image 20251212013637.png]]

![[Pasted image 20251212013754.png]]
