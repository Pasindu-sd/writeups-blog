![[Pasted image 20260209190255.png]]


![[Pasted image 20260209195459.png]]


![[Pasted image 20260209195558.png]]


![[Pasted image 20260209195526.png]]


Without any interesting data to look at, I’ll check for `xp_cmdshell`:
![[Pasted image 20260209195645.png]]
It’s disabled, and scott doesn’t have permissions to enable it.

I’ll check for impersonation and linked servers, but neither show anything interesting:
![[Pasted image 20260209195825.png]]

That output confirms the current server name, DC01, which I can also see with `@@SERVERNAME`:

![[Pasted image 20260209200011.png]]


Listing the logins,'

![[Pasted image 20260209200050.png]]


![[Pasted image 20260209213152.png]]
It looks like scott can run the command, but doesn’t have permissions to read any of the files.

MSSQL provides the mechanism to get information about domain users. For example, I can look up the SID for the domain Administrator account:


![[Pasted image 20260209214509.png]]

![[Pasted image 20260209214530.png]]


after :There’s a hash in Responder:
![[Pasted image 20260209214551.png]]

![[Pasted image 20260209215520.png]]


![[Pasted image 20260209221541.png]]


It is still showing guest privileges. This account is not an admin:
![[Pasted image 20260209221627.png]]


![[Pasted image 20260209222206.png]]


![[Pasted image 20260209222547.png]]

![[Pasted image 20260209223950.png]]


![[Pasted image 20260209224004.png]]







