
# #PortSwigger


![[Pasted image 20260427133544.png]]


**Description**
	*This lab contains a DOM-based cross-site scripting vulnerability in the search blog functionality. It uses an `innerHTML` assignment, which changes the HTML contents of a `div` element, using data from `location.search`.*


# Steps

1. Access the Lab and locate the search box.

![[Pasted image 20260427223405.png]]


2. First, test with a random string (eg. `haha`) in the search box and submit it.
3. Rigth-click and inspect the element to see where your input appears.

![[Pasted image 20260427225329.png]]


4. **Enter the following payload** into the search box:

```
<img src=1 onerror=alert(1)>
```

*Because* :
- *- `<script>alert(1)</script>` often **does NOT execute** when injected via `innerHTML` because modern browsers **block script execution** for security.
- *`<img src=1 onerror=alert(1)>` **works** because:*
    - *It’s a normal HTML element
    - *The `onerror` event **still fires**
    - *Browser executes the JavaScript inside the event handler*

![[Pasted image 20260427230243.png]]

5. **Click "Search"**.
6. 1. **The alert fires**, and the lab is marked as **Solved**.

![[Pasted image 20260427230257.png]]

## Solved
![[Pasted image 20260427230400.png]]




# Deeper Learning

## Why This Works

- The `innerHTML` property allows insertion of HTML tags.
- Unlike `document.write`, `innerHTML` does **not** execute `<script>` tags for security reasons.
- However, event handlers like `onerror` on HTML elements **will** execute JavaScript.
- `<img src=1 onerror=alert(1)>` works because:
    - `src=1` is an invalid image source → triggers error
    - `onerror` event fires → executes `alert(1)`

## Comparison with Previous Lab

## Why This Works

- The `innerHTML` property allows insertion of HTML tags.
- Unlike `document.write`, `innerHTML` does **not** execute `<script>` tags for security reasons.
- However, event handlers like `onerror` on HTML elements **will** execute JavaScript.
- `<img src=1 onerror=alert(1)>` works because:
    - `src=1` is an invalid image source → triggers error
    - `onerror` event fires → executes `alert(1)`

## Comparison with `document.write`

|Lab|Sink|Restriction|Recommended Payload|
|---|---|---|---|
|DOM XSS in `document.write`|`document.write()`|Inside `img src` attribute|`"><svg onload=alert(1)>`|
|DOM XSS in `innerHTML`|`innerHTML`|`<script>` blocked|`<img src=1 onerror=alert(1)>`|
