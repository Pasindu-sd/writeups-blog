
# #PortSwigger 


![[Pasted image 20251216233305.png]]


## Lab Description

> This lab doesn't adequately validate user input. You can exploit a logic flaw in its purchasing workflow to buy items for an unintended price. To solve the lab, buy a "Lightweight l33t leather jacket".
> 
> - **Your credentials:** `wiener:peter

---
---

## Step 1: Understanding the Vulnerability

**The low-level logic flaw:**
- The server uses an integer to store the total price (in cents)
- When the total price exceeds the maximum integer value (`2,147,483,647`), it **wraps around** to the minimum negative value (`-2,147,483,648`)
- This is an **integer overflow** vulnerability

**The attack:**
1. Add many jackets to cart (enough to cause integer overflow)
2. The total price wraps to a large negative number
3. Add more items to adjust the total to a positive amount within store credit
4. Purchase the jacket(s) for very low cost

---

## Step 2: Reconnaissance

### Step 2.1: Log In

1. Log in with `wiener:peter`
2. Initial store credit: **$100.00**

### Step 2.2: Check Jacket Price

**Lightweight l33t leather jacket:**
- Price: $1337.00
- In cents: **133,700** (for integer calculation)


![[Pasted image 20251217014412.png]]


### Step 2.3: Understand the Integer Limit

|Value|Amount|
|---|---|
|Maximum 32-bit signed integer|2,147,483,647|
|Minimum 32-bit signed integer|-2,147,483,648|

---

## Step 3: Calculating the Overflow

### Step 3.1: Find How Many Jackets to Overflow

We need the total cents to exceed 2,147,483,647:
```
133,700 × N > 2,147,483,647
N > 2,147,483,647 ÷ 133,700
N > 16,062.99
```

So **16,063 jackets** will exceed the maximum integer value.

### Step 3.2: Calculate the Overflow Amount

If we add 16,063 jackets:
```
Total cents = 16,063 × 133,700 = 2,147,623,100
```

Overflow amount:
```
2,147,623,100 - 2,147,483,647 = 139,453 cents over the maximum
```

When it overflows, it wraps to:
```
-2,147,483,648 + 139,453 = -2,147,344,195 cents
```

That's approximately **-$21,473,441.95** (large negative number!)



![[Pasted image 20251217014501.png]]


---

## Step 4: First Attack - Testing the Overflow

### Step 4.1: Setup Intruder

1. Capture the `POST /cart` request for the jacket
2. Send to Intruder
3. Set the `quantity` parameter to `99` (max allowed per request)

### Step 4.2: Configure Payload

|Setting|Value|
|---|---|
|Payload type|Null payloads|
|Generate|Continue indefinitely|

### Step 4.3: Resource Pool

Set **Maximum concurrent requests = 1** (send sequentially)

### Step 4.4: Start Attack

While the attack runs, refresh the cart page periodically.

**Observed behavior (from your screenshots):**
- Price starts at $1337.00
- Price increases with each added jacket
- Suddenly, the price jumps to a **large negative number** (e.g., -$19,830,312.96)
- The price then starts counting **up** towards zero

Integer overflow occurred!

![[Pasted image 20251217020512.png]]


![[Pasted image 20251217021318.png]]
![[Pasted image 20251217021347.png]]


---

## Step 5: Calculating the Exact Payload

### Step 5.1: Find the Overflow Point

From your screenshots:
- After **18,369** jackets, total = **-$19,830,312.96**
- After **64,247** jackets, total = **$18.92** (after adding other items)

We need the total to be between $0 and $100 (store credit).

### Step 5.2: Calculate Required Jackets

From the lab solution: **323 payloads** of quantity 99 each
```
323 × 99 = 31,977 jackets
```

Then add **47 more jackets**:
```
31,977 + 47 = 32,024 jackets
```


**From your screenshot:** The final successful order used **64,247 jackets** for the jacket and **12 pairs of shoes** to fine-tune the total to **$18.92**.

---

## Step 6: Fine-Tuning the Total

### Step 6.1: Add Second Item

After the overflow, the total is a large negative number. Add a **second cheap item** (e.g., Baby Minding Shoes at $93.82) in suitable quantity to bring the total into the positive range ($0 - $100).

**From your screenshot:**
- Jackets: 64,247 × $1337.00
- Shoes: 12 × $93.82
- Total: **$18.92** 

### Step 6.2: Place Order

With the total below your store credit ($100), place the order.

---

## Step 7: Lab Solved

Success message displayed:

![[Pasted image 20251217023159.png]]

---
---
