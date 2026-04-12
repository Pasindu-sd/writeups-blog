![[Pasted image 20251213214204.png]]



![[Pasted image 20251213214145.png]]


##  Embedded credentials trick

```
http://username@stock.weliketoshop.net/
```

📌 Important concept:

```
protocol://username@hostname/
```

- Browser / server URL parser එකට:
    
    - **Hostname** = `stock.weliketoshop.net`
        
    - `username` = credential part
        

✔️ Whitelist check pass වෙනවා  
➡️ This proves **URL parser supports embedded credentials**


##### Double URL encode # (Key trick 🔥)

```
#  → %23 
%23 → %2523
```


![[Pasted image 20251213215307.png]]



![[Pasted image 20251213215550.png]]


![[Pasted image 20251213215643.png]]


![[Pasted image 20251213215657.png]]


![[Pasted image 20251213215720.png]]


![[Pasted image 20251213215746.png]]
