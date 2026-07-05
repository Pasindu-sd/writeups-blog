
# #PortSwigger 


![[Pasted image 20251217145632.png]]


## Lab Description (from PortSwigger)

> This lab makes flawed assumptions about the sequence of events in the purchasing workflow. To solve the lab, exploit this flaw to buy a "Lightweight l33t leather jacket".
> 
> - **Your credentials:** `wiener:peter`

---
---

## Step 1: Understanding the Vulnerability

**The workflow validation flaw:**
- The application assumes users will follow a specific sequence: add to cart → checkout → confirm order
- The order confirmation request (`GET /cart/order-confirmation?order-confirmed=true`) is not properly validated
- It can be called directly, bypassing the checkout process

**The attack:**
1. Add the expensive jacket to cart
2. Directly call the order confirmation endpoint
3. The order is completed without deducting store credit

---

## Step 2: Reconnaissance

### Step 2.1: Log In

1. Log in with `wiener:peter`
2. Note your store credit (e.g., $100)

### Step 2.2: Study Normal Workflow

1. Buy an affordable item (something you can afford)
2. Capture the requests in Burp Proxy

**Normal sequence:**
```
POST /cart (add item)
POST /cart/checkout (checkout)
GET /cart/order-confirmation?order-confirmed=true (confirm order)
```

### Step 2.3: Identify the Vulnerable Endpoint

Send the `GET /cart/order-confirmation?order-confirmed=true` request to Repeater.

---

## Step 3: Exploiting the Flaw

### Step 3.1: Add the Jacket to Cart

Add the **"Lightweight l33t leather jacket"** to your cart (do NOT checkout normally).


![[Pasted image 20251217145619.png]]


### Step 3.2: Bypass Checkout

In Burp Repeater, send the saved order confirmation request:
```
GET /cart/order-confirmation?order-confirmed=true HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: session=YOUR_SESSION
```


![[Pasted image 20251217145959.png]]


### Step 3.3: Observe the Result

**Response:** `302 Found` redirecting to order confirmation page.

The jacket is now purchased!

Check your store credit - it hasn't changed.

![[Pasted image 20251217150110.png]]


---

## Step 4: Lab Solved

Success message displayed:

![[Pasted image 20251217150138.png]]

---
---
