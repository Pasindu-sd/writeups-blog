
## 🧭 Gobuster කියන්නේ මොකද්ද?

**Gobuster** කියන්නේ web server එකක:

- folders
    
- files
    
- hidden paths
    

brute-force කරලා හොයන්න භාවිතා කරන tool එකක්.


උදාහරණයක්:
```
/admin 
/login 
/config 
/backup
```


## 🔑 `-x` switch එකේ වැඩේ මොකද්ද?

`-x`

👉 `-x` කියන්නේ **file extensions specify කරන්න**.

💡 සාමාන්‍ය gobuster scan එක:
```
/admin 
/login
```

💡 `-x` දාලා scan කළොත්:

```
/admin.php 
/admin.txt 
/admin.bak
```


## 🧠 Simple Sinhala example

### ❌ Without `-x`

`gobuster dir -u http://10.129.49.130 -w words.txt`

Gobuster try කරන්නේ:

`/admin /login`

---

### ✅ With `-x`

`gobuster dir -u http://10.129.49.130 -w words.txt -x php,txt`

Gobuster try කරන්නේ:

`/admin /admin.php /admin.txt  /login /login.php /login.txt`

👉 **එක word එකකට extensions එකතු කරලා scan කරනවා**


---

## 🧪 Gobuster VHOST Command – Sinhala Explanation

`gobuster vhost -w ~/Downloads/subdomains-top1mil-20000.txt -u http://10.129.227.248`

### 🔹 `gobuster`

👉 **Gobuster** කියන්නේ brute-force tool එකක්.  
Web application වල **hidden resources**, **subdomains**, **virtual hosts (vhost)** හොයන්න භාවිතා කරනවා.

---

### 🔹 `vhost`

👉 මේ mode එක භාවිතා කරන්නේ  
**Virtual Hosts (VHOST)** හොයන්න.

🧠 **Virtual Host කියන්නේ මොකක්ද?**  
එකම IP address එකකට:

- site1.example.com
    
- admin.example.com
    
- test.example.com
    

වගේ **domain names ගොඩක්** map කරලා තියෙන්න පුළුවන්.

`vhost` mode එකෙන්:  
➡️ `Host:` header එක වෙනස් කරලා  
➡️ server එක respond කරනවද කියලා test කරනවා



---
---
----


## 🔹 What can Gobuster find?

### 1️⃣ Hidden folders & files

`gobuster dir -u http://target -w common.txt`

➡️ Finds: `/admin`, `/backup`, `/login.php`



### 2️⃣ Files with extensions

`gobuster dir -u http://target -w list.txt -x php,zip,bak`

➡️ Finds: `config.php`, `backup.zip`



### 3️⃣ Subdomains (DNS)

`gobuster dns -d example.com -w subdomains.txt`

➡️ Finds: `admin.example.com`



### 4️⃣ Virtual Hosts (same IP)

`gobuster vhost -u http://10.10.10.10 -w subdomains.txt`

➡️ Finds: `dev.site.local`

