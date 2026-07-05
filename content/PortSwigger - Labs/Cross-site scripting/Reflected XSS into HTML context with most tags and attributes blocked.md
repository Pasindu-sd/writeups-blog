
# #PortSwigger 


![[Pasted image 20260505215728.png]]




**Description**
	*Perform a cross-site scripting attack that bypasses the WAF and calls the `print()` function **without any user interaction**.*




## Solution Steps


### Step 1: Test Standard XSS Vector

First, Test standard payload:
```
<script>alert(1)</script>
```

![[Pasted image 20260505222302.png]]
- Click Search button
![[Pasted image 20260505222342.png]]
- **Result:** Gets blocked by the WAF (400 response).




### Step 2: Identify Allowed Tags (Burp Intruder)

1. **Send the search request to Burp Intruder**
2. **Replace the search term with:** `<>`
3. **Add payload position:** `<§§>` (cursor between angle brackets)
4. **From XSS cheat sheet, copy tags to clipboard**
![[Pasted image 20260505223147.png]]

5. **Paste tags into payloads list** (Payload configuration)
6. **Start attack**

**Results:**
- Most payloads → `400` response (blocked)
- **`<body>` payload → `200` response** (allowed!)

![[Pasted image 20260505224956.png]]





### Step 3: Identify Allowed Attributes (Burp Intruder)

1. **Replace search term with:** `<body%20=1>`
2. **Add payload position:** `<body%20§§=1>` (before `=`)
3. **From XSS cheat sheet, copy events to clipboard**
4. **Clear previous payloads and paste events**
![[Pasted image 20260505225348.png]]

5. **Start attack**

**Results:**
- Most payloads → `400` response (blocked)
- **`onresize` attribute → `200` response** (allowed!)

![[Pasted image 20260505225515.png]]





### Step 4: Construct the Exploit

Now we know:
- Allowed tag: `<body>`
- Allowed attribute: `onresize`

**Final payload:**
```
<body onresize=print()>
```

URL encode version:
```
%3Cbody%20onresize=print()%3E
```





### Step 5: Create the Exploit on Exploit Server

**Go to the exploit server** and paste the following:
```
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
```
- **Replace `YOUR-LAB-ID` with your actual lab ID.**

![[Pasted image 20260505230931.png]]




### Step 6: Deliver the Exploit

1. Click **"Store"**
2. Click **"Deliver exploit to victim"**
3. The `print()` function is called
4. Lab solved!

![[Pasted image 20260505231031.png]]



---

### How the Iframe Exploit Works

|Part|Purpose|
|---|---|
|`<iframe src="...">`|Loads the vulnerable page with XSS payload|
|`?search=%22%3E%3Cbody%20onresize=print()%3E`|URL-encoded payload|
|`onload=this.style.width='100px'`|Trigger resize event by changing iframe width|
|`onresize=print()`|Executes `print()` when resize occurs|



## Why This Works

|Obstacle|Bypass Method|
|---|---|
|WAF blocks most tags|Found `<body>` tag is allowed|
|WAF blocks most attributes|Found `onresize` attribute is allowed|
|Need auto-execution (no click)|Iframe triggers `onresize` by changing width|
|Need to avoid user interaction|`onload` event changes iframe size automatically|
