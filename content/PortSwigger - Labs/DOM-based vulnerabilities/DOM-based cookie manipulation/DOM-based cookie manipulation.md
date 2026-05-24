
# #PortSwigger 


![[Pasted image 20260525014406.png]]


## Lab Description (from PortSwigger)

> This lab demonstrates DOM-based client-side cookie manipulation.  
> **Objective:** Inject a cookie that will cause XSS on a different page and call the `print()` function. You will need to use the exploit server to direct the victim to the correct pages.

---

## Step 1: Understanding the Vulnerability

**DOM-based cookie manipulation** occurs when:
- Client-side JavaScript reads data from an attacker-controlled source (e.g., URL)
- That data is written to a cookie without sanitization
- Another page reads that cookie and uses it in a dangerous sink (e.g., `innerHTML`)

**In this lab:**
- The home page uses a client-side cookie called `lastViewedProduct`
- The cookie stores the URL of the last product page the user visited
- A different page (or the same page on load) uses this cookie value unsafely    





## Step 2: Reconnaissance

1. Open the lab homepage
2. Browse to a product page (e.g., `/product?productId=1`)
3. Check browser cookies (Developer Tools → Application → Cookies)
	**Observed behavior:**
	- A cookie named `lastViewedProduct` is created
	- Its value is the URL of the visited product page

4. Look for where this cookie is used. Check the homepage source for:

![[Pasted image 20260525015255.png]]






## Step 3: Crafting the Payload

**Goal:** Call `print()` via XSS.

The vulnerability exists if:
1. The cookie value is **taken from the URL** (attacker can control it)    
2. The cookie is later **written to the DOM** without sanitization

**Strategy:** Poison the `lastViewedProduct` cookie with a JavaScript payload that will execute when the homepage loads.

### Step 3.1: Finding the Injection Point

The product page URL might look like:
```
/product?productId=1
```

![[Pasted image 20260525015431.png]]

**Potential injection:**
```
/product?productId=1'><script>print()</script>
```

**Why this works:**

- The entire URL becomes the cookie value
- If the homepage inserts this URL into an HTML attribute without escaping, the `'` breaks out of the attribute, and `><script>` injects a new script tag

### Step 3.2: Complete Malicious URL

```
https://YOUR-LAB-ID.web-security-academy.net/product?productId=1&'><script>print()</script>
```






## Step 4: Building the Exploit

The exploit needs to:
1. Send the victim to the malicious URL to poison the cookie
2. Redirect the victim to the homepage without them noticing
3. The homepage then triggers the XSS

#### Exploit Code:

```
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/product?productId=1&'><script>print()</script>" 
        onload="if(!window.x)this.src='https://YOUR-LAB-ID.web-security-academy.net';window.x=1;">
</iframe>
```






## Step 5: Testing the Exploit

1. Go to the **Exploit server**
2. In the **Body** section, paste:
```
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/product?productId=1&'><script>print()</script>" 
        onload="if(!window.x)this.src='https://YOUR-LAB-ID.web-security-academy.net';window.x=1;">
</iframe>
```

3. Replace `YOUR-LAB-ID` with your actual lab ID
4. Click **Store**
5. Click **View exploit**

**Expected result:** The `print()` dialog appears.

![[Pasted image 20260525020050.png]]

![[Pasted image 20260525020031.png]]






## Step 6: Delivering to the Victim

1. Click **Deliver exploit to victim**
2. The lab solves when the victim's browser executes the payload

![[Pasted image 20260525020133.png]]




## Step 7: Lab Solved

Success message displayed:

![[Pasted image 20260525020151.png]]


---

