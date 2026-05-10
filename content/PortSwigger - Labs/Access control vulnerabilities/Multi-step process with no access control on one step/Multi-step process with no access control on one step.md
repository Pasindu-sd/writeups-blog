
# #PortSwigger 



![[Pasted image 20251216214851.png]]



**Description**
	*The admin panel uses a **multi-step process** to change a user's role. While some steps may check for admin privileges, **not all steps have access control**. This allows a non-admin user to skip the restricted steps and directly submit the final privileged request.*



## Solution Steps

### Step 1: Log in as Administrator (Reconnaissance)
1. Log in using `administrator:admin`
2. Browse to the **admin panel**
3. Find the functionality to promote a user (e.g., promote `carlos`)
4. Use Burp Proxy to capture the **confirmation HTTP request** when promoting the user

The request likely looks like:

![[Pasted image 20251216215450.png]]





### Step 2: Send Request to Repeater

1. Send this captured request to **Burp Repeater**
2. Keep this tab open






### Step 3: Log in as Non-Admin User

1. Open a **private/incognito browser window**
2. Log in using `wiener:peter`
3. Copy the **non-admin user's session cookie** from this browser

*wiener session cookie* 
![[Pasted image 20251216215737.png]]





### Step 4: Modify the Request in Repeater

1. Go back to Burp Repeater
2. Replace the admin session cookie with the **non-admin session cookie**
3. Change the `username` parameter from `carlos` to `wiener` (or your username)
4. Keep the `role=administrator` parameter    
5. Send the request

![[Pasted image 20251216215832.png]]





### Step 5: Verify the Exploit

**Observe:** The request is **accepted** even though it came from a non-admin user!

The final step of the multi-step process does not verify that the user is an administrator.





### Step 6: Solve the Lab

1. The lab is marked as **Solved** when you successfully promote yourself
2. You are now an administrator

![[Pasted image 20251216215844.png]]
