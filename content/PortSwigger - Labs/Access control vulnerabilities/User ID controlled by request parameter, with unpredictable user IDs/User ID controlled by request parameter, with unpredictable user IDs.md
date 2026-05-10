
# #PortSwigger 



![[Pasted image 20251216200236.png]]




## Background

This lab uses **GUIDs** (globally unique identifiers) instead of sequential integers for user IDs. GUIDs are hard to guess — but if they are **exposed anywhere** (e.g., in blog post URLs, links, or public profiles), they offer no real protection. The vulnerability is still **horizontal privilege escalation**: any logged-in user can access another user's account page if they know their GUID.


---



## Step 1 — Log In and Explore

Log in with `wiener:peter`.  
Your account page URL looks like:  
`/my-account?id=1e4c61464-4ee2-4e15-8d0e-9c75d76978be`

That GUID is **unpredictable**, but the lab gives you another way to find `carlos`'s GUID.

![[Pasted image 20251216200354.png]]






## Step 2 — Find Carlos's GUID via a Blog Post

Navigate to the blog section. Find a post by `carlos`.  
Click on `carlos` (the author name/link) — observe the URL in your browser or Burp:

```
/blogs?userId=48802904-9da5-444f-8397-e5bcfdd684d8
```

![[Pasted image 20251216200253.png]]
![[Pasted image 20251216200320.png]]

- That `userId` parameter contains **carlos's GUID**.




## Step 3 — Access Carlos's Account Page

Now visit your own account page but replace your GUID with carlos's GUID:

```
GET /my-account?id=48802904-9da5-444f-8397-e5bcfdd684d8
```

The server responds with **carlos's account page**, including his API key.

![[Pasted image 20251216200506.png]]





## Step 4 — Extract the API Key

In the response, locate the API key:

```
Your API Key is: 4iALtvJSwMONO79DpZkiVLvqXG3exE5d
```
- Copy this value.

![[Pasted image 20251216200537.png]]





## Step 5 — Submit the API Key

Go back to the lab description page.  
Enter the API key in the submission field (or on the "Submit solution" button if present).





## Step 6 — Lab Solved 

You'll see:
	**Congratulations, you solved the lab!**

![[Pasted image 20251216200606.png]]


---

## hy This Works

|Assumption|Reality|
|---|---|
|GUIDs are hard to guess|True — but they are **exposed** in blog URLs|
|Hidden fields are safe|The API key is only hidden by obscurity|
|Unpredictable IDs prevent enumeration|They do, but **off-platform disclosure** breaks this|

This is a classic case of **security by obscurity** failing because the "secret" ID is leaked elsewhere on the same application.

---
