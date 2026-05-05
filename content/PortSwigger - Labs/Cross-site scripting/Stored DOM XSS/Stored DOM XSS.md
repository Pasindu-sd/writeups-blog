
# #PortSwigger 


![[Pasted image 20260505194727.png]]




**Description**
	*This lab demonstrates a stored DOM vulnerability in the blog comment functionality. To solve this lab, exploit this vulnerability to call the `alert()` function.*




## Solution Steps


### Step 1: Identify the Vulnerability

1. try the common string symbols to check encoded character
![[Pasted image 20260505195644.png]]


2. checks where the user feedback process in JS

![[Pasted image 20260505201822.png]]

- **The problem:** `replace()` with a string only targets the **first match**.

|Input|Result|Issue|
|---|---|---|
|`<script>`|`&lt;script>`|Only first `<` replaced, `>` remains!|

But more importantly: If you put **two sets** of angle brackets, the first gets encoded, the second survives!




### Step 2: Construct the Payload

Since the filter only removes the **first** `<` and `>`, you can use:

```
<><img src=1 onerror=alert(1)>
```

Payload work flow:
```
Original payload:
<><img src=1 onerror=alert(1)>
↑ ↑
│ └─ This > might get replaced (depends on implementation)
└─ This < is the FIRST occurrence

After replace('<', '&lt;'):
&lt;><img src=1 onerror=alert(1)>
   ↑
   └─ This > is still here! Not replaced!

After replace('>', '&gt;') - if implemented similarly:
&lt;&gt;<img src=1 onerror=alert(1)>
      ↑
      └─ Wait, now the first > is gone, but what about the > after img?

But the key point: The first angle brackets are sacrificial.
The actual payload <img src=1 onerror=alert(1)> survives!
```




### Step 3: Enter the Comment

1. Go to any blog post
2. Scroll down to the comment section
3. Fill in the comment form:
    - **Name:** Any name
    - **Email:** Any email
    - **Comment:** `<><img src=1 onerror=alert(1)>`
4. Click **"Post Comment"**

![[Pasted image 20260505212611.png]]



### Step 4: Solved the Lab
1. The page reloads with your comment
2. The `alert(1)` fires immediately
3. The lab is marked as **Solved**

![[Pasted image 20260505212640.png]]


![[Pasted image 20260505212656.png]]






## Key Takeaway

| Concept                | Explanation                                       |
| ---------------------- | ------------------------------------------------- |
| **Vulnerability Type** | Stored DOM XSS                                    |
| **Flawed Function**    | `replace()` with string instead of regex          |
| **Why it's flawed**    | Only replaces FIRST occurrence                    |
| **Bypass technique**   | Add extra angle brackets at the beginning         |
| **Why it works**       | First brackets get encoded, real payload survives |
