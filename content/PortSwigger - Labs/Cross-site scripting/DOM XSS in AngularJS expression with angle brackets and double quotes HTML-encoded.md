
# #PortSwigger 


![[Pasted image 20260504230923.png]]



**Description**
	*Perform a DOM-based XSS attack that executes an AngularJS expression and calls the `alert` function. Angle brackets (`< >`) and double quotes (`"`) are HTML-encoded, so you cannot use traditional HTML/JavaScript injection.*


**Vulnerability Explanation**
	*The search functionality uses **AngularJS** with the `ng-app` directive. AngularJS evaluates expressions inside double curly braces `{{ }}`. This allows JavaScript execution without using angle brackets or quotes.*



## Solution Steps


### Step 1: Test with random string

Enter a random alphanumeric string into the search box (e.g., `haha`).

![[Pasted image 20260504232436.png]]




### Step 2: View the page source

Right-click and select **"View Page Source"** or press `Ctrl+U`. Observe that your random string is enclosed in an `ng-app` directive.

![[Pasted image 20260504232622.png]]
- can identify it using AngularJS




### Step 3: Understand AngularJS Expressions

AngularJS evaluate anything inside `{{ }}` as a JavaScript expression. For example:
```
{{5+5}}     → Displays: 10
{{'hello'}} → Displays: hello
```

![[Pasted image 20260504233817.png]]

![[Pasted image 20260504233913.png]]




### Step 4: Construct a Angular Payload

Since we can execute JavaScript inside `{{ }}`, we need to call the `alert` function.
**Payload :**
```
{{$on.constructor('alert(1)')()}}
```

![[Pasted image 20260504233943.png]]

- check the `alert` function displayed.
![[Pasted image 20260504233959.png]]




### Step 5: Solved Lab

![[Pasted image 20260504234232.png]]
