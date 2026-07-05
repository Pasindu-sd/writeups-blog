
# #PortSwigger 



![[Pasted image 20260527142531.png]]


## Lab Description

> This lab has some account functionality that is protected by a CSRF token and also has a confirmation dialog to protect against Clickjacking. To solve this lab construct an attack that fools the user into clicking the delete account button and the confirmation dialog by clicking on "Click me first" and "Click me next" decoy actions. You will need to use two elements for this lab.
> 
> **Objective:** Delete the user's account by tricking them into clicking two sequential decoy buttons.
> 
> **Credentials:** `wiener:peter`
> 
> **Note:** The victim will be using Chrome so test your exploit on that browser.

---
---

## Step 1: Understanding Multistep Clickjacking

**Multistep clickjacking** involves:
- A **multi-step process** that requires multiple clicks (e.g., delete account → confirm deletion)
- Each step has its own UI element that must be clicked
- The attacker creates **multiple decoy buttons** positioned over each step's target

**Why this is more complex:**
- Single decoy is not enough — need to chain clicks
- The confirmation dialog appears **after** the first click
- The second decoy must align with the confirmation dialog's button

**The protection being bypassed:**
- Confirmation dialogs are meant to prevent accidental destructive actions
- Clickjacking can still fool users into clicking both the action and its confirmation

---

## Step 2: Reconnaissance

### Step 2.1: Log in to Your Account

1. Log in with credentials: `wiener:peter`
2. Go to **My account** page

![[Pasted image 20260527142649.png]]

### Step 2.2: Identify the Multi-Step Process

Look for a destructive action with confirmation:

1. **Delete account** button (first click)
2. **Confirmation dialog** appears (e.g., "Are you sure? Yes/No")
3. **"Yes" button** (second click) confirms deletion
![[Pasted image 20260527142928.png]]

**Note the positions:**

- Position of "Delete account" button
- Position of "Yes" button in the confirmation dialog

***Caution:** If aligned correctly, this will delete YOUR account! Use a test account or be prepared to reset.*
### Step 2.3: Check for Frame Protections

- No `X-Frame-Options` header (or allows framing)
- No CSP `frame-ancestors` restrictions
- Confirmation dialog is **not** frame-busted

***Caution:** If aligned correctly, this will delete YOUR account! Use a test account or be prepared to reset.*

![[Pasted image 20260527143318.png]]


---

## Step 3: Understanding the Two-Step Attack

```
Step 1: Victim clicks first decoy
              ↓
    Clicks "Delete account" button
              ↓
    Confirmation dialog appears
              ↓
Step 2: Victim clicks second decoy
              ↓
    Clicks "Yes" in confirmation dialog
              ↓
    Account is deleted!
```

**Key challenge:** The confirmation dialog's position may be:
- Centered on screen (position varies)
- Below the delete button
- In a fixed location    

**Position values need careful calibration.**

---

## Step 4: Crafting the Multistep Clickjacking Exploit

### Step 4.1: HTML Template with Two Decoys

```
<style>
    iframe {
        position: relative;
        width: 500px;
        height: 700px;
        opacity: 0.0001;
        z-index: 1;
    }
    .firstClick {
        position: absolute;
        top: 330px;
        left: 50px;
        z-index: 2;
        cursor: pointer;
    }
    .secondClick {
        position: absolute;
        top: 285px;
        left: 225px;
        z-index: 2;
        cursor: pointer;
    }
</style>

<div class="firstClick">Click me first</div>
<div class="secondClick">Click me next</div>

<iframe src="https://YOUR-LAB-ID.web-security-academy.net/my-account"></iframe>
```

---

## Step 5: Positioning the Decoys

### Step 5.1: Initial Test with High Opacity

Start with `opacity: 0.1` so you can see both layers:
```
iframe {
    opacity: 0.1;  /* Start visible for alignment */
}
```

### Step 5.2: Align First Decoy

1. **Click View exploit**
2. **Hover over "Click me first"**
3. If the cursor changes to a hand, you're over a clickable element
4. **Adjust `top` and `left` values** until the decoy is exactly over the "Delete account" button

### Step 5.3: Align Second Decoy

**Important:** The confirmation dialog appears **after** the first click. You need to:
1. **Click "Click me first"** (this will click the delete button)
2. **Confirmation dialog appears**
3. **Hover over "Click me next"**
4. **Adjust `top` and `left`** until aligned with the "Yes" button

---

### Step 6: Verification Process

1. **Set opacity to 0.1** (visible iframe)
2. **Click "Click me first"** — watch where the click lands
3. **Wait for confirmation dialog**
4. **Click "Click me next"** — watch where the second click lands
5. **Adjust values** until both clicks hit the correct targets
6. **Set opacity to 0.0001** for final exploit


---

## Step 7: Final Exploit Code

```
<style>
    iframe {
        position: relative;
        width: 500px;
        height: 700px;
        opacity: 0.0001;
        z-index: 1;
    }
    .firstClick {
        position: absolute;
        top: 330px;
        left: 50px;
        z-index: 2;
        cursor: pointer;
    }
    .secondClick {
        position: absolute;
        top: 285px;
        left: 225px;
        z-index: 2;
        cursor: pointer;
    }
</style>

<div class="firstClick">Click me first</div>
<div class="secondClick">Click me next</div>

<iframe src="https://YOUR-LAB-ID.web-security-academy.net/my-account"></iframe>
```

- Replace `YOUR-LAB-ID` with your actual lab ID.
![[Pasted image 20260528001603.png]]

---

## Step 8: Testing the Exploit

1. **Store** the exploit
2. **View exploit**
3. **Hover over "Click me first"** - cursor should change to hand
4. **Click "Click me first"** - this clicks the delete button
5. **Confirmation dialog appears**
6. **Hover over "Click me next"** - cursor should change to hand
7. **Click "Click me next"** - this clicks "Yes" in confirmation

![[Pasted image 20260528001709.png]]

**Caution:** If aligned correctly, this will delete YOUR account! Use a test account or be prepared to reset.

---

## Step 9: Delivering to the Victim

1. Change text to user-friendly decoys (optional but recommended)
2. Click **Store**
3. Click **Deliver exploit to victim**
![[Pasted image 20260528001621.png]]

4. The victim clicks both decoys in sequence
5. Their account is deleted --> Lab solved

---

## Step 10: Lab Solved

Success message displayed:

![[Pasted image 20260528001516.png]]

---
---
