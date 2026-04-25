
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


3. The application reflects the input without encoding, executing the script and triggering an alert box.

![[Pasted image 20260425130401.png]]

### Solved Lab

5. Once the alert appears, the lab is marked as Solved.

![[Pasted image 20260425130456.png]]




### #Key_Takeaway
The vulnerability exists because the search parameter is echoed directly into the HTML response without any sanitization or encoding, allowing arbitrary JavaScript execution.

Lab solved with a simple `<script>alert(1)</script>` payload.