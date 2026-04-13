![[Pasted image 20251230070915.png]]

![[Pasted image 20251230080017.png]]


scan with NMAP
![[Pasted image 20251230080308.png]]


find login page in source code
![[Pasted image 20251230080635.png]]


Access the logging page 
![[Pasted image 20251230080733.png]]


Capture the login GET request and Edit this one 'gues' into 'admin', then can log admin panel
![[Pasted image 20251230081155.png]]


client ID
![[Pasted image 20251230082219.png]]


Change users with chnging user id NO in url , 
![[Pasted image 20251230082423.png]]


![[Pasted image 20251230082845.png]]


change the cookie access ID - 34322, and then can get Admin access
![[Pasted image 20251230085424.png]]



https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php


![[Pasted image 20251230090602.png]]





![[Pasted image 20251230090627.png]]


![[Pasted image 20251230090753.png]]


![[Pasted image 20251230091050.png]]


![[Pasted image 20251230092845.png]]


![[Pasted image 20251230093212.png]]


![[Pasted image 20251230095529.png]]

We indeed got the password: MEGACORP_4dm1n!! . We can check the available users are on the system by reading the /etc/passwd file so we can try a password reuse of this password:
![[Pasted image 20251230095851.png]]


![[Pasted image 20251230100002.png]]


![[Pasted image 20251230121454.png]]


![[Pasted image 20251230122224.png]]

![[Pasted image 20251230132053.png]]
