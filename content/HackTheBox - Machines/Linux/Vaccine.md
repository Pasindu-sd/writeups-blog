
![[Pasted image 20251231001836.png]]

Scan the ports
![[Pasted image 20251231002553.png]]


ftp vulnerabilty in here
![[Pasted image 20251231002757.png]]


![[Pasted image 20251231002954.png]]


backup.zip file is protected with password
![[Pasted image 20251231003029.png]]


- In order to successfully crack the password, we will have to convert the ZIP into the hash using the zip2john module that comes within John the Ripper:
![[Pasted image 20251231003436.png]]
![[Pasted image 20251231003452.png]]



then using john the ripper, we can find password
![[Pasted image 20251231003819.png]]


unzip the backup.zip file
![[Pasted image 20251231004501.png]]


We can see the credentials of admin:2cb42f8734ea607eefed3b70af13bbd3 , which we might be able to use. But the password seems hashed.
![[Pasted image 20251231004654.png]]


We will try to identify the hash type & crack it with the hashcat: It provides a huge list of possible hashes, however, we will go with MD5 first:
![[Pasted image 20251231004825.png]]


![[Pasted image 20251231005021.png]]


using johnTheRipper, found 'admin' password
![[Pasted image 20251231005638.png]]


Using the credentials , Login 
![[Pasted image 20251231010150.png]]


find SQL vulnerabilty in we
![[Pasted image 20251231010644.png]]


### Optional detail
![[Pasted image 20251231011001.png]]



get the sessinID
![[Pasted image 20251231011511.png]]

scan via sqlmap
![[Pasted image 20251231011858.png]]
![[Pasted image 20251231011928.png]]


then can get reverse shell
![[Pasted image 20251231012445.png]]


Got execution
![[Pasted image 20251231012848.png]]
![[Pasted image 20251231012911.png]]

![[Pasted image 20251231104630.png]]
![[Pasted image 20251231104658.png]]
![[Pasted image 20251231103355.png]]

![[Pasted image 20251231104747.png]]


![[Pasted image 20251231104859.png]]


![[Pasted image 20251231105015.png]]



![[Pasted image 20251231104537.png]]
