
![[Pasted image 20251224094956.png]]

Logging with social media account
![[Pasted image 20251224095445.png]]


![[Pasted image 20251224095723.png]]


![[Pasted image 20251224095829.png]]

1. Notice that the client application (the blog website) receives some basic information about the user from the OAuth service. It then logs the user in by sending a `POST` request containing this information to its own `/authenticate` endpoint, along with the access token.

![[Pasted image 20251224100603.png]]


![[Pasted image 20251224100634.png]]


1. Send the `POST /authenticate` request to Burp Repeater. In Repeater, change the email address to `carlos@carlos-montoya.net` and send the request. Observe that you do not encounter an error.
2. Right-click on the `POST` request and select "Request in browser" > "In original session". Copy this URL and visit it in the browser. You are logged in as Carlos and the lab is solved.

![[Pasted image 20251224100731.png]]
