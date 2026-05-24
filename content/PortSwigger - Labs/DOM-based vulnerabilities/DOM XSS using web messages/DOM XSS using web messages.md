
# #PortSwigger 


![[Pasted image 20260523224239.png]]


## Lab Description

> This lab demonstrates a simple web message vulnerability.  
> **Objective:** Use the exploit server to post a message to the target site that causes the `print()` function to be called.

---

## Step 1: Understanding the Vulnerability

**Web Message Vulnerability** occurs when:
- A page uses `window.addEventListener('message', ...)` to receive messages from other origins
- The page does **not validate the origin** of incoming messages
- The page does **not sanitize** the message content before inserting it into the DOM
- An attacker can send a malicious message from any origin

**In this lab:**
- The homepage has an event listener waiting for web messages
- The listener is intended to serve ads
- It inserts message content into a `div` without sanitization    



## Step 2: Reconnaissance

1. Open the lab homepage
2. Right-click → **View Page Source** or open **Browser Developer Tools** → **Console**
3. Look for an event listener:

```
window.addEventListener('message', function(event) {
    // Intended to serve ads
    document.getElementById('ads').innerHTML = event.data;
});
```

![[Pasted image 20260523230447.png]]

**Key observations:**
- No origin check (`event.origin` is not validated)
- No sanitization (message content is directly inserted into the DOM)






## Step 3: Crafting the Exploit Payload

The goal is to call `print()`. We'll use an `<img>` tag with an invalid `src` attribute to trigger the `onerror` event.

**Malicious message:**
```
<img src=1 onerror=print()>
```

**Why this works:**
- `src=1` is an invalid image source
- The browser attempts to load the image and fails
- The `onerror` event executes the `print()` function






## Step 4: Building the Exploit Page

1. Go to the **Exploit server** (provided in the lab)
2. In the **Body** section, paste the following HTML:

```
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/" 
        onload="this.contentWindow.postMessage('<img src=1 onerror=print()>','*')">
</iframe>
```

3. Replace `YOUR-LAB-ID` with your actual lab ID (e.g., `abc123.web-security-academy.net`)

![[Pasted image 20260523230730.png]]






## Step 6: Testing the Exploit

1. Click **View exploit** (simulates visiting your malicious page)
2. Observe that the `print()` dialog appears

**Success indicator:** Your browser's print dialog pops up. The same will happen to the victim.

![[Pasted image 20260523230759.png]]





## Step 7: Delivering to the Victim

1. Click **Store** to save the exploit
2. Click **Deliver exploit to victim**
3. The lab solves when the victim's browser executes the payload

![[Pasted image 20260523230906.png]]





## Step 8: Lab Solved

Success message displayed:

![[Pasted image 20260523230852.png]]

---

