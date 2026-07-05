
# #PortSwigger 


![[Pasted image 20260506213326.png]]



**Description**
	*Perform a reflected XSS attack that calls the `alert` function inside a JavaScript **template string** (template literal). All dangerous characters are escaped, but template literal interpolation `${...}` still works.*



## Solution Steps


### Step 1: Test with a random string

Enter a random alphanumeric string in the search box (e.g., `# "\'<>;haha`).
![[Pasted image 20260506213712.png]]




### Step 2: Observe the Reflection

Using Burp Suite or DevTools, you'll see your input reflected inside a JavaScript template string:
![[Pasted image 20260506213807.png]]





### Step 3: Understand Template Literal Interpolation

In JavaScript template literals, `${...}` allows you to embed expressions:
```
var name = "Thunder";
var greeting = `Hello ${name}!`;  // "Hello Thunder!"
```




### Step 4: Construct the Payload

Since `${...}` executes JavaScript, you can simply put `alert(1)` inside it:
```
${alert(1)}
```





### Step 5: Test the Exploit
1. Enter `${alert(1)}` in the search box
![[Pasted image 20260506214307.png]]

2. Click "Search"
![[Pasted image 20260506214409.png]]

3. The alert fires immediately
![[Pasted image 20260506214333.png]]

4. The lab is solved
![[Pasted image 20260506214428.png]]
