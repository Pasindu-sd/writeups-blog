
# #PortSwigger 


![[Pasted image 20260505001259.png]]


**Description**
	*The server reflects your search input in a JSON response. A script (`searchResults.js`) then processes this JSON using `eval()`. Backslashes (`\`) are not escaped, allowing you to break out of the JSON string and inject JavaScript.*




## Solution Steps


### Step 1: Test with Random String

1. Use the search bar to search for a random test string (e.g., `haha`)
2. Intercept the request using Burp Suite (Proxy → Intercept on)

![[Pasted image 20260505004944.png]]

![[Pasted image 20260505005005.png]]




### Step 2: Analyze the Response

Forward the request and observe that your string is reflected in a JSON response:
![[Pasted image 20260505005129.png]]




### Step 3: Examine the JavaScript

From the Site Map, open `searchResults.js` and notice that the JSON response is used with an `eval()` function call:
![[Pasted image 20260505005637.png]]

![[Pasted image 20260505005236.png]]




### Step 4: Understand the Escaping Behavior

- The JSON response **escapes quotation marks** (`"` → `\"`)
- **Backslashes (`\`) are NOT escaped**

This is the key vulnerability!




### Step 5: Construct the Payload

Since backslashes aren't escaped, you can inject a backslash to cancel out the escaping:
**Payload:**
```
\"-alert(1)}//
```




### Step 6: Enter the Payload
1. Enter the payload into the search box:
```
\"-alert(1)}//
```

2. Click Search
3. The alert fires




### Step 7: Solved Lab

Once the alert appears, the lab is marked as **Solved**.

![[Pasted image 20260505013020.png]]

![[Pasted image 20260505013038.png]]








## Why This Works - Simplified

| Step | What happens                                       |
| ---- | -------------------------------------------------- |
| 1    | You send `\"-alert(1)}//`                          |
| 2    | Server escapes `"` to `\"` → becomes `\\"`         |
| 3    | JSON becomes `{"searchTerm":"\\"-alert(1)}//"...}` |
| 4    | `eval()` executes the JSON                         |
| 5    | The quote closes the string early                  |
| 6    | `-alert(1)` executes as JavaScript                 |
| 7    | `}//` comments out the remaining JSON              |
