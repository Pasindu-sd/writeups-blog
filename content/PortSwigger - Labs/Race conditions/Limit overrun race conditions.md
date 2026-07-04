
# #PortSwigger 


![[Pasted image 20260704163413.png]]


## Lab Description

This lab's purchasing flow contains a race condition that enables you to purchase items for an unintended price.

**Objective:** Successfully purchase a Lightweight "l33t" Leather Jacket.

**Credentials:** `wiener:peter`

**Note:** Solving this lab requires Burp Suite 2023.9 or higher.

---
---


### Step 1: Log In and Study the Flow

1. Log in as `wiener:peter`
2. Add a cheap item to cart
3. Apply the discount code (e.g., `SIGNUP30`)
4. Study the purchasing flow in Burp Proxy

**Key endpoints:**
- `POST /cart` --> Add items
- `POST /cart/coupon` --> Apply discount

---

### Step 2: Understand the Restriction

**Test applying discount twice:**

1. Apply the discount code once --> Success
2. Apply it again --> `Coupon already applied`

The restriction is server-side.

---

### Step 3: Confirm State is Session-Based

1. Send `GET /cart` with session cookie --> Shows your cart

![[Pasted image 20260704164145.png]]

2. Send `GET /cart` without session cookie --> Empty cart    

![[Pasted image 20260704164240.png]]

Cart state is stored server-side, keyed by session.

---

### Step 4: Send Parallel Requests

1. Make sure **no discount** is applied
2. Send the `POST /cart/coupon` request to Repeater

**Create multiple tabs:**
1. Right-click the tab --> **Add to group**
2. Duplicate the tab 19 times (20 total)

**Send in parallel:**
- Select **Send group in parallel (separate connections)**
- Click **Send**

![[Pasted image 20260704165533.png]]


---

### Step 5: Analyze Results

**Expected response:**
- Multiple requests show `Coupon applied successfully` 
- Some show `Coupon already applied`

The discount was applied multiple times in the race window.

---

### Step 6: Verify in Browser

1. Refresh the cart
2. The discount has been applied multiple times
3. Total price is significantly reduced

---

### Step 7: Attack the Jacket

1. Clear the cart
2. Add the **leather jacket** to cart
3. Send the parallel discount requests again
4. Refresh the cart

**If total is below store credit:**
- Click **Place order**

**If total is still too high:**
- Repeat the attack with more parallel requests


![[Pasted image 20260704170537.png]]

---

### Step 8: Lab Solved

![[Pasted image 20260704170611.png]]

---
---

