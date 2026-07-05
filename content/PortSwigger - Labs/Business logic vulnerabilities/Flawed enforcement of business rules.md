
# #PortSwigger 


![[Pasted image 20260605160532.png]]


## Lab Description

> This lab has a logic flaw in its purchasing workflow. To solve the lab, exploit this flaw to buy a "Lightweight l33t leather jacket".
> 
> - **Your credentials:** `wiener:peter`

---
---

## Step 1: Understanding the Vulnerability

**The business logic flaw:**
- The application has two coupon codes: `NEWCUST5` (5% off) and `SIGNUP30` (30% off)
- The same coupon cannot be applied twice in a row
- However, **alternating** between the two coupons bypasses this check

**The attack:**
- Apply `NEWCUST5` -> discount
- Apply `SIGNUP30` -> discount
- Apply `NEWCUST5` again -> discount (not the same as previous!)
- Repeat until price is $0 or less

---

## Step 2: Reconnaissance

### Step 2.1: Log In

1. Log in with `wiener:peter`
2. Note your initial store credit (usually $100)

### Step 2.2: Get the Coupon Codes

| Coupon     | How to Get                             | Discount |
| ---------- | -------------------------------------- | -------- |
| `NEWCUST5` | New customer (automatically available) | 5% off   |
| `SIGNUP30` | Sign up for newsletter                 | 30% off  |

*NEWCUTS  ***- New customer (automatically available)**
![[Pasted image 20251217152232.png]]

*`SIGNUP30`** - Sign up for newsletter***
![[Pasted image 20251217155752.png]]

### Step 2.3: Check Jacket Price

Find the **"Lightweight l33t leather jacket"**:
- Price: ~$1337 (or similar)
- Initial credit: $100

Need to reduce price below credit amount.

---

## Step 3: Testing the Coupon Logic

### Step 3.1: Add Jacket to Cart

Add the leather jacket to your shopping cart.

### Step 3.2: Apply Coupons Normally

Go to checkout and apply:
1. `NEWCUST5` -> price reduces by 5%
2. `SIGNUP30` -> price reduces further

### Step 3.3: Test Reapplication

**Try applying the same coupon twice:**
```
Apply NEWCUST5 -> Accepted
Apply NEWCUST5 again -> Rejected ("Coupon already applied")
```

**Try alternating:**
```
Apply NEWCUST5 -> Accepted
Apply SIGNUP30 -> Accepted
Apply NEWCUST5 -> Accepted! (Previous coupon was SIGNUP30, not NEWCUST5)
Apply SIGNUP30 -> Accepted! (Previous coupon was NEWCUST5, not SIGNUP30)
```

- The validation only checks if the **previous** coupon is the same, not if the coupon was ever used before.

![[Pasted image 20251217160618.png]]


---

## Step 4: Exploiting the Flaw

### Step 4.1: Calculate Discounts

Each full cycle of both coupons:
- First coupon: 5% off current price
- Second coupon: 30% off new price
- Net effect: Multiply price by `0.95 × 0.70 = 0.665` (33.5% off)

|Cycle|Price|Discount Applied|
|---|---|---|
|Start|$1337.00|-|
|NEWCUST5|$1270.15|5% off|
|SIGNUP30|$889.11|30% off|
|NEWCUST5|$844.65|5% off|
|SIGNUP30|$591.26|30% off|
|NEWCUST5|$561.70|5% off|
|SIGNUP30|$393.19|30% off|
|...|...|...|

Continue until price ≤ $100 (your store credit).

![[Pasted image 20251217160649.png]]

### Step 4.2: Automation Method

**Manual approach:**
1. Apply `NEWCUST5`
2. Apply `SIGNUP30`
3. Repeat steps 1-2 until price is low enough

**Using Burp Intruder:** (advanced)
- Capture the POST requests for applying coupons
- Create a payload that alternates between the two codes
- Send sequentially

---

## Step 5: Completing the Purchase

### Step 5.1: Reduce Price Below Credit

After enough alternating applications, the price will drop below $100.

### Step 5.2: Checkout

1. Click **Complete order**
2. The purchase will use your store credit    
3. The jacket is purchased!

![[Pasted image 20251217160726.png]]


---

## Step 6: Lab Solved

Success message displayed:

![[Pasted image 20251217160744.png]]

---
---
