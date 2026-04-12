
## **වැදගත් flags (කොඩි):**

- `-L` = List shares (ෂෙයාර් ලැයිස්තුගත කරන්න)
    
- `-U` = Username (පරිශීලක නාමය)
    
- `-N` = No password (මුරපදය නොමැතිව)
    
- `-W` = Workgroup (වැඩසමූහය)
    
- `-p` = Port (පොර්ට් අංකය)


## **ක්‍රියාකාරී භාවිත (Penetration Testing එකේදී):**

### **1. Anonymous access එක සදහා පරීක්ෂා කිරීම:**

```
smbclient -L //192.168.1.10 -N
```
`-N` = No password (password නැතුව උත්සාහ කරනවා)

### **2. Specific credentials සමග:**


```
smbclient -L //target_ip -U administrator%Password123
```

### **3. Workgroup specify කරන්න:**


```
smbclient -L //target_ip -W MYDOMAIN -U user
```


Example :
```
┌──(kali㉿kali)-[~]
└─$ smbclient -L //192.168.1.100 -N

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        public          Disk      Public Share
        IPC$            IPC       Remote IPC
```


```
┌──(root㉿kali)-[/home/thunder/tmp]
└─# smbclient \\\\10.129.43.28\\workshares
Password for [WORKGROUP\root]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Mar 29 04:22:01 2021
  ..                                  D        0  Mon Mar 29 04:22:01 2021
  Amy.J                               D        0  Mon Mar 29 05:08:24 2021
  James.P                             D        0  Thu Jun  3 04:38:03 2021

                5114111 blocks of size 4096. 1753186 blocks available

```


