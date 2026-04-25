
![[Pasted image 20260425125347.png]]


## #Steps 

**Description**
	*This lab contains a simple reflected cross-site scripting vulnerability in the search functionality.
To solve the lab, perform a cross-site scripting attack that calls the `alert` function.*



![[Pasted image 20260425130112.png]]

Vulnerability in the search functionality.

### Step 1: insert a Script

1. Copy and paste the following into the search box:
    `<script>alert(1)</script>`
    
2. Click "Search".

![[Pasted image 20260425130319.png]]


- check Pop-up the alert message

![[Pasted image 20260425130401.png]]

### Solved Lab

![[Pasted image 20260425130456.png]]
