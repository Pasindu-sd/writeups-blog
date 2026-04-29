
# #PortSwigger 


![[Pasted image 20260429183319.png]]


**Description**
	**


### Solution Steps

### Step 1: Indentify Vulnerability
1. Access lab and point search functionality
![[Pasted image 20260429183452.png]]


2. input common string into the search field
![[Pasted image 20260429184011.png]]

3. check where the input string reflect in the page source
![[Pasted image 20260429184050.png]]

4. check `encodeURIComponent(searchTerms)` what do, which character convert HTML-safe format(it can see in the URL )
![[Pasted image 20260429184527.png]]
	only `/` Character convert.


### Step 2: Construct Payload
1. passes encoded character and use
```
';alert(1);//
```

![[Pasted image 20260429190254.png]]

2. Click search and see alert message
![[Pasted image 20260429190057.png]]



### Step 3: Solve Lab

![[Pasted image 20260429190115.png]]



