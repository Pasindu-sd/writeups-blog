
# #PortSwigger 


![[Pasted image 20260429131645.png]]


**Description**
	*This lab contains a stored cross-site scripting vulnerability in the comment functionality. To solve this lab, submit a comment that calls the `alert` function when the comment author name is clicked.*


### Solve Steps

### Step 1: Identify Vulnerability
1. Access the Lab and iput any alphabetic character in the comment section
2. Using DevTools , check where the input save
![[Pasted image 20260429134632.png]]



### Step 2: Break test
1. try simple payload:
		`test"123`
2. check where the input save and HTML break
![[Pasted image 20260429141301.png]]
	
	it means :
		No input sanitize
		Vulnerability chance HIGH




### Step 3: Construct the Payload
Since you're inside an `href` attribute, you can use the `javascript:` pseudo-protocol:

**Enter this in the Website field:**
```
javascript:alert(1)
```

![[Pasted image 20260429141619.png]]

![[Pasted image 20260429141720.png]]


### Step 4: Test the Exploit
1. Post the comment with the payload
2. View the blog post
3. **Click on the author name** above your comment
4. The alert pops up

![[Pasted image 20260429141744.png]]




### Step 5: Solve the Lab
Once the alert appears when clicking the author name, the lab is marked as **Solved**.

![[Pasted image 20260429142123.png]]



## Important Note

Unlike `onmouseover` or `onclick` events, `javascript:` URIs in `href` attributes execute **when the link is clicked**, not automatically. The user must click the author name.