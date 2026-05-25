
# #PortSwigger 


![[Pasted image 20260526001306.png]]


## Lab Description (from PortSwigger)

> This lab uses the HTMLJanitor library, which is vulnerable to DOM clobbering.  
> **Objective:** Construct a vector that bypasses the filter and uses DOM clobbering to inject a vector that calls the `print()` function. You may need to use the exploit server in order to make your vector auto-execute in the victim's browser.

- **Note:** The intended solution to this lab will not work in Firefox. We recommend using Chrome to complete this lab.

---

## Step 1: Understanding the Vulnerability

**This lab combines:**
1. **HTMLJanitor** - a client-side HTML sanitizer
2. **DOM clobbering** - polluting global variables/properties
3. **Attribute clobbering** - specifically clobbering the `attributes` property of a DOM element

**The key insight:**
- HTMLJanitor checks the `attributes` property of elements to filter dangerous attributes
- If we can **clobber** the `attributes` property, the filter breaks
- We can then inject arbitrary attributes (like `onfocus`)

