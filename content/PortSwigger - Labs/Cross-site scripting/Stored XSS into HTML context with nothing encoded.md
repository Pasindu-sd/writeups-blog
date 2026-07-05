# #PortSwigger

![[Pasted image 20260425134053.png]]

## Steps 

**Description**
	*Submit a comment that calls the `alert` function when the blog post is viewed.*


## Solution Steps

1. **Access the lab** and open any blog post.
2. **Scroll down to the comment section**.
3. **Fill in the comment form** with the following payload:
    - **Comment:** `<script>alert(1)</script>`
    - **Name:** Any name (e.g., `thunder`)
    - **Email:** Any valid format (e.g., `haha@haha.com`)
    - **Website:** Can be left blank or any value

![[Pasted image 20260425134656.png]]


4. **Click "Post comment"**.
5. The page reloads with the new comment stored.
6. **View the blog post again** (or refresh the page) — the alert will fire.

![[Pasted image 20260425134757.png]]


7. The lab is now **Solved**.

![[Pasted image 20260425134732.png]]



## Why This Works

The comment functionality stores user input and later renders it in the HTML without encoding or sanitization. When any user views the blog post, the `<script>` tag executes in their browser.