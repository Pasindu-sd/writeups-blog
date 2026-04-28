
# #PortSwigger 


![[Pasted image 20260428223433.png]]


**Description**
	*Perform a reflected XSS attack that injects an attribute and calls the `alert` function. Angle brackets (`< >`) are HTML-encoded, so you cannot use `<script>` or `<tag>` tags.*

| Character | Encoded As |
| --------- | ---------- |
| `<`       | `&lt;`     |
| `>`       | `&gt;`     |

This prevents you from injecting new HTML tags. However, you can still break out of an existing attribute and inject event handlers




## Solution Steps

### Step 1: Test with a Random String
1. Enter a random alphanumeric string in the search box (e.g., `test123`)
2. Submit the search
3. Inspect the page to see where your input appears

You'll notice your input is reflected **inside a quoted attribute**, like this:
![[Pasted image 20260428224457.png]]


### Step 2: Construct the Payload

inside a `value` attribute, you need to:
1. **Close the quote** (`"`)
2. **Inject an event handler** that will execute JavaScript

Enter this payload in the search box:
```
" onmouseover="alert(1)
```

![[Pasted image 20260428230005.png]]


After the Enter:
![[Pasted image 20260428230553.png]]



### Step 3: Test the Exploit
1. **Copy the URL** from the address bar (right-click → Copy URL)
2. **Paste it in a new browser tab**
3. **Move your mouse** over the search box or the reflected element
4. **Alert pops up** confirming the XSS works




### Step 4: Solve the Lab
Once the alert appears on mouse hover, the lab is marked as **Solved**.

![[Pasted image 20260428230327.png]]