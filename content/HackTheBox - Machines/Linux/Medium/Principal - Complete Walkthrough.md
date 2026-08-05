
# #HTB 


![[Pasted image 20260805234633.png|281]]



 
![[Pasted image 20260805223420.png]]



![[Pasted image 20260805223441.png]]


Taking a look at the page source, it loads `static/js/app.js`, which handles the auth. At the top, it defines the JWT structure in comments and some endpoints:

![[Pasted image 20260805223825.png]]



![[Pasted image 20260805223540.png]]


![[Pasted image 20260805225532.png]]


![[Pasted image 20260805225516.png]]


![[Pasted image 20260805225557.png]]


![[Pasted image 20260805230825.png]]


![[Pasted image 20260805230847.png]]


#### Password Spray

I’ll save the usernames from the dashboard into `users.txt` and spray the encryptionKey as a password against SSH. `netexec` can do this spray, but it runs serially, waiting for each attempt to timeout before doing the next, so `hydra` is a better tool here:

![[Pasted image 20260805230810.png]]


It finds a match using the `encryptionKey` for the svc-deploy account.

#### Shell

I’ll use the creds to get a shell using SSH:

![[Pasted image 20260805230944.png]]


And grab the user flag:

![[Pasted image 20260805231005.png]]


![[Pasted image 20260805233338.png]]


![[Pasted image 20260805233314.png]]



![[Pasted image 20260805233530.png]]


![[Pasted image 20260805233723.png]]


![[Pasted image 20260805233743.png|700]]


