
# #PortSwigger 


![[Pasted image 20260504212033.png]]


**Description:**
	*The stock checker functionality uses `document.write()` to write data from `location.search` (the `storeId` parameter) **inside a select element**. You need to break out of the select element to inject your payload.*


## Solution Steps


### Step 1: Understand the Vulnerability

On product pages, JavaScript extracts a `storeId` parameter from the URL and uses `document.write()` to create a new option inside a select element.



### Step 2: Test with a Random String

Add a `storeId` query parameter to the URL with a random alphanumeric string:

![[Pasted image 20260504214157.png]]




### Step 3: Inspect the HTML

Right-click and inspect the drop-down list. You'll see something like:

![[Pasted image 20260504214251.png]]
Your input is inside a **select element** as an `<option>` tag.




### Step 4: Construct payload

To break out of the select element and execute JavaScript, you need to:
1. **Close the current option tag** (`">`)
2. **Close the select element** (`</select>`)
3. **Inject your XSS payload** (`<img src=1 onerror=alert(1)>`)
    
**Final payload:**
```
"></select><img src=1 onerror=alert(1)>
```

![[Pasted image 20260504215149.png]]




### Step 5: Solve the lab

1. Enter the malicious URL in your browser
2. The alert should fire immediately
3. The lab is marked as **Solved**

![[Pasted image 20260504215129.png]]


![[Pasted image 20260504215311.png]]










## Alternative Payloads

1. 
```
"></select><svg onload=alert(1)>
```

 2. 
```
"></select><body onload=alert(1)>
```

3. 
```
"></select><script>alert(1)</script>  <!-- If script tags not blocked -->
```
