
# #PortSwigger 


![[Pasted image 20260615111845.png]]


## Lab Description (from PortSwigger)

> This lab contains multiple independent vulnerabilities, including cache key injection. A user regularly visits this site's home page using Chrome.
> 
> **Objective:** Combine the vulnerabilities to execute `alert(1)` in the victim's browser. You will need to make use of the `Pragma: x-get-cache-key` header to solve this lab.

---

## Step 1: Understanding the Vulnerability Chain

**This lab requires combining four vulnerabilities:**

1. **Flawed regex in cache key** - `utm_content` parameter is excluded using a flawed regex, allowing appending unkeyed content to the `lang` parameter
    
2. **Client-side parameter pollution** - `/js/localize.js` doesn't URL-encode the `lang` parameter value
    
3. **Response header injection** - `/js/localize.js` allows injecting headers via the `Origin` header when `cors=1`
    
4. **Cache key injection** - The header injection can be triggered via a crafted URL (cache key injection)
    

---

## Step 2: Vulnerabilities Explained

### Vulnerability 1: Flawed Regex in Cache Key

The cache excludes `utm_content` using a regex:
```
/login?lang=en?utm_content=anything
```

