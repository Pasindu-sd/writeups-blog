# HTB Machine Writeup - Strutted 
[Date - 2025.12.9]

## Enumeration

Machine IP - 10.10.11.59

![[Pasted image 20251209213044.png]]


![[Pasted image 20251209213002.png]]

![[Pasted image 20251209222930.png]]


```
┌──(kali㉿kali)-[~/Strutted/strutted]
└─$ tree                     
.
├── mvnw
├── mvnw.cmd
├── pom.xml
├── src
│   └── main
│       ├── java
│       │   └── org
│       │       └── strutted
│       │           └── htb
│       │               ├── AboutAction.java
│       │               ├── DatabaseUtil.java
│       │               ├── HowAction.java
│       │               ├── Upload.java
│       │               ├── URLMapping.java
│       │               └── URLUtil.java
│       ├── resources
│       │   └── struts.xml
│       └── webapp
│           └── WEB-INF
│               ├── about.jsp
│               ├── error.jsp
│               ├── how.jsp
│               ├── showImage.jsp
│               ├── success.jsp
│               ├── upload.jsp
│               └── web.xml
└── target
    ├── classes
    │   ├── org
    │   │   └── strutted
    │   │       └── htb
    │   │           ├── AboutAction.class
    │   │           ├── DatabaseUtil.class
    │   │           ├── HowAction.class
    │   │           ├── Upload.class
    │   │           ├── URLMapping.class
    │   │           └── URLUtil.class
    │   └── struts.xml
    ├── generated-sources
    │   └── annotations
    ├── maven-archiver
    │   └── pom.properties
    ├── maven-status
    │   └── maven-compiler-plugin
    │       └── compile
    │           └── default-compile
    │               ├── createdFiles.lst
    │               └── inputFiles.lst
    ├── strutted-1.0.0
    │   ├── META-INF
    │   └── WEB-INF
    │       ├── about.jsp
    │       ├── classes
    │       │   ├── org
    │       │   │   └── strutted
    │       │   │       └── htb
    │       │   │           ├── AboutAction.class
    │       │   │           ├── DatabaseUtil.class
    │       │   │           ├── HowAction.class
    │       │   │           ├── Upload.class
    │       │   │           ├── URLMapping.class
    │       │   │           └── URLUtil.class
    │       │   └── struts.xml
    │       ├── error.jsp
    │       ├── how.jsp
    │       ├── lib
    │       │   ├── commons-fileupload-1.5.jar
    │       │   ├── commons-io-2.13.0.jar
    │       │   ├── commons-lang3-3.13.0.jar
    │       │   ├── commons-text-1.10.0.jar
    │       │   ├── freemarker-2.3.32.jar
    │       │   ├── javassist-3.29.0-GA.jar
    │       │   ├── javax.servlet-api-4.0.1.jar
    │       │   ├── log4j-api-2.20.0.jar
    │       │   ├── ognl-3.3.4.jar
    │       │   ├── sqlite-jdbc-3.47.1.0.jar
    │       │   └── struts2-core-6.3.0.1.jar
    │       ├── success.jsp
    │       ├── upload.jsp
    │       └── web.xml
    └── strutted-1.0.0.war

30 directories, 52 files

```


 ![[Pasted image 20251209231132.png]]

![[Pasted image 20251210133429.png]]

![[Pasted image 20251210134554.png]]


![[Pasted image 20251210134523.png]]


![[Pasted image 20251210135041.png]]


```
>> peass
>> cd linpeas
>> scp linpeas.sh james@strutted.sh:/home/james
```

```
>> ./linpeas.sh
```


![[Pasted image 20251210142456.png]]

![[Pasted image 20251210145021.png]]
