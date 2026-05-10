
# #PortSwigger 



![[Pasted image 20251216220100.png]]


---

## Description

This lab controls access to certain admin functionality based on the `Referer` header. The server trusts the client-provided `Referer` header instead of properly authenticating the user's session.

**Credentials:**
- Admin: `administrator:admin` (to understand the process)
- Your account: `wiener:peter`

**Objective:** Promote yourself (wiener) to administrator.


## Vulnerability Explanation

The server uses the `Referer` header as the sole mechanism for access control:
- Requests **with** a `Referer` header from the admin panel → **Allowed**
- Requests **without** the correct `Referer` header → **Blocked**

The flaw is that the `Referer` header is **client-controlled** and can be added to any request. The server never verifies that the user making the request is actually an administrator.

---

## Solution Steps

### Step 1: Log in as Administrator (Reconnaissance)
1. Log in using `administrator:admin`
2. Browse to the **admin panel**
3. Find the functionality to promote a user (e.g., upgrade `carlos`)
4. Use **Burp Proxy** to capture the HTTP request

**Captured Request:**

![[Pasted image 20251216221000.png]]





### Step 2: Send Request to Repeater

1. Send this captured request to **Burp Repeater**
2. Keep this tab open for modification






### Step 3: Test Non-Admin Access (No Referer)

1. Open a **private/incognito browser window**
2. Log in using `wiener:peter`
3. Try to browse directly to:

```
/admin-roles?username=carlos&action=upgrade
```

- **Observe:** The request is **unauthorized** because there is no `Referer` header.





### Step 4: Copy Non-Admin Session Cookie

From the incognito browser, copy the non-admin user's session cookie:

```
Cookie: session=WIENER_SESSION_COOKIE
```

![[Pasted image 20251216220851.png]]






### Step 5: Modify the Request in Repeater

1. Go back to Burp Repeater
2. Replace the admin session cookie with the **non-admin session cookie**
3. Change `username=carlos` to `username=wiener`
4. **Keep the `Referer` header** (this is the key!)
5. Send the request

**Modified Request:**
```
GET /admin-roles?username=wiener&action=upgrade HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Referer: https://YOUR-LAB-ID.web-security-academy.net/admin
Cookie: session=WIENER_SESSION_COOKIE
```

![[Pasted image 20251216221132.png]]





### Step 6: Verify the Exploit

**Observe:** The request is **accepted** even though the session is from a non-admin user!

The server only checked that the `Referer` header was present and matched the admin page — it did NOT verify that the user is actually an administrator.

![[Pasted image 20251216221217.png]]






### Step 7: Solve the Lab

The lab is marked as **Solved** when you successfully promote yourself to administrator.

![[Pasted image 20251216221228.png]]


---

## Request Comparison Table

| Component      | Original Admin Request  | Modified Non-Admin Request     |
| -------------- | ----------------------- | ------------------------------ |
| Method         | `GET`                   | `GET` (same)                   |
| Endpoint       | `/admin-roles`          | `/admin-roles` (same)          |
| Username       | `carlos`                | `wiener` (changed)             |
| Action         | `upgrade`               | `upgrade` (same)               |
| Referer        | `https://LAB.net/admin` | `https://LAB.net/admin` (kept) |
| Session Cookie | Admin session           | **Wiener session** (changed)   |

**The server only checks:** "Does the Referer header match the admin page?"  
**It does NOT check:** "Is the user actually an administrator?"

---
