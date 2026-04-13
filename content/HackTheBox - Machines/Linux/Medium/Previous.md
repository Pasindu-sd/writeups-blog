![[Pasted image 20260116110918.png]]


![[Pasted image 20260116110947.png]]


![[Pasted image 20260116111544.png]]


![[Pasted image 20260116120749.png]]

CVE-2025-29927


![[Pasted image 20260116183953.png]]
Add x-middleware-subrequest:
![[Pasted image 20260116184103.png]]


![[Pasted image 20260116185832.png]]


![[Pasted image 20260116190006.png]]


![[Pasted image 20260116190236.png]]


![[Pasted image 20260116190450.png]]


![[Pasted image 20260116191435.png]]


![[Pasted image 20260116191711.png]]


- Username = `jeremy`
- Password = `MyNameIsJeremyAndILovePancakes`

![[Pasted image 20260116192135.png]]


![[Pasted image 20260116192423.png]]

![[Pasted image 20260116192512.png]]

user Flag
![[Pasted image 20260116192615.png]]



![[Pasted image 20260116210922.png]]

This command forces the current working directory to be /opt/examples and then runs apply, which creates or updates infrastructure.
![[Pasted image 20260116211531.png]]


```
>>jeremy@previous:~$ find / -name main.tf
>>/opt/examples/main.tf
```
![[Pasted image 20260117120238.png]]


![[Pasted image 20260117120357.png]]



make fake path
![[Pasted image 20260117142430.png]]


![[Pasted image 20260117150058.png]]


![[Pasted image 20260117150429.png]]
![[Pasted image 20260117150445.png]]


![[Pasted image 20260117150841.png]]


```

jeremy@previous:~$ cat /home/jeremy/docker/previous/public/examples/id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----


```


```
┌──(sn0x㉿sn0x)-[~/HTB/Previous]
└─$ cat > root_key << 'EOF'
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAmxhpS4UBVdbNosrMXPuKzRSbCOTgUH0/Tp/Yb32hyiMyMT68JuwK
[... FULL KEY CONTENT ...]
jR+NWPfpk9KHMAAAANcm9vdEBwcmV2aW91cwECAwQFBg==
-----END OPENSSH PRIVATE KEY-----
EOF
```
![[Pasted image 20260117151503.png]]



![[Pasted image 20260117151631.png]]


![[Pasted image 20260117151723.png]]


![[Pasted image 20260117151749.png]]



![[Pasted image 20260117151817.png]]
