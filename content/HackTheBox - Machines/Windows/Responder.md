![[Pasted image 20251229075455.png]]


![[Pasted image 20251229084051.png]]


![[Pasted image 20251229084116.png]]


it may be have FLI vulnarability
![[Pasted image 20251229085211.png]]


![[Pasted image 20251229085350.png]]


![[Pasted image 20251229090741.png]]


my ip adress
![[Pasted image 20251229091025.png]]


capture hash
![[Pasted image 20251229091046.png]]


## 🟢 Final One-Line Answer (Very Important)

> **“NetNTLMv2 hash කියන්නේ `Administrator::RESPONDER:challenge:response:blob` කියන entire string එක.”**



![[Pasted image 20251229091919.png]]



![[Pasted image 20251229092932.png]]


Target machine එකේ **WinRM (5985)** open තිබ්බා remember?
```
evil-winrm -i <TARGET_IP> -u Administrator -p badminton
```


![[Pasted image 20251229093408.png]]


found some users
![[Pasted image 20251229093802.png]]


find flag in 'mike' user. 
![[Pasted image 20251229093929.png]]


![[Pasted image 20251229094004.png]]


![[Pasted image 20251229094624.png]]
