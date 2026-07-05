
# #PortSwigger 


![[Pasted image 20260506093758.png]]



**Description**
	*This lab has a simple reflected XSS vulnerability. The site is blocking common tags but misses some SVG tags and events. To solve the lab, perform a cross-site scripting attack that calls the `alert()` function.*




## Solution Steps


### Step 1: Indentify the Vulneraility

Test standard XSS vector:
```
<script>alert(1)</script>
```
- **Result:** Gets blocked (400 response)

![[Pasted image 20260506095154.png]]





### Step 2: Identify Allowed Tags (Burp Intruder)

1. **Send the search request to Burp Intruder**
2. **Replace the search term with:** `<>`
3. **Add payload position:** `<§§>` (cursor between angle brackets)
4. **From XSS cheat sheet, copy tags to clipboard**
![[Pasted image 20260506095446.png]]
5. **Paste tags into payloads list**
6. **Start attack**

**Results:**
- Most payloads → `400` response (blocked)
- **Allowed tags (200 response)**
![[Pasted image 20260506100542.png]]




### Step 3: Identify Allowed Attributes (Burp Intruder)

1. **Replace search term with:** `<svg><animatetransform%20=1>` (`animatetrannsform` is `svg` related tag)
2. **Add payload position:** `<svg><animatetransform%20§§=1>` (before `=`)
3. **From XSS cheat sheet, copy events to clipboard**
4. **Clear previous payloads and paste events**
![[Pasted image 20260506101015.png]]
5. **Start attack**

**Results:**
- Most payloads → `400` response (blocked)
- **`onbegin` attribute → `200` response** (allowed!)
![[Pasted image 20260506101100.png]]





### Step 4: Construct the Payload

Now we know:
- Allowed tag: `<svg>` and `<animatetransform>`
- Allowed event: `onbegin`

**Final payload:**
```
<svg><animatetransform onbegin=alert(1)>
```

![[Pasted image 20260506101439.png]]

- Enter the payload into search box and Click Search button




### Step 6: Solve the Lab

Once the alert appears, the lab is marked as **Solved**.

![[Pasted image 20260506101309.png]]

![[Pasted image 20260506101336.png]]



--- 

## How the Payload Works

|Part|Purpose|
|---|---|
|`"`|Closes any existing attribute|
|`>`|Closes any existing tag|
|`<svg>`|Starts an SVG element (allowed)|
|`<animatetransform>`|SVG animation tag (allowed)|
|`onbegin=alert(1)`|Event that fires when animation begins|
|`>`|Closes the tag|


