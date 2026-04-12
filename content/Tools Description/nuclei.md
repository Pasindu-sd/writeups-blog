**Nuclei කියන්නේ මොනවද? (Simple Explanation)**

**Nuclei** කියන්නේ 👉 **automated vulnerability scanner** එකක්.  
ඒ කියන්නේ website / server / API වල තියෙන **security vulnerabilities ඉක්මනින් හඳුනාගන්න** භාවිතා කරන **open-source cybersecurity tool** එකක්.

Since you’re learning **cybersecurity practically** 🔐  
👉 Nuclei is **must-learn tool** for:

- Bug bounty
- Pentesting
- Blue team scanning
- CVE research

---

## 🔍 Simple definition

> **Nuclei = Template-based vulnerability scanner**

Templates භාවිතා කරලා known vulnerabilities scan කරන tool එකක්.


## 🛠️ Nuclei මොනවට භාවිතා කරන්නේ?

Nuclei භාවිතා කරලා detect කරගන්න පුළුවන්👇

- 🔓 **SQL Injection**
    
- ❌ **XSS (Cross-Site Scripting)**
    
- 🔐 **Exposed admin panels**
    
- 📁 **Sensitive files** (`.env`, `.git`, `backup.zip`)
    
- ☠️ **RCE (Remote Code Execution)**
    
- 🧠 **Misconfigurations**
    
- 🧪 **Outdated software vulnerabilities**
    
- ⚠️ **CVE-based vulnerabilities**
    

---

## 🧱 How Nuclei works (easy way)

1️⃣ Target එක දෙන්න (URL / domain / IP)  
2️⃣ Templates run කරනවා  
3️⃣ Each template = one vulnerability test  
4️⃣ Match වෙන response එකක් තියෙනවද කියලා check  
5️⃣ Result output කරනවා

👉 **Fast + Accurate + Scalable**


## 📄 Templates කියන්නේ මොනවද?

Templates කියන්නේ **YAML files**.

📌 Example:

```

id: sql-injection-test
info:
  name: Basic SQL Injection
requests:
  - method: GET
    path:
      - "{{BaseURL}}/product.php?id=1' OR '1'='1"
    matchers:
      - type: word
        words:
          - "sql"


```

👉 මේ template එක SQLi check කරනවා.


## 🚀 Simple Nuclei command example

`nuclei -u https://example.com`

👉 default templates run වෙනවා.


Target list file එකක්:

`nuclei -l targets.txt`

Specific category:

`nuclei -u https://example.com -tags sql,xss`

---

## 🆚 Nuclei vs Other tools

|Tool|Type|
|---|---|
|**Nuclei**|Fast template-based scanner|
|Burp Suite|Manual + semi-auto|
|sqlmap|SQL injection only|
|Nikto|Old-school web scan|

👉 **Nuclei = modern + customizable**