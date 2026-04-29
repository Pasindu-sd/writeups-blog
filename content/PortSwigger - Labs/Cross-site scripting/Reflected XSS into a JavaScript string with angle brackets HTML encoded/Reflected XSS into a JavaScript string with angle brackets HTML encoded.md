---
title: Pasindu's Cybersecurity Blog
---
---

# #PortSwigger 


![[Pasted image 20260429183319.png]]


**Description**
	*This lab contains a reflected cross-site scripting vulnerability in the search query tracking functionality where angle brackets are encoded. The reflection occurs inside a JavaScript string. To solve this lab, perform a cross-site scripting attack that breaks out of the JavaScript string and calls the `alert` function.*




## Solution Steps


### Step 1: Identify Vulnerability
1.  **Access the lab** and locate the search functionality.
![[Pasted image 20260429183452.png]]


2. **Input a common string** into the search field to test where the input is reflected
![[Pasted image 20260429184011.png]]


3. **Check where the input string reflects** in the page source
	- In the source code, you can see:
![[Pasted image 20260429184050.png]]
****Observation:** The input is reflected inside a JavaScript string variable (`searchTerms`)*

4. 4. **Check what `encodeURIComponent()` does** – which characters are converted to HTML-safe format (visible in the URL).
![[Pasted image 20260429184527.png]]
	*Only the `/` character is encoded (to `%2F`). Other characters like quotes (`"`) remain unchanged.*




### Step 2: Construct Payload
Since the input is inside a JavaScript string delimited by **single quotes** (`'`), we need to:
1. Close the string with a single quote
2. Terminate the statement with a semicolon
3. Inject our JavaScript code (`alert(1)`)
4. Comment out the remaining code with `//`

**Payload:**
```
';alert(1);//
```

![[Pasted image 20260429190254.png]]


2. 5. **Click search** and the alert message appears.

![[Pasted image 20260429190057.png]]




### Step 3: Solve Lab
Once the alert triggers, the lab is marked as **Solved**.

![[Pasted image 20260429190115.png]]



---
## Alternative Payloads
The following payloads also work in this context:

| Payload          | Result             |
| ---------------- | ------------------ |
| `'-alert(1)-'`   | `''-alert(1)-''`   |
| `';alert(1)//`   | `'';alert(1)//'`   |
| `'\|alert(1)\|'` | `''\|alert(1)\|''` |

---
