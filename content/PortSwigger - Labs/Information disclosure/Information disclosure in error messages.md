
# #PortSwigger 


![[Pasted image 20260528222734.png]]


## Lab Description

> This lab's verbose error messages reveal that it is using a vulnerable version of a third-party framework. To solve the lab, obtain and submit the version number of this framework.
> 
> **Objective:** Trigger an error message that discloses the framework version.

---
---


## Step 1: Understanding Information Disclosure

**Information disclosure** occurs when:
- Error messages are too detailed
- Stack traces reveal internal system information
- Version numbers expose known vulnerabilities

**Why this is dangerous:**
- Attackers can identify vulnerable software versions
- Known CVEs can be exploited
- Internal paths and system architecture are exposed

**In this lab:**
- Product pages accept a `productId` parameter
- Sending invalid data triggers an exception
- The stack trace reveals the Apache Struts version

---

## Step 2: Reconnaissance

### Step 2.1: Explore the Application

1. Open the lab homepage
2. Browse to a product page (e.g., click on any product)
3. Observe the URL structure:

![[Pasted image 20260528223718.png]]

### Step 2.2: Identify User Inputs

The `productId` parameter appears to accept numeric values:
- `productId=1` --> Leather Jacket
- `productId=2` --> Running Shoes    
- `productId=3` --> Gift Card

This parameter is a potential injection point.

---

## Step 3: Triggering an Error

### Step 3.1: Send Request to Repeater

1. In **Burp Suite**, go to **Proxy --> HTTP history**
2. Find the `GET /product?productId=1` request
3. Right-click --> **Send to Repeater**

### Step 3.2: Modify the Parameter

Change the `productId` parameter from an integer to a string:

**Original request:**
![[Pasted image 20260528224240.png]]

**Modified request:**
![[Pasted image 20260528224331.png]]

### Step 3.3: Send the Request

Click **Send** in Repeater.

---


## Step 4: Analyzing the Error Response

### Step 4.1: Observe the Stack Trace

The response should contain a detailed error message, similar to:
![[Pasted image 20260528224506.png]]

### Step 4.2: Identify the Framework Version

Look for lines containing version information:

![[Pasted image 20260528224558.png]]

**The version number:** `2.3.31`
```
Apache Struts 2 2.3.31
```


---

## Step 5: Submitting the Solution

1. Go back to the lab page
2. Click **Submit solution**
3. Enter the version number: `2 2.3.31`
![[Pasted image 20260528224821.png]]

4. Click **Submit**

---

## Step 6: Lab Solved

Success message displayed:

![[Pasted image 20260528224841.png]]

---
---
