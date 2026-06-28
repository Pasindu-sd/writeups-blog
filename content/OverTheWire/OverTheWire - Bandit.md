

OverTheWire Bandit is the perfect wargame for beginners to learn Linux basics and command-line skills. Each level requires finding an SSH password to progress to the next level.

- **Host:** `bandit.labs.overthewire.org`
- **Port:** `2220`
- **Connect:** `ssh banditX@bandit.labs.overthewire.org -p 2220`

---

## Level 0 → Level 1

**Goal:** Log into the game using SSH and read the password from the `readme` file in the home directory.

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
# password: bandit0

cat readme
```

---

## Level 1 → Level 2

**Password:** `ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If`

**Goal:** Read a file named `-` (dash). Since `-` is interpreted as stdin by most commands, you need to specify the path explicitly.

```bash
cat ./-
```

---

## Level 2 → Level 3

**Password:** `263JcdLucbFp900kkjkLuao74GwNAMTa`

**Goal:** Read a file that has spaces in its filename.

```bash
cat ./"--spaces in this filename--"
```

---

## Level 3 → Level 4

**Password:** `2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ`

**Goal:** Find and read a hidden file inside the `inhere` directory.

```bash
ls -la inhere/
cat inhere/.hidden
```

**What we learned:** Files starting with `.` are hidden in Linux. Use `ls -la` to reveal them.

---

## Level 4 → Level 5

**Password:** `4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw`

**Goal:** Find the only human-readable file in the `inhere` directory among multiple files.

```bash
file inhere/-file0*
cat inhere/-file07
```

**What we learned:** The `file` command identifies file types. Human-readable = ASCII text.

---

## Level 5 → Level 6

**Password:** `HWasnPhtq9AVKe0dmk45nxy20cvUa6EG`

**Goal:** Find a file with these properties: human-readable, 1033 bytes, not executable.

```bash
find inhere/ -size 1033c ! -executable -readable
cat inhere/maybehere07/.file2
```

**What we learned:** `find` command with multiple filters can narrow down files efficiently.

---

## Level 6 → Level 7

**Password:** `morbNTDkSW6jIlUc0ymOdMaLnOlFVAaj`

**Goal:** Find a file somewhere on the server owned by user `bandit7`, group `bandit6`, and exactly 33 bytes in size.

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
cat /var/lib/dpkg/info/bandit7.password
```

**What we learned:** `2>/dev/null` suppresses permission-denied errors to clean up output.

---

## Level 7 → Level 8

**Password:** `dfwvzFQi4mU0wfNbFOe9RoWskMLg7eEc`

**Goal:** Find the password stored next to the word "millionth" in `data.txt`.

```bash
grep "millionth" data.txt
```

**What we learned:** `grep` is powerful for searching specific strings inside large files.

---

## Level 8 → Level 9

**Password:** `4CKMh1JI91bUIZZPXDqGanal4xvAg0JM`

**Goal:** Find the line that appears only once in `data.txt`.

```bash
sort data.txt | uniq -u
```

**What we learned:** `sort` then `uniq -u` isolates unique lines from duplicates.

---

## Level 9 → Level 10

**Password:** `FGUW5ilLVJrxX9kMYMmlN4MgbpfMiqey`

**Goal:** The password is one of the few human-readable strings in `data.txt`, preceded by several `=` characters.

```bash
strings data.txt | grep "=="
```

**What we learned:** `strings` extracts readable text from binary files.

---

## Level 10 → Level 11

**Password:** `dtR173fZKb0RRsDFSGsg2RWnpNVj3qRr`

**Goal:** `data.txt` contains Base64 encoded data. Decode it to find the password.

```bash
base64 -d data.txt
```

**What we learned:** Base64 is an encoding scheme commonly used to represent binary data as text.

---

## Level 11 → Level 12

**Password:** `7x16WNeHIi5YkIhWsfFIqoognUTyj9Q4`

**Goal:** `data.txt` has been encoded with ROT13. All letters have been rotated by 13 positions.

```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

**What we learned:** ROT13 is a simple Caesar cipher. `tr` can perform character substitution.

---

## Level 12 → Level 13

**Password:** `FO5dwFsc0cbaIiH0h8J2eUks2vdTDwAn`

**Goal:** `data.txt` is a hexdump of a file that has been repeatedly compressed. Reverse the hexdump and decompress multiple times.

```bash
mkdir /tmp/bandit12work
cp data.txt /tmp/bandit12work/
cd /tmp/bandit12work/

# Convert hexdump back to binary
xxd -r data.txt > data.bin

# Check file type and decompress accordingly
file data.bin
# Repeat: gzip -d / bzip2 -d / tar -xf until you reach ASCII text
```

**What we learned:** Files can be compressed multiple times with different algorithms. Always check with `file` before decompressing.

---

## Level 13 → Level 14

**Password:** SSH private key file (`sshkey.private`)

**Goal:** Use the private SSH key found in the home directory to log into bandit14.

```bash
ssh -i sshkey.private bandit14@localhost -p 2220
cat /etc/bandit_pass/bandit14
```

**What we learned:** SSH supports key-based authentication as a more secure alternative to passwords.

---

## Level 14 → Level 15

**Password:** `MU4VWeTyJk8ROof1qqmcBPaLh7lDCPvS`

**Goal:** Submit the current level's password to port 30000 on localhost to receive the next password.

```bash
echo "MU4VWeTyJk8ROof1qqmcBPaLh7lDCPvS" | nc localhost 30000
```

**What we learned:** `nc` (Netcat) is a utility for reading/writing to network connections — useful for interacting with services directly.

---

## Level 15 → Level 16

**Password:** `8xCjnmgoKbGLhHFAZlGE5Tmu4M2tKJQo`

**Goal:** Submit the password to port 30001 on localhost using SSL/TLS encryption.

```bash
echo "8xCjnmgoKbGLhHFAZlGE5Tmu4M2tKJQo" | openssl s_client -connect localhost:30001 -quiet
```

**What we learned:** `openssl s_client` allows you to connect to SSL/TLS encrypted services from the command line.

---

## Key Commands Reference

|Command|Purpose|
|---|---|
|`ssh`|Secure remote login|
|`cat`|Read file contents|
|`ls -la`|List all files including hidden|
|`find`|Search for files with filters|
|`grep`|Search text within files|
|`file`|Identify file type|
|`sort \| uniq`|Find unique lines|
|`strings`|Extract readable text from binaries|
|`base64 -d`|Decode Base64|
|`tr`|Translate/rotate characters|
|`xxd -r`|Reverse hexdump|
|`nc`|Netcat — raw network connections|
|`openssl s_client`|SSL/TLS connections|
