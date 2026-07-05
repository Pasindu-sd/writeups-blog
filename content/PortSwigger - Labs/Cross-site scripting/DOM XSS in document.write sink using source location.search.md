
# #PortSwigger

![[Pasted image 20260426094604.png]]


**Description**
	*This lab contains a DOM-based cross-site scripting vulnerability in the search query tracking functionality. It uses the JavaScript `document.write` function, which writes data out to the page. The `document.write` function is called with data from `location.search`, which you can control using the website URL.*


## Steps 


### Solution Steps

1. **Access the lab** and locate the search box.
2. First, test with a random string (eg. `haha`) in the search box and submit it.
3. Rigth-click and inspect the element to see where your input appears. You should find it inside an `img` `src` attribute, like :

`document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">');`

![[Pasted image 20260426103457.png]]


4. Construct the payload to break out of the `src` attribute and inject JavaScript.
		Enter this as your search term:
```
"><svg onload=alert(1)>
```


5. **Click "Search"** or submit the query via the URL.
		The resulting HTML becomes:
	```
	<img src="/resources/images/tracker.gif?searchTerms="><svg onload=alert(1)>">
	```

6. **The alert fires**, and the lab is marked as **Solved**

## Solved
![[Pasted image 20260426111721.png]]




## Why This Works

- The `document.write` function writes user-controlled `location.search` data directly into the page.
- The data is placed inside an `img src` attribute without proper escaping.
- By injecting `">`, you close the attribute and the `img` tag.
- The `<svg onload=alert(1)>` injects a new element whose `onload` event executes JavaScript.