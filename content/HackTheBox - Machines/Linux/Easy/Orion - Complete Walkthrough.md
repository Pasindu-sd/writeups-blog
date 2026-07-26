
# #HTB 

![[Pasted image 20260726151334.png|281]]


# HTB: Orion

**Machine IP:** `10.129.1.6`  
**Difficulty:** Easy  
**OS:** Linux  

---

---

## Step 1: Reconnaissance - Port Scanning

### Nmap Results:

```
nmap -n -Pn -sV -sC 10.129.244.146
```


![[Pasted image 20260726151935.png]]

**Open Ports:**

|Port|Service|Version|
|---|---|---|
|22/tcp|SSH|OpenSSH 9.6p1 Ubuntu|
|80/tcp|HTTP|nginx 1.18.0 (Ubuntu)|

**Important Finding:** HTTP redirects to `http://orion.htb/`

```bash
# Add to /etc/hosts
echo "10.129.244.146 orion.htb" | sudo tee -a /etc/hosts
```



![[Pasted image 20260726152025.png]]


![[Pasted image 20260726152123.png]]


![[Pasted image 20260726152155.png]]


![[Pasted image 20260726152241.png]]


![[Pasted image 20260726152257.png]]


![[Pasted image 20260726152311.png]]


![[Pasted image 20260726152345.png]]


![[Pasted image 20260726152404.png]]


![[Pasted image 20260726152506.png]]


