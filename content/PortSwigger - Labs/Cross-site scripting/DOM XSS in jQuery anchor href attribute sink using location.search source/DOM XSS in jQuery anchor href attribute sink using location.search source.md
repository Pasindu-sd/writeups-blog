
# #PortSwigger 



![[Pasted image 20260428001223.png]]



**Description** :
	*Make the "back" link execute `alert(document.cookie)` by exploiting a DOM-based XSS vulnerability in the jQuery `$` selector function that changes an anchor element's `href` attribute using data from `location.search`.*


## Solution Steps

1. **Access the lab** and navigate to the **Submit feedback** page.

![[Pasted image 20260428002927.png]]


2. 1. **Look for the "back" link** on the page.
    
3. **Modify the URL** by changing the `returnPath` query parameter.  
    Start by testing with a random alphanumeric string:

```
https://YOUR-LAB-ID.web-security-academy.net/feedback?returnPath=/test123
```

![[Pasted image 20260428003402.png]]


4. **Right-click and inspect** the "back" link to verify your input appears inside an `a href` attribute.
    
5. **Change the `returnPath` parameter** to a `javascript:` pseudo-protocol payload:

```
javascript:alert(document.cookie)
```

![[Pasted image 20260428004232.png]]


Full URL Example:
![[Pasted image 20260428004001.png]]


6. **Press Enter** to load the modified URL.
    
7. **Click the "back" link** on the page.
    
8. **The alert fires** showing the document cookie, and the lab is marked as **Solved**.


### Solved
![[Pasted image 20260428004118.png]]
