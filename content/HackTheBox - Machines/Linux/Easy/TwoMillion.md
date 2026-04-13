![[Pasted image 20260104135744.png]]

![[Pasted image 20260104135824.png]]



invite page
![[Pasted image 20260104135907.png]]


![[Pasted image 20260104140217.png]]


Open the inviteapi.min.js file in new tab
![[Pasted image 20260104140301.png]]


මෙය **JavaScript obfuscated code** එකක්. ඒක deliberately අමුතු ලෙස ලියා තිබෙන්නේ code එක **hidden / protect** කරන්න


![[Pasted image 20260104140406.png]]


![[Pasted image 20260104143007.png]]


![[Pasted image 20260104143030.png]]


![[Pasted image 20260104143233.png]]


![[Pasted image 20260104143353.png]]


![[Pasted image 20260104143457.png]]


![[Pasted image 20260104143541.png]]


![[Pasted image 20260104143617.png]]


![[Pasted image 20260104145535.png]]



![[Pasted image 20260104145433.png]]




We get a list of quite a few endpoints that are available in the API, with some of the most interesting ones being the admin specific endpoints. As a test we can hit the /admin/auth endpoint to check if we are an admin user.
![[Pasted image 20260104150855.png]]


![[Pasted image 20260104151036.png]]We get a 401 Unauthorised error, most probably because we are not an admin. Let's move to the final administrative endpoint, /admin/settings/update . We note that this request needs to be a PUT as shown in the output from /api/v1 .

![[Pasted image 20260104151242.png]]
Interestingly enough, this time we do not get an Unauthorized error, but instead the API replies with Invalid content type


![[Pasted image 20260104152712.png]]


![[Pasted image 20260104152728.png]]


![[Pasted image 20260104155318.png]]

```

┌──(thunder㉿kali)-[~]
└─$ nc -lvnp 1337
listening on [any] 1337 ...


```


![[Pasted image 20260104155408.png]]


![[Pasted image 20260104155424.png]]


![[Pasted image 20260104155527.png]]


![[Pasted image 20260104155631.png]]


![[Pasted image 20260104160301.png]]


![[Pasted image 20260104160314.png]]


![[Pasted image 20260104161052.png]]


# The OverlayFS vulnerability CVE-2023-0386: Overview, detection, and remediation
![[Pasted image 20260104161607.png]]


![[Pasted image 20260104161619.png]]


![[Pasted image 20260104161808.png]]

![[Pasted image 20260104162047.png]]
![[Pasted image 20260104162024.png]]



![[Pasted image 20260104162236.png]]



![[Pasted image 20260104162246.png]]


